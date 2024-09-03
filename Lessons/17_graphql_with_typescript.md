# Lesson 17: GraphQL with TypeScript

In this lesson, we will explore how to effectively integrate GraphQL into your Next.js applications using TypeScript. GraphQL is a powerful query language for APIs that allows clients to request exactly the data they need, improving efficiency and flexibility in data handling. We will cover the fundamentals of GraphQL, setting up a GraphQL server with Apollo, creating type-safe GraphQL queries and mutations, integrating GraphQL with Next.js pages and API routes, and exploring advanced GraphQL patterns and best practices. By the end of this lesson, you will have a comprehensive understanding of how to leverage GraphQL in your Next.js applications.

## Introduction to GraphQL and Its Benefits

GraphQL is a query language for APIs and a runtime for executing those queries by using a type system you define for your data. It provides a more efficient and powerful alternative to traditional REST APIs.

### Key Benefits of GraphQL

1. **Efficient Data Retrieval**: Clients can request exactly the data they need, avoiding over-fetching or under-fetching of data. This results in optimized data retrieval and reduced payload sizes.

2. **Single Request for Multiple Resources**: GraphQL allows clients to consolidate multiple queries into a single request, reducing latency and minimizing the number of network requests.

3. **Strong Typing**: GraphQL provides a flexible and self-documenting schema, ensuring a clear contract between the client and the server. This strong typing helps catch errors early in the development process.

4. **Real-time Data with Subscriptions**: GraphQL supports real-time data updates through subscriptions, making it ideal for applications that require live data, such as chat applications or collaborative tools.

5. **Evolving APIs without Versioning**: GraphQL enables seamless evolution of APIs by allowing the addition of new fields or types without breaking existing clients, eliminating the need for versioning.

### When to Use GraphQL

- When you need flexible data requirements and want to avoid over-fetching or under-fetching.
- When your application requires real-time updates.
- When you anticipate frequent changes to your API without breaking existing clients.

## Setting Up a GraphQL Server with Apollo

Apollo Server is a community-driven, open-source GraphQL server that works with any GraphQL schema. It is easy to set up and integrates well with Next.js.

### Installing Apollo Server

1. **Install Apollo Server**: Begin by installing the necessary packages.

   ```bash
   npm install apollo-server-micro graphql
   ```

2. **Create a GraphQL Server**: Create a file named `graphql.ts` in the `pages/api` directory to define your GraphQL server.

   **Example: pages/api/graphql.ts**
   ```typescript
   import { ApolloServer, gql } from 'apollo-server-micro';

   // Define your GraphQL schema
   const typeDefs = gql`
     type User {
       id: ID!
       name: String!
       email: String!
     }

     type Query {
       users: [User!]!
     }
   `;

   // Define your resolvers
   const resolvers = {
     Query: {
       users: () => [
         { id: '1', name: 'John Doe', email: 'john@example.com' },
         { id: '2', name: 'Jane Doe', email: 'jane@example.com' },
       ],
     },
   };

   const apolloServer = new ApolloServer({ typeDefs, resolvers });

   export const config = {
     api: {
       bodyParser: false,
     },
   };

   export default apolloServer.createHandler({ path: '/api/graphql' });
   ```

### Testing Your GraphQL Server

You can test your GraphQL server using tools like GraphiQL or Postman.

1. **Run Your Next.js Application**:
   ```bash
   npm run dev
   ```

2. **Access the GraphQL Endpoint**: Open your browser and navigate to `http://localhost:3000/api/graphql`. You should see the GraphiQL interface where you can run queries.

3. **Example Query**:
   ```graphql
   query {
     users {
       id
       name
       email
     }
   }
   ```

## Creating Type-Safe GraphQL Queries and Mutations

Using TypeScript with GraphQL allows you to define types for your queries and mutations, ensuring type safety throughout your application.

### Generating Type Definitions

You can use tools like `graphql-codegen` to automatically generate TypeScript types from your GraphQL schema.

1. **Install GraphQL Code Generator**:
   ```bash
   npm install --save-dev @graphql-codegen/cli @graphql-codegen/typescript
   ```

2. **Create a Codegen Configuration**: Create a file named `codegen.yml` in the root of your project.

   **Example: codegen.yml**
   ```yaml
   schema: "http://localhost:3000/api/graphql"
   documents: "src/**/*.graphql"
   generates:
     src/generated/graphql.ts:
       plugins:
         - "typescript"
   ```

3. **Run the Code Generator**:
   ```bash
   npx graphql-codegen
   ```

### Using Generated Types

