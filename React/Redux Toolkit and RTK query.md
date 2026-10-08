## 1. Redux vs Redux Toolkit --- the big picture

### What is Redux?

Redux is a predictable state management library used to keep shared
application state in a centralized store.

The traditional Redux flow is:

``` text
Component
   ↓
dispatch(action)
   ↓
Middleware
   ↓
Reducer
   ↓
New immutable state
   ↓
Store updates
   ↓
Subscribed components read updated state
```

Redux Toolkit (RTK) is the **official recommended way to write Redux
today**.

It does not replace the Redux architecture. It provides APIs and
defaults that make Redux much easier to write correctly.

The three APIs to know especially well for interviews are:

``` text
configureStore
createSlice
createApi (RTK Query)
```

------------------------------------------------------------------------

# 2. Why Redux Toolkit is better than traditional Redux

Traditional Redux often required a lot of boilerplate.

Suppose we want a counter.

## Traditional Redux

### Action types

```
const INCREMENT = "counter/increment";
const DECREMENT = "counter/decrement";
```

### Action creators

```
const increment = () => ({
  type: INCREMENT
});

const decrement = () => ({
  type: DECREMENT
});
```

### Reducer

```
const initialState = {
  value: 0
};

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return {
        ...state,
        value: state.value + 1
      };

    case DECREMENT:
      return {
        ...state,
        value: state.value - 1
      };

    default:
      return state;
  }
}
```

### Store

```
import { createStore } from "redux";

const store = createStore(counterReducer);
```

There is nothing inherently wrong with this, but there is a lot of
repetitive code.

------------------------------------------------------------------------

# 3. The same thing with Redux Toolkit

```
import { createSlice, configureStore } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",

  initialState: {
    value: 0
  },

  reducers: {
    increment: (state) => {
      state.value += 1;
    },

    decrement: (state) => {
      state.value -= 1;
    }
  }
});

export const { increment, decrement } = counterSlice.actions;

const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
});
```

Much less code.

More importantly, RTK provides useful defaults and helps prevent common
Redux mistakes.

------------------------------------------------------------------------

# 4. Main advantages of Redux Toolkit

## 4.1 Less boilerplate

Traditional Redux usually requires:

``` text
Action type
    ↓
Action creator
    ↓
Reducer
    ↓
Switch statement
```

With `createSlice`:

``` text
createSlice
    ↓
Action types
Action creators
Reducer
```

are generated from one definition.

------------------------------------------------------------------------

## 4.2 Immer allows mutation-like reducer code

Traditional Redux requires immutable updates.

For example:

```
return {
  ...state,
  items: [...state.items, action.payload]
};
```

With RTK:

```
state.items.push(action.payload);
```

This looks like mutation, but RTK uses **Immer internally**.

Immer gives the reducer a draft state and tracks the changes. It then
produces a new immutable state.

So:

```
state.items.push(item);
```

does NOT mean Redux is directly mutating the actual Redux state.

Interview answer:

> Redux Toolkit uses Immer internally, which allows us to write
> mutation-like reducer logic while still producing immutable state
> updates.

------------------------------------------------------------------------

# 5. configureStore

`configureStore` is the recommended way to create the Redux store.

```
const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer
  }
});
```

Instead of manually configuring everything, RTK provides sensible
defaults.

## What configureStore gives us

### 1. Combines reducers

```
const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer,
    cart: cartReducer
  }
});
```

This creates the Redux state shape:

```
{
  counter: {...},
  user: {...},
  cart: {...}
}
```

------------------------------------------------------------------------

### 2. Middleware configuration

RTK automatically includes useful middleware, including Redux Thunk.

You can also add custom middleware.

```
const store = configureStore({
  reducer: {
    counter: counterReducer
  },

  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(myMiddleware)
});
```

------------------------------------------------------------------------

### 3. Redux DevTools

`configureStore` automatically enables Redux DevTools integration in
normal development usage.

This makes it easy to inspect:

``` text
Actions
State changes
Previous state
Next state
Action payload
```

------------------------------------------------------------------------

### 4. Development checks

RTK's default middleware also performs useful development-time checks
such as detecting accidental mutations and problematic non-serializable
values.

------------------------------------------------------------------------

# 6. createSlice

