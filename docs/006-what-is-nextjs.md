# 006 — What Is Next.js?

---

## Purpose of This Topic

This is where the course truly begins.  
Everything you've learned about React so far is the foundation.  
Now we're building the house on top of that foundation.

In this file, you'll understand what Next.js is, why it exists,  
and why it has become the dominant way to build React applications in production.

---

## Why Does This Topic Exist?

React is a library. It solves one problem beautifully: building user interfaces.  
But when you try to build a real, production-grade web application with React alone,  
you immediately run into problems React doesn't solve:

```
Problem 1: SEO
  React renders in the browser.
  Search engine crawlers see an empty HTML page.
  Your content is invisible to Google.

Problem 2: Performance (Initial Load)
  Users download a large JavaScript bundle.
  The browser runs it. React renders.
  Only then does the user see anything.
  This takes 2-5 seconds on average connections.

Problem 3: Routing
  React has no built-in router.
  You need to install react-router-dom.
  Configure it. Handle nested routes manually.

Problem 4: Data Fetching
  React has no built-in data fetching.
  You write useEffect + fetch on every page.
  No caching. No deduplication. No server-side loading.

Problem 5: API
  React is frontend only.
  To connect to a database, you need a separate backend.
  Node.js + Express? Nest.js? A completely separate repo?

Problem 6: Build Configuration
  Webpack, Babel, TypeScript, ESLint, environment variables...
  Each one needs configuration. This takes days to set up correctly.
```

Next.js solves **all of these problems** with a single, cohesive framework.

---

## Where Is This Used in Real Projects?

Next.js is used by some of the most visited websites in the world:

```
Vercel         → Their own product
TikTok         → For web experience
Twitch         → Clip pages
Netflix         → Jobs site and marketing pages
GitHub         → Various pages
Shopify        → Documentation and marketing
Hulu           → Web application
AT&T           → Customer portal
HashiCorp      → Documentation site
Notion         → Marketing site
Loom           → Product marketing
```

In 2024-2025, Next.js is the most popular React framework by a significant margin.  
On npm, it's downloaded over 7 million times per week.

---

## Real-Life Analogy

Think of the difference between raw building materials and a pre-fabricated house kit.

```
React (raw materials):
  You have concrete, bricks, steel, wood.
  You can build any house you want.
  But you need to figure out everything yourself:
  - How to lay the foundation
  - How to wire electricity
  - How to plumb the pipes
  - How to insulate the walls

Next.js (pre-fabricated house kit):
  You receive all the materials AND:
  - A detailed blueprint
  - Pre-cut pieces that fit together
  - Built-in electrical conduits
  - Pre-installed plumbing routes
  - Insulation included

  You still build the house (write your code).
  But all the infrastructure decisions are made for you.
  You focus on what makes YOUR house unique.
```

Next.js gives you React, plus all the infrastructure decisions made and configured  
correctly by a team of experts at Vercel.

---

## Simple Explanation

Next.js is a framework for building web applications with React.  
It adds things React doesn't have:

```
React gives you:
  ✓ Components
  ✓ State management
  ✓ Event handling
  ✓ The Virtual DOM

Next.js additionally gives you:
  ✓ File-based routing (no router configuration)
  ✓ Server-Side Rendering (SEO + performance)
  ✓ Static Site Generation (blazing fast pages)
  ✓ API Routes (backend endpoints in the same project)
  ✓ Image optimization
  ✓ Font optimization
  ✓ Built-in TypeScript support
  ✓ Built-in ESLint configuration
  ✓ Middleware
  ✓ Edge Runtime
  ✓ Server Actions (server functions called from the client)
  ✓ Incremental Static Regeneration
  ✓ Automatic code splitting
  ✓ Zero-config Webpack/Turbopack setup
```

All of this, out of the box, with one command.

---

## Technical Explanation

### Library vs Framework

Understanding the difference between a library and a framework is crucial:

