# 004 — React Lifecycle and Hooks

---

## Purpose of This Topic

Every React component has a lifecycle — it's born, it changes, and it disappears.  
Understanding this lifecycle is fundamental to writing correct, bug-free React code.

Hooks are the mechanism that lets you tap into this lifecycle from function components.  
Almost every React pattern you'll encounter in real projects depends on  
understanding Hooks deeply.

---

## Why Does This Topic Exist?

Before React 16.8 (February 2019), class components had lifecycle methods:  
`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`, and others.

These methods were powerful but had problems:
- Unrelated logic was often mixed together in a single lifecycle method
- Related logic was split across multiple lifecycle methods
- Sharing stateful logic between components required complex patterns (HOC, Render Props)
- Class syntax was verbose and confusing (especially `this` binding)

Hooks solved all of these problems.  
They let you use state and lifecycle features in function components,  
and they let you extract and share stateful logic between components cleanly.

Today, **all modern React code uses Hooks exclusively.**  
Class components exist in legacy codebases. You should recognize them,  
but you'll write everything with Hooks.

---

## Where Is This Used in Real Projects?

Hooks are used **everywhere** in React development:

```
useState    → Every interactive UI element (inputs, toggles, counters)
useEffect   → Data fetching, subscriptions, DOM manipulation, timers
useRef      → Focus management, measuring elements, storing mutable values
useContext  → Sharing theme, auth state, language settings globally
useReducer  → Complex state logic (shopping carts, multi-step forms)
useMemo     → Expensive calculations that shouldn't re-run on every render
useCallback → Stable function references to prevent child re-renders
```

In a typical enterprise Next.js application, you'll use all of these daily.

---

## Real-Life Analogy

The lifecycle of a React component is like the lifecycle of a light bulb:

```
Mount (Birth):
  The bulb is screwed into the socket.
  Power flows. It lights up.
  → componentDidMount / useEffect (empty deps)

Update:
  Someone adjusts the dimmer switch.
  The bulb gets brighter or dimmer.
  → componentDidUpdate / useEffect (with deps)

Unmount (Death):
  The bulb is unscrewed.
  Power is cut. It goes dark.
  → componentWillUnmount / useEffect cleanup function
```

With Hooks, all three phases are handled in a single, clean `useEffect` call.

---

## Simple Explanation

### The Component Lifecycle

```
1. MOUNTING
   Component appears on screen for the first time.
   → useEffect with empty dependency array runs once.

2. UPDATING
   State or props change.
   Component re-renders.
   → useEffect with dependencies runs when those dependencies change.

3. UNMOUNTING
   Component is removed from the screen.
   → useEffect's cleanup function runs.
```

### What Are Hooks?

Hooks are special functions that start with `use`.  
They let you "hook into" React's internal features from function components.

```
Built-in Hooks (most common):
  useState       → Manage local state
  useEffect      → Run code after render / manage side effects
  useRef         → Store a mutable value without triggering re-renders
  useContext     → Read a context value
  useReducer     → More complex state management
  useMemo        → Memoize expensive values
  useCallback    → Memoize functions
  useId          → Generate unique IDs
  useTransition  → Mark updates as non-urgent
```

---

## Technical Explanation

### Rules of Hooks

React has two mandatory rules for Hooks:

**Rule 1: Only call Hooks at the top level**

```tsx
function BadComponent() {
  if (someCondition) {
    const [count, setCount] = useState(0);
  }
  return <div />;
}
```

Never call Hooks inside conditions, loops, or nested functions.  
React relies on the **order of Hook calls** to map state to the right Hook.  
If the order changes between renders, everything breaks.

```tsx
function GoodComponent() {
  const [count, setCount] = useState(0);
  const [isVisible, setIsVisible] = useState(true);

  return <div>{isVisible && <p>{count}</p>}</div>;
}
```

Correct: All Hooks are called unconditionally at the top level.

**Rule 2: Only call Hooks from React functions**

