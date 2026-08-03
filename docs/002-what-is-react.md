# 002 — What Is React?

---

## Purpose of This Topic

You can't understand Next.js without understanding React first.  
Next.js is a framework built on top of React.  
That's why we start with React — to build a solid foundation.

In this file, you'll learn what React is, why it was invented, and what problem it solves.

---

## Why Does This Topic Exist?

Before 2013, web sites were built very differently.  
Tools like jQuery, vanilla JavaScript, and Backbone.js were used.  
Building large, complex applications with these tools turned into a nightmare.

Facebook engineers experienced this problem firsthand.  
News feed, like button, comments, notifications...  
Everything affected everything else.  
Change something in one place, and something else breaks.

They created React to solve this problem.  
They open-sourced it in 2013.  
And the world of web development changed forever.

---

## Where Is This Used in Real Projects?

React is used to build user interfaces (UI).  
Today, millions of websites around the world are built with React:

```
Facebook       → Where React was born
Instagram      → Meta's other product
Netflix        → Content discovery interface
Airbnb         → Booking platform
Twitter/X      → Timeline and notifications
Dropbox        → File management interface
Khan Academy   → Education platform
```

---

## Real-Life Analogy

To understand React, think of LEGO.

```
Without LEGO:
┌─────────────────────────────────────────────┐
│ You want to build a large sculpture.        │
│ But you only have one giant solid block.    │
│ If you want to change it, you have to       │
│ rebuild the whole thing from scratch.       │
└─────────────────────────────────────────────┘

With LEGO (React):
┌─────────────────────────────────────────────┐
│ You build the same sculpture with LEGO      │
│ pieces. If you only want to change the head │
│ → detach the head, attach a new one.        │
│ You don't need to touch the rest.           │
└─────────────────────────────────────────────┘
```

In React, each LEGO piece is a **Component**.  
Every component works independently.  
When one component changes, the others aren't affected.

---

## The World Before React

Before React was invented, web pages were built like this:

### The HTML + jQuery Era

```
User clicks a button
       ↓
jQuery updates the DOM
       ↓
Sometimes the whole page re-renders
       ↓
Another button changes something
       ↓
Another DOM update
       ↓
Code entangles and gets messy
       ↓
Very hard to debug
```

**Problems:**
- Code becomes harder to manage as it grows
- It's unclear what code is changing what
- Tracking the application's state is difficult
- Writing tests is nearly impossible
- Teamwork is very challenging

### The World With React

```
User clicks a button
       ↓
Component updates its own state
       ↓
React re-renders only the part that changed
       ↓
Other components are unaffected
       ↓
Code is clear, readable, and testable
```

---

## Simple Explanation

React is built on three core ideas:

### 1. Component-Based Architecture

Everything is a component.  
A button is a component.  
A navbar is a component.  
A page is a component.

```
Page
├── Navbar (component)
│   ├── Logo (component)
│   ├── Menu (component)
│   └── UserAvatar (component)
├── MainContent (component)
│   ├── ArticleCard (component)
│   ├── ArticleCard (component)
│   └── ArticleCard (component)
└── Footer (component)
```

### 2. Declarative Approach

You tell React **what you want**, not **how to do it**.

```
Old way (Imperative):
"Go find that div, change the textContent of the span inside it,
then set the background-color of that div to red,
then set display to none..."

React way (Declarative):
"When this condition is true, the component looks like this.
When this condition is false, the component looks like that."
```

### 3. One-Way Data Flow

Data always flows from top to bottom.

```
Parent Component
      │
      │ props
      ▼
Child Component
      │
      │ props
      ▼
Grandchild Component
```

This means you always know where data is changing.  
Debugging becomes much easier.

---

## Technical Explanation

### What Is JSX?

React uses a special syntax called **JSX**.  
JSX lets you write HTML-like code inside JavaScript.

```tsx
function Greeting() {
  const userName = "Sarah";

  return (
    <div className="greeting-container">
      <h1>Hello, {userName}!</h1>
      <p>You are learning React.</p>
    </div>
  );
}
```

JSX is actually JavaScript under the hood.  
Before it reaches the browser, it gets compiled into this:

```tsx
function Greeting() {
  const userName = "Sarah";

  return React.createElement(
    "div",
    { className: "greeting-container" },
    React.createElement("h1", null, "Hello, ", userName, "!"),
    React.createElement("p", null, "You are learning React.")
  );
}
```

You don't do this conversion yourself.  
Babel or the TypeScript compiler does it automatically.  
You just write JSX.

