# Lesson 18: Advanced Patterns and Best Practices

In this lesson, we will explore advanced patterns and best practices for building robust, maintainable, and scalable applications using Next.js and TypeScript. These patterns will help you leverage the full potential of Next.js while ensuring your codebase remains clean and efficient. We will cover custom server setups for complex use cases, micro-frontends, implementing design patterns such as Higher-Order Components (HOCs) and Render Props, optimizing for Core Web Vitals and SEO, and using error boundaries and graceful degradation techniques. By the end of this lesson, you will be equipped with the knowledge to implement these advanced patterns in your Next.js applications.

## Custom Server Setup for Complex Use Cases

While Next.js comes with a built-in server, there are scenarios where you may need to create a custom server to handle specific requirements, such as integrating with existing backend services or implementing custom routing logic.

### Creating a Custom Server

1. **Setting Up the Server**: Create a file named `server.js` in the root of your Next.js project.

   **Example: server.js**
   ```javascript
   const express = require('express');
   const next = require('next');

   const dev = process.env.NODE_ENV !== 'production';
   const app = next({ dev });
   const handle = app.getRequestHandler();

   app.prepare().then(() => {
     const server = express();

     // Custom route example
     server.get('/custom', (req, res) => {
       return app.render(req, res, '/customPage', req.query);
     });

     // Handle all other routes with Next.js
     server.all('*', (req, res) => {
       return handle(req, res);
     });

     server.listen(3000, (err) => {
       if (err) throw err;
       console.log('> Ready on http://localhost:3000');
     });
   });
   ```

2. **Updating package.json**: Modify your `package.json` to include a script for starting the custom server.

   ```json
   "scripts": {
     "dev": "node server.js",
     "build": "next build",
     "start": "NODE_ENV=production node server.js"
   }
   ```

3. **Running the Custom Server**: Start your application using the following command:

   ```bash
   npm run dev
   ```

### Considerations for Custom Servers

- **Performance**: Custom servers can remove some of the performance optimizations provided by Next.js. Use them only when necessary.
- **Deployment**: Custom servers cannot be deployed on Vercel. Consider using platforms like Heroku or AWS for deployment.
- **Maintainability**: Ensure your custom server code is well-documented and maintainable, as it can become complex over time.

## Micro-Frontends with Next.js

Micro-frontends is an architectural style where independently deliverable frontend applications are composed into a larger application. This approach allows teams to work on different parts of the application independently, promoting scalability and maintainability.

### Implementing Micro-Frontends

1. **Choosing a Micro-Frontend Framework**: You can use frameworks like Module Federation (Webpack 5) or single-spa to implement micro-frontends in your Next.js application.

2. **Setting Up Module Federation**: If using Webpack 5, configure module federation in your Next.js application.

   **Example: next.config.js**
   ```javascript
   const { withModuleFederation } = require('next-module-federation');

   module.exports = withModuleFederation({
     name: 'app1',
     filename: 'static/chunks/remoteEntry.js',
     exposes: {
       './Component': './components/Component',
     },
     shared: {
       react: {
         singleton: true,
         eager: true,
         requiredVersion: false,
       },
       'react-dom': {
         singleton: true,
         eager: true,
         requiredVersion: false,
       },
     },
   });
   ```

3. **Consuming Micro-Frontends**: Use dynamic imports to load micro-frontend components from other applications.

   **Example: Loading a Micro-Frontend Component**
   ```typescript
   import dynamic from 'next/dynamic';

   const RemoteComponent = dynamic(() => import('app2/Component'));

   const MyPage = () => {
     return (
       <div>
         <h1>Main Application</h1>
         <RemoteComponent />
       </div>
     );
   };

   export default MyPage;
   ```

## Implementing Design Patterns in Next.js

Design patterns help structure your code in a way that promotes reusability and maintainability. In Next.js, you can implement various design patterns, including Higher-Order Components (HOCs) and Render Props.

### Higher-Order Components (HOCs)

HOCs are functions that take a component and return a new component, allowing you to add additional functionality or modify behavior.

#### Example: Creating a Higher-Order Component
```typescript
// hoc/withAuth.tsx
import React from 'react';
import { useSession } from 'next-auth/react';

const withAuth = (WrappedComponent: React.ComponentType) => {
  return (props: any) => {
    const { data: session, status } = useSession();

    if (status === 'loading') {
      return <div>Loading...</div>;
    }

    if (!session) {
      return <div>Please log in to access this page.</div>;
    }

    return <WrappedComponent {...props} />;
  };
};

export default withAuth;
```