```
LIBRARY:
  You are in control.
  You call the library's functions when you need them.
  React is a library.
  
  You decide:
    - When to render
    - What routing library to use
    - How to fetch data
    - How to handle the build

FRAMEWORK:
  The framework is in control.
  The framework calls your code at the right time.
  Next.js is a framework.
  
  Next.js decides:
    - When server code runs
    - When static pages are generated
    - How routing works
    - How the build is structured
  
  You fill in the blanks: your components, your data, your logic.
```

This is called the **Inversion of Control** principle.  
You write the code; the framework decides when to call it.

### Rendering Strategies

This is Next.js's superpower. You can choose how each page is rendered:

```
CSR — Client-Side Rendering
────────────────────────────
  How it works:
    Server sends an empty HTML shell.
    Browser downloads JavaScript.
    JavaScript runs. React renders.
    User sees content.
  
  Speed:    Slow initial load
  SEO:      Poor (empty HTML)
  Use case: Dashboards, apps behind login (SEO doesn't matter)

SSG — Static Site Generation
────────────────────────────
  How it works:
    HTML is generated at BUILD time.
    Generated files are served from CDN.
    User gets pre-built HTML instantly.
  
  Speed:    Fastest possible
  SEO:      Excellent (full HTML)
  Use case: Blog, documentation, marketing pages

SSR — Server-Side Rendering
────────────────────────────
  How it works:
    User requests a page.
    Server runs React and generates HTML.
    Server sends full HTML to browser.
    Browser shows page. JavaScript hydrates.
  
  Speed:    Fast initial load (slower than SSG)
  SEO:      Excellent (full HTML)
  Use case: Product pages with live prices, user-specific pages

ISR — Incremental Static Regeneration
────────────────────────────────────────
  How it works:
    Page is statically generated at build time.
    After a time interval (e.g., 60 seconds), the page regenerates.
    Users always get a fast response (cached version).
    Background revalidation keeps content fresh.
  
  Speed:    Very fast (CDN-cached)
  SEO:      Excellent
  Use case: News sites, e-commerce with slowly changing prices
```

In Next.js's App Router (which you'll use throughout this course),  
these strategies are applied per-component or per-route through simple declarations.  
No configuration files. No boilerplate.

### The App Router vs Pages Router

Next.js has two routing systems. This course uses the **App Router** exclusively.

```
Pages Router (Old — Next.js 9 to 12):
  Introduced in 2019.
  Files in /pages become routes.
  getServerSideProps() for SSR.
  getStaticProps() for SSG.
  Simple but limited.
  
App Router (New — Next.js 13+):
  Introduced in 2022, stable in Next.js 14.
  Files in /app become routes.
  React Server Components by default.
  No need for getServerSideProps() — just use async components.
  Much more powerful and flexible.
  The future of Next.js.
```

If you work at a company with an older Next.js codebase, you may encounter  
the Pages Router. But all new projects start with the App Router.  
File 007 covers the differences in detail.

### React Server Components (RSC)

The App Router is built on **React Server Components** — a game-changing feature.

```
Traditional React (Client Components):
  Component runs in the browser.
  JavaScript bundle is sent to the client.
  User's device executes the code.
  
React Server Components:
  Component runs on the server.
  Only HTML is sent to the client.
  No JavaScript bundle for that component.
  Can directly access databases, files, environment variables.
```

Example of what Server Components make possible:

```tsx
import { db } from "@/lib/db";

async function ProductList() {
  const products = await db.product.findMany({
    orderBy: { createdAt: "desc" },
    take: 10,
  });

  return (
    <ul>
      {products.map((product) => (
        <li key={product.id}>
          {product.name} — ${product.price}
        </li>
      ))}
    </ul>
  );
}
```

This component:
- Runs on the server
- Directly queries the database
- Returns HTML to the browser
- Has zero client-side JavaScript
- Can never be inspected or manipulated by users

Before Server Components, this required an API route, a `useEffect`, and state management.  
Now it's just an `async` function component.

---

## Next.js Architecture

### How a Request Flows Through Next.js