Call Hooks only from:
- React function components
- Custom Hooks (functions that start with `use`)

Never call Hooks from regular JavaScript functions, class methods, or event handlers.

---

## useState — Deep Dive

`useState` is the most fundamental Hook.  
It gives your component memory — the ability to hold and update a value across renders.

### Basic Usage

```tsx
import { useState } from "react";

function TemperatureConverter() {
  const [celsius, setCelsius] = useState<number>(0);

  const fahrenheit = (celsius * 9) / 5 + 32;

  return (
    <div className="converter">
      <label htmlFor="celsius-input">Celsius:</label>
      <input
        id="celsius-input"
        type="number"
        value={celsius}
        onChange={(e) => setCelsius(Number(e.target.value))}
      />
      <p>{celsius}°C = {fahrenheit.toFixed(1)}°F</p>
    </div>
  );
}
```

**Line-by-line:**

`useState<number>(0)` → Creates state of type `number` with initial value `0`.  
`const fahrenheit = (celsius * 9) / 5 + 32` → Derived value. No state needed — just calculated from state.  
`Number(e.target.value)` → Input values are strings; we convert to number.

### Functional Updates

When the new state depends on the previous state, use the functional form:

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  const incrementThreeTimes = () => {
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={incrementThreeTimes}>Add 3</button>
    </div>
  );
}
```

**Why the functional form?**  
If you wrote `setCount(count + 1)` three times, React batches the updates.  
`count` is still `0` in all three calls (it's a stale closure).  
Result: count goes from `0` to `1`, not `3`.

With `(prev) => prev + 1`, each call gets the latest value.  
Result: count goes from `0` to `3`. Correct.

### State With Objects

```tsx
type UserSettings = {
  theme: "light" | "dark";
  language: "en" | "es" | "fr";
  notificationsEnabled: boolean;
  fontSize: number;
};

function SettingsPanel() {
  const [settings, setSettings] = useState<UserSettings>({
    theme: "light",
    language: "en",
    notificationsEnabled: true,
    fontSize: 16,
  });

  const updateTheme = (theme: UserSettings["theme"]) => {
    setSettings((prev) => ({ ...prev, theme }));
  };

  const updateFontSize = (fontSize: number) => {
    setSettings((prev) => ({ ...prev, fontSize }));
  };

  return (
    <div className="settings-panel">
      <div>
        <label>Theme:</label>
        <select
          value={settings.theme}
          onChange={(e) => updateTheme(e.target.value as UserSettings["theme"])}
        >
          <option value="light">Light</option>
          <option value="dark">Dark</option>
        </select>
      </div>
      <div>
        <label>Font Size: {settings.fontSize}px</label>
        <input
          type="range"
          min={12}
          max={24}
          value={settings.fontSize}
          onChange={(e) => updateFontSize(Number(e.target.value))}
        />
      </div>
    </div>
  );
}
```

**Key point:** `{ ...prev, theme }` spreads the previous state and overrides only `theme`.  
You must spread the previous state when updating object state — otherwise you lose other fields.

---

## useEffect — Deep Dive

`useEffect` is where you handle **side effects** — anything that reaches outside  
the pure render cycle: data fetching, subscriptions, DOM mutations, timers.

### The Three Dependency Patterns

```tsx
import { useEffect } from "react";

function ExampleComponent() {
  useEffect(() => {
    // Pattern 1: No dependency array
    // Runs after EVERY render (initial + every re-render)
    // Rarely what you want
  });

  useEffect(() => {
    // Pattern 2: Empty dependency array []
    // Runs ONCE after the initial render
    // Equivalent to componentDidMount
  }, []);

  useEffect(() => {
    // Pattern 3: With dependencies [dep1, dep2]
    // Runs after initial render AND whenever dep1 or dep2 changes
    // Equivalent to componentDidUpdate for those specific values
  }, []);

  return <div />;
}
```

### Data Fetching Pattern

```tsx
"use client";

import { useState, useEffect } from "react";

