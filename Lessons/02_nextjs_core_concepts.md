# Lesson 02: Next.js Core Concepts

## Understanding the Next.js File Structure and Conventions

Next.js employs a structured file system that simplifies the development process by using conventions to define routes, components, and application behavior. Understanding this structure is crucial for efficiently navigating and building applications with Next.js.

### Top-Level Directory Structure
When you create a new Next.js project, the following top-level directories and files are typically generated:

- **`/pages`**: Contains the application's pages. Each file in this directory corresponds to a route based on its filename.
- **`/public`**: This directory is for static assets such as images, fonts, and other files that should be served directly.
- **`/styles`**: Contains global styles and CSS modules for styling your application.
- **`/api`**: (Optional) Can be used for API routes if you choose to place them in a separate directory.
- **`/src`**: (Optional) A source directory for organizing your application code, separating it from configuration files.
- **`next.config.js`**: Configuration file for customizing Next.js settings.
- **`package.json`**: Contains project metadata, dependencies, and scripts.
- **`.env`**: Environment variables for configuration.

### Pages Directory and Routing
The `/pages` directory is the heart of routing in a Next.js application. Each file within this directory automatically becomes a route.

#### File Naming Conventions
- **`index.js`**: Represents the root route (`/`).
- **`about.js`**: Represents the `/about` route.
- **`blog/[id].js`**: Represents dynamic routes, allowing for parameters (e.g., `/blog/1`, `/blog/2`).
- **`404.js`**: Custom error page for handling not found routes.

### App Directory (Next.js 13+)
In Next.js 13 and later, the introduction of the `/app` directory allows for more advanced routing features, such as nested layouts and improved organization.

- **`/app`**: Contains route segments. Each folder represents a route segment.
- **`/app/layout.js`**: Defines a common layout for the pages within that segment.
- **`/app/page.js`**: Represents the main page for that segment.
- **`/app/error.js`**: Custom error handling for that segment.

## Pages and Routing: From Basic to Advanced

Next.js simplifies routing through its file-based routing system. This section will cover basic routing concepts and advanced routing techniques.

### Basic Routing
To create a basic page, simply add a new file to the `/pages` directory. For example, creating `contact.js` will automatically create a route at `/contact`.

### Dynamic Routing
Dynamic routes allow you to create pages that can accept parameters. For instance, creating a file named `[id].js` in the `/pages/blog` directory will enable routes like `/blog/1`, `/blog/2`, etc.

#### Example of Dynamic Routing
```javascript
// pages/blog/[id].js
import { useRouter } from 'next/router';

const BlogPost = () => {
  const router = useRouter();
  const { id } = router.query;

  return <h1>Blog Post: {id}</h1>;
};

export default BlogPost;
```

### Nested Routes
You can create nested routes by organizing files into subdirectories. For example:
- `/pages/blog/index.js` for `/blog`
- `/pages/blog/[id].js` for `/blog/[id]`

### Catch-All Routes
To match multiple segments, use a catch-all route. For example, `[...slug].js` will match `/a`, `/a/b`, `/a/b/c`, etc.

#### Example of Catch-All Routes
```javascript
// pages/docs/[...slug].js
const Docs = () => {
  return <h1>Documentation</h1>;
};

export default Docs;
```

## Server-Side Rendering (SSR) vs. Static Site Generation (SSG)

Next.js provides two primary methods for rendering pages: Server-Side Rendering (SSR) and Static Site Generation (SSG). Understanding when to use each method is crucial for optimizing performance and SEO.

### Server-Side Rendering (SSR)
SSR generates the HTML for a page on each request. This is useful for pages that require up-to-date data.

#### Example of SSR
```javascript
// pages/users.js
export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/users');
  const users = await res.json();

  return { props: { users } };
}

const Users = ({ users }) => {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};

export default Users;
```

### Static Site Generation (SSG)
SSG generates the HTML at build time, serving static pages. This is ideal for pages that do not change often.

#### Example of SSG
```javascript
// pages/posts.js
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return { props: { posts } };
}

const Posts = ({ posts }) => {
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
};

export default Posts;
```

### Incremental Static Regeneration (ISR)
ISR allows you to update static pages after the build without rebuilding the entire site. You can specify a revalidation time.

#### Example of ISR
```javascript
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: { posts },
    revalidate: 10, // Revalidate every 10 seconds
  };
}
```

## Client-Side Rendering and Hydration

Client-side rendering (CSR) occurs when JavaScript runs in the browser to render content. Next.js uses hydration to enhance the performance of CSR.

### Hydration
Hydration is the process of taking a server-rendered page and making it interactive by attaching event listeners and state management. This allows for a seamless transition from server-rendered content to a fully interactive application.

### Example of Hydration
```javascript
// pages/index.js
import { useState } from 'react';

const Home = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

export default Home;
```

## The Next.js Component Lifecycle

Understanding the component lifecycle in Next.js is essential for managing data fetching, rendering, and state.

### Lifecycle Methods
Next.js components follow the React component lifecycle, including:
- **Mounting**: When the component is first rendered.
- **Updating**: When state or props change.
- **Unmounting**: When the component is removed from the DOM.

### Data Fetching Methods
Next.js provides specific methods for data fetching that align with the component lifecycle:
- **getStaticProps**: Runs at build time for static generation.
- **getServerSideProps**: Runs on each request for server-side rendering.
- **getStaticPaths**: Used with dynamic routes to specify which paths to pre-render.

### Example of Using Lifecycle Methods
```javascript
// pages/profile.js
import { useEffect } from 'react';

const Profile = ({ user }) => {
  useEffect(() => {
    // Perform side effects here
  }, []);

  return <h1>{user.name}'s Profile</h1>;
};

export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/user');
  const user = await res.json();

  return { props: { user } };
}

export default Profile;
```

## Conclusion

In this lesson, you learned about the core concepts of Next.js, including its file structure, routing mechanisms, rendering methods, and component lifecycle. Understanding these concepts is essential for building efficient and scalable web applications with Next.js and TypeScript. In the next lesson, we will delve deeper into advanced routing techniques and data fetching strategies to enhance your applications further.

[Next Lesson](./03_advanced_routing_and_navigation.md)