```
Browser (User)
      │
      │ HTTP Request: GET /products/42
      │
      ▼
Middleware (edge)
      │
      │ Check auth, redirect, set headers
      │
      ▼
Next.js Router
      │
      │ Match: /app/products/[id]/page.tsx
      │
      ▼
Layout Tree
      │
      │ Root Layout → Products Layout → Product Layout
      │
      ▼
Page Component (Server Component by default)
      │
      │ async function, can fetch data directly
      │
      ▼
React renders to HTML
      │
      ▼
HTML sent to browser
      │
      ▼
Browser displays page (user sees content immediately)
      │
      ▼
JavaScript bundle downloads
      │
      ▼
React hydrates (page becomes interactive)
```

### Project Structure Overview

```
my-next-app/
├── app/
│   ├── layout.tsx          → Root layout (wraps everything)
│   ├── page.tsx            → Home page (/)
│   ├── globals.css         → Global styles
│   ├── about/
│   │   └── page.tsx        → /about
│   ├── blog/
│   │   ├── layout.tsx      → Blog layout (wraps all blog pages)
│   │   ├── page.tsx        → /blog
│   │   └── [slug]/
│   │       └── page.tsx    → /blog/my-post-title
│   └── api/
│       └── users/
│           └── route.ts    → /api/users (GET, POST)
├── components/
│   ├── ui/
│   │   └── Button.tsx
│   └── features/
│       └── ProductCard.tsx
├── lib/
│   ├── db.ts
│   └── utils.ts
├── public/
│   └── images/
├── next.config.ts
├── tsconfig.json
├── tailwind.config.ts
└── package.json
```

You'll learn every one of these folders and files in the sections ahead.

---

## Code Example: First Next.js Concepts

Let's look at what your first Next.js files look like.  
You'll create these yourself in the next file (008 — Installation and Setup).  
For now, just read and understand the structure.

### Root Layout (`app/layout.tsx`)

```tsx
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: {
    template: "%s | My App",
    default: "My App",
  },
  description: "A Next.js application",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

This file wraps every page in your application.  
`metadata` is exported to set SEO data.  
`children` is the current page's content.

### Home Page (`app/page.tsx`)

```tsx
export default function HomePage() {
  return (
    <main>
      <h1>Welcome to My App</h1>
      <p>Built with Next.js and TypeScript.</p>
    </main>
  );
}
```

Just a simple function component. No routing library. No configuration.  
Because the file is at `app/page.tsx`, Next.js automatically serves it at `/`.

### A Page With Data Fetching

```tsx
type GitHubUser = {
  login: string;
  name: string;
  avatar_url: string;
  public_repos: number;
  followers: number;
};

async function getUserData(username: string): Promise<GitHubUser> {
  const response = await fetch(
    `https://api.github.com/users/${username}`,
    { next: { revalidate: 3600 } }
  );

  if (!response.ok) {
    throw new Error("Failed to fetch user data");
  }

  return response.json();
}

export default async function GitHubProfilePage() {
  const user = await getUserData("vercel");

  return (
    <main className="profile-page">
      <img
        src={user.avatar_url}
        alt={`${user.login}'s avatar`}
        width={96}
        height={96}
      />
      <h1>{user.name}</h1>
      <p>@{user.login}</p>
      <div className="stats">
        <span>{user.public_repos} repos</span>
        <span>{user.followers} followers</span>
      </div>
    </main>
  );
}
```

`async function` in a page component — this is a Server Component.  
It fetches data on the server, builds HTML, and sends it to the browser.  
`next: { revalidate: 3600 }` — This is ISR: the data refreshes every hour.

Look at how simple this is. No `useEffect`. No `useState`. No loading state.  
The server fetches, the server renders, the user sees the complete page.

---

## The Ecosystem Around Next.js

Next.js doesn't exist in isolation. It's part of a rich ecosystem:

```
VERCEL (the company behind Next.js)
  ├── Next.js         → The framework
  ├── Turbopack       → Rust-based bundler (replaces Webpack)
  ├── Vercel Platform → Deployment with zero configuration
  ├── v0              → AI-powered UI generation
  └── AI SDK          → Building AI-powered apps

