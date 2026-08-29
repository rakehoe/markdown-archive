# React Layout Template Pattern

A simple guide for building a React application using the **Layout Template Pattern** with **React Router**.

This approach is recommended when multiple pages share common UI elements such as:

- Header / Navigation
- Footer
- Sidebar
- Page container
- Global layout styles

The layout is defined once and reused across multiple pages.

---

# Option 1: The Layout Template Pattern (Recommended)

Instead of repeating the same header, footer, and navigation code on every page, create a reusable `Layout` component.

The basic structure looks like this:

```text
App
└── Layout
    ├── Header
    ├── Navigation
    ├── Main Content
    │   ├── Home
    │   ├── About
    │   └── Contact Us
    └── Footer
```

React Router will determine which page is displayed inside the layout.

---

# Step 1: Install React Router

If you are using an existing React project, install `react-router-dom`:

```bash
npm install react-router-dom
```

For a new Vite React project:

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm install react-router-dom
npm run dev
```

---

# Step 2: Establish Your File Structure

Create a folder structure similar to the following:

```text
src/
│
├── components/
│   ├── Header.jsx
│   ├── Footer.jsx
│   └── Navigation.jsx
│
├── layouts/
│   └── Layout.jsx
│
├── pages/
│   ├── Home.jsx
│   ├── About.jsx
│   ├── Contact.jsx
│   └── NotFound.jsx
│
├── App.jsx
├── main.jsx
└── index.css
```

### Purpose of each folder

| Folder        | Purpose                       |
| ------------- | ----------------------------- |
| `components/` | Reusable UI components        |
| `layouts/`    | Shared page layouts           |
| `pages/`      | Individual application pages  |
| `App.jsx`     | Application routing           |
| `main.jsx`    | React application entry point |
| `index.css`   | Global styles                 |

---

# Step 3: Create the Layout Component

Create:

```text
src/layouts/Layout.jsx
```

Add the following:

```jsx
import { Outlet } from "react-router-dom";
import Header from "../components/Header";
import Footer from "../components/Footer";

function Layout() {
  return (
    <>
      <Header />

      <main>
        <Outlet />
      </main>

      <Footer />
    </>
  );
}

export default Layout;
```

## How `Outlet` works

`<Outlet />` is the location where React Router renders the current page.

For example:

```text
Layout
│
├── Header
│
├── Outlet
│     └── About Page
│
└── Footer
```

When the user visits `/about`, the `About` component will appear inside `<Outlet />`.

---

# Step 4: Create the Header

Create:

```text
src/components/Header.jsx
```

Example:

```jsx
import { Link } from "react-router-dom";

function Header() {
  return (
    <header>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/contact">Contact Us</Link>
      </nav>
    </header>
  );
}

export default Header;
```

Using `Link` instead of a normal `<a>` tag allows React Router to navigate between pages without completely reloading the application.

---

# Step 5: Create the Footer

Create:

```text
src/components/Footer.jsx
```

Example:

```jsx
function Footer() {
  return (
    <footer>
      <p>&copy; 2026 My Website. All rights reserved.</p>
    </footer>
  );
}

export default Footer;
```

---

# Step 6: Create Your Pages

## Home Page

Create:

```text
src/pages/Home.jsx
```

```jsx
function Home() {
  return (
    <section>
      <h1>Welcome to Our Website</h1>

      <p>
        This is the home page of our React application.
      </p>
    </section>
  );
}

export default Home;
```

---

# About Page Template

Create:

```text
src/pages/About.jsx
```

```jsx
function About() {
  return (
    <section>
      <h1>About Us</h1>

      <p>
        Learn more about our company, team, mission, and values.
      </p>

      <h2>Our Mission</h2>

      <p>
        Our mission is to provide high-quality products and services
        to our customers.
      </p>

      <h2>Our Vision</h2>

      <p>
        We aim to create a better experience for our customers
        through innovation and excellent service.
      </p>
    </section>
  );
}

export default About;
```

---

# Contact Us Page Template

Create:

```text
src/pages/Contact.jsx
```

```jsx
function Contact() {
  return (
    <section>
      <h1>Contact Us</h1>

      <p>
        Have a question? Send us a message using the form below.
      </p>

      <form>
        <div>
          <label htmlFor="name">Name</label>
          <input
            type="text"
            id="name"
            name="name"
            placeholder="Enter your name"
          />
        </div>

        <div>
          <label htmlFor="email">Email</label>
          <input
            type="email"
            id="email"
            name="email"
            placeholder="Enter your email"
          />
        </div>

        <div>
          <label htmlFor="message">Message</label>
          <textarea
            id="message"
            name="message"
            placeholder="Enter your message"
            rows="5"
          />
        </div>

        <button type="submit">
          Send Message
        </button>
      </form>
    </section>
  );
}

export default Contact;
```

> **Note:** This is only the front-end form template. To actually send messages, connect the form to your backend, API, email service, or form-handling service.

---

# Step 7: Create a 404 / Not Found Page

Create:

```text
src/pages/NotFound.jsx
```

```jsx
import { Link } from "react-router-dom";

function NotFound() {
  return (
    <section>
      <h1>404</h1>

      <h2>Page Not Found</h2>

      <p>
        Sorry, the page you are looking for does not exist.
      </p>

      <Link to="/">
        Go Back Home
      </Link>
    </section>
  );
}