You can now use the generated types in your components and API routes.

#### Example: Using Generated Types in a Component
```typescript
// components/UserList.tsx
import { useQuery } from '@apollo/client';
import { gql } from 'graphql-tag';
import { User } from '../generated/graphql';

const GET_USERS = gql`
  query {
    users {
      id
      name
      email
    }
  }
`;

const UserList: React.FC = () => {
  const { loading, error, data } = useQuery<{ users: User[] }>(GET_USERS);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data?.users.map((user) => (
        <li key={user.id}>{user.name} - {user.email}</li>
      ))}
    </ul>
  );
};

export default UserList;
```

## Integrating GraphQL with Next.js Pages and API Routes

You can seamlessly integrate GraphQL into your Next.js pages and API routes to fetch data and manage application state.

### Fetching Data in Pages

You can use Apollo Client to fetch data in your Next.js pages, either during server-side rendering or client-side fetching.

#### Example: Fetching Data in a Page Component
```typescript
// pages/users.tsx
import { ApolloClient, InMemoryCache, ApolloProvider, useQuery } from '@apollo/client';
import { gql } from 'graphql-tag';
import UserList from '../components/UserList';

const client = new ApolloClient({
  uri: 'http://localhost:3000/api/graphql',
  cache: new InMemoryCache(),
});

const GET_USERS = gql`
  query {
    users {
      id
      name
      email
    }
  }
`;

const UsersPage: React.FC = () => {
  const { loading, error, data } = useQuery(GET_USERS);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ApolloProvider client={client}>
      <UserList />
    </ApolloProvider>
  );
};

export default UsersPage;
```

### Using GraphQL in API Routes

You can also use GraphQL in your API routes to perform server-side operations.

#### Example: Using GraphQL in an API Route
```typescript
// pages/api/addUser.ts
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';
import client from '../../lib/apolloClient';

export default async function handler(req, res) {
  if (req.method === 'POST') {
    const { name, email } = req.body;

    const ADD_USER = gql`
      mutation AddUser($name: String!, $email: String!) {
        addUser(name: $name, email: $email) {
          id
          name
          email
        }
      }
    `;

    try {
      const { data } = await client.mutate({
        mutation: ADD_USER,
        variables: { name, email },
      });
      res.status(201).json(data.addUser);
    } catch (error) {
      res.status(500).json({ error: 'Failed to add user' });
    }
  } else {
    res.setHeader('Allow', ['POST']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}
```

## Advanced GraphQL Patterns and Best Practices

### 1. Pagination

Implement pagination in your GraphQL queries to handle large datasets efficiently. Use arguments to limit the number of results returned.

#### Example: Implementing Pagination
```typescript
// GraphQL Schema
type Query {
  users(limit: Int, offset: Int): [User!]!
}

// Resolver
users: async (_, { limit, offset }) => {
  return await User.find().limit(limit).skip(offset);
}
```

### 2. Caching with Apollo Client

Utilize Apollo Client's caching capabilities to improve performance and reduce unnecessary network requests.

#### Example: Configuring Cache Policies
```typescript
const client = new ApolloClient({
  uri: 'http://localhost:3000/api/graphql',
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          users: {
            merge(existing = [], incoming) {
              return [...existing, ...incoming];
            },
          },
        },
      },
    },
  }),
});
```

### 3. Error Handling

Implement robust error handling in your GraphQL resolvers and client-side queries to provide meaningful feedback to users.

#### Example: Handling Errors in Resolvers
```typescript
const resolvers = {
  Query: {
    users: async () => {
      try {
        return await User.find();
      } catch (error) {
        throw new Error('Failed to fetch users');
      }
    },
  },
};
```

### 4. Subscriptions for Real-Time Data

Use GraphQL subscriptions to enable real-time updates in your application, allowing clients to receive updates as changes occur on the server.

#### Example: Setting Up Subscriptions
```typescript
const subscription = gql`
  subscription {
    userAdded {
      id
      name
      email
    }
  }
`;

const { data, loading } = useSubscription(subscription);
```

## Conclusion

In this lesson, you learned how to integrate GraphQL into your Next.js applications using TypeScript. We covered the benefits of GraphQL, setting up a GraphQL server with Apollo, creating type-safe queries and mutations, integrating GraphQL with Next.js pages and API routes, and exploring advanced patterns and best practices. By leveraging GraphQL, you can build efficient and flexible applications that provide a seamless experience for users. In the next lesson, we will summarize the course and discuss next steps in your journey with Next.js and TypeScript.

[Next Lesson](./18_advanced_patterns_and_best_practices.md)