COMMUNITY TOOLS
  ├── Shadcn/ui       → Copy-paste component library
  ├── Radix UI        → Accessible component primitives
  ├── Framer Motion   → Animation library
  ├── next-themes     → Dark/light mode management
  └── next-mdx-remote → MDX content in Next.js

DATA LAYER
  ├── Prisma          → Most popular Next.js ORM
  ├── Drizzle         → Lightweight alternative
  ├── Neon            → Serverless PostgreSQL
  ├── PlanetScale     → MySQL for Next.js
  └── Supabase        → PostgreSQL + Auth + Storage
```

---

## Why Next.js Over Alternatives?

Let's look at the alternatives and why Next.js wins for most use cases:

```
Plain React (Create React App / Vite):
  ✓ Full control
  ✗ No SSR/SSG (bad for SEO)
  ✗ No file-based routing
  ✗ No API routes
  ✗ You configure everything yourself
  Best for: Simple SPAs behind a login

Remix:
  ✓ Great data model
  ✓ Progressive enhancement
  ✗ Smaller community
  ✗ Less ecosystem support
  Best for: Form-heavy applications

Gatsby:
  ✓ Excellent for static sites
  ✗ Poor for dynamic content
  ✗ Slow build times for large sites
  Best for: Pure static marketing sites

Nuxt (Vue ecosystem):
  ✓ Great if you prefer Vue
  ✗ React developers can't use it
  Best for: Vue.js projects

Next.js:
  ✓ Best-in-class SSR, SSG, ISR, RSC
  ✓ Enormous community
  ✓ Vercel platform integration
  ✓ Backed by a well-funded company
  ✓ Used by the largest websites
  ✓ Most job listings
  ✓ Best TypeScript support
  Best for: Everything from blogs to enterprise applications