`createSlice` is one of the most important RTK APIs.

It lets us define:

``` text
slice name
initial state
reducers
```

in one place.

Example:

```
const cartSlice = createSlice({
  name: "cart",

  initialState: {
    items: [],
    total: 0
  },

  reducers: {
    addItem: (state, action) => {
      state.items.push(action.payload);
    },

    removeItem: (state, action) => {
      state.items = state.items.filter(
        item => item.id !== action.payload
      );
    }
  }
});
```

RTK generates action creators automatically:

```
export const {
  addItem,
  removeItem
} = cartSlice.actions;
```

And the reducer:

```
cartSlice.reducer
```

------------------------------------------------------------------------

# 7. What createSlice generates

Given:

```
const counterSlice = createSlice({
  name: "counter",

  initialState: {
    value: 0
  },

  reducers: {
    increment: (state) => {
      state.value += 1;
    }
  }
});
```

RTK effectively gives us:

```
counterSlice.actions.increment
```

which creates an action similar to:

```
{
  type: "counter/increment"
}
```

And:

```
counterSlice.reducer
```

is the reducer responsible for handling that action.

So `createSlice` removes the need to manually write:

```
const INCREMENT = "counter/increment";
```

and:

```
const increment = () => ({
  type: INCREMENT
});
```

and:

```
switch(action.type) {
   ...
}
```

------------------------------------------------------------------------

# 8. createSlice with payload

Example:

```
const userSlice = createSlice({
  name: "user",

  initialState: {
    users: []
  },

  reducers: {
    addUser: (state, action) => {
      state.users.push(action.payload);
    }
  }
});
```

Dispatch:

```
dispatch(
  addUser({
    id: 1,
    name: "Vinay"
  })
);
```

The action looks conceptually like:

```
{
  type: "user/addUser",
  payload: {
    id: 1,
    name: "Vinay"
  }
}
```

The reducer receives:

```
state
action
```

and accesses the data through:

```
action.payload
```

------------------------------------------------------------------------

# 9. Complete Redux Toolkit flow

Example:

```
dispatch(addItem(product));
```

The flow is:

``` text
Component
    ↓
dispatch(addItem(product))
    ↓
Action created
    ↓
Middleware
    ↓
Reducer
    ↓
Immer produces new immutable state
    ↓
Redux store updated
    ↓
useSelector reads selected state
    ↓
Component re-renders if selected result changed
```

A strong interview explanation:

> A component dispatches an action. The action passes through middleware
> and reaches the reducer. The reducer calculates the next state. With
> Redux Toolkit, Immer allows mutation-like reducer syntax while
> producing immutable state. The store is updated, and components using
> the affected selected state can re-render.

------------------------------------------------------------------------

# 10. RTK Query

RTK Query is part of Redux Toolkit, but it solves a different problem.

Traditional Redux is commonly used for **client/application state**.

RTK Query is designed for **server/API state**.

Examples of server state:

``` text
Users from API
Products from API
Orders from API
Account details
Search results
```

Instead of manually writing:

``` text
loading state
error state
API call
success state
cache
refetch
duplicate request handling
```

RTK Query manages these concerns for us.

------------------------------------------------------------------------

# 11. createApi

`createApi` is the main API definition function for RTK Query.

Example:

```
import {
  createApi,
  fetchBaseQuery
} from "@reduxjs/toolkit/query/react";

export const api = createApi({
  reducerPath: "api",

  baseQuery: fetchBaseQuery({
    baseUrl: "/api"
  }),

  endpoints: (builder) => ({
    getUsers: builder.query({
      query: () => "/users"
    }),

    createUser: builder.mutation({
      query: (user) => ({
        url: "/users",
        method: "POST",
        body: user
      })
    })
  })
});
```

------------------------------------------------------------------------

# 12. builder.query vs builder.mutation

This is a common interview question.

## Query

Used primarily for retrieving server data.

```
getUsers: builder.query({
  query: () => "/users"
})
```

Usually corresponds to:

``` text
GET
```

Examples:

``` text
GET /users
GET /products
GET /orders/123
```

------------------------------------------------------------------------

## Mutation

Used when changing server-side data.

