# Lesson 06: Advanced TypeScript Techniques in Next.js

In this lesson, we will delve into advanced TypeScript techniques that enhance your development experience when building applications with Next.js. These techniques will help you leverage TypeScript's powerful features to create type-safe, maintainable, and scalable applications. We will cover generic components and hooks, advanced type inference and type guards, mapped types and conditional types, creating type-safe API clients, and performance optimizations with TypeScript.

## Generic Components and Hooks

Generics in TypeScript allow you to create reusable components and hooks that can work with different types while maintaining type safety. This is particularly useful in Next.js, where you may have components that need to handle various data types.

### Creating Generic Components

To create a generic component, you define a type parameter that can be used within the component.

#### Example of a Generic Component
```typescript
// components/List.tsx
import React from 'react';

interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

const List = <T,>({ items, renderItem }: ListProps<T>) => {
  return <ul>{items.map((item, index) => <li key={index}>{renderItem(item)}</li>)}</ul>;
};

export default List;
```

In this example, the `List` component accepts an array of items of any type `T` and a render function to display each item.

### Using Generic Components
You can use the `List` component with different data types, such as strings or objects.

#### Example of Using a Generic Component
```typescript
// pages/index.tsx
import List from '../components/List';

const Home = () => {
  const stringItems = ['Apple', 'Banana', 'Cherry'];

  return (
    <div>
      <h1>Fruits</h1>
      <List items={stringItems} renderItem={(item) => <span>{item}</span>} />
    </div>
  );
};

export default Home;
```

## Advanced Type Inference and Type Guards

Type inference in TypeScript allows the compiler to automatically deduce types based on the values you provide. This can simplify your code and reduce the need for explicit type annotations.

### Type Guards

Type guards are a way to narrow down the type of a variable within a conditional block. They help ensure that you are working with the correct type before performing operations.

#### Example of Type Guards
```typescript
interface Dog {
  bark: () => void;
}

interface Cat {
  meow: () => void;
}

type Pet = Dog | Cat;

const isDog = (pet: Pet): pet is Dog => {
  return (pet as Dog).bark !== undefined;
};

const handlePet = (pet: Pet) => {
  if (isDog(pet)) {
    pet.bark(); // TypeScript knows pet is a Dog here
  } else {
    pet.meow(); // TypeScript knows pet is a Cat here
  }
};
```

In this example, the `isDog` function acts as a type guard, allowing TypeScript to infer the correct type of `pet` within the conditional blocks.

## Mapped Types and Conditional Types in React Props

Mapped types and conditional types are powerful features in TypeScript that allow you to create new types based on existing ones.

### Mapped Types

Mapped types allow you to create new types by transforming properties of an existing type.

#### Example of Mapped Types
```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

type ReadonlyUser = {
  readonly [K in keyof User]: User[K];
};

// ReadonlyUser is equivalent to:
// {
//   readonly id: number;
//   readonly name: string;
//   readonly email: string;
// }
```

### Conditional Types

Conditional types enable you to create types based on conditions. They are defined using the `extends` keyword.

#### Example of Conditional Types
```typescript
type IsString<T> = T extends string ? 'Yes' : 'No';

type Test1 = IsString<string>; // 'Yes'
type Test2 = IsString<number>; // 'No'
```

### Using Mapped and Conditional Types in React Props

You can combine mapped and conditional types to create flexible and type-safe props for your components.

#### Example of Using Mapped and Conditional Types
```typescript
type ButtonProps<T> = {
  label: string;
  onClick: (value: T) => void;
};

type StringButtonProps = ButtonProps<string>;

const StringButton: React.FC<StringButtonProps> = ({ label, onClick }) => {
  return <button onClick={() => onClick('clicked')}>{label}</button>;
};
```

## Creating Type-Safe API Clients

When working with APIs, it’s essential to ensure that your API clients are type-safe. This can help prevent runtime errors and improve the developer experience.

### Defining API Response Types

Start by defining the expected response types from your API.

#### Example of Defining API Response Types
```typescript
// types/api.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export interface ApiResponse<T> {
  data: T;
  error?: string;
}
```

### Creating a Type-Safe API Client

You can create a function that fetches data from your API and returns a type-safe response.

#### Example of a Type-Safe API Client
```typescript
// lib/apiClient.ts
import { User, ApiResponse } from '../types/api';

const fetcher = async (url: string): Promise<ApiResponse<User[]>> => {
  const response = await fetch(url);
  if (!response.ok) {
    return { data: [], error: 'Failed to fetch data' };
  }
  const data = await response.json();
  return { data };
};

export const getUsers = async (): Promise<ApiResponse<User[]>> => {
  return fetcher('https://api.example.com/users');
};
```

### Using the Type-Safe API Client

You can now use the type-safe API client in your components.

#### Example of Using the API Client
```typescript
// pages/users.tsx
import { useEffect, useState } from 'react';
import { getUsers } from '../lib/apiClient';
import { User } from '../types/api';

const UsersPage = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUsers = async () => {
      const response = await getUsers();
      if (response.error) {
        setError(response.error);
      } else {
        setUsers(response.data);
      }
    };

    fetchUsers();
  }, []);

  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};

export default UsersPage;
```

## Performance Optimizations with TypeScript

TypeScript can help improve the performance of your Next.js applications by enabling better code organization, type safety, and compile-time checks. Here are some performance optimization techniques:

### 1. Avoiding Unnecessary Re-renders

Using TypeScript's type checks can help you ensure that your components only re-render when necessary. For example, using `React.memo` with typed props can prevent unnecessary re-renders.

#### Example of Using React.memo
```typescript
import React from 'react';

type UserProps = {
  user: User;
};

const UserCard: React.FC<UserProps> = React.memo(({ user }) => {
  return <div>{user.name}</div>;
});
```

### 2. Leveraging Type Inference

TypeScript's type inference can help reduce boilerplate code and improve performance by allowing you to avoid explicit type annotations where they are unnecessary.

#### Example of Type Inference
```typescript
const users = await fetchUsers(); // TypeScript infers the type of users
```

### 3. Using TypeScript with Next.js Features

Take advantage of Next.js features that work well with TypeScript, such as `getStaticProps` and `getServerSideProps`, to optimize data fetching and rendering strategies.

### 4. Optimizing TypeScript Configuration

You can optimize the TypeScript configuration in your `tsconfig.json` file to improve performance and reduce build times. For example, enabling `skipLibCheck` can speed up compilation by skipping type checking of declaration files.

#### Example of tsconfig.json Optimization
```json
{
  "compilerOptions": {
    "skipLibCheck": true,
    "strict": true,
    "noImplicitAny": true,
    "jsx": "preserve",
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "esModuleInterop": true
  }
}
```

## Conclusion

In this lesson, you explored advanced TypeScript techniques that enhance your Next.js development experience. You learned about generic components and hooks, advanced type inference and type guards, mapped types and conditional types, creating type-safe API clients, and performance optimizations with TypeScript. With these techniques, you can build robust, maintainable, and scalable applications using Next.js and TypeScript. In the next lesson, we will cover styling and UI development techniques to enhance the user experience further.

[Next Lesson](./07_styling_and_ui_development.md)