```

---

## Common Mistakes (Conceptual)

### Mistake 1: Thinking Next.js Is Just React

Next.js is not "React with a router added."  
It fundamentally changes how your application works:
- Most code runs on the server, not the browser
- Data fetching is not done with `useEffect` anymore
- "Client Components" and "Server Components" are two different worlds

Take time to understand these differences. They're the hardest mental shift  
you'll make in this course.

### Mistake 2: Using Pages Router Tutorials With App Router

Many Next.js tutorials on YouTube and blogs were written before the App Router.  
They show `getServerSideProps()`, `getStaticProps()`, and files in `/pages`.  
This does not apply to the App Router.

Always check that a tutorial targets Next.js 13+ App Router before following it.

### Mistake 3: Making Everything a Client Component

When learning Next.js, beginners often add `"use client"` to everything  
because it makes the component feel more like "regular React."

This is wrong. Server Components are more powerful, more secure, and more  
performant. Only add `"use client"` when you absolutely need browser APIs  
or interactivity (useState, useEffect, event handlers).

---

## Best Practices

### Choose Next.js When...

```
✓ You need SEO (blogs, e-commerce, marketing pages)
✓ You need fast initial page loads
✓ You're building a full-stack application
✓ Your team knows React
✓ You want to deploy on Vercel
✓ You're building an enterprise application
```

### Set Up From Day One

Always configure these when starting a new Next.js project:
- TypeScript (non-negotiable)
- ESLint with Next.js rules
- Prettier
- Tailwind CSS (or your CSS solution)
- Absolute imports with `@/` prefix

---

## Real Company Examples

### Tiktok's Web Experience

TikTok's web platform uses Next.js for:
- Server-side rendering of video feeds (SEO + performance)
- Static generation of creator profile pages
- API routes for client-side interactions
- Edge middleware for geographic routing

### Twitch Clip Pages

Twitch uses Next.js to render clip pages server-side:
- Full HTML is sent to the browser on load
- Clips are indexed by search engines
- Fast sharing on social media (OpenGraph metadata works)

### GitHub Copilot Marketing Site

GitHub's Copilot landing page is built with Next.js:
- Static generation for maximum speed
- ISR for content updates without rebuilding
- Tailwind CSS for styling

---

## Mini Exercise

No code to write yet — you'll write your first Next.js code in file 008.  
For now, do this conceptual exercise:

**Think about a website you use frequently.**

Answer these questions for that website:
1. Does it need SEO? (Can you find it on Google?)
2. Does the content change frequently?
3. Does it show different content per user?
4. Does it have interactive parts (forms, buttons that update content)?

Now answer:
- Which rendering strategy would work best? (CSR / SSG / SSR / ISR)
- What would be Server Components?
- What would need to be Client Components?

Example — Twitter/X:
- SEO: Yes (tweets are indexed by Google)
- Content changes: Very frequently (new tweets every second)
- Per-user content: Yes (your timeline is unique)
- Interactive: Highly (likes, retweets, replies)

Answer: SSR for tweet pages (SEO), CSR for the timeline (real-time), Server Components  
for static parts (header, sidebar counts), Client Components for interactive elements  
(like button, reply form).

---

## Mini Quiz

**Question 1:** What is the primary problem React alone has that Next.js solves?

```
A) React is too slow
B) React doesn't support TypeScript
C) React has no built-in routing, SSR, or data fetching — making it hard to build production apps
D) React can't handle state management
```

*Answer: C*

---

**Question 2:** What is the difference between a library and a framework?

```
A) Libraries are free; frameworks are paid
B) In a library, YOU call the code; in a framework, the FRAMEWORK calls your code
C) Libraries are for frontend; frameworks are for backend
D) There is no difference
```

*Answer: B*

---

**Question 3:** Which rendering strategy generates HTML at BUILD time?

```
A) CSR (Client-Side Rendering)
B) SSR (Server-Side Rendering)
C) SSG (Static Site Generation)
D) None of the above
```

*Answer: C*

---

**Question 4:** What is a React Server Component?

```
A) A component that runs in a separate React version
B) A component that runs on the server, sends only HTML to the browser, and has no client-side JavaScript
C) A component that fetches data using useEffect
D) A component only available in Next.js 13+
```

*Answer: B*

---

**Question 5:** The App Router was introduced in which version of Next.js?

```
A) Next.js 9
B) Next.js 11
C) Next.js 13
D) Next.js 15
```

*Answer: C (stable in Next.js 14)*

---

**Question 6:** When should you NOT use Next.js?

```
A) When you need SEO
B) When you're building a simple SPA that's entirely behind a login and doesn't need SSR
C) When you need TypeScript
D) When you need API routes
```

*Answer: B*

---

## Summary

| Concept | Explanation |
|---------|-------------|
| Next.js | Full-stack React framework by Vercel |
| Framework vs Library | Frameworks call your code; you call libraries |
| CSR | Browser renders everything — bad for SEO |
| SSG | Build-time rendering — fastest, great for SEO |
| SSR | Per-request server rendering — SEO + dynamic content |
| ISR | Static + periodic regeneration — best of both worlds |
| App Router | New (Next.js 13+) routing system, uses Server Components |
| Pages Router | Old routing system, uses `getServerSideProps` |
| React Server Components | Components that run on the server, send HTML only |
| File-based routing | `app/page.tsx` → `/`, `app/about/page.tsx` → `/about` |

---

## Preparation for the Next Topic

Next file: **007-nextjs-vs-react.md**

In that file, you'll learn:
- A detailed comparison of working with plain React vs Next.js
- How every React pattern maps to a Next.js equivalent
- The mental model shift required to think in Next.js
- When the Pages Router makes more sense than the App Router
- Real code examples: same feature in React vs Next.js

After that, file 008 takes you hands-on: installing Node.js, creating your first  
Next.js project, and understanding every generated file.

---

*Next: [007-nextjs-vs-react.md](007-nextjs-vs-react.md)*
