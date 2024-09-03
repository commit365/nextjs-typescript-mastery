# Lesson 03: Advanced Routing and Navigation

## Dynamic Routes and Route Parameters

Dynamic routing in Next.js allows you to create pages that can accept variable parameters in the URL. This is particularly useful for applications that display content based on user input or dynamic data, such as blog posts, product pages, or user profiles.

### Creating Dynamic Routes

To create a dynamic route, you simply wrap the filename in square brackets within the `pages` directory. For example, if you want to create a blog post page that can accept a `slug` parameter, you would create a file named `[slug].js` in the `pages/blog` directory.

#### Example
```javascript
// pages/blog/[slug].js
import { useRouter } from 'next/router';

const BlogPost = () => {
  const router = useRouter();
  const { slug } = router.query;

  return <h1>Blog Post: {slug}</h1>;
};

export default BlogPost;
```

In this example, when a user navigates to `/blog/my-first-post`, the `slug` variable will be populated with `my-first-post`.

### Accessing Route Parameters

You can access the dynamic parameters using the `useRouter` hook from Next.js. The `query` object contains all the parameters defined in your dynamic route.

#### Example
```javascript
const BlogPost = () => {
  const router = useRouter();
  const { slug } = router.query;

  return (
    <div>
      <h1>Blog Post: {slug}</h1>
      <p>This is the content for the post with slug: {slug}</p>
    </div>
  );
};
```

### Catch-All Routes

Catch-all routes allow you to match multiple segments in a URL. You can create a catch-all route by using an ellipsis (`...`) inside the square brackets.

#### Example
To create a catch-all route for a shop that can handle nested categories:
```javascript
// pages/shop/[...slug].js
const Shop = () => {
  const router = useRouter();
  const { slug } = router.query;

  return <h1>Shop Categories: {slug.join(' > ')}</h1>;
};

export default Shop;
```

In this case, navigating to `/shop/clothes/tops` will result in `slug` being `['clothes', 'tops']`.

### Optional Catch-All Routes

You can make catch-all routes optional by wrapping the parameter in double square brackets. This allows the route to match even if no parameters are provided.

#### Example
```javascript
// pages/shop/[[...slug]].js
const Shop = () => {
  const router = useRouter();
  const { slug } = router.query;

  return <h1>Shop Categories: {slug ? slug.join(' > ') : 'Home'}</h1>;
};

export default Shop;
```

Now, navigating to `/shop` will display "Home", while `/shop/clothes/tops` will display "Shop Categories: clothes > tops".

## Programmatic Navigation with useRouter

Next.js provides the `useRouter` hook to enable programmatic navigation within your application. This is useful for redirecting users or navigating based on certain actions, such as form submissions.

### Using useRouter for Navigation

To navigate programmatically, you can use the `push` or `replace` methods provided by the `router` object.

#### Example
```javascript
import { useRouter } from 'next/router';

const MyComponent = () => {
  const router = useRouter();

  const handleNavigation = () => {
    router.push('/blog/my-first-post');
  };

  return <button onClick={handleNavigation}>Go to My First Post</button>;
};

export default MyComponent;
```

In this example, clicking the button will navigate the user to the specified blog post.

### Replacing the Current Route

If you want to replace the current route instead of pushing a new entry onto the history stack, you can use the `replace` method.

#### Example
```javascript
const handleReplace = () => {
  router.replace('/blog/my-first-post');
};
```

## Custom Link Components and Prefetching Strategies

Next.js provides a built-in `Link` component for navigating between pages. This component enables client-side navigation and supports prefetching for improved performance.

### Using the Link Component

To use the `Link` component, import it from `next/link` and wrap it around the element you want to make clickable.

#### Example
```javascript
import Link from 'next/link';

const Navigation = () => {
  return (
    <nav>
      <Link href="/blog/my-first-post">
        <a>My First Post</a>
      </Link>
      <Link href="/about">
        <a>About Us</a>
      </Link>
    </nav>
  );
};

export default Navigation;
```

### Prefetching Links

By default, Next.js prefetches linked pages in the background when they are visible in the viewport. This means that when a user clicks a link, the page will load almost instantly.

You can disable prefetching by setting the `prefetch` prop to `false`:
```javascript
<Link href="/blog/my-first-post" prefetch={false}>
  <a>My First Post</a>
</Link>
```

## Internationalized Routing and Language Detection

Next.js supports internationalization (i18n) out of the box, allowing you to create applications that cater to multiple languages and locales.

### Setting Up Internationalized Routing

To enable internationalized routing, you need to configure the `i18n` property in your `next.config.js` file. You can specify the supported locales and the default locale.

#### Example
```javascript
// next.config.js
module.exports = {
  i18n: {
    locales: ['en-US', 'fr', 'es'],
    defaultLocale: 'en-US',
  },
};
```

### Language Detection

Next.js automatically detects the user's preferred language based on their browser settings. You can access the current locale using the `useRouter` hook.

#### Example
```javascript
import { useRouter } from 'next/router';

const MyComponent = () => {
  const router = useRouter();
  const { locale } = router;

  return <h1>Current Language: {locale}</h1>;
};
```

### Creating Localized Routes

You can create localized routes by using the locale in your links. The `Link` component will automatically handle the locale based on the current settings.

#### Example
```javascript
<Link href="/about" locale="fr">
  <a>À propos</a>
</Link>
```

## Conclusion

In this lesson, you explored advanced routing and navigation techniques in Next.js, including dynamic routes, nested routes, programmatic navigation, custom link components, and internationalized routing. Mastering these concepts is essential for building complex applications that require flexible navigation and dynamic content handling. In the next lesson, we will delve into data fetching strategies to enhance your applications further.

[Next Lesson](./04_data_fetching_and_state_management.md)