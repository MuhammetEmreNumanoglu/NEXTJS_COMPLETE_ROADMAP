# 005 — Props, State, and Context

---

## Purpose of This Topic

Props, state, and context are the three pillars of data management in React.  
Understanding the difference between them — and knowing when to use each one —  
is what separates junior developers from senior ones.

In this file, you'll master all three in depth, with patterns used in real production codebases.

---

## Why Does This Topic Exist?

Data in a React application can be described by three questions:

1. Where does this data live?
2. Who needs to read it?
3. Who needs to change it?

The answers to these questions determine whether to use props, state, or context.  
Getting this wrong leads to:
- Prop drilling (passing data through 5 layers of components that don't need it)
- Over-rendering (state placed too high causes the entire tree to re-render)
- Under-rendering (state placed too low prevents necessary updates)
- Hard-to-maintain code

---

## Where Is This Used in Real Projects?

```
Props    → Everywhere. Any data passed between parent and child components.
State    → Any interactive UI: forms, modals, tabs, counters, toggles.
Context  → Auth state, theme (dark/light), language/locale, shopping cart.
```

You'll use all three dozens of times per day as a React developer.

---

## Real-Life Analogy

Think of a restaurant:

```
PROPS: The menu card
  The kitchen (parent) prepares the information.
  The waiter (component) carries it to the customer.
  The customer can read it but cannot change it.
  Information flows in one direction only.

STATE: The order pad
  The waiter owns it.
  The waiter writes on it and can change it.
  It's local to the waiter — other waiters have their own pads.

CONTEXT: The restaurant's Wi-Fi password
  Written on the wall. Visible to everyone in the restaurant.
  Any customer (component) can read it without asking the waiter.
  Changed by the manager (provider), and everyone sees the change.
```

---

## Simple Explanation

```
PROPS
  ─────────────────────────────────────────────────────
  Data passed FROM a parent TO a child component.
  Think: function arguments.
  Rules: Read-only. Never modify props inside the child.

STATE
  ─────────────────────────────────────────────────────
  Data that lives INSIDE a component.
  Think: a variable the component owns and manages.
  Rules: Change only via the setter function.
  When it changes, the component re-renders.

CONTEXT
  ─────────────────────────────────────────────────────
  Data shared across many components without prop drilling.
  Think: a global variable, but managed by React.
  Rules: Wrap the tree with a Provider. Any descendant can read it.
  When context value changes, all consumers re-render.
```

---

## Technical Explanation: Props

### Basic Props

```tsx
type PriceBadgeProps = {
  amount: number;
  currency: "USD" | "EUR" | "GBP" | "TRY";
  size: "sm" | "md" | "lg";
};

function PriceBadge({ amount, currency, size }: PriceBadgeProps) {
  const formattedPrice = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency,
  }).format(amount);

  return (
    <span className={`price-badge price-badge--${size}`}>
      {formattedPrice}
    </span>
  );
}
```

### Default Props

```tsx
type AvatarProps = {
  src?: string;
  alt: string;
  size?: 32 | 48 | 64 | 96 | 128;
  shape?: "circle" | "square";
};

function Avatar({
  src,
  alt,
  size = 48,
  shape = "circle",
}: AvatarProps) {
  const fallbackSrc = `https://ui-avatars.com/api/?name=${encodeURIComponent(alt)}&size=${size}`;

  return (
    <img
      src={src ?? fallbackSrc}
      alt={alt}
      width={size}
      height={size}
      className={`avatar avatar--${shape}`}
      style={{ borderRadius: shape === "circle" ? "50%" : "8px" }}
    />
  );
}
```

`size = 48` → Default parameter value. If no `size` prop is passed, it defaults to `48`.  
`src ?? fallbackSrc` → Nullish coalescing: if `src` is `undefined`, use the generated avatar.

### The `children` Prop

`children` is a special prop that represents content placed between component tags.

```tsx
type CardProps = {
  title: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
  className?: string;
};