type Post = {
  id: number;
  title: string;
  body: string;
  userId: number;
};

type FetchState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };

function PostViewer({ postId }: { postId: number }) {
  const [fetchState, setFetchState] = useState<FetchState<Post>>({
    status: "idle",
  });

  useEffect(() => {
    if (postId <= 0) return;

    let cancelled = false;

    setFetchState({ status: "loading" });

    fetch(`https://jsonplaceholder.typicode.com/posts/${postId}`)
      .then((response) => {
        if (!response.ok) {
          throw new Error(`HTTP error: ${response.status}`);
        }
        return response.json() as Promise<Post>;
      })
      .then((data) => {
        if (!cancelled) {
          setFetchState({ status: "success", data });
        }
      })
      .catch((error: Error) => {
        if (!cancelled) {
          setFetchState({ status: "error", error: error.message });
        }
      });

    return () => {
      cancelled = true;
    };
  }, [postId]);

  if (fetchState.status === "idle") return <p>Select a post.</p>;
  if (fetchState.status === "loading") return <p>Loading...</p>;
  if (fetchState.status === "error") return <p>Error: {fetchState.error}</p>;

  const { data: post } = fetchState;

  return (
    <article>
      <h2>{post.title}</h2>
      <p>{post.body}</p>
    </article>
  );
}
```

**Line-by-line explanation:**

`let cancelled = false` → A flag to handle the race condition.  
The cleanup function sets `cancelled = true`.  
If `postId` changes before the fetch completes, the old effect cleans up.  
The `if (!cancelled)` checks prevent the old response from updating state  
after the component has moved on to a new request.

This is the correct pattern for async operations in `useEffect`.

### Subscription Pattern (Cleanup Required)

```tsx
"use client";

import { useState, useEffect } from "react";

function WindowSizeTracker() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return (
    <p>
      Window: {windowSize.width}px × {windowSize.height}px
    </p>
  );
}
```

**The cleanup function (`return () => {...}`) is critical here.**  
Without it, every time the component mounts, a new event listener is added.  
When the component unmounts, the listener stays attached — a memory leak.  
The cleanup function removes the listener on unmount.

### Timer Pattern

```tsx
"use client";

import { useState, useEffect } from "react";

function CountdownTimer({ startSeconds }: { startSeconds: number }) {
  const [secondsLeft, setSecondsLeft] = useState(startSeconds);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;
    if (secondsLeft <= 0) {
      setIsRunning(false);
      return;
    }

    const intervalId = setInterval(() => {
      setSecondsLeft((prev) => prev - 1);
    }, 1000);

    return () => {
      clearInterval(intervalId);
    };
  }, [isRunning, secondsLeft]);

  const minutes = Math.floor(secondsLeft / 60);
  const seconds = secondsLeft % 60;

  return (
    <div className="countdown">
      <p className="time-display">
        {String(minutes).padStart(2, "0")}:{String(seconds).padStart(2, "0")}
      </p>
      <button onClick={() => setIsRunning((prev) => !prev)}>
        {isRunning ? "Pause" : "Start"}
      </button>
      <button onClick={() => { setSecondsLeft(startSeconds); setIsRunning(false); }}>
        Reset
      </button>
    </div>
  );
}
```

**Cleanup:** `clearInterval(intervalId)` removes the timer when the effect re-runs  
or when the component unmounts. Without this, multiple intervals would stack up.

---

## useRef — Deep Dive

`useRef` stores a mutable value that:
1. Persists across renders (like state)
2. Does NOT trigger a re-render when changed (unlike state)

It's also the standard way to access DOM elements directly.

### DOM Access

```tsx
"use client";

import { useRef, useEffect } from "react";

