# Lesson 16: Integration with External Services

In this lesson, we will explore how to integrate various external services with your Next.js applications using TypeScript. Integrating external services can enhance your application’s functionality, allowing you to manage content, handle payments, authenticate users, and more. We will cover headless CMS integration (e.g., Contentful, Strapi), e-commerce solutions (e.g., Shopify, Stripe), authentication providers (e.g., Auth0, Firebase), serverless databases (e.g., FaunaDB, Supabase), and implementing real-time features with WebSockets and Socket.io. By the end of this lesson, you will have a comprehensive understanding of how to effectively integrate these services into your Next.js applications.

## Headless CMS Integration (e.g., Contentful, Strapi)

Headless Content Management Systems (CMS) allow you to manage content separately from the front-end presentation layer. This separation enables you to use any front-end framework, including Next.js, to build your application.

### Integrating Contentful

1. **Create a Contentful Account**: Sign up at [Contentful](https://www.contentful.com/) and create a new space.

2. **Set Up Content Model**: Define your content types (e.g., blog posts, products) and their fields in the Contentful dashboard.

3. **Install Contentful SDK**: Install the Contentful SDK in your Next.js project.

   ```bash
   npm install contentful
   ```

4. **Fetching Data from Contentful**: Create a utility function to fetch data from Contentful.

   **Example: lib/contentful.ts**
   ```typescript
   import { createClient } from 'contentful';

   const client = createClient({
     space: process.env.CONTENTFUL_SPACE_ID,
     accessToken: process.env.CONTENTFUL_ACCESS_TOKEN,
   });

   export const fetchEntries = async () => {
     const entries = await client.getEntries();
     return entries.items;
   };
   ```

5. **Using Fetched Data in a Component**:
   ```typescript
   // pages/index.tsx
   import { fetchEntries } from '../lib/contentful';

   const HomePage = async () => {
     const entries = await fetchEntries();

     return (
       <div>
         <h1>Contentful Entries</h1>
         <ul>
           {entries.map((entry) => (
             <li key={entry.sys.id}>{entry.fields.title}</li>
           ))}
         </ul>
       </div>
     );
   };

   export default HomePage;
   ```

### Integrating Strapi

Strapi is an open-source headless CMS that provides a flexible API for managing content.

1. **Set Up a Strapi Project**: Follow the documentation to create a new Strapi project.

2. **Install Axios**: Use Axios to fetch data from your Strapi API.

   ```bash
   npm install axios
   ```

3. **Fetching Data from Strapi**:
   ```typescript
   // lib/strapi.ts
   import axios from 'axios';

   const API_URL = process.env.STRAPI_API_URL;

   export const fetchPosts = async () => {
     const response = await axios.get(`${API_URL}/posts`);
     return response.data;
   };
   ```

4. **Using Fetched Data in a Component**:
   ```typescript
   // pages/posts.tsx
   import { fetchPosts } from '../lib/strapi';

   const PostsPage = async () => {
     const posts = await fetchPosts();

     return (
       <div>
         <h1>Strapi Posts</h1>
         <ul>
           {posts.map((post) => (
             <li key={post.id}>{post.title}</li>
           ))}
         </ul>
       </div>
     );
   };

   export default PostsPage;
   ```

## E-commerce Solutions (e.g., Shopify, Stripe)

Integrating e-commerce solutions allows you to manage products, handle payments, and create a seamless shopping experience for users.

### Integrating Shopify

1. **Create a Shopify Store**: Sign up at [Shopify](https://www.shopify.com/) and set up your store.

2. **Install Shopify Buy SDK**: Use the Shopify Buy SDK to interact with your store.

   ```bash
   npm install shopify-buy
   ```

3. **Setting Up the Shopify Client**:
   ```typescript
   // lib/shopify.ts
   import Client from 'shopify-buy';

   const client = Client.buildClient({
     domain: process.env.SHOPIFY_DOMAIN,
     storefrontAccessToken: process.env.SHOPIFY_ACCESS_TOKEN,
   });

   export const fetchProducts = async () => {
     const products = await client.product.fetchAll();
     return products;
   };
   ```

4. **Using Fetched Products in a Component**:
   ```typescript
   // pages/products.tsx
   import { fetchProducts } from '../lib/shopify';

   const ProductsPage = async () => {
     const products = await fetchProducts();

     return (
       <div>
         <h1>Shopify Products</h1>
         <ul>
           {products.map((product) => (
             <li key={product.id}>{product.title}</li>
           ))}
         </ul>
       </div>
     );
   };

   export default ProductsPage;
   ```

### Integrating Stripe for Payments

1. **Set Up a Stripe Account**: Sign up at [Stripe](https://stripe.com/) and create a new account.

2. **Install Stripe SDK**: Install the Stripe SDK for handling payments.

   ```bash
   npm install @stripe/stripe-js
   ```

3. **Creating a Checkout Session**:
   ```typescript
   // pages/api/create-checkout-session.ts
   import { NextApiRequest, NextApiResponse } from 'next';
   import Stripe from 'stripe';

   const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
     apiVersion: '2022-11-15',
   });

   export default async function handler(req: NextApiRequest, res: NextApiResponse) {
     const session = await stripe.checkout.sessions.create({
       payment_method_types: ['card'],
       line_items: [
         {
           price_data: {
             currency: 'usd',
             product_data: {
               name: 'T-shirt',
             },
             unit_amount: 2000,
           },
           quantity: 1,
         },
       ],
       mode: 'payment',
       success_url: `${req.headers.origin}/success`,
       cancel_url: `${req.headers.origin}/cancel`,
     });

     res.status(200).json({ id: session.id });
   }
   ```

4. **Redirecting to Checkout**:
   ```typescript
   // pages/checkout.tsx
   import { loadStripe } from '@stripe/stripe-js';

   const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY);

   const CheckoutPage = () => {
     const handleCheckout = async () => {
       const stripe = await stripePromise;
       const response = await fetch('/api/create-checkout-session', {
         method: 'POST',
       });
       const session = await response.json();
       await stripe.redirectToCheckout({ sessionId: session.id });
     };

     return <button onClick={handleCheckout}>Checkout</button>;
   };

   export default CheckoutPage;
   ```

## Authentication Providers (e.g., Auth0, Firebase)

Integrating authentication providers allows you to manage user authentication securely and efficiently.

### Integrating Auth0

1. **Set Up an Auth0 Account**: Sign up at [Auth0](https://auth0.com/) and create a new application.

2. **Install Auth0 SDK**:
   ```bash
   npm install @auth0/nextjs-auth0
   ```

3. **Configure Auth0**: Create a file named `auth0.ts` in the `lib` directory to configure Auth0.

   **Example: lib/auth0.ts**
   ```typescript
   import { initAuth0 } from '@auth0/nextjs-auth0';

   const auth0 = initAuth0({
     domain: process.env.AUTH0_DOMAIN,
     clientId: process.env.AUTH0_CLIENT_ID,
     clientSecret: process.env.AUTH0_CLIENT_SECRET,
     scope: 'openid profile',
     redirectUri: 'http://localhost:3000/api/auth/callback',
     postLogoutRedirectUri: 'http://localhost:3000/',
     session: {
       cookieSecret: process.env.COOKIE_SECRET,
       cookieLifetime: 60 * 60 * 8, // 8 hours
     },
   });

   export default auth0;
   ```

4. **Creating Authentication API Routes**: Create API routes for login, logout, and callback.

   **Example: pages/api/auth/[...auth0].ts**
   ```typescript
   import auth0 from '../../../lib/auth0';

   export default auth0.handleAuth();
   ```

5. **Using Auth0 in Your Application**:
   ```typescript
   // pages/profile.tsx
   import { useUser } from '@auth0/nextjs-auth0';

   const ProfilePage = () => {
     const { user, error, isLoading } = useUser();

     if (isLoading) return <div>Loading...</div>;
     if (error) return <div>{error.message}</div>;

     return (
       <div>
         <h1>Profile</h1>
         <img src={user.picture} alt={user.name} />
         <p>{user.name}</p>
         <p>{user.email}</p>
       </div>
     );
   };

   export default ProfilePage;
   ```

### Integrating Firebase

Firebase provides a comprehensive suite of tools for authentication, database management, and hosting.

1. **Set Up a Firebase Project**: Create a new project in the [Firebase Console](https://console.firebase.google.com/).

2. **Install Firebase SDK**:
   ```bash
   npm install firebase
   ```

3. **Configure Firebase**: Create a `firebase.ts` file in the `lib` directory.

   **Example: lib/firebase.ts**
   ```typescript
   import firebase from 'firebase/app';
   import 'firebase/auth';

   const firebaseConfig = {
     apiKey: process.env.FIREBASE_API_KEY,
     authDomain: process.env.FIREBASE_AUTH_DOMAIN,
     projectId: process.env.FIREBASE_PROJECT_ID,
     storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
     messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
     appId: process.env.FIREBASE_APP_ID,
   };

   if (!firebase.apps.length) {
     firebase.initializeApp(firebaseConfig);
   }

   export const auth = firebase.auth();
   ```

4. **Using Firebase Authentication**:
   ```typescript
   // pages/login.tsx
   import { auth } from '../lib/firebase';

   const LoginPage = () => {
     const handleLogin = async () => {
       const provider = new firebase.auth.GoogleAuthProvider();
       await auth.signInWithPopup(provider);
     };

     return <button onClick={handleLogin}>Login with Google</button>;
   };

   export default LoginPage;
   ```

## Serverless Databases (e.g., FaunaDB, Supabase)

Serverless databases provide scalable and flexible data storage solutions without the need for managing infrastructure. They are ideal for modern web applications.

### Integrating FaunaDB

1. **Set Up a FaunaDB Account**: Sign up at [FaunaDB](https://fauna.com/) and create a new database.

2. **Install FaunaDB Client**:
   ```bash
   npm install faunadb
   ```

3. **Configuring FaunaDB**: Create a `fauna.ts` file in the `lib` directory.

   **Example: lib/fauna.ts**
   ```typescript
   import faunadb from 'faunadb';

   const client = new faunadb.Client({
     secret: process.env.FAUNADB_SECRET,
   });

   export const query = faunadb.query;
   export default client;
   ```

4. **Using FaunaDB in Your Application**:
   ```typescript
   // pages/api/users.ts
   import client, { query } from '../../lib/fauna';

   export default async function handler(req, res) {
     const { name, email } = req.body;

     try {
       const result = await client.query(
         query.Create(query.Collection('users'), {
           data: { name, email },
         })
       );
       res.status(201).json(result);
     } catch (error) {
       res.status(500).json({ error: 'Error creating user' });
     }
   }
   ```

### Integrating Supabase

Supabase is an open-source Firebase alternative that provides a backend-as-a-service with a PostgreSQL database.

1. **Set Up a Supabase Project**: Create a new project at [Supabase](https://supabase.io/).

2. **Install Supabase Client**:
   ```bash
   npm install @supabase/supabase-js
   ```

3. **Configuring Supabase**: Create a `supabase.ts` file in the `lib` directory.

   **Example: lib/supabase.ts**
   ```typescript
   import { createClient } from '@supabase/supabase-js';

   const supabaseUrl = process.env.SUPABASE_URL!;
   const supabaseAnonKey = process.env.SUPABASE_ANON_KEY!;

   export const supabase = createClient(supabaseUrl, supabaseAnonKey);
   ```

4. **Using Supabase in Your Application**:
   ```typescript
   // pages/api/users.ts
   import { supabase } from '../../lib/supabase';

   export default async function handler(req, res) {
     const { name, email } = req.body;

     const { data, error } = await supabase
       .from('users')
       .insert([{ name, email }]);

     if (error) {
       return res.status(500).json({ error: error.message });
     }

     res.status(201).json(data);
   }
   ```

## Real-time Features with WebSockets and Socket.io

Real-time features enhance user experience by allowing instant communication between the client and server. Socket.io is a popular library for implementing WebSockets in JavaScript applications.

### Setting Up Socket.io

1. **Install Socket.io**:
   ```bash
   npm install socket.io socket.io-client
   ```

2. **Creating a Socket.io Server**:
   ```typescript
   // server.js
   import express from 'express';
   import { createServer } from 'http';
   import { Server } from 'socket.io';

   const app = express();
   const server = createServer(app);
   const io = new Server(server);

   io.on('connection', (socket) => {
     console.log('A user connected');
     socket.on('disconnect', () => {
       console.log('User disconnected');
     });
   });

   server.listen(3001, () => {
     console.log('Socket.io server running on port 3001');
   });
   ```

3. **Connecting to Socket.io from Next.js**:
   ```typescript
   // pages/index.tsx
   import { useEffect } from 'react';
   import io from 'socket.io-client';

   const socket = io('http://localhost:3001');

   const HomePage = () => {
     useEffect(() => {
       socket.on('connect', () => {
         console.log('Connected to Socket.io server');
       });

       return () => {
         socket.off('connect');
       };
     }, []);

     return <h1>Welcome to My Next.js App with Socket.io</h1>;
   };

   export default HomePage;
   ```

## Conclusion

In this lesson, you learned how to integrate various external services with your Next.js applications using TypeScript. We covered headless CMS integration (Contentful and Strapi), e-commerce solutions (Shopify and Stripe), authentication providers (Auth0 and Firebase), serverless databases (FaunaDB and Supabase), and implementing real-time features with WebSockets and Socket.io. By leveraging these integrations, you can enhance the functionality and user experience of your applications. In the next lesson, we will summarize the course and discuss next steps in your journey with Next.js and TypeScript.

[Next Lesson](./17_graphql_with_typescript.md)