#### Example: Using the HOC
```typescript
// pages/protected.tsx
import withAuth from '../hoc/withAuth';

const ProtectedPage = () => {
  return <h1>This is a protected page.</h1>;
};

export default withAuth(ProtectedPage);
```

### Render Props

Render Props is a pattern where a component's behavior is defined by a function that returns a React element.

#### Example: Creating a Render Props Component
```typescript
// components/Toggle.tsx
import React, { useState } from 'react';

const Toggle: React.FC<{ render: (isOpen: boolean, toggle: () => void) => JSX.Element }> = ({ render }) => {
  const [isOpen, setIsOpen] = useState(false);

  const toggle = () => setIsOpen((prev) => !prev);

  return render(isOpen, toggle);
};

export default Toggle;
```

#### Example: Using Render Props
```typescript
// pages/index.tsx
import Toggle from '../components/Toggle';

const HomePage = () => {
  return (
    <Toggle
      render={(isOpen, toggle) => (
        <div>
          <button onClick={toggle}>{isOpen ? 'Close' : 'Open'} Menu</button>
          {isOpen && <div>Menu Content</div>}
        </div>
      )}
    />
  );
};

export default HomePage;
```

## Optimizing for Core Web Vitals and SEO

Core Web Vitals are essential metrics that measure the performance and user experience of your web applications. Optimizing for these metrics can improve your site's SEO and user engagement.

### Key Core Web Vitals

1. **Largest Contentful Paint (LCP)**: Measures loading performance. Aim for an LCP of less than 2.5 seconds.
2. **First Input Delay (FID)**: Measures interactivity. Aim for an FID of less than 100 milliseconds.
3. **Cumulative Layout Shift (CLS)**: Measures visual stability. Aim for a CLS score of less than 0.1.

### Optimizing Core Web Vitals

1. **Optimize Images**: Use the Next.js `Image` component for automatic optimization.
2. **Minimize JavaScript**: Use code splitting and dynamic imports to reduce JavaScript payloads.
3. **Reduce Server Response Times**: Optimize API routes and use caching strategies.
4. **Preload Key Resources**: Use `<link rel="preload">` to preload critical resources.

### SEO Best Practices

1. **Use Semantic HTML**: Ensure your HTML structure is semantic and meaningful.
2. **Optimize Meta Tags**: Use dynamic meta tags for each page to improve visibility.
3. **Implement Structured Data**: Use structured data (JSON-LD) to help search engines understand your content better.

## Error Boundaries and Graceful Degradation

Error boundaries are a React feature that allows you to catch JavaScript errors in components and display a fallback UI instead of crashing the entire application. Graceful degradation ensures that your application remains functional, even when certain features fail.

### Implementing Error Boundaries

1. **Create an Error Boundary Component**:
```typescript
// components/ErrorBoundary.tsx
import React, { Component } from 'react';

interface State {
  hasError: boolean;
}

class ErrorBoundary extends Component<{}, State> {
  constructor(props: {}) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught in ErrorBoundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}

export default ErrorBoundary;
```

2. **Using the Error Boundary**:
```typescript
// pages/index.tsx
import ErrorBoundary from '../components/ErrorBoundary';

const HomePage = () => {
  return (
    <ErrorBoundary>
      <h1>Welcome to My Next.js App</h1>
      {/* Other components */}
    </ErrorBoundary>
  );
};

export default HomePage;
```

### Graceful Degradation

Graceful degradation involves designing your application to provide a basic level of functionality even when certain features are not available. This can be achieved by:

- **Feature Detection**: Use feature detection libraries like Modernizr to check for browser capabilities and provide fallbacks.
- **Progressive Enhancement**: Start with a basic experience and enhance it for users with modern browsers.

## Conclusion

In this lesson, you learned about advanced patterns and best practices for building Next.js applications using TypeScript. We covered custom server setups, micro-frontends, implementing design patterns like HOCs and Render Props, optimizing for Core Web Vitals and SEO, and using error boundaries and graceful degradation techniques. By applying these patterns and best practices, you can create robust, maintainable, and scalable applications that provide an excellent user experience. In the next lesson, we will summarize the course and discuss next steps in your journey with Next.js and TypeScript.

[Next Lesson](./19_capstone_project.md)