function SearchBar({ autoFocus = false }: { autoFocus?: boolean }) {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    if (autoFocus && inputRef.current) {
      inputRef.current.focus();
    }
  }, [autoFocus]);

  const handleClear = () => {
    if (inputRef.current) {
      inputRef.current.value = "";
      inputRef.current.focus();
    }
  };

  return (
    <div className="search-bar">
      <input
        ref={inputRef}
        type="search"
        placeholder="Search..."
        className="search-input"
      />
      <button onClick={handleClear} className="clear-button">
        Clear
      </button>
    </div>
  );
}
```

`useRef<HTMLInputElement>(null)` → Creates a ref typed to an HTMLInputElement.  
`ref={inputRef}` → Attaches the ref to the DOM element. After mount, `inputRef.current` points to the real `<input>` element.  
`inputRef.current.focus()` → Directly calls the DOM's focus method.

### Storing Previous Values

```tsx
"use client";

import { useState, useEffect, useRef } from "react";

function PriceTracker({ currentPrice }: { currentPrice: number }) {
  const previousPriceRef = useRef<number>(currentPrice);

  useEffect(() => {
    previousPriceRef.current = currentPrice;
  });

  const previousPrice = previousPriceRef.current;
  const priceChange = currentPrice - previousPrice;
  const isIncreasing = priceChange > 0;
  const isDecreasing = priceChange < 0;

  return (
    <div className="price-tracker">
      <span className="current-price">${currentPrice.toFixed(2)}</span>
      {isIncreasing && (
        <span className="price-change price-change--up">▲ +${priceChange.toFixed(2)}</span>
      )}
      {isDecreasing && (
        <span className="price-change price-change--down">▼ ${priceChange.toFixed(2)}</span>
      )}
    </div>
  );
}
```

**Why ref instead of state?**  
We need to remember the previous value, but changing it shouldn't cause a re-render.  
If we used state for `previousPrice`, updating it would trigger an extra render.

---

## useContext — Deep Dive

`useContext` lets you share data across many components without prop drilling.

### The Problem: Prop Drilling

```
App (has user data)
  │
  ├── Header (receives user prop but doesn't use it)
  │     └── UserMenu (receives user prop and displays it)
  │
  └── Sidebar (receives user prop but doesn't use it)
        └── UserProfile (receives user prop and displays it)
```

`Header` and `Sidebar` pass `user` down even though they don't need it.  
This is **prop drilling** — it makes code harder to maintain.

### The Solution: Context

```tsx
import { createContext, useContext, useState } from "react";

type User = {
  id: string;
  name: string;
  email: string;
  role: "admin" | "user";
};

type AuthContextValue = {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
};

const AuthContext = createContext<AuthContextValue | null>(null);

function useAuth(): AuthContextValue {
  const context = useContext(AuthContext);
  if (context === null) {
    throw new Error("useAuth must be used inside AuthProvider");
  }
  return context;
}

function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = (userData: User) => {
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function UserMenu() {
  const { user, logout } = useAuth();

  if (!user) {
    return <a href="/login">Log in</a>;
  }

  return (
    <div className="user-menu">
      <span>{user.name}</span>
      <button onClick={logout}>Log out</button>
    </div>
  );
}

function AdminBadge() {
  const { user } = useAuth();

  if (!user || user.role !== "admin") return null;

  return <span className="admin-badge">Admin</span>;
}

function App() {
  return (
    <AuthProvider>
      <header>
        <UserMenu />
        <AdminBadge />
      </header>
    </AuthProvider>
  );
}
```

**Key points:**

`createContext<AuthContextValue | null>(null)` → The `null` default means "no provider found."  
`useAuth()` → A custom Hook that wraps `useContext`. The `null` check provides a  
helpful error if you accidentally use `useAuth` outside the `AuthProvider`.  
`AuthContext.Provider value={...}` → Any component inside this tree can access the value.

---

## useReducer — Deep Dive

`useReducer` is an alternative to `useState` for complex state logic.  
It's inspired by Redux. You define a **reducer function** that handles different **actions**.

### When to Use `useReducer` Instead of `useState`

```
Use useState when:
  - Simple on/off toggle
  - Single counter
  - Simple string/number updates

Use useReducer when:
  - Multiple sub-values in one state object
  - Next state depends on previous state in complex ways
  - Many different state transitions to handle
  - State logic you want to extract and test separately
```

### Shopping Cart Example

```tsx
"use client";

import { useReducer } from "react";

type CartItem = {
  id: string;
  name: string;
  price: number;
  quantity: number;
};

type CartState = {
  items: CartItem[];
  isOpen: boolean;
};

type CartAction =
  | { type: "ADD_ITEM"; item: Omit<CartItem, "quantity"> }
  | { type: "REMOVE_ITEM"; itemId: string }
  | { type: "UPDATE_QUANTITY"; itemId: string; quantity: number }
  | { type: "CLEAR_CART" }
  | { type: "TOGGLE_CART" };

function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case "ADD_ITEM": {
      const existingItemIndex = state.items.findIndex(
        (item) => item.id === action.item.id
      );

      if (existingItemIndex !== -1) {
        const updatedItems = state.items.map((item, index) =>
          index === existingItemIndex
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
        return { ...state, items: updatedItems };
      }

      return {
        ...state,
        items: [...state.items, { ...action.item, quantity: 1 }],
      };
    }

    case "REMOVE_ITEM": {
      return {
        ...state,
        items: state.items.filter((item) => item.id !== action.itemId),
      };
    }

    case "UPDATE_QUANTITY": {
      if (action.quantity <= 0) {
        return {
          ...state,
          items: state.items.filter((item) => item.id !== action.itemId),
        };
      }

      return {
        ...state,
        items: state.items.map((item) =>
          item.id === action.itemId
            ? { ...item, quantity: action.quantity }
            : item
        ),
      };
    }

    case "CLEAR_CART": {
      return { ...state, items: [] };
    }

    case "TOGGLE_CART": {
      return { ...state, isOpen: !state.isOpen };
    }
  }
}

function ShoppingCart() {
  const [cart, dispatch] = useReducer(cartReducer, {
    items: [],
    isOpen: false,
  });

  const totalPrice = cart.items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );

  const totalItems = cart.items.reduce((sum, item) => sum + item.quantity, 0);

  return (
    <div>
      <button onClick={() => dispatch({ type: "TOGGLE_CART" })}>
        Cart ({totalItems})
      </button>

      {cart.isOpen && (
        <div className="cart-panel">
          {cart.items.length === 0 ? (
            <p>Your cart is empty.</p>
          ) : (
            <>
              {cart.items.map((item) => (
                <div key={item.id} className="cart-item">
                  <span>{item.name}</span>
                  <input
                    type="number"
                    min={0}
                    value={item.quantity}
                    onChange={(e) =>
                      dispatch({
                        type: "UPDATE_QUANTITY",
                        itemId: item.id,
                        quantity: Number(e.target.value),
                      })
                    }
                  />
                  <span>${(item.price * item.quantity).toFixed(2)}</span>
                  <button
                    onClick={() =>
                      dispatch({ type: "REMOVE_ITEM", itemId: item.id })
                    }
                  >
                    Remove
                  </button>
                </div>
              ))}
              <div className="cart-total">Total: ${totalPrice.toFixed(2)}</div>
              <button onClick={() => dispatch({ type: "CLEAR_CART" })}>
                Clear Cart
              </button>
            </>
          )}
        </div>
      )}
    </div>
  );
}
```

**Key points:**

`CartAction` is a **discriminated union** — each action has a `type` that narrows TypeScript's type.  
When you write `case "ADD_ITEM"`, TypeScript knows `action.item` exists.  
The reducer is a **pure function** — same inputs always produce same output, no side effects.  
This makes it easy to test in isolation.

---

## useMemo and useCallback

These two Hooks are optimization tools.  
They prevent unnecessary work on re-renders.

### useMemo — Memoize Expensive Values

```tsx
import { useState, useMemo } from "react";

type Product = {
  id: string;
  name: string;
  price: number;
  category: string;
  rating: number;
};

function ProductDashboard({ products }: { products: Product[] }) {
  const [searchQuery, setSearchQuery] = useState("");
  const [minRating, setMinRating] = useState(0);
  const [sortBy, setSortBy] = useState<"price" | "rating" | "name">("name");

  const filteredAndSortedProducts = useMemo(() => {
    const lowercaseQuery = searchQuery.toLowerCase();

    const filtered = products.filter(
      (product) =>
        product.name.toLowerCase().includes(lowercaseQuery) &&
        product.rating >= minRating
    );

    return filtered.sort((a, b) => {
      if (sortBy === "price") return a.price - b.price;
      if (sortBy === "rating") return b.rating - a.rating;
      return a.name.localeCompare(b.name);
    });
  }, [products, searchQuery, minRating, sortBy]);

  return (
    <div>
      <input
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        placeholder="Search products..."
      />
      <input
        type="range"
        min={0}
        max={5}
        step={0.5}
        value={minRating}
        onChange={(e) => setMinRating(Number(e.target.value))}
      />
      <select
        value={sortBy}
        onChange={(e) => setSortBy(e.target.value as typeof sortBy)}
      >
        <option value="name">Name</option>
        <option value="price">Price</option>
        <option value="rating">Rating</option>
      </select>
      <p>{filteredAndSortedProducts.length} products found</p>
      <ul>
        {filteredAndSortedProducts.map((product) => (
          <li key={product.id}>
            {product.name} — ${product.price} — ★{product.rating}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

Without `useMemo`, the filter and sort would re-run on every render —  
even if the user is doing something completely unrelated to products.  
With `useMemo`, it only re-runs when `products`, `searchQuery`, `minRating`, or `sortBy` change.

### useCallback — Memoize Functions

```tsx
import { useState, useCallback, memo } from "react";

type TodoItem = {
  id: string;
  text: string;
  completed: boolean;
};

const TodoItemComponent = memo(function TodoItemComponent({
  item,
  onToggle,
  onDelete,
}: {
  item: TodoItem;
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
}) {
  return (
    <li className={item.completed ? "completed" : ""}>
      <span onClick={() => onToggle(item.id)}>{item.text}</span>
      <button onClick={() => onDelete(item.id)}>Delete</button>
    </li>
  );
});

function TodoList() {
  const [todos, setTodos] = useState<TodoItem[]>([]);
  const [inputValue, setInputValue] = useState("");

  const handleToggle = useCallback((id: string) => {
    setTodos((prev) =>
      prev.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []);

  const handleDelete = useCallback((id: string) => {
    setTodos((prev) => prev.filter((todo) => todo.id !== id));
  }, []);

  const handleAdd = () => {
    if (!inputValue.trim()) return;
    const newTodo: TodoItem = {
      id: crypto.randomUUID(),
      text: inputValue.trim(),
      completed: false,
    };
    setTodos((prev) => [...prev, newTodo]);
    setInputValue("");
  };

  return (
    <div>
      <div>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyDown={(e) => e.key === "Enter" && handleAdd()}
          placeholder="Add a todo..."
        />
        <button onClick={handleAdd}>Add</button>
      </div>
      <ul>
        {todos.map((todo) => (
          <TodoItemComponent
            key={todo.id}
            item={todo}
            onToggle={handleToggle}
            onDelete={handleDelete}
          />
        ))}
      </ul>
    </div>
  );
}
```

`useCallback(fn, [])` → The function is only created once (empty deps). The same reference is passed to `TodoItemComponent` on every render.  
`memo(Component)` → `TodoItemComponent` only re-renders if its props actually changed. Since `handleToggle` and `handleDelete` are stable, typing in the input no longer re-renders all todo items.

---

## Custom Hooks

Custom Hooks let you extract stateful logic from components and reuse it.  
Any function starting with `use` that calls other Hooks is a custom Hook.

```tsx
import { useState, useEffect, useCallback } from "react";

type FetchResult<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };

function useFetch<T>(url: string): FetchResult<T> & { refetch: () => void } {
  const [result, setResult] = useState<FetchResult<T>>({ status: "idle" });
  const [refetchTrigger, setRefetchTrigger] = useState(0);

  const refetch = useCallback(() => {
    setRefetchTrigger((prev) => prev + 1);
  }, []);

  useEffect(() => {
    if (!url) return;

    let cancelled = false;
    setResult({ status: "loading" });

    fetch(url)
      .then((response) => {
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json() as Promise<T>;
      })
      .then((data) => {
        if (!cancelled) setResult({ status: "success", data });
      })
      .catch((error: Error) => {
        if (!cancelled) setResult({ status: "error", error: error.message });
      });

    return () => {
      cancelled = true;
    };
  }, [url, refetchTrigger]);

  return { ...result, refetch };
}

type Post = {
  id: number;
  title: string;
  body: string;
};

function PostList() {
  const { status, refetch, ...state } =
    useFetch<Post[]>("https://jsonplaceholder.typicode.com/posts?_limit=5");

  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      {status === "loading" && <p>Loading...</p>}
      {status === "error" && "error" in state && <p>Error: {state.error}</p>}
      {status === "success" && "data" in state && (
        <ul>
          {state.data.map((post) => (
            <li key={post.id}>{post.title}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

`useFetch` can now be used in any component that needs to fetch data.  
Zero duplication. All the loading/error/success logic lives in one place.

---

## Common Mistakes

### Mistake 1: Missing Dependencies in useEffect

```tsx
function BadComponent({ userId }: { userId: string }) {
  const [userData, setUserData] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(setUserData);
  }, []);
}
```

**Problem:** `userId` is used inside the effect but not listed as a dependency.  
If `userId` changes, the effect won't re-run. You'll show stale data.

```tsx
function GoodComponent({ userId }: { userId: string }) {
  const [userData, setUserData] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(setUserData);
  }, [userId]);
}
```

**Correct:** Every value used inside `useEffect` must be in the dependency array.

---

### Mistake 2: Not Cleaning Up Effects

```tsx
function BadTimerComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setInterval(() => {
      setCount((prev) => prev + 1);
    }, 1000);
  }, []);

  return <p>{count}</p>;
}
```

**Problem:** No cleanup. Every time the component mounts, a new interval starts.  
If the component mounts and unmounts repeatedly, multiple intervals stack up.  
The count increments faster and faster.

```tsx
function GoodTimerComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount((prev) => prev + 1);
    }, 1000);

    return () => {
      clearInterval(intervalId);
    };
  }, []);

  return <p>{count}</p>;
}
```

**Correct:** Always return a cleanup function from effects that set up subscriptions or timers.

---

### Mistake 3: Overusing useMemo and useCallback

```tsx
function BadComponent({ name }: { name: string }) {
  const greeting = useMemo(() => `Hello, ${name}!`, [name]);

  return <p>{greeting}</p>;
}
```

**Problem:** `useMemo` has overhead. Creating a string template literal is trivially cheap.  
`useMemo` here is slower than just computing the value directly.

```tsx
function GoodComponent({ name }: { name: string }) {
  const greeting = `Hello, ${name}!`;

  return <p>{greeting}</p>;
}
```

**Rule of thumb:** Only use `useMemo` and `useCallback` when:
- You have measured a real performance problem
- The computation is genuinely expensive (complex sort/filter, heavy calculation)
- You're preventing unnecessary re-renders of memoized child components

---

## Best Practices

### Group Related State

```tsx
function GoodForm() {
  const [formData, setFormData] = useState({
    firstName: "",
    lastName: "",
    email: "",
    age: 0,
  });

  const updateField = <K extends keyof typeof formData>(
    field: K,
    value: (typeof formData)[K]
  ) => {
    setFormData((prev) => ({ ...prev, [field]: value }));
  };

  return (
    <form>
      <input
        value={formData.firstName}
        onChange={(e) => updateField("firstName", e.target.value)}
        placeholder="First name"
      />
      <input
        value={formData.email}
        onChange={(e) => updateField("email", e.target.value)}
        placeholder="Email"
      />
    </form>
  );
}
```

Related state lives together. One update function handles all fields.  
But keep unrelated state separate — don't put everything in one giant object.

---

## Real Company Examples

### Stripe Dashboard

Stripe's dashboard uses `useReducer` for payment form state:
- Multiple fields (card number, expiry, CVC)
- Complex validation logic per field
- Submission states (idle, validating, submitting, success, error)
- The reducer handles all transitions cleanly

### Vercel Deployment UI

Vercel's deployment page uses `useEffect` + WebSocket subscription:
- Connects to a WebSocket on mount
- Receives log entries in real time
- Disconnects and cleans up on unmount
- Re-connects if the deployment ID changes

---

## Mini Exercise

Build a `useLocalStorage` custom Hook:

```
Requirements:
1. Takes a key (string) and an initial value
2. Reads from localStorage on first render
3. Writes to localStorage whenever the value changes
4. Returns [value, setValue] like useState
5. Handles JSON serialization/deserialization
6. Handles errors gracefully (localStorage may be unavailable)
7. Types everything with TypeScript generics
```

The signature should be:
```tsx
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void]
```

---

## Mini Quiz

**Question 1:** What are the two rules of Hooks?

```
A) Call at top level; call only from React functions
B) Call with useEffect; call with useState
C) Call in classes; call in functions
D) Call once; call twice
```

*Answer: A*

---

**Question 2:** What does the cleanup function in `useEffect` do?

```
A) Clears the component's state
B) Removes the component from the DOM
C) Runs before the next effect and on unmount, cleaning up subscriptions/timers
D) Fetches fresh data
```

*Answer: C*

---

**Question 3:** When should you use `useReducer` instead of `useState`?

```
A) When you want the code to look more like Redux
B) When state is simple (a toggle or counter)
C) When state logic is complex with many transitions, or you need testable state logic
D) When you need to fetch data
```

*Answer: C*

---

**Question 4:** What is `useMemo` used for?

```
A) Memorizing your component's render history
B) Caching expensive computed values so they don't re-compute on every render
C) Making your component render faster by skipping it entirely
D) Storing values in localStorage
```

*Answer: B*

---

**Question 5:** What is a custom Hook?

```
A) A Hook provided by a third-party library
B) A regular function that starts with "use" and calls other Hooks
C) A class-based alternative to function components
D) A Hook that only works in TypeScript
```

*Answer: B*

---

**Question 6:** What is the functional update pattern in `useState`?

```
A) setCount(count + 1) — always use the current count
B) setCount((prev) => prev + 1) — always use the latest state value
C) setCount(useState(count + 1)) — reinitialize state
D) setCount(null) — clear the state
```

*Answer: B*

---

## Summary

| Hook | Purpose | When to Use |
|------|---------|-------------|
| `useState` | Local state | Any value that changes and triggers UI updates |
| `useEffect` | Side effects | Data fetching, subscriptions, timers, DOM mutation |
| `useRef` | Mutable value / DOM access | When you need to mutate without re-rendering, or access DOM |
| `useContext` | Consume context | Avoid prop drilling for global values |
| `useReducer` | Complex state | Multiple transitions, complex logic, testable reducers |
| `useMemo` | Cache values | Expensive calculations |
| `useCallback` | Cache functions | Stable references for memoized children |
| Custom Hooks | Reusable logic | Extract shared stateful behavior |

---

## Preparation for the Next Topic

Next file: **005-props-state-context.md**

In that file, you'll do a focused deep dive on:
- Props patterns: default props, spread props, children, render props
- State lifting strategies
- Context API patterns used in production
- How to structure shared state in a large application
- The difference between local, shared, and global state

Before reading: Think about how data flows in an application you've seen or built.  
Where does the data live? How does it get to where it's needed?

---

*Next: [005-props-state-context.md](005-props-state-context.md)*