```
createUser: builder.mutation({
  query: (user) => ({
    url: "/users",
    method: "POST",
    body: user
  })
})
```

Examples:

``` text
POST
PUT
PATCH
DELETE
```

Typical distinction:

``` text
Query     → read server state
Mutation  → change server state
```

------------------------------------------------------------------------

# 13. Generated hooks

RTK Query automatically generates React hooks from the endpoints.

For:

```
getUsers: builder.query({
  query: () => "/users"
})
```

we get:

```
useGetUsersQuery()
```

Usage:

``` tsx
const {
  data,
  isLoading,
  error
} = useGetUsersQuery();
```

RTK Query manages the request lifecycle.

------------------------------------------------------------------------

# 14. Mutation hooks

For:

```
createUser: builder.mutation({
  query: (user) => ({
    url: "/users",
    method: "POST",
    body: user
  })
})
```

we get:

```
useCreateUserMutation()
```

Usage:

``` tsx
const [
  createUser,
  {
    isLoading,
    error
  }
] = useCreateUserMutation();
```

Then:

``` tsx
const handleSubmit = async () => {
  try {
    const result = await createUser(user).unwrap();

    console.log(result);
  } catch (error) {
    console.error(error);
  }
};
```

------------------------------------------------------------------------

# 15. What RTK Query manages for us

Without RTK Query, we might write:

``` text
useState(data)
useState(isLoading)
useState(error)

useEffect(...)
fetch(...)
try/catch
cache handling
refetch handling
duplicate request handling
```

RTK Query handles many of these concerns.

It provides features such as:

-   Loading state
-   Error state
-   Request lifecycle
-   Caching
-   Cache subscriptions
-   Deduplication
-   Refetching
-   Cache invalidation
-   Generated hooks

------------------------------------------------------------------------

# 16. RTK Query caching

Suppose two components use:

``` tsx
const { data } = useGetUsersQuery();
```

RTK Query can share the cached result instead of treating each component
as an entirely independent API request.

Conceptually:

``` text
Component A ──┐
              ├──> RTK Query cache ──> /users
Component B ──┘
```

This is one of the major benefits of RTK Query.

------------------------------------------------------------------------

# 17. Cache invalidation with tags

One of the most important RTK Query interview topics.

Define tag types:

```
export const api = createApi({
  reducerPath: "api",

  baseQuery: fetchBaseQuery({
    baseUrl: "/api"
  }),

  tagTypes: ["Users"],

  endpoints: (builder) => ({
    getUsers: builder.query({
      query: () => "/users",
      providesTags: ["Users"]
    }),

    createUser: builder.mutation({
      query: (user) => ({
        url: "/users",
        method: "POST",
        body: user
      }),

      invalidatesTags: ["Users"]
    })
  })
});
```

Flow:

``` text
GET /users
    ↓
Query provides "Users" tag
    ↓
Result cached

POST /users
    ↓
Mutation invalidates "Users"
    ↓
Existing Users query becomes stale
    ↓
Active subscription generally refetches
```

Important interview wording:

> `invalidatesTags` does not simply mean "delete the cache." It marks
> related cached data as invalid, and active queries using those tags
> can refetch.

------------------------------------------------------------------------

# 18. More granular cache invalidation

Instead of invalidating every user query, we can use IDs.

```
getUsers: builder.query({
  query: () => "/users",

  providesTags: (result) =>
    result
      ? [
          ...result.map(user => ({
            type: "Users",
            id: user.id
          })),
          {
            type: "Users",
            id: "LIST"
          }
        ]
      : [
          {
            type: "Users",
            id: "LIST"
          }
        ]
})
```

Then:

```
updateUser: builder.mutation({
  query: ({ id, ...body }) => ({
    url: `/users/${id}`,
    method: "PUT",
    body
  }),

  invalidatesTags: (result, error, arg) => [
    {
      type: "Users",
      id: arg.id
    }
  ]
})
```

This allows more precise invalidation.

------------------------------------------------------------------------

# 19. RTK vs RTK Query

These are related but should not be confused.

### Redux Toolkit

Provides tools for writing Redux:

``` text
configureStore
createSlice
createReducer
createAction
createAsyncThunk
createEntityAdapter
createApi
...
```

### RTK Query

Is the API/server-state portion of Redux Toolkit.