function Card({ title, children, footer, className = "" }: CardProps) {
  return (
    <div className={`card ${className}`}>
      <div className="card-header">
        <h3 className="card-title">{title}</h3>
      </div>
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}
```

Usage:

```tsx
function HomePage() {
  return (
    <div>
      <Card
        title="Latest Updates"
        footer={<a href="/updates">See all updates →</a>}
      >
        <p>Version 2.0 has been released with major performance improvements.</p>
        <p>Check out the changelog for details.</p>
      </Card>

      <Card title="Quick Stats">
        <ul>
          <li>1,284 active users</li>
          <li>98.7% uptime this month</li>
          <li>142 issues resolved</li>
        </ul>
      </Card>
    </div>
  );
}
```

`React.ReactNode` is the widest type — it accepts JSX, strings, numbers, arrays, `null`, and more.

### Prop Spreading (Use Carefully)

```tsx
type InputProps = React.InputHTMLAttributes<HTMLInputElement> & {
  label: string;
  error?: string;
};

function FormInput({ label, error, ...inputProps }: InputProps) {
  const inputId = `input-${label.toLowerCase().replace(/\s+/g, "-")}`;

  return (
    <div className="form-field">
      <label htmlFor={inputId}>{label}</label>
      <input
        id={inputId}
        className={`form-input ${error ? "form-input--error" : ""}`}
        {...inputProps}
      />
      {error && <p className="form-error">{error}</p>}
    </div>
  );
}
```

Usage:

```tsx
function ContactForm() {
  return (
    <form>
      <FormInput
        label="Email Address"
        type="email"
        placeholder="you@example.com"
        required
        autoComplete="email"
        error="Please enter a valid email"
      />
      <FormInput
        label="Phone Number"
        type="tel"
        placeholder="+1 (555) 000-0000"
      />
    </form>
  );
}
```

`React.InputHTMLAttributes<HTMLInputElement>` → All standard HTML `<input>` attributes.  
`...inputProps` → Spread the remaining props onto the native `<input>`. This way `FormInput`  
automatically supports `type`, `placeholder`, `required`, `autoComplete`, etc.

### Render Props Pattern

Render props let a parent component pass a function that returns JSX to a child,  
giving the child control over what to render.

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  renderEmpty?: () => React.ReactNode;
  keyExtractor: (item: T) => string;
};

function List<T>({
  items,
  renderItem,
  renderEmpty,
  keyExtractor,
}: ListProps<T>) {
  if (items.length === 0) {
    return <>{renderEmpty ? renderEmpty() : <p>No items.</p>}</>;
  }

  return (
    <ul className="list">
      {items.map((item, index) => (
        <li key={keyExtractor(item)} className="list-item">
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}
```

Usage:

```tsx
type User = { id: string; name: string; email: string };

function UserList({ users }: { users: User[] }) {
  return (
    <List
      items={users}
      keyExtractor={(user) => user.id}
      renderItem={(user) => (
        <div className="user-row">
          <strong>{user.name}</strong>
          <span>{user.email}</span>
        </div>
      )}
      renderEmpty={() => (
        <div className="empty-state">
          <p>No users found.</p>
          <a href="/users/new">Add the first user</a>
        </div>
      )}
    />
  );
}
```

The generic `<T>` makes `List` reusable with any data type — users, products, posts, etc.

---

## Technical Explanation: State

### Derived State vs State

One of the most important state decisions: **should this be state or derived?**

```tsx
type CartItem = {
  id: string;
  name: string;
  price: number;
  quantity: number;
};

function ShoppingCartSummary() {
  const [items, setItems] = useState<CartItem[]>([]);

  const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const tax = subtotal * 0.08;
  const total = subtotal + tax;
  const itemCount = items.reduce((sum, item) => sum + item.quantity, 0);

  return (
    <div className="cart-summary">
      <p>Items: {itemCount}</p>
      <p>Subtotal: ${subtotal.toFixed(2)}</p>
      <p>Tax (8%): ${tax.toFixed(2)}</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

**`subtotal`, `tax`, `total`, `itemCount` are NOT state — they're derived.**  
They're always computed from `items`. There's no need to store them.  
If you stored them as state, you'd have to keep them synchronized with `items` — a recipe for bugs.

**Rule: If a value can be computed from existing state, it should be derived, not stored.**

### State Lifting

When two sibling components need to share state, lift the state to their common parent.

```tsx
type TemperatureUnit = "celsius" | "fahrenheit";

function TemperatureConverter() {
  const [celsius, setCelsius] = useState<number>(0);

  const fahrenheit = (celsius * 9) / 5 + 32;

  return (
    <div className="converter">
      <TemperatureInput
        label="Celsius"
        value={celsius}
        onChange={setCelsius}
      />
      <TemperatureInput
        label="Fahrenheit"
        value={fahrenheit}
        onChange={(f) => setCelsius(((f - 32) * 5) / 9)}
      />
      <p className="boiling-indicator">
        {celsius >= 100 ? "Water is boiling! 🌡️" : "Water is not boiling."}
      </p>
    </div>
  );
}

type TemperatureInputProps = {
  label: string;
  value: number;
  onChange: (value: number) => void;
};

function TemperatureInput({ label, value, onChange }: TemperatureInputProps) {
  return (
    <div className="temperature-input">
      <label>{label}:</label>
      <input
        type="number"
        value={value.toFixed(2)}
        onChange={(e) => onChange(Number(e.target.value))}
      />
    </div>
  );
}
```

Both `TemperatureInput` components share the same state — `celsius` — which lives  
in their parent, `TemperatureConverter`. Changing either input updates both.

### State Colocation

The opposite of lifting: keep state as close to where it's needed as possible.

```tsx
function ProductPage() {
  return (
    <div>
      <ProductGallery />
      <ProductDetails />
      <RelatedProducts />
    </div>
  );
}

function ProductGallery() {
  const [selectedImageIndex, setSelectedImageIndex] = useState(0);

  return (
    <div className="gallery">
      <img src={`/images/product-${selectedImageIndex}.jpg`} alt="Product" />
      <div className="thumbnails">
        {[0, 1, 2, 3].map((index) => (
          <button key={index} onClick={() => setSelectedImageIndex(index)}>
            <img src={`/images/product-${index}-thumb.jpg`} alt={`View ${index + 1}`} />
          </button>
        ))}
      </div>
    </div>
  );
}
```

`selectedImageIndex` is only used inside `ProductGallery`.  
There's no reason to lift it to `ProductPage`.  
If `ProductPage` re-renders for some other reason, `ProductGallery`'s selected image  
won't reset because the state is local to `ProductGallery`.

---

## Technical Explanation: Context

### When Context Is Right

```
Use Context for:
✓ Authentication state (current user, roles)
✓ Theme (dark/light mode)
✓ Language/locale settings
✓ Feature flags
✓ Shopping cart (if you don't have a dedicated state manager)
✓ Toast/notification system

Do NOT use Context for:
✗ Frequently changing state (e.g., mouse position, animation frames)
  → Every consumer re-renders on every change
✗ State that only a few components need
  → Just lift the state to the common parent
✗ Replacing a proper state manager in large apps
  → Use Zustand or Redux Toolkit instead (covered in Section 4)
```

### Theme Context: Full Example

```tsx
import { createContext, useContext, useState, useCallback } from "react";

type Theme = "light" | "dark" | "system";

type ThemeContextValue = {
  theme: Theme;
  resolvedTheme: "light" | "dark";
  setTheme: (theme: Theme) => void;
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextValue | null>(null);

function useTheme(): ThemeContextValue {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }
  return context;
}

function getSystemTheme(): "light" | "dark" {
  if (typeof window === "undefined") return "light";
  return window.matchMedia("(prefers-color-scheme: dark)").matches
    ? "dark"
    : "light";
}

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setThemeState] = useState<Theme>("system");

  const resolvedTheme: "light" | "dark" =
    theme === "system" ? getSystemTheme() : theme;

  const setTheme = useCallback((newTheme: Theme) => {
    setThemeState(newTheme);
    if (typeof document !== "undefined") {
      document.documentElement.setAttribute("data-theme", newTheme === "system" ? getSystemTheme() : newTheme);
    }
  }, []);

  const toggleTheme = useCallback(() => {
    setTheme(resolvedTheme === "light" ? "dark" : "light");
  }, [resolvedTheme, setTheme]);

  return (
    <ThemeContext.Provider value={{ theme, resolvedTheme, setTheme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemeToggleButton() {
  const { resolvedTheme, toggleTheme } = useTheme();

  return (
    <button
      onClick={toggleTheme}
      className="theme-toggle"
      aria-label={`Switch to ${resolvedTheme === "light" ? "dark" : "light"} mode`}
    >
      {resolvedTheme === "light" ? "🌙 Dark" : "☀️ Light"}
    </button>
  );
}

function ThemeIndicator() {
  const { theme, resolvedTheme } = useTheme();

  return (
    <p className="theme-indicator">
      Setting: <strong>{theme}</strong> | Resolved: <strong>{resolvedTheme}</strong>
    </p>
  );
}
```

### Context Performance Optimization

A common problem with Context: every consumer re-renders when **any part** of the  
context value changes.

```tsx
type AppContextValue = {
  user: User | null;
  notifications: Notification[];
  theme: Theme;
};

const AppContext = createContext<AppContextValue | null>(null);
```

**Problem:** If `notifications` changes every second (new notifications coming in),  
every component using `useContext(AppContext)` re-renders every second —  
even components that only care about `user` or `theme`.

**Solution: Split into multiple focused contexts:**

```tsx
const UserContext = createContext<User | null>(null);
const NotificationContext = createContext<Notification[]>([]);
const ThemeContext = createContext<ThemeContextValue | null>(null);

function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <UserProvider>
      <NotificationProvider>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </NotificationProvider>
    </UserProvider>
  );
}
```

Now a component that only calls `useTheme()` won't re-render when notifications change.

### Notification Context: Full Example

```tsx
import { createContext, useContext, useState, useCallback, useId } from "react";

type NotificationVariant = "success" | "error" | "warning" | "info";

type Notification = {
  id: string;
  message: string;
  variant: NotificationVariant;
  duration: number;
};

type NotificationContextValue = {
  notifications: Notification[];
  addNotification: (
    message: string,
    variant?: NotificationVariant,
    duration?: number
  ) => void;
  removeNotification: (id: string) => void;
};

const NotificationContext = createContext<NotificationContextValue | null>(null);

function useNotifications(): NotificationContextValue {
  const context = useContext(NotificationContext);
  if (!context) {
    throw new Error(
      "useNotifications must be used within a NotificationProvider"
    );
  }
  return context;
}

function NotificationProvider({ children }: { children: React.ReactNode }) {
  const [notifications, setNotifications] = useState<Notification[]>([]);

  const removeNotification = useCallback((id: string) => {
    setNotifications((prev) => prev.filter((n) => n.id !== id));
  }, []);

  const addNotification = useCallback(
    (
      message: string,
      variant: NotificationVariant = "info",
      duration: number = 4000
    ) => {
      const id = crypto.randomUUID();
      const notification: Notification = { id, message, variant, duration };

      setNotifications((prev) => [...prev, notification]);

      setTimeout(() => {
        removeNotification(id);
      }, duration);
    },
    [removeNotification]
  );

  return (
    <NotificationContext.Provider
      value={{ notifications, addNotification, removeNotification }}
    >
      {children}
      <NotificationContainer notifications={notifications} onRemove={removeNotification} />
    </NotificationContext.Provider>
  );
}

type NotificationContainerProps = {
  notifications: Notification[];
  onRemove: (id: string) => void;
};

function NotificationContainer({ notifications, onRemove }: NotificationContainerProps) {
  return (
    <div className="notification-container" aria-live="polite">
      {notifications.map((notification) => (
        <NotificationToast
          key={notification.id}
          notification={notification}
          onRemove={onRemove}
        />
      ))}
    </div>
  );
}

type NotificationToastProps = {
  notification: Notification;
  onRemove: (id: string) => void;
};

function NotificationToast({ notification, onRemove }: NotificationToastProps) {
  return (
    <div className={`notification notification--${notification.variant}`}>
      <p>{notification.message}</p>
      <button
        onClick={() => onRemove(notification.id)}
        aria-label="Dismiss notification"
      >
        ✕
      </button>
    </div>
  );
}

function SaveButton() {
  const { addNotification } = useNotifications();

  const handleSave = async () => {
    try {
      await saveSomething();
      addNotification("Changes saved successfully!", "success");
    } catch {
      addNotification("Failed to save changes. Please try again.", "error");
    }
  };

  return <button onClick={handleSave}>Save Changes</button>;
}

async function saveSomething(): Promise<void> {
  await new Promise((resolve) => setTimeout(resolve, 500));
}
```

---

## State vs Props vs Context: Decision Guide

```
┌─────────────────────────────────────────────────────────────┐
│              DATA MANAGEMENT DECISION GUIDE                   │
└─────────────────────────────────────────────────────────────┘

Is the data only needed in one component?
  └─ YES → useState (local state)

Is the data needed in a child component?
  └─ YES → Is it 1-2 levels deep?
        └─ YES → Pass as props
        └─ NO  → How many components need it?
              └─ FEW (3-5) → Lift state to common parent
              └─ MANY      → Context or state manager

Does the data change very frequently?
  └─ YES → Avoid Context (use local state or Zustand)

Is the data truly global (auth, theme, locale)?
  └─ YES → Context

Is the state complex with many transitions?
  └─ YES → useReducer instead of useState

Is the state shared across many pages/routes?
  └─ YES → Zustand, Redux Toolkit, or Jotai
           (covered in Section 4)
```

---

## Common Mistakes

### Mistake 1: Modifying Props

```tsx
function BadComponent({ user }: { user: User }) {
  user.name = "John";
  return <p>{user.name}</p>;
}
```

**Problem:** Props are read-only. This mutates the original object.  
The parent component (which owns `user`) has no idea this happened.

```tsx
function GoodComponent({ user }: { user: User }) {
  return <p>{user.name}</p>;
}
```

Never modify props. If you need a modified version, create a new variable.

---

### Mistake 2: Mirroring Props Into State

```tsx
type BadProfileProps = {
  initialName: string;
};

function BadProfile({ initialName }: BadProfileProps) {
  const [name, setName] = useState(initialName);

  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

**Problem:** If `initialName` changes (parent re-renders with a new name),  
the state won't update — it only uses `initialName` once during initialization.

```tsx
type GoodProfileProps = {
  name: string;
  onNameChange: (newName: string) => void;
};

function GoodProfile({ name, onNameChange }: GoodProfileProps) {
  return (
    <input
      value={name}
      onChange={(e) => onNameChange(e.target.value)}
    />
  );
}
```

**Better:** Make the component controlled — the parent owns the state.

---

### Mistake 3: Context for Frequently Changing Data

```tsx
function BadMouseTrackerProvider({ children }: { children: React.ReactNode }) {
  const [mousePosition, setMousePosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setMousePosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener("mousemove", handleMouseMove);
    return () => window.removeEventListener("mousemove", handleMouseMove);
  }, []);

  return (
    <MousePositionContext.Provider value={mousePosition}>
      {children}
    </MousePositionContext.Provider>
  );
}
```

**Problem:** Mouse position updates hundreds of times per second.  
Every component consuming `MousePositionContext` re-renders on every mouse move.

```tsx
function useMousePosition() {
  const [mousePosition, setMousePosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setMousePosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener("mousemove", handleMouseMove);
    return () => window.removeEventListener("mousemove", handleMouseMove);
  }, []);

  return mousePosition;
}
```

**Better:** A custom Hook. Only components that call `useMousePosition()` re-render.  
No unnecessary re-renders in components that don't care about mouse position.

---

### Mistake 4: Stale Props in Callbacks

```tsx
"use client";

import { useState, useEffect } from "react";

function BadTimer({ interval }: { interval: number }) {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount((prev) => prev + interval);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <p>{count}</p>;
}
```

**Problem:** `interval` is used inside the effect but not in the dependency array.  
If the parent passes a new `interval` value, the timer still uses the old one.

```tsx
function GoodTimer({ interval }: { interval: number }) {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount((prev) => prev + interval);
    }, 1000);
    return () => clearInterval(id);
  }, [interval]);

  return <p>{count}</p>;
}
```

**Correct:** List all values used inside effects in the dependency array.

---

## Best Practices

### Use Discriminated Unions for Complex State

Instead of many booleans, use a single state with multiple cases:

```tsx
type RequestState<T> =
  | { phase: "idle" }
  | { phase: "loading" }
  | { phase: "success"; data: T }
  | { phase: "error"; message: string };

function UserProfile({ userId }: { userId: string }) {
  const [requestState, setRequestState] = useState<RequestState<User>>({
    phase: "idle",
  });

  useEffect(() => {
    setRequestState({ phase: "loading" });

    fetchUser(userId)
      .then((data) => setRequestState({ phase: "success", data }))
      .catch((error: Error) =>
        setRequestState({ phase: "error", message: error.message })
      );
  }, [userId]);

  if (requestState.phase === "idle") return null;
  if (requestState.phase === "loading") return <LoadingSpinner />;
  if (requestState.phase === "error") return <ErrorMessage message={requestState.message} />;

  return <UserCard user={requestState.data} />;
}

function LoadingSpinner() {
  return <div className="spinner" aria-label="Loading..." />;
}

function ErrorMessage({ message }: { message: string }) {
  return <p className="error-message" role="alert">{message}</p>;
}

type User = { id: string; name: string; email: string };
function UserCard({ user }: { user: User }) {
  return <div><h2>{user.name}</h2><p>{user.email}</p></div>;
}
async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}
```

Instead of `isLoading: boolean`, `isError: boolean`, `data: User | null`, `error: string | null`  
(which can be in impossible combinations like `isLoading: true` AND `isError: true`),  
you have a single discriminated union that's always in a valid state.

---

## Real Company Examples

### Stripe: Context for Feature Flags

Stripe uses Context to provide feature flags throughout their dashboard:

```tsx
type FeatureFlags = {
  newCheckoutFlow: boolean;
  betaDashboard: boolean;
  experimentalAnalytics: boolean;
};

const FeatureFlagContext = createContext<FeatureFlags>({
  newCheckoutFlow: false,
  betaDashboard: false,
  experimentalAnalytics: false,
});

function useFeatureFlag(flag: keyof FeatureFlags): boolean {
  return useContext(FeatureFlagContext)[flag];
}

function CheckoutButton() {
  const newCheckoutFlow = useFeatureFlag("newCheckoutFlow");

  return newCheckoutFlow ? <NewCheckoutButton /> : <LegacyCheckoutButton />;
}

function NewCheckoutButton() {
  return <button className="checkout-btn-v2">Checkout →</button>;
}

function LegacyCheckoutButton() {
  return <button className="checkout-btn">Proceed to Checkout</button>;
}
```

### Airbnb: Lifted State for Search Filters

Airbnb lifts search filter state to a `SearchPage` component:

```tsx
type SearchFilters = {
  location: string;
  checkIn: Date | null;
  checkOut: Date | null;
  guests: number;
  minPrice: number;
  maxPrice: number;
};

function SearchPage() {
  const [filters, setFilters] = useState<SearchFilters>({
    location: "",
    checkIn: null,
    checkOut: null,
    guests: 1,
    minPrice: 0,
    maxPrice: 1000,
  });

  const updateFilter = <K extends keyof SearchFilters>(
    key: K,
    value: SearchFilters[K]
  ) => {
    setFilters((prev) => ({ ...prev, [key]: value }));
  };

  return (
    <div className="search-page">
      <SearchBar filters={filters} onFilterChange={updateFilter} />
      <FilterPanel filters={filters} onFilterChange={updateFilter} />
      <SearchResults filters={filters} />
    </div>
  );
}
```

All three children — `SearchBar`, `FilterPanel`, `SearchResults` — receive the  
same `filters` state. When any filter changes, all three update automatically.

---

## Mini Exercise

Build a multi-step form with shared state:

```
Requirements:
  Step 1: Personal Info (name, email)
  Step 2: Account Setup (username, password)
  Step 3: Preferences (theme, language, notifications)
  Step 4: Review & Submit (show all collected data)

The form must:
  - Keep a shared FormData state at the top level
  - Each step receives only its relevant fields as props
  - A progress indicator shows which step is active
  - "Back" and "Next" buttons navigate between steps
  - "Back" should NOT lose previously entered data
  - Final step shows a summary of all entered data
  - Everything typed with TypeScript
```

---

## Mini Quiz

**Question 1:** What does "prop drilling" mean?

```
A) A tool for creating custom props
B) Passing props through multiple intermediate components that don't use them
C) A performance optimization technique
D) A way to connect to a database
```

*Answer: B*

---

**Question 2:** When should you derive a value instead of storing it in state?

```
A) When the value is a string
B) Always — never store anything in state
C) When the value can always be computed from existing state or props
D) When the value is used in more than one component
```

*Answer: C*

---

**Question 3:** What is the primary downside of using Context for frequently changing data?

```
A) Context doesn't support TypeScript
B) Every component consuming the context re-renders on every change
C) Context can only hold one value
D) Context is not available in Next.js
```

*Answer: B*

---

**Question 4:** What is the `children` prop?

```
A) A list of all child components
B) The content placed between a component's opening and closing tags
C) A TypeScript utility type
D) An HTML attribute for nested elements
```

*Answer: B*

---

**Question 5:** What does "state lifting" mean?

```
A) Moving state from a parent to a child
B) Moving state upward to a common ancestor so sibling components can share it
C) Converting state to props
D) Removing state entirely
```

*Answer: B*

---

**Question 6:** Why is a discriminated union better than multiple boolean flags for request state?

```
A) It's shorter to write
B) It's faster to execute
C) It prevents impossible state combinations (e.g., isLoading AND isError both being true)
D) TypeScript requires it
```

*Answer: C*

---

## Summary

| Concept | What It Is | When to Use |
|---------|-----------|-------------|
| Props | Data from parent to child | Sharing data between adjacent components |
| Children prop | JSX between component tags | Building wrapper/layout components |
| Default props | Fallback prop values | Optional props with sensible defaults |
| State | Component-owned data | Any value that changes and drives UI |
| Derived state | Value computed from state | Avoid duplicating state |
| State lifting | Moving state up to common parent | Siblings sharing the same data |
| State colocation | Keeping state near where it's used | Avoiding unnecessary re-renders |
| Context | Global shared data | Auth, theme, locale, notifications |
| Context splitting | Multiple small contexts | Preventing unnecessary re-renders |
| Render props | Function prop returning JSX | Highly reusable, flexible components |

---

## Preparation for the Next Topic

Next file: **006-what-is-nextjs.md**

You've now mastered the React foundation.  
In the next file, you'll finally step into the main topic of this course: **Next.js**.

You'll learn:
- What problem Next.js solves that React alone can't
- The history of Next.js and why Vercel built it
- What "full-stack" means in the context of Next.js
- The difference between a library and a framework
- Why major companies choose Next.js over plain React

---

*Next: [006-what-is-nextjs.md](006-what-is-nextjs.md)*