export default NotFound;
```

---

# Step 8: Configure React Router

Update:

```text
src/App.jsx
```

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

import Layout from "./layouts/Layout";

import Home from "./pages/Home";
import About from "./pages/About";
import Contact from "./pages/Contact";
import NotFound from "./pages/NotFound";

function App() {
  return (
    <BrowserRouter>
      <Routes>

        {/* Shared Layout */}
        <Route element={<Layout />}>

          {/* Pages */}
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />

          {/* 404 Page */}
          <Route path="*" element={<NotFound />} />

        </Route>

      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

---

# Step 9: Application Entry Point

Your:

```text
src/main.jsx
```

can remain simple:

```jsx
import React from "react";
import ReactDOM from "react-dom/client";

import App from "./App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

# Step 10: Add Basic Styling

Example:

```text
src/index.css
```

```css
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
}

header {
  padding: 1rem 2rem;
  border-bottom: 1px solid #ddd;
}

nav {
  display: flex;
  gap: 1rem;
}

nav a {
  text-decoration: none;
}

main {
  min-height: 70vh;
  padding: 2rem;
}

footer {
  padding: 1rem 2rem;
  border-top: 1px solid #ddd;
  text-align: center;
}

form {
  max-width: 500px;
}

form div {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
}

input,
textarea {
  width: 100%;
  padding: 0.75rem;
}

button {
  padding: 0.75rem 1.25rem;
  cursor: pointer;
}
```

---

# Complete File Structure

After completing the steps, your project should look like:

```text
my-app/
│
├── public/
│
├── src/
│   │
│   ├── components/
│   │   ├── Header.jsx
│   │   └── Footer.jsx
│   │
│   ├── layouts/
│   │   └── Layout.jsx
│   │
│   ├── pages/
│   │   ├── Home.jsx
│   │   ├── About.jsx
│   │   ├── Contact.jsx
│   │   └── NotFound.jsx
│   │
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
│
├── package.json
└── README.md
```

---

# Adding More Pages

The Layout Template Pattern makes it easy to add additional pages.

For example:

```text
pages/
├── Home.jsx
├── About.jsx
├── Contact.jsx
├── Services.jsx
├── Products.jsx
├── FAQ.jsx
├── Blog.jsx
└── NotFound.jsx
```

Then add the route in `App.jsx`:

```jsx
<Route path="/services" element={<Services />} />
<Route path="/products" element={<Products />} />
<Route path="/faq" element={<FAQ />} />
<Route path="/blog" element={<Blog />} />
```

Add the corresponding navigation links:

```jsx
<nav>
  <Link to="/">Home</Link>
  <Link to="/about">About</Link>
  <Link to="/services">Services</Link>
  <Link to="/products">Products</Link>
  <Link to="/faq">FAQ</Link>
  <Link to="/blog">Blog</Link>
  <Link to="/contact">Contact Us</Link>
</nav>
```

---

# Why Use the Layout Template Pattern?

Without a shared layout, you might end up repeating this on every page:

```jsx
<Header />

<PageContent />

<Footer />
```

This can become difficult to maintain.

With the Layout Pattern:

```jsx
<Layout>
  <Outlet />
</Layout>
```

the shared UI is defined once.

### Benefits

- Less duplicated code
- Easier maintenance
- Consistent navigation
- Consistent header and footer
- Easier to add new pages
- Cleaner page components
- Better separation of concerns

---

# Page Template

When creating a new page, use this basic template:

```jsx
function ExamplePage() {
  return (
    <section>
      <h1>Page Title</h1>

      <p>
        Page content goes here.
      </p>
    </section>
  );
}

export default ExamplePage;
```

Then add it to your router:

```jsx
<Route
  path="/example"
  element={<ExamplePage />}
/>
```

---

# Recommended Development Pattern

Keep your application organized using these responsibilities:

```text
components/
    Reusable UI

layouts/
    Shared page structures

pages/
    Route-specific content

App.jsx
    Routing configuration

main.jsx
    Application entry point
```

For example:

```text
components/
├── Header.jsx
├── Footer.jsx
├── Button.jsx
└── Card.jsx

layouts/
├── Layout.jsx
└── DashboardLayout.jsx

pages/
├── Home.jsx
├── About.jsx
├── Contact.jsx
├── Services.jsx
└── NotFound.jsx
```

This structure makes the application easier to scale as more pages and features are added.

---

# Quick Setup Checklist

- [ ] Install `react-router-dom`
- [ ] Create the `components` folder
- [ ] Create the `layouts` folder
- [ ] Create the `pages` folder
- [ ] Create `Layout.jsx`
- [ ] Create `Header.jsx`
- [ ] Create `Footer.jsx`
- [ ] Create `Home.jsx`
- [ ] Create `About.jsx`
- [ ] Create `Contact.jsx`
- [ ] Create `NotFound.jsx`
- [ ] Configure routes in `App.jsx`
- [ ] Add navigation links
- [ ] Add global styles
- [ ] Run the application

Start the development server with:

```bash
npm run dev
```

Then test:

```text
/
 /about
 /contact
```

The header and footer should remain consistent while the content inside the layout changes based on the current URL.
