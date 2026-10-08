# TypeScript + React Interview Notes

## TypeScript

TypeScript provides static type checking on top of JavaScript. Types are
primarily compile-time/development-time information.

``` ts
function add(a: number, b: number): number {
  return a + b;
}
```

## Optional vs nullable

``` ts
department: string | null // property exists, value may be null
department?: string       // property may be omitted/undefined
department?: string | null // absent/undefined or null
```

## Interface vs type

Interfaces are strong for object contracts and support
extension/declaration merging. Types are flexible for unions,
intersections, tuples and other compositions.

``` ts
interface User {
  id: number;
  name: string;
}

type Status = "PENDING" | "SUCCESS" | "FAILED";
```

## Generics

``` ts
interface ApiResponse<T> {
  data: T;
  status: number;
}

type UserResponse = ApiResponse<User[]>;

function getFirst<T>(items: T[]): T {
  return items[0];
}
```

## `keyof`

``` ts
type UserKey = keyof User;
```

Produces a union of property names.

``` ts
function getProperty<T, K extends keyof T>(
  obj: T,
  key: K
) {
  return obj[key];
}
```

`K extends keyof T` constrains K to valid keys; it does not mean class
inheritance.

## `typeof`

``` ts
const user = { id: 1, name: "Vinay" };

type User = typeof user;
type UserKey = keyof typeof user;
```

In a type position, TypeScript `typeof` derives a type from a value.
JavaScript `typeof` is a runtime operator.

## React state typing

``` tsx
const [count, setCount] = useState(0);
const [user, setUser] = useState<User | null>(null);
```

## React events

``` tsx
const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {};
const onClick = (e: React.MouseEvent<HTMLButtonElement>) => {};
const onSubmit = (e: React.FormEvent<HTMLFormElement>) => {};
const onKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {};
const onFocus = (e: React.FocusEvent<HTMLInputElement>) => {};
```

Prefer exact callback signatures instead of `Function`.

## Props

``` ts
interface UserCardProps {
  name: string;
  department?: string;
  onSelect: (id: string) => void;
}
```

Children:

``` ts
children?: React.ReactNode;
```
