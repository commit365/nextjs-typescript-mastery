# Lesson 05: API Routes and Backend Integration

In this lesson, we will explore how to create and manage API routes in Next.js, allowing you to build backend functionality directly within your application. We will cover the creation of RESTful API routes, the use of middleware, connecting to databases, implementing authentication and authorization, and handling errors and validation. By the end of this lesson, you will have a solid understanding of how to integrate backend functionality into your Next.js applications.

## Creating RESTful API Routes

Next.js allows you to create API routes that can be accessed via HTTP requests. These routes are stored in the `pages/api` directory and can respond to various HTTP methods such as GET, POST, PUT, and DELETE.

### Creating a Simple API Route

To create a basic API route, simply create a JavaScript (or TypeScript) file in the `pages/api` directory. For example, to create a route for fetching a list of users, create a file named `users.js`.

#### Example of a Simple API Route
```javascript
// pages/api/users.js
export default function handler(req, res) {
  const users = [
    { id: 1, name: 'John Doe' },
    { id: 2, name: 'Jane Smith' },
  ];

  res.status(200).json(users);
}
```

In this example, when a client sends a GET request to `/api/users`, the server responds with a JSON array of users.

### Handling Different HTTP Methods

You can handle different HTTP methods within a single API route by checking the `req.method` property.

#### Example of Handling Multiple Methods
```javascript
// pages/api/users.js
export default function handler(req, res) {
  if (req.method === 'GET') {
    const users = [
      { id: 1, name: 'John Doe' },
      { id: 2, name: 'Jane Smith' },
    ];
    res.status(200).json(users);
  } else if (req.method === 'POST') {
    const newUser = req.body; // Assume user data is sent in the request body
    // Logic to save the new user goes here
    res.status(201).json(newUser);
  } else {
    res.setHeader('Allow', ['GET', 'POST']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}
```

In this example, the API route handles both GET and POST requests, responding appropriately based on the method used.

## Middleware and API Route Handlers

Middleware in Next.js API routes allows you to run code before your request is processed. This is useful for tasks such as authentication, logging, and validation.

### Creating Middleware

Middleware can be created as a separate function that takes `req`, `res`, and `next` as arguments. You can then use this middleware in your API routes.

#### Example of Middleware
```javascript
// middleware/auth.js
export function auth(req, res, next) {
  const token = req.headers.authorization;

  if (!token || token !== 'your-secret-token') {
    return res.status(401).json({ message: 'Unauthorized' });
  }

  next(); // Proceed to the next middleware or route handler
}
```

### Using Middleware in API Routes

You can apply middleware to your API routes by calling it within the route handler.

#### Example of Using Middleware
```javascript
// pages/api/users.js
import { auth } from '../../middleware/auth';

export default function handler(req, res) {
  auth(req, res, () => {
    // Your route logic here
    if (req.method === 'GET') {
      const users = [
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' },
      ];
      res.status(200).json(users);
    }
  });
}
```

In this example, the `auth` middleware checks for a valid token before processing the request.

## Connecting to Databases: SQL and NoSQL Options

Next.js API routes can connect to various databases, allowing you to store and retrieve data for your application. You can use both SQL and NoSQL databases based on your requirements.

### Connecting to a SQL Database

To connect to a SQL database (e.g., PostgreSQL, MySQL), you can use libraries like `pg` for PostgreSQL or `mysql2` for MySQL.

#### Example of Connecting to PostgreSQL
```bash
npm install pg
```

```javascript
// pages/api/users.js
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

export default async function handler(req, res) {
  if (req.method === 'GET') {
    const result = await pool.query('SELECT * FROM users');
    res.status(200).json(result.rows);
  }
}
```

### Connecting to a NoSQL Database

For NoSQL databases like MongoDB, you can use the `mongodb` package.

#### Example of Connecting to MongoDB
```bash
npm install mongodb
```

```javascript
// pages/api/users.js
import { MongoClient } from 'mongodb';

const uri = process.env.MONGODB_URI;
let client;
let db;

async function connectToDatabase() {
  if (!client) {
    client = new MongoClient(uri);
    await client.connect();
    db = client.db('your-database-name');
  }
  return { db, client };
}

export default async function handler(req, res) {
  const { db } = await connectToDatabase();

  if (req.method === 'GET') {
    const users = await db.collection('users').find({}).toArray();
    res.status(200).json(users);
  }
}
```

## Authentication and Authorization in API Routes

Implementing authentication and authorization in your API routes is crucial for securing your application. You can use libraries like `jsonwebtoken` for token-based authentication.

### Example of Token-Based Authentication

1. **Install the necessary package**:
   ```bash
   npm install jsonwebtoken
   ```

2. **Create an API route for login**:
```javascript
// pages/api/login.js
import jwt from 'jsonwebtoken';

export default function handler(req, res) {
  const { username, password } = req.body;

  // Validate user credentials (this is just an example)
  if (username === 'admin' && password === 'password') {
    const token = jwt.sign({ username }, process.env.JWT_SECRET, {
      expiresIn: '1h',
    });
    return res.status(200).json({ token });
  }

  res.status(401).json({ message: 'Invalid credentials' });
}
```

3. **Protecting API routes**:
```javascript
// middleware/auth.js
import jwt from 'jsonwebtoken';

export function auth(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ message: 'Unauthorized' });
  }

  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ message: 'Forbidden' });
    }
    req.user = user;
    next();
  });
}
```

## Error Handling and Validation in API Routes

Proper error handling and validation are essential for building robust APIs. You can use libraries like `joi` or `express-validator` for validation.

### Example of Error Handling

You can handle errors in your API routes by using try-catch blocks and sending appropriate HTTP status codes.

```javascript
// pages/api/users.js
export default async function handler(req, res) {
  try {
    if (req.method === 'GET') {
      // Fetch users from the database
      const users = await fetchUsersFromDatabase();
      res.status(200).json(users);
    } else {
      res.setHeader('Allow', ['GET']);
      res.status(405).end(`Method ${req.method} Not Allowed`);
    }
  } catch (error) {
    console.error(error);
    res.status(500).json({ message: 'Internal Server Error' });
  }
}
```

### Example of Validation

You can validate incoming request data before processing it.

```javascript
import Joi from 'joi';

// Define a schema for user data
const userSchema = Joi.object({
  name: Joi.string().min(3).required(),
  email: Joi.string().email().required(),
});

// Validate incoming data
export default async function handler(req, res) {
  if (req.method === 'POST') {
    const { error } = userSchema.validate(req.body);
    if (error) {
      return res.status(400).json({ message: error.details[0].message });
    }

    // Proceed with creating the user
    const newUser = await createUser(req.body);
    res.status(201).json(newUser);
  }
}
```

## Conclusion

In this lesson, you learned how to create API routes in Next.js, handle different HTTP methods, and implement middleware for authentication and authorization. You also explored how to connect to SQL and NoSQL databases, and how to handle errors and validate incoming data. With these skills, you can effectively integrate backend functionality into your Next.js applications. In the next lesson, we will cover styling and UI development techniques to enhance the user experience.

[Next Lesson](./06_advanced_typescript_techniques_in_nextjs.md)