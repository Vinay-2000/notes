# Architecture + HTML/CSS + Build + Performance Interview Notes

## React architecture

Feature-based structure:

``` text
src/
├── features/
│   ├── auth/
│   ├── orders/
│   └── portfolio/
├── components/common/
├── services/
├── store/
├── routes/
└── utils/
```

Mental model:

``` text
Component
 ↓
Hook / RTK Query
 ↓
API layer
 ↓
Backend
```

Separate presentation, state management, API communication and business
logic appropriately.

## Shared WebSocket architecture

``` text
One WebSocket
 ↓
service/middleware
 ↓
Redux store
 ↓
granular selectors
 ↓
Chart / OrderBook
```

Do not create separate socket connections in each component.

## Granular Redux subscriptions

``` tsx
const btcPrice = useSelector(
  state => state.marketData.btc
);

const balance = useSelector(
  state => state.wallet.balance
);
```

A BTC price update should not force Wallet to re-render if its selected
value remains unchanged.

For very high-frequency streams, consider buffering/batching or another
specialized external store.

## Reconnection

Guard against duplicate connections:

``` js
if (
  socket &&
  (
    socket.readyState === WebSocket.OPEN ||
    socket.readyState === WebSocket.CONNECTING
  )
) {
  return;
}
```

On close/error, schedule reconnect. Production systems commonly use
exponential backoff and jitter.

## Persistent WebSocket

If it must survive route changes, manage the connection at
application/provider/service level, not inside a page that unmounts.

``` tsx
function WebSocketProvider({ children }) {
  useEffect(() => {
    connect();
    return () => disconnect();
  }, []);

  return children;
}
```

## Idempotency

Prevent duplicate orders with an idempotency key:

``` http
POST /orders
Idempotency-Key: 7f3c9a...
```

Backend stores the key/result. Duplicate requests return the existing
result.

Disabling the frontend button is useful UX but cannot guarantee backend
idempotency.

# HTML / Accessibility

## Semantic HTML

Prefer:

``` html
<header></header>
<nav></nav>
<main></main>
<section></section>
<footer></footer>
```

instead of using `<div>` for everything.

## Buttons

Prefer:

``` html
<button>Submit Order</button>
```

instead of:

``` html
<div onclick="submitOrder()">Submit Order</div>
```

Native buttons provide semantics, keyboard behavior and assistive
technology support.

## Labels

HTML:

``` html
<label for="email">Email</label>
<input id="email" type="email">
```

React:

``` jsx
<label htmlFor="email">Email</label>
<input id="email" type="email" />
```

## ARIA

Use ARIA to supplement semantic HTML.

``` html
<button aria-label="Delete order">
  🗑️
</button>
```

`aria-label` provides the name directly.

`aria-labelledby` references another element:

``` html
<h2 id="order-title">Order Details</h2>
<section aria-labelledby="order-title">
```

## Keyboard accessibility

For a custom interactive element:

``` jsx
<div
  role="button"
  tabIndex={0}
  onKeyDown={e => {
    if (e.key === "Enter" || e.key === " ") {
      toggle();
    }
  }}
>
  Select Account
</div>
```

Prefer a native `<button>` whenever possible.

# CSS

## Flexbox

``` css
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

Accurate rule:

``` text
justify-content → main axis
align-items     → cross axis
```

## Grid

``` css
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr 300px;
}
```

Flexbox is mainly one-dimensional. Grid is two-dimensional.

## Responsive design

``` css
.container {
  display: grid;
  grid-template-columns: 1fr 1fr;
}

@media (max-width: 768px) {
  .container {
    grid-template-columns: 1fr;
  }
}
```

Choose breakpoints based on layout requirements, not individual device
models.

Bootstrap provides predefined responsive utilities/breakpoints.

## Specificity

Simplified:

``` text
element
class
ID
inline
!important (special cascade priority)
```

``` css
button { color: blue; }
.primary { color: green; }
#submit { color: red; }
```

`#submit` wins.

`!important` can override normal declarations, but avoid unnecessary
use.

# SCSS

## Nesting

``` scss
.card {
  padding: 20px;

  .title {
    font-size: 20px;
  }

  &:hover {
    box-shadow: 0 2px 10px rgba(0,0,0,.2);
  }

  &.active {
    border: 2px solid green;
  }
}
```

## Variables

``` scss
$primary: #1976d2;
$spacing: 16px;

.card {
  color: $primary;
  padding: $spacing;
}
```

## Mixins

``` scss
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.header {
  @include flex-center;
}
```

Parameterized:

``` scss
@mixin flex-container($direction, $gap) {
  display: flex;
  flex-direction: $direction;
  gap: $gap;
}

.container {
  @include flex-container(row, 20px);
}
```

## `@extend`

``` scss
.button {
  padding: 10px 20px;
  border-radius: 5px;
}

.primary-button {
  @extend .button;
  background: blue;
}
```

SCSS is not Java-style inheritance; it provides nesting, variables,
mixins, `@extend`, functions and modular organization.

## CSS Modules

`Card.module.scss`:

``` scss
.card {
  color: red;
}
```

React:

``` tsx
import styles from "./Card.module.scss";

<div className={styles.card}>
```

SCSS improves CSS authoring; CSS Modules provide local class scoping.

# Vite / Build

## Vite

Vite is a frontend build tool and development server.

``` bash
npm create vite@latest
```

During development, Vite serves/transforms modules on demand rather than
bundling the entire application upfront, improving startup and update
speed.

HMR updates affected modules without a full page reload.

Browsers do not understand JSX directly; Vite transforms source into
browser-understandable JavaScript.

Production builds perform optimization such as bundling, tree shaking,
code splitting and asset optimization.

## Environment variables

``` text
.env
.env.development
.env.production
.env.staging
```

Client-exposed Vite variables use the `VITE_` prefix:

``` env
VITE_API_URL=https://api.example.com
```

``` js
const apiUrl = import.meta.env.VITE_API_URL;
```

Vite uses modes:

``` bash
npm run dev
npm run build
npm run build -- --mode staging
```

Custom staging mode can load `.env.staging`.

Never put secrets such as DB passwords/private keys into `VITE_`
variables because client-side values are inspectable.

Keep secrets in the backend, e.g.:

``` text
React
 ↓
Spring Boot
 ↓
AWS Secrets Manager / Parameter Store
```

## Tree shaking

Removes unused code from production bundles.

``` js
export function add(a,b) {}
export function subtract(a,b) {}
```

If only `add` is imported, unused exports may be removed.

``` text
Tree shaking → remove unused code
Code splitting → divide code into chunks
```

# Performance

## React Profiler

Use React DevTools Profiler to identify:

-   which components render
-   how often
-   render duration
-   potential causes

Check state, props, parent renders, context and effects.

## Large lists

For 50,000 records:

**Pagination** reduces fetched data:

``` text
GET /transactions?page=1&size=50
```

**Virtualization** reduces DOM/rendering:

``` text
50,000 available
 ↓
visible rows + overscan
 ↓
DOM
```

They can be combined.

## Bundle optimization

For an 8 MB bundle:

1.  Analyze the bundle.
2.  Remove unnecessary dependencies.
3.  Tree shake.
4.  Code split.
5.  Lazy-load routes/features.
6.  Optimize large libraries.
7.  Optimize images.
8.  Use caching/CDN/compression.

SSR can improve initial content delivery but is not the direct solution
to reducing JavaScript bundle size.

## Image optimization

Use:

-   WebP/AVIF
-   compression
-   correct dimensions
-   lazy loading
-   responsive images

``` html
<img src="chart.webp" loading="lazy" alt="Price chart">
```

## Network optimization

Use:

-   browser caching
-   CDN
-   Gzip/Brotli
-   HTTP/2 or HTTP/3 where supported
-   smaller API payloads
-   pagination
-   debounce
-   API caching
-   avoiding unnecessary calls

Use Chrome DevTools/Lighthouse to identify bottlenecks.

## Search

Debounce user input:

``` text
a → ap → app → appl → apple
                    ↓
               wait 300–500ms
                    ↓
                  API
```

If old requests remain in flight, use cancellation/race handling so
stale results cannot overwrite newer ones.

# Interview Mental Model

For architecture/performance questions:

``` text
1. Who owns the data?
2. Client state or server state?
3. Who needs it?
4. How frequently does it change?
5. What is the rendering cost?
6. Where should lifecycle live?
7. How do we handle failure/retry?
8. How do we avoid unnecessary work?
```

# High-value distinctions

\`\`\`text useState → local state useReducer → complex local transitions
Context → shared values Redux Toolkit → shared client/application state
RTK Query → server/API state

React.memo → skip child render when props are shallowly equal useMemo →
memoized value useCallback → memoized function useRef → persistent
mutable value without render

Pagination → reduce fetched data Virtualization → reduce DOM/rendering

Debounce → wait for burst to stop Throttle → limit execution frequency

SSR → server generates initial HTML Hydration → React makes SSR HTML
interactive

Tree shaking → remove unused code Code splitting → split code into
loadable chunks
