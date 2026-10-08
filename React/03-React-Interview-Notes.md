# React Interview Notes

## Rendering

Rendering means React executes components to determine the UI. It does
not necessarily mean the DOM changes.

``` text
setState → schedule → render → reconciliation → commit → DOM
```

## State snapshot

Repeated direct updates use the same render snapshot:

``` tsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

Use functional updates when depending on previous state:

``` tsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

## Memoization

``` tsx
const Chart = React.memo(ChartComponent);

const config = useMemo(
  () => ({ timeframe, indicators }),
  [timeframe, indicators]
);

const onClick = useCallback(
  () => submitOrder(orderId),
  [orderId]
);
```

-   `React.memo` → component render optimization based on props
-   `useMemo` → memoized value
-   `useCallback` → memoized function reference

Memoization does not automatically stop the parent from rendering.

## useEffect

Use for synchronization with external systems: APIs, timers,
subscriptions, WebSockets, event listeners.

``` tsx
useEffect(() => {
  const timer = setInterval(refresh, 5000);
  return () => clearInterval(timer);
}, []);
```

Cleanup runs on unmount and before rerunning when dependencies change.

Avoid effects for simple derived values.

## State management

``` text
local UI             → useState/useReducer
simple shared       → Context
complex client      → Redux Toolkit
server/API state    → RTK Query
```

Context distributes values; it does not itself manage state.

## Redux Toolkit

``` ts
const walletSlice = createSlice({
  name: "wallet",
  initialState: { balance: 0 },
  reducers: {
    setBalance(state, action) {
      state.balance = action.payload;
    }
  }
});
```

RTK uses Immer for immutable updates.

## RTK Query

``` ts
const api = createApi({
  reducerPath: "api",
  baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
  tagTypes: ["Wallet"],
  endpoints: builder => ({
    getWallet: builder.query({
      query: () => "/wallet",
      providesTags: ["Wallet"]
    }),
    placeOrder: builder.mutation({
      query: order => ({
        url: "/orders",
        method: "POST",
        body: order
      }),
      invalidatesTags: ["Wallet"]
    })
  })
});
```

RTK Query manages caching, loading/error state, refetching, invalidation
and deduplication.

If wallet balance is backend-owned, prefer RTK Query over duplicating it
in a normal Redux slice.

## Keys

``` tsx
users.map(user => (
  <UserCard key={user.id} user={user} />
))
```

Keys identify item identity across renders. Avoid array indexes for
dynamic/reordered lists.

## Controlled vs uncontrolled

Controlled:

``` tsx
const [name, setName] = useState("");

<input
  value={name}
  onChange={e => setName(e.target.value)}
/>
```

Uncontrolled:

``` tsx
const inputRef = useRef<HTMLInputElement>(null);
<input ref={inputRef} />
```

## useRef

`useRef` persists a mutable value across renders. Changing `.current`
does not trigger a render.

## Custom hooks

Custom hooks reuse stateful logic, not state itself.

``` tsx
function useUser(userId: string) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(setUser);
  }, [userId]);

  return user;
}
```

Rules: hooks at top level; only components/custom hooks; no
conditional/looped hook calls.

## useReducer

Useful for complex related local state transitions. Redux is broader
application architecture.

## useLayoutEffect

Timing:

``` text
render → DOM commit → useLayoutEffect → paint → useEffect
```

Use for layout measurement or visual synchronization. Expensive work
blocks paint.

## Refs

`forwardRef` passes a ref through a custom component.
`useImperativeHandle` controls the imperative API exposed to the parent.

## Error boundaries

Catch certain rendering/lifecycle/constructor errors in child trees.

``` tsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error(error, info);
  }

  render() {
    return this.state.hasError
      ? <h1>Something went wrong</h1>
      : this.props.children;
  }
}
```

They do not automatically catch event-handler or arbitrary async
callback errors.

## Lazy loading

``` tsx
const AdminPage = React.lazy(() => import("./AdminPage"));

<Suspense fallback={<Loading />}>
  <AdminPage />
</Suspense>
```

Commonly used for route-level code splitting.

## Suspense

Suspense provides fallback UI while supported content is not ready.
`React.lazy` is a common use case. Normal `useEffect` fetching is not
automatically Suspense-enabled.

## Strict Mode

Development-time checks. React may intentionally invoke certain logic
more than once, including extra effect setup/cleanup. Do not say the
production app always renders twice.

## Fiber

React's internal reconciliation architecture introduced in React 16. It
allows React to break rendering work into units and gives React more
control over scheduling/prioritization.

``` text
Virtual DOM → UI representation
Reconciliation → determine changes
Fiber → internal architecture for rendering/reconciliation
```

## SSR / CSR / Hydration

CSR:

``` text
browser → download JS → React renders
```

SSR:

``` text
browser → server renders React → HTML → browser
```

Hydration:

``` text
SSR HTML → React JS loads → React attaches behavior → interactive
```

SSR can improve initial content/SEO; hydration makes the server-rendered
HTML interactive.

## Server Components

Server Components execute on the server and don't require their
component code to be shipped as client-side JavaScript.

``` tsx
export default async function ProductsPage() {
  const products = await db.product.findMany();

  return products.map(p => (
    <div key={p.id}>{p.name}</div>
  ));
}
```

Interactive components can be client components:

``` tsx
"use client";

function AddToCart() {
  const [added, setAdded] = useState(false);

  return (
    <button onClick={() => setAdded(true)}>
      {added ? "Added" : "Add"}
    </button>
  );
}
```

RSC and SSR are different concepts and can be used together.

## API integration

Fetch does not reject merely because HTTP status is 400/500:

``` js
const response = await fetch("/api/users");
if (!response.ok) throw new Error("Request failed");
```

Axios normally rejects non-2xx.

## Axios cancellation

``` js
const controller = new AbortController();

axios.get("/api/search", {
  signal: controller.signal
});

controller.abort();
```

## REST vs GraphQL

REST is resource-oriented with server-defined response shapes. GraphQL
lets the client request fields from a schema, usually through one
endpoint.

## Authentication

OAuth2 = authorization framework. OIDC = authentication/identity layer
on OAuth2. SSO = architecture/experience.

JWT commonly has:

``` text
header.payload.signature
```

Payload is not encrypted by default.

Validate issuer, audience, expiry, signature and authorization claims as
appropriate.

## Concurrent token refresh

Use one shared refresh Promise:

``` js
let refreshPromise = null;

async function refreshToken() {
  if (!refreshPromise) {
    refreshPromise = axios
      .post("/auth/refresh")
      .then(r => setAccessToken(r.data.accessToken))
      .finally(() => {
        refreshPromise = null;
      });
  }

  return refreshPromise;
}
```

Multiple 401s wait for the same refresh instead of starting multiple
refresh requests.

## Forms

Controlled forms are perfectly valid.

``` tsx
const [email, setEmail] = useState("");

<input
  value={email}
  onChange={e => setEmail(e.target.value)}
/>
```

Backend validation remains mandatory.

## Testing

Vitest is common with Vite; React Testing Library focuses on user
behavior.

``` text
getBy    → expected now
queryBy  → check absence
findBy   → async/wait
```

Prefer role/label-based queries.

Test loading, success, error, empty states and user interactions.
