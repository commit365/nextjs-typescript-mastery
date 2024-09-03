# Lesson 04: Data Fetching and State Management

In this lesson, we will explore the various methods of data fetching in Next.js, including static and server-side rendering techniques, as well as client-side data fetching strategies. We will also discuss state management solutions, including Redux and the Context API, to manage application state effectively. By the end of this lesson, you will have a solid understanding of how to fetch data and manage state in your Next.js applications.

## getStaticProps, getServerSideProps, and getInitialProps

Next.js provides several methods for fetching data, each suited for different use cases. Understanding when and how to use these methods is crucial for optimizing performance and user experience.

### 1. getStaticProps

`getStaticProps` is used for static site generation (SSG). It allows you to fetch data at build time, which is then used to generate static HTML pages. This method is ideal for pages that do not change frequently and can be pre-rendered.

#### Example of getStaticProps
```javascript
// pages/products.js
import React from 'react';

export async function getStaticProps() {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();

  return {
    props: {
      products,
    },
  };
}

const Products = ({ products }) => {
  return (
    <div>
      <h1>Products</h1>
      <ul>
        {products.map((product) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  );
};

export default Products;
```

### 2. getServerSideProps

`getServerSideProps` is used for server-side rendering (SSR). It fetches data on each request, making it suitable for pages that require up-to-date information.

#### Example of getServerSideProps
```javascript
// pages/users.js
import React from 'react';

export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/users');
  const users = await res.json();

  return {
    props: {
      users,
    },
  };
}

const Users = ({ users }) => {
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

export default Users;
```

### 3. getInitialProps

`getInitialProps` is an older data fetching method that can be used in both pages and custom App components. It runs on the server during the initial page load and on the client for subsequent navigation. However, it is less preferred in favor of `getStaticProps` and `getServerSideProps` due to its complexity and performance implications.

#### Example of getInitialProps
```javascript
// pages/about.js
import React from 'react';

const About = ({ data }) => {
  return <h1>About: {data}</h1>;
};

About.getInitialProps = async () => {
  const res = await fetch('https://api.example.com/about');
  const data = await res.json();
  return { data };
};

export default About;
```

## Incremental Static Regeneration (ISR) and Revalidation Strategies

Incremental Static Regeneration (ISR) allows you to update static pages after the site has been built. This is particularly useful for pages that require periodic updates without needing a full rebuild.

### Using ISR with getStaticProps

You can implement ISR by using the `revalidate` property in the return object of `getStaticProps`. This property specifies the number of seconds after which a page re-generation can occur.

#### Example of ISR
```javascript
// pages/posts.js
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: {
      posts,
    },
    revalidate: 10, // Revalidate every 10 seconds
  };
}

const Posts = ({ posts }) => {
  return (
    <div>
      <h1>Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
};

export default Posts;
```

### Revalidation Strategies

- **On-demand Revalidation**: You can trigger a revalidation programmatically using Next.js API routes. This is useful for scenarios where you want to refresh data based on specific events (e.g., after a form submission).

- **Fallback Pages**: When using ISR, you can specify a fallback option to serve a static version of a page while the new version is being generated.

## SWR for Client-Side Data Fetching and Caching

SWR (stale-while-revalidate) is a React Hooks library developed by Vercel for data fetching. It provides a simple and efficient way to handle client-side data fetching, caching, and revalidation.

### Installing SWR

To use SWR, you first need to install it:
```bash
npm install swr
```

### Basic Usage of SWR

You can use the `useSWR` hook to fetch data and manage caching automatically.

#### Example of Using SWR
```javascript
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then((res) => res.json());

const Products = () => {
  const { data, error } = useSWR('https://api.example.com/products', fetcher);

  if (error) return <div>Error loading products.</div>;
  if (!data) return <div>Loading...</div>;

  return (
    <div>
      <h1>Products</h1>
      <ul>
        {data.map((product) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  );
};

export default Products;
```

### Features of SWR

- **Caching**: SWR caches the data fetched from the API and serves it instantly on subsequent requests.
- **Revalidation**: Automatically revalidates the data in the background, ensuring the UI is always up-to-date.
- **Error Handling**: Provides built-in error handling capabilities.

