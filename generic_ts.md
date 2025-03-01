Advanced generics in TypeScript allow for powerful type manipulations, constraints, and inference. Here are some key concepts with examples:

---

## 1. **Generic Constraints (`extends`)**
You can restrict a generic type to certain types using `extends`.

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "Alice", age: 25 };
const name = getProperty(person, "name"); // Valid
// const invalid = getProperty(person, "gender"); // ❌ Error: "gender" does not exist in person
```

---

## 2. **Default Generic Types**
You can provide default types to generics.

```typescript
type ApiResponse<T = any> = {
  data: T;
  status: number;
};

const response: ApiResponse<string> = { data: "Success", status: 200 };
const defaultResponse: ApiResponse = { data: { key: "value" }, status: 200 };
```

---

## 3. **Using `keyof` for Dynamic Keys**
You can extract keys dynamically from a type.

```typescript
type User = { id: number; name: string; age: number };
type UserKeys = keyof User; // "id" | "name" | "age"

function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = { id: 1, name: "John", age: 30 };
console.log(getValue(user, "name")); // "John"
```

---

## 4. **Conditional Types (`extends ? :`)**
You can use conditional types to dynamically determine the output type.

```typescript
type IsString<T> = T extends string ? "Yes" : "No";

type A = IsString<string>; // "Yes"
type B = IsString<number>; // "No"
```

---

## 5. **Mapped Types (`in keyof`)**
You can create new types dynamically from existing types.

```typescript
type ReadonlyUser<T> = {
  readonly [K in keyof T]: T[K];
};

type User = { id: number; name: string };
type ReadonlyUserType = ReadonlyUser<User>;

const user: ReadonlyUserType = { id: 1, name: "Alice" };
// user.id = 2; // ❌ Error: Cannot assign to 'id' because it is a read-only property.
```

---

## 6. **Infer Keyword (`infer`)**
The `infer` keyword allows TypeScript to extract types dynamically.

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function greet(): string {
  return "Hello!";
}

type GreetReturnType = ReturnType<typeof greet>; // string
```

---

## 7. **Utility Types (`Partial<T>`, `Pick<T, K>`, `Omit<T, K>`)**
You can modify object types dynamically.

```typescript
type User = { id: number; name: string; age: number };

// Partial: All properties become optional
type PartialUser = Partial<User>;

// Pick: Select specific keys
type NameOnly = Pick<User, "name">;

// Omit: Remove specific keys
type WithoutAge = Omit<User, "age">;
```

---

## 8. **Tuple to Union (`T[number]`)**
Convert a tuple into a union type.

```typescript
type Colors = ["red", "green", "blue"];
type ColorUnion = Colors[number]; // "red" | "green" | "blue"
```

---

## 9. **Generic Interface with Multiple Parameters**
You can define interfaces with multiple generic parameters.

```typescript
interface Pair<K, V> {
  key: K;
  value: V;
}

const item: Pair<string, number> = { key: "age", value: 30 };
```

---

## 10. **Recursive Generics**
You can use generics recursively, such as in a tree structure.

```typescript
type TreeNode<T> = {
  value: T;
  children?: TreeNode<T>[];
};

const tree: TreeNode<number> = {
  value: 1,
  children: [{ value: 2 }, { value: 3, children: [{ value: 4 }] }],
};
```

---