### Component Types

There are two types of components in React:

**1. Function Component (Modern, recommended)**
```tsx
function WelcomeCard({ name }: { name: string }) {
  return (
    <div>
      <h2>Welcome, {name}!</h2>
    </div>
  );
}
```

**2. Class Component (Old, no longer used)**
```tsx
class WelcomeCard extends React.Component<{ name: string }> {
  render() {
    return (
      <div>
        <h2>Welcome, {this.props.name}!</h2>
      </div>
    );
  }
}
```

**Only function components are used today.**  
You don't need to know class components.  
I mentioned them only so you'll recognize them in legacy codebases.

---

## Your First React Component

### A Simple Component

```tsx
type UserProfileProps = {
  fullName: string;
  email: string;
  role: "admin" | "user" | "moderator";
};

function UserProfile({ fullName, email, role }: UserProfileProps) {
  return (
    <div className="user-profile">
      <h2>{fullName}</h2>
      <p>{email}</p>
      <span className={`role-badge role-badge--${role}`}>{role}</span>
    </div>
  );
}
```

**Line-by-line explanation:**

`type UserProfileProps` → We defined the shape of the data this component will receive using TypeScript.  
`fullName: string` → fullName must be a string.  
`email: string` → email must be a string.  
`role: "admin" | "user" | "moderator"` → role can only be one of these three values.  
`function UserProfile(...)` → We defined a function component.  
`{ fullName, email, role }` → We destructured the props.  
`return (...)` → We're returning JSX.  
`className` → Used instead of HTML's `class` keyword (because `class` is a reserved word in JavaScript).  
`` `role-badge role-badge--${role}` `` → We created a dynamic CSS class using a template literal.

### Using the Component

```tsx
function App() {
  return (
    <div>
      <UserProfile
        fullName="John Smith"
        email="john@example.com"
        role="admin"
      />
      <UserProfile
        fullName="Jane Doe"
        email="jane@example.com"
        role="user"
      />
    </div>
  );
}
```

Back to the LEGO analogy:  
`UserProfile` is a LEGO piece we created.  
We used it as many times as we wanted inside `App`.  
Each uses different props but the same template.

---

## What Is State?

State is the information a component "remembers."

```
Think of a counter component:
- Initial value is 0
- User clicked the + button → value became 1
- User clicked the + button → value became 2
- User clicked the - button → value became 1

This changing number → state
```

When state changes, React re-renders the relevant component.

```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Value: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**Line-by-line explanation:**

`import { useState } from "react"` → We imported the useState hook from React.  
`useState(0)` → Creates a state with an initial value of 0.  
`const [count, setCount]` → `count` is the current value; `setCount` is the function that changes it.  
`onClick={() => setCount(count + 1)}` → When the button is clicked, increment count by 1.  
React sees that `count` changed and re-renders the component.

---

## What Are Props?

Props (properties) are how you pass data into a component from the outside.

```tsx
type ButtonProps = {
  label: string;
  variant: "primary" | "secondary" | "danger";
  onClick: () => void;
  disabled?: boolean;
};