## Implementing Redux with Next.js and TypeScript

Redux is a popular state management library that can be integrated with Next.js to manage global state across your application.

### Installing Redux

To use Redux in your Next.js application, install the necessary packages:
```bash
npm install redux react-redux @reduxjs/toolkit
```

### Setting Up Redux Store

Create a `store.js` file to configure your Redux store.

#### Example of Redux Store
```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './reducers'; // Import your root reducer

const store = configureStore({
  reducer: rootReducer,
});

export default store;
```

### Providing the Store to Your Application

Wrap your application with the Redux Provider in the custom `_app.js` file.

#### Example of _app.js
```javascript
// pages/_app.js
import { Provider } from 'react-redux';
import store from '../store';

const MyApp = ({ Component, pageProps }) => {
  return (
    <Provider store={store}>
      <Component {...pageProps} />
    </Provider>
  );
};

export default MyApp;
```

### Creating a Redux Slice

You can create slices of state using Redux Toolkit.

#### Example of a Redux Slice
```javascript
// reducers/userSlice.js
import { createSlice } from '@reduxjs/toolkit';

const userSlice = createSlice({
  name: 'user',
  initialState: { name: '', age: 0 },
  reducers: {
    setUser: (state, action) => {
      state.name = action.payload.name;
      state.age = action.payload.age;
    },
  },
});

export const { setUser } = userSlice.actions;
export default userSlice.reducer;
```

### Using Redux State in Components

You can access and update the Redux state using hooks provided by `react-redux`.

#### Example of Using Redux in a Component
```javascript
import { useSelector, useDispatch } from 'react-redux';
import { setUser } from '../reducers/userSlice';

const UserProfile = () => {
  const user = useSelector((state) => state.user);
  const dispatch = useDispatch();

  const updateUser = () => {
    dispatch(setUser({ name: 'John Doe', age: 30 }));
  };

  return (
    <div>
      <h1>User Profile</h1>
      <p>Name: {user.name}</p>
      <p>Age: {user.age}</p>
      <button onClick={updateUser}>Update User</button>
    </div>
  );
};

export default UserProfile;
```

## Context API and Custom Hooks for State Management

The Context API is a built-in feature of React that allows you to manage global state without the need for external libraries like Redux. It is particularly useful for smaller applications or specific use cases.

### Creating a Context

You can create a context using `React.createContext()`.

#### Example of Creating a Context
```javascript
// context/UserContext.js
import { createContext, useContext, useState } from 'react';

const UserContext = createContext();

export const UserProvider = ({ children }) => {
  const [user, setUser] = useState({ name: '', age: 0 });

  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
};

export const useUser = () => useContext(UserContext);
```

### Using the Context Provider

Wrap your application with the context provider in `_app.js`.

#### Example of Using UserProvider
```javascript
// pages/_app.js
import { UserProvider } from '../context/UserContext';

const MyApp = ({ Component, pageProps }) => {
  return (
    <UserProvider>
      <Component {...pageProps} />
    </UserProvider>
  );
};

export default MyApp;
```

### Accessing Context in Components

You can access the context values using the custom hook.

#### Example of Using the Context in a Component
```javascript
import { useUser } from '../context/UserContext';

const UserProfile = () => {
  const { user, setUser } = useUser();

  const updateUser = () => {
    setUser({ name: 'Jane Doe', age: 25 });
  };

  return (
    <div>
      <h1>User Profile</h1>
      <p>Name: {user.name}</p>
      <p>Age: {user.age}</p>
      <button onClick={updateUser}>Update User</button>
    </div>
  );
};

export default UserProfile;
```

## Conclusion

In this lesson, you learned about various data fetching methods in Next.js, including `getStaticProps`, `getServerSideProps`, and `getInitialProps`. You also explored Incremental Static Regeneration (ISR) and client-side data fetching using SWR. Additionally, you learned how to implement state management using Redux and the Context API, along with custom hooks for managing application state. With this knowledge, you are well-equipped to handle data fetching and state management in your Next.js applications. In the next lesson, we will dive into styling and UI development techniques to enhance the user experience.

[Next Lesson](./05_api_routes_and_backend_integration.md)