``` text
createApi
    ↓
Queries
Mutations
Caching
Refetching
Invalidation
Generated hooks
```

Interview answer:

> Redux Toolkit is the recommended way to write Redux logic, while RTK
> Query is a data-fetching and caching solution within Redux Toolkit for
> managing server state.

------------------------------------------------------------------------

# 20. Redux Toolkit vs manually using Redux for API calls

### Traditional Redux approach

You might have:

``` text
FETCH_USERS_REQUEST
FETCH_USERS_SUCCESS
FETCH_USERS_FAILURE
```

Then:

```
dispatch(fetchUsers());
```

and maintain:

```
{
  users: [],
  loading: false,
  error: null
}
```

You also need to implement:

``` text
API call
loading state
error handling
success handling
cache
refetch
invalidation
```

### RTK Query

You can define:

```
getUsers: builder.query({
  query: () => "/users"
})
```

and use:

``` tsx
const {
  data,
  isLoading,
  error
} = useGetUsersQuery();
```

This removes a lot of manual server-state management.

------------------------------------------------------------------------

# 21. What should go into Redux vs RTK Query?

This is a very important architecture interview question.

## Redux slice

Use a slice for client/application state.

Examples:

``` text
Theme
Modal state
Shopping cart
UI preferences
Selected filters
Wizard state
Local application workflow state
```

Example:

```
const cartSlice = createSlice({
  name: "cart",

  initialState: {
    items: []
  },

  reducers: {
    addItem: (state, action) => {
      state.items.push(action.payload);
    }
  }
});
```

## RTK Query

Use RTK Query for server state.

Examples:

``` text
Users
Products
Orders
Transactions
Account information
API search results
```

A useful mental model:

``` text
                 Application State
                       |
              ┌────────┴────────┐
              ↓                 ↓
        Client/UI state     Server state
              ↓                 ↓
        createSlice         RTK Query
```

------------------------------------------------------------------------

# 22. Complete example

## API

```
import {
  createApi,
  fetchBaseQuery
} from "@reduxjs/toolkit/query/react";

export const api = createApi({
  reducerPath: "api",

  baseQuery: fetchBaseQuery({
    baseUrl: "/api"
  }),

  tagTypes: ["Products"],

  endpoints: (builder) => ({
    getProducts: builder.query({
      query: () => "/products",
      providesTags: ["Products"]
    }),

    createProduct: builder.mutation({
      query: (product) => ({
        url: "/products",
        method: "POST",
        body: product
      }),

      invalidatesTags: ["Products"]
    })
  })
});

export const {
  useGetProductsQuery,
  useCreateProductMutation
} = api;
```

## Store

```
import { configureStore } from "@reduxjs/toolkit";
import cartReducer from "./cartSlice";
import { api } from "./api";

export const store = configureStore({
  reducer: {
    cart: cartReducer,

    [api.reducerPath]: api.reducer
  },

  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware)
});
```

Two important RTK Query store integrations are:

```
[api.reducerPath]: api.reducer
```

and:

```
getDefaultMiddleware().concat(api.middleware)
```

Without integrating the API reducer and middleware into the store, RTK
Query will not work correctly.

------------------------------------------------------------------------

# 23. React usage

``` tsx
function Products() {
  const {
    data: products,
    isLoading,
    error
  } = useGetProductsQuery();

  if (isLoading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Failed to load products</p>;
  }

  return (
    <div>
      {products?.map(product => (
        <div key={product.id}>
          {product.name}
        </div>
      ))}
    </div>
  );
}
```

Mutation:

``` tsx
function AddProduct() {
  const [
    createProduct,
    { isLoading }
  ] = useCreateProductMutation();

  const handleSubmit = async () => {
    await createProduct({
      name: "Laptop",
      price: 50000
    }).unwrap();
  };

  return (
    <button
      disabled={isLoading}
      onClick={handleSubmit}
    >
      Add Product
    </button>
  );
}
```

Because the mutation invalidates the `Products` tag, an active
`getProducts` query can refetch automatically.

------------------------------------------------------------------------

# 24. Interview comparison table

  -----------------------------------------------------------------------
  Concern                 Traditional Redux       Redux Toolkit
  ----------------------- ----------------------- -----------------------
  Store setup             Manual                  `configureStore()`

  Action types            Manual constants        Generated by
                                                  `createSlice()`

  Action creators         Manual                  Generated

  Reducers                Switch statements       Slice reducers

  Immutable updates       Manual spread/copy      Immer

  Middleware setup        More manual             Sensible defaults

  DevTools                Manual/configuration    Integrated by default

  Async logic             Commonly thunk/custom   Thunk included;
                          setup                   `createAsyncThunk`
                                                  available

  API fetching            Manual Redux logic      RTK Query

  Caching                 Usually custom          RTK Query

  Cache invalidation      Usually custom          RTK Query tags

  Boilerplate             High                    Much lower
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 25. Top interview questions

## Q1. What is Redux Toolkit?

> Redux Toolkit is the official recommended way to write Redux logic. It
> reduces boilerplate and provides utilities such as `configureStore`
> and `createSlice`, uses Immer for immutable updates, provides useful
> middleware defaults, and includes RTK Query for server-state fetching
> and caching.

------------------------------------------------------------------------

## Q2. What does configureStore do?

> `configureStore` creates the Redux store with sensible defaults. It
> combines reducers, configures middleware, integrates Redux DevTools,
> and enables useful development checks.

------------------------------------------------------------------------

## Q3. What is createSlice?

> `createSlice` lets us define a slice name, initial state, and reducers
> together. It automatically generates the corresponding action creators
> and action types, reducing Redux boilerplate.

------------------------------------------------------------------------

## Q4. How can you mutate state inside an RTK reducer?

> RTK uses Immer internally. The reducer receives a draft state, so we
> can write mutation-like operations such as `state.items.push(item)`.
> Immer converts those changes into a new immutable state.

------------------------------------------------------------------------

## Q5. What is RTK Query?

> RTK Query is the data-fetching and caching solution included with
> Redux Toolkit. It manages server-state concerns such as API requests,
> loading and error states, caching, refetching, deduplication, and
> cache invalidation.

------------------------------------------------------------------------

## Q6. Difference between query and mutation?

> A query is primarily used to retrieve server data, while a mutation is
> used to change server-side data. Queries are commonly associated with
> GET requests, while mutations are commonly used for POST, PUT, PATCH,
> and DELETE operations.

------------------------------------------------------------------------

## Q7. How does RTK Query invalidate cache?

> A query can declare `providesTags`, and a mutation can declare
> `invalidatesTags`. When the mutation invalidates a tag used by an
> active query, RTK Query marks the cached data stale and can refetch
> the query.

------------------------------------------------------------------------

## Q8. Redux Toolkit vs RTK Query?

> Redux Toolkit is the broader library for writing Redux logic. RTK
> Query is one part of Redux Toolkit focused specifically on API data
> fetching, caching, synchronization, and server state.

------------------------------------------------------------------------

# 26. One-minute interview answer

If an interviewer asks:

**"Why do you use Redux Toolkit instead of traditional Redux?"**

Answer:

> We use Redux Toolkit because it is the recommended approach for
> writing Redux and significantly reduces boilerplate. With
> `configureStore`, store and middleware setup is simplified. With
> `createSlice`, action types, action creators, and reducers can be
> defined together. RTK also uses Immer, so reducers can use
> mutation-like syntax while maintaining immutable state internally. For
> server-side data, we can use RTK Query through `createApi`, which
> provides generated hooks, caching, loading and error handling,
> refetching, and cache invalidation. So it gives us a much simpler and
> more maintainable way to handle both client-side Redux state and
> server state.

------------------------------------------------------------------------

# 27. The three APIs to remember

For interviews, keep this mental model:

``` text
configureStore
      ↓
Creates/configures Redux store

createSlice
      ↓
Creates Redux state logic
(actions + reducers)

createApi
      ↓
RTK Query
(API calls + caching + server state)
```

And the architecture:

``` text
                         React App
                            |
             ┌──────────────┴──────────────┐
             ↓                             ↓
       Client/UI State               Server State
             ↓                             ↓
       createSlice                    createApi
             ↓                             ↓
       Redux Store                  RTK Query Cache
             └──────────────┬──────────────┘
                            ↓
                     configureStore
```