function Button({ label, variant, onClick, disabled = false }: ButtonProps) {
  return (
    <button
      className={`btn btn--${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
}
```

Usage:

```tsx
function PaymentPage() {
  const handlePayment = () => {
    console.log("Payment processed");
  };

  return (
    <div>
      <Button label="Pay Now" variant="primary" onClick={handlePayment} />
      <Button label="Cancel" variant="secondary" onClick={() => {}} />
      <Button label="Delete" variant="danger" onClick={() => {}} disabled={true} />
    </div>
  );
}
```

**The golden rule of props:** Props only flow from top to bottom.  
A child component cannot directly change a parent's state.  
A child can send a message to the parent via a function prop.

---

## How React Works

```
1. Application starts
         │
         ▼
2. React renders all components
         │
         ▼
3. Virtual DOM is created (in memory)
         │
         ▼
4. Virtual DOM → painted to the real DOM (visible in browser)
         │
         ▼
5. User clicks something → State changes
         │
         ▼
6. React creates a new Virtual DOM
         │
         ▼
7. Old Virtual DOM is compared with new Virtual DOM (Diffing)
         │
         ▼
8. Only the different parts are updated in the real DOM
         │
         ▼
9. User sees the update
```

This process is called **Reconciliation**.  
You'll learn about Virtual DOM and Rendering in detail in the next file (003).

---

## Common Mistakes

### Mistake 1: Mutating State Directly

```tsx
function BadCounter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    count = count + 1;
  };

  return <button onClick={increment}>{count}</button>;
}
```

**Problem:** `count = count + 1` mutates state directly.  
React doesn't see this change. The screen never updates.

```tsx
function GoodCounter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(count + 1);
  };

  return <button onClick={increment}>{count}</button>;
}
```

**Correct:** Always use `setCount`. React only sees the change this way.

---

### Mistake 2: Using If-Else Blocks Inside JSX

```tsx
function BadConditional({ isLoggedIn }: { isLoggedIn: boolean }) {
  return (
    <div>
      {if (isLoggedIn) { <p>Welcome!</p> } else { <p>Please log in</p> }}
    </div>
  );
}
```

**Problem:** You cannot use an if-else block inside JSX.

```tsx
function GoodConditional({ isLoggedIn }: { isLoggedIn: boolean }) {
  return (
    <div>
      {isLoggedIn ? <p>Welcome!</p> : <p>Please log in</p>}
    </div>
  );
}
```

**Correct:** Use the ternary operator.

---

### Mistake 3: Not Using `key` When Rendering Lists

```tsx
function BadList({ items }: { items: string[] }) {
  return (
    <ul>
      {items.map((item) => (
        <li>{item}</li>
      ))}
    </ul>
  );
}
```

**Problem:** The `key` prop is missing on each list item. React will warn you, and performance issues may arise.

```tsx
function GoodList({ items }: { items: Array<{ id: string; label: string }> }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.label}</li>
      ))}
    </ul>
  );
}
```

**Correct:** Give each list item a unique and stable `key`.  
Don't use the index as the key (`key={index}`) — it causes bugs if the list order changes.

---

### Mistake 4: Starting Component Names With a Lowercase Letter

```tsx
function myButton() {
  return <button>Click me</button>;
}

function App() {
  return <myButton />;
}
```

**Problem:** React treats a lowercase name as an HTML element (like `<mybutton>`).

```tsx
function MyButton() {
  return <button>Click me</button>;
}

function App() {
  return <MyButton />;
}
```

**Correct:** Component names must always start with a capital letter (PascalCase).

---

### Mistake 5: Not Typing Props With TypeScript

```tsx
function ProductCard({ name, price, onAddToCart }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{price}</p>
      <button onClick={onAddToCart}>Add to Cart</button>
    </div>
  );
}
```

**Problem:** No TypeScript types. You have no idea what types `name`, `price`, or `onAddToCart` should be. This leads to runtime errors.

```tsx
type ProductCardProps = {
  name: string;
  price: number;
  onAddToCart: (productName: string) => void;
};

function ProductCard({ name, price, onAddToCart }: ProductCardProps) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{price} USD</p>
      <button onClick={() => onAddToCart(name)}>Add to Cart</button>
    </div>
  );
}
```

**Correct:** Always define TypeScript types for every component's props.

---

## Best Practices

### Component Size

Keep components small.  
A component should do one thing.  
If a component grows too large, break it into smaller pieces.

```
Bad:  One 300-line component
Good: Many small 10-50 line components
```

### Props With TypeScript

Define a TypeScript type for every component.  
Don't use `any`.  
Clearly specify the shape of the props.

```tsx
type ArticleCardProps = {
  id: string;
  title: string;
  excerpt: string;
  authorName: string;
  publishedAt: Date;
  readTimeMinutes: number;
  tags: string[];
  isFeatured: boolean;
};

function ArticleCard({
  id,
  title,
  excerpt,
  authorName,
  publishedAt,
  readTimeMinutes,
  tags,
  isFeatured,
}: ArticleCardProps) {
  return (
    <article className={`article-card ${isFeatured ? "article-card--featured" : ""}`}>
      <header>
        <h2>{title}</h2>
        <div className="article-meta">
          <span>{authorName}</span>
          <time dateTime={publishedAt.toISOString()}>
            {publishedAt.toLocaleDateString("en-US")}
          </time>
          <span>{readTimeMinutes} min read</span>
        </div>
      </header>
      <p>{excerpt}</p>
      <footer>
        {tags.map((tag) => (
          <span key={tag} className="tag">
            {tag}
          </span>
        ))}
      </footer>
    </article>
  );
}
```

---

### Recommended Folder Structure

```
src/
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Card.tsx
│   └── features/
│       ├── ProductCard.tsx
│       └── UserProfile.tsx
├── pages/
│   └── Home.tsx
└── App.tsx
```

---

## Real Company Examples

### Facebook News Feed Component Tree

```
NewsFeed (Component)
├── StoryBar (Component)
│   └── StoryItem (Component) × N
├── PostComposer (Component)
└── Post (Component) × N
    ├── PostHeader (Component)
    ├── PostContent (Component)
    ├── PostMedia (Component)
    ├── ReactionBar (Component)
    └── CommentSection (Component)
        └── Comment (Component) × N
```

Each is an independent component.  
When ReactionBar inside Post changes, the other parts of Post aren't affected.

### E-Commerce Product List

```
ProductListPage
├── FilterSidebar
│   ├── CategoryFilter
│   ├── PriceFilter
│   └── BrandFilter
├── ProductGrid
│   └── ProductCard × N
│       ├── ProductImage
│       ├── ProductInfo
│       │   ├── ProductName
│       │   ├── ProductPrice
│       │   └── ProductRating
│       └── AddToCartButton
└── Pagination
```

### Dashboard Application

```
DashboardPage
├── Sidebar
│   ├── NavLogo
│   ├── NavMenu
│   │   └── NavItem × N
│   └── UserInfo
├── TopBar
│   ├── SearchBar
│   ├── NotificationBell
│   └── UserMenu
└── MainContent
    ├── StatsGrid
    │   └── StatCard × 4
    ├── RevenueChart
    └── RecentOrdersTable
        └── OrderRow × N
```

---

## Mini Exercise

Write the following component with TypeScript:

**Task:**  
Write a `MovieCard` component.  
This component should accept:
- `title` (string)
- `director` (string)
- `year` (number)
- `genre` ("action" | "comedy" | "drama" | "thriller" | "sci-fi")
- `rating` (number — between 0 and 10)
- `durationMinutes` (number)
- `isNowPlaying` (boolean)

The component should display:
- Movie title as heading
- Director and year
- Genre badge
- Star rating (show `rating / 10`)
- Duration in hours and minutes
- "Now Playing" badge if `isNowPlaying` is true

**Bonus:** If `rating` is above 8.0, add a "Top Rated" badge.

---

## Mini Quiz

**Question 1:** React was developed by which company?

```
A) Google
B) Microsoft
C) Facebook (Meta)
D) Netflix
```

*Answer: C*

---

**Question 2:** What is JSX?

```
A) A new programming language
B) A syntax for writing HTML-like code inside JavaScript
C) A CSS system specific to React
D) A database query language
```

*Answer: B*

---

**Question 3:** Why is state changed with `setState` (or `setCount`)?

```
A) Because directly mutating is forbidden by the language
B) Because React only detects the change and re-renders this way
C) Because setState is faster
D) Because TypeScript enforces it
```

*Answer: B*

---

**Question 4:** Which of the following is an invalid component name?

```
A) ProductCard
B) userAvatar
C) NavigationMenu
D) ArticleList
```

*Answer: B — Starts with a lowercase letter*

---

**Question 5:** What is the fundamental difference between Props and State?

```
A) Props come from outside and are immutable; State is managed internally and can change
B) They are the same thing
C) State comes from outside; Props are managed internally
D) Props are faster
```

*Answer: A*

---

**Question 6:** What is the one-way data flow rule in React?

```
A) Data flows from child to parent
B) Data flows from parent to child via props
C) Data flows in both directions simultaneously
D) Data doesn't flow at all; it stays in place
```

*Answer: B*

---

## Summary

| Concept | Explanation |
|---------|-------------|
| React | UI library developed by Facebook |
| Component | Independent, reusable UI piece |
| JSX | HTML-like syntax inside JavaScript |
| Props | Data passed into a component from outside (immutable) |
| State | Data managed by the component itself (mutable) |
| useState | React hook that creates state |
| Reconciliation | React updating only the parts that changed |
| PascalCase | Requirement that component names start with a capital letter |
| One-way data flow | Data flows only from parent to child via props |

---

## Preparation for the Next Topic

Next file: **003-virtual-dom-and-rendering.md**

In that file, you'll learn:
- What exactly is the Virtual DOM?
- How is it different from the real DOM?
- How does the Reconciliation algorithm work?
- What is rendering? What is re-rendering?
- Where does React's performance advantage come from?

Before reading that file, think about this question:  
"Why might changing a single text element on a web page take time?"

---

*Next: [003-virtual-dom-and-rendering.md](003-virtual-dom-and-rendering.md)*
