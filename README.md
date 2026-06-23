# StoreShop

![React](https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-7-646CFF?style=flat-square&logo=vite&logoColor=white)
![React Router](https://img.shields.io/badge/React_Router-6-CA4245?style=flat-square&logo=reactrouter&logoColor=white)
![Axios](https://img.shields.io/badge/Axios-1.x-5A29E4?style=flat-square&logo=axios&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

A single-page e-commerce storefront built with React 19 and Vite. It consumes the public [Fake Store API](https://fakestoreapi.com) to display a live product catalogue with category filtering, keyword search, product detail views, and a fully functional shopping cart — all managed through React Context and `useReducer`.

---

## Overview

StoreShop demonstrates a production-style React architecture: clean separation of concerns between pages, shared layout, context providers, reusable components, and a utility layer. URL state is synchronised with the browser via `useSearchParams`, so category and search filters survive a page refresh or a shared link. Cart state persists for the lifetime of the session and drives a live item counter in the header.

---

## Features

- **Product catalogue** — fetches all products from Fake Store API on mount; displays them as a responsive card grid
- **Keyword search** — filters by product title; search term is reflected in the URL query string
- **Category sidebar** — one-click filtering by `electronics`, `jewelery`, `men's clothing`, or `women's clothing`; combines with active search
- **URL-synced filters** — `?search=` and `?category=` params are read on load, so deep links work correctly
- **Product detail page** — dynamic route (`/products/:id`) shows the full description, category, and price
- **Shopping cart** — add, remove, increase, decrease quantity; cart icon in the header shows a live item count badge
- **Checkout view** — summary sidebar (total price, quantity, status) alongside a list of all cart items
- **Loading states** — animated spinner (via `react-loader-spinner`) while data is in flight
- **404 page** — catch-all route for unmatched URLs

---

## Tech Stack

| Layer | Library / Tool | Version |
|---|---|---|
| UI framework | React | ^19.1.1 |
| Build tool | Vite | ^7.1.7 |
| Routing | React Router DOM | ^6.30.1 |
| HTTP client | Axios | ^1.12.2 |
| Icons | React Icons | ^5.5.0 |
| Spinner | React Loader Spinner | ^7.0.3 |
| Linting | ESLint + plugins | ^9.36.0 |

Styling is handled entirely with **CSS Modules** — no CSS-in-JS runtime, no Tailwind dependency.

---

## Installation

**Prerequisites:** Node.js ≥ 20.19.0 (required by Vite 7).

```bash
# 1. Clone the repository
git clone https://github.com/your-username/store-project.git
cd store-project

# 2. Install dependencies
npm install
```

---

## Getting Started

```bash
# Start the development server (with HMR)
npm run dev

# Build for production
npm run build

# Preview the production build locally
npm run preview

# Run ESLint
npm run lint
```

The dev server starts at `http://localhost:5173` by default.

The app proxies no requests — it hits `https://fakestoreapi.com` directly from the browser. An active internet connection is required.

---

## Project Structure

```
store-project/
├── index.html
├── vite.config.js
├── eslint.config.js
├── package.json
└── src/
    ├── main.jsx                  # React root; mounts BrowserRouter
    ├── App.jsx                   # Route definitions; wraps providers
    ├── global.css                # Reset + root-level variables
    │
    ├── pages/
    │   ├── ProductsPage.jsx      # Catalogue with search + filter
    │   ├── ProductsPage.module.css
    │   ├── DetailsPage.jsx       # Single product view (/products/:id)
    │   ├── DetailsPage.module.css
    │   ├── CheckoutPage.jsx      # Cart summary + item list
    │   ├── CheckoutPage.module.css
    │   └── 404.jsx               # Catch-all not-found page
    │
    ├── layout/
    │   ├── Layout.jsx            # Header (nav + cart badge) + footer
    │   └── Layout.module.css
    │
    ├── components/
    │   ├── Card.jsx              # Product card with add/remove controls
    │   ├── Card.module.css
    │   ├── BasketCard.jsx        # Cart item row (qty controls + delete)
    │   ├── BasketCard.module.css
    │   ├── BasketSidebar.jsx     # Order summary panel in checkout
    │   ├── BasketSidebar.module.css
    │   ├── SearchBox.jsx         # Search input + trigger button
    │   ├── SearchBox.module.css
    │   ├── Sidebar.jsx           # Category filter list
    │   ├── Sidebar.module.css
    │   ├── Loader.jsx            # Centred spinner wrapper
    │   └── Loader.module.css
    │
    ├── context/
    │   ├── CartContext.jsx       # useReducer cart; exposes useCart()
    │   └── ProducContext.jsx     # Fetches products; exposes useProducts(), useProductDetails()
    │
    ├── services/
    │   └── config.js             # Axios instance (baseURL + response interceptor)
    │
    ├── helper/
    │   └── helper.js             # Pure utility functions
    │
    └── constants/
        └── list.js               # Category definitions
```

---

## Key Components & Modules

### Context

**`CartContext.jsx`**

Manages the entire cart lifecycle with `useReducer`. The reducer handles five action types:

| Action | Effect |
|---|---|
| `ADD_ITEM` | Adds product with `quantity: 1` if not already present |
| `REMOVE_ITEM` | Removes the item entirely |
| `INCREASE` | Increments item quantity |
| `DECREASE` | Decrements item quantity |
| `CHECKOUT` | Clears the cart and sets `checkout: true` |

`sumProducts` is called after every mutation to keep `itemsCounter` and `total` derived and consistent. The `useCart()` hook exposes `[state, dispatch]`.

**`ProducContext.jsx`**

Fetches `/products` once on mount and stores the result in state. Exposes two hooks:

- `useProducts()` — returns the full product array
- `useProductDetails(id)` — finds a single product by numeric id

### Pages

**`ProductsPage.jsx`**

Orchestrates search and filter state. Two `useEffect` hooks keep URL params and displayed products in sync:

- The first runs when the products load, reading initial query params from the URL.
- The second runs whenever `query` changes, updating the URL, re-running `searchProducts`, and re-running `filterProducts`.

**`DetailsPage.jsx`**

Reads `:id` from the URL, looks up the product via `useProductDetails`, and shows a loader until the context is populated.

**`CheckoutPage.jsx`**

Renders an empty-state message when the cart is empty. Otherwise composes `BasketSidebar` (totals + checkout action) and a list of `BasketCard` rows.

### Utilities (`helper/helper.js`)

| Function | Purpose |
|---|---|
| `shortenText(text)` | Trims a product title to the first three words |
| `searchProducts(products, search)` | Case-insensitive title filter |
| `filterProducts(products, category)` | Exact category match |
| `createQueryObject(current, next)` | Merges or removes query keys (handles `"all"` and `""` sentinel values) |
| `getInitialQuery(searchParams)` | Hydrates the query object from `URLSearchParams` |
| `sumProducts(products)` | Derives `itemsCounter` and `total` from the cart array |
| `productQuantity(state, id)` | Returns quantity for a given product id, or 0 |

### Services (`services/config.js`)

An Axios instance pointed at `https://fakestoreapi.com`. A response interceptor unwraps `response.data` so callers receive the payload directly (no `.data` access required at the call site).

---

## Screenshots

> Add screenshots here after running the app locally.

| View | Screenshot |
|---|---|
| Products page | _coming soon_ |
| Category filter active | _coming soon_ |
| Product detail | _coming soon_ |
| Checkout | _coming soon_ |

---

## Future Enhancements

- **Persistent cart** — save cart state to `localStorage` so items survive a page refresh
- **Authentication** — integrate Fake Store API login endpoints for user sessions
- **Pagination or infinite scroll** — the API supports limit/offset; currently all products load at once
- **Sorting** — price ascending/descending, rating, alphabetical
- **Checkout form** — shipping and payment fields instead of the current one-click checkout action
- **Toast notifications** — feedback when items are added to or removed from the cart
- **Unit tests** — cover the reducer, utility functions, and key components with Vitest + React Testing Library
- **TypeScript migration** — the dev dependencies already include `@types/react` and `@types/react-dom`

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m "feat: describe your change"`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a pull request

Please run `npm run lint` before submitting and keep each PR focused on a single concern.

---

## License

Distributed under the **MIT License**. See `LICENSE` for details.

---

## Author

Developed by **Hossein Douzandeh**