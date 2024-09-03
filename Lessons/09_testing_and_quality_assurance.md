# Lesson 09: Testing and Quality Assurance

In this lesson, we will cover the essential aspects of testing and quality assurance in Next.js applications using TypeScript. We will explore unit testing with Jest and React Testing Library, integration testing of components and pages, end-to-end testing with Cypress, performance testing with Lighthouse CI, and accessibility testing. By the end of this lesson, you will have a solid foundation in testing methodologies and practices that will help ensure the reliability and quality of your applications.

## Unit Testing with Jest and React Testing Library

Unit testing involves testing individual components or functions in isolation to ensure they behave as expected. Jest is a popular testing framework for JavaScript applications, and React Testing Library provides utilities for testing React components.

### Setting Up Jest and React Testing Library

1. **Install Dependencies**: Start by installing Jest and React Testing Library in your Next.js project.

   ```bash
   npm install --save-dev jest @testing-library/react @testing-library/jest-dom
   ```

2. **Create Jest Configuration**: Create a `jest.config.js` file in the root of your project to configure Jest.

   ```javascript
   // jest.config.js
   const nextJest = require('next/jest');

   const createJestConfig = nextJest({
     dir: './',
   });

   const customJestConfig = {
     testEnvironment: 'jsdom',
     setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
   };

   module.exports = createJestConfig(customJestConfig);
   ```

3. **Setup Testing Environment**: Create a `jest.setup.js` file to configure the testing environment.

   ```javascript
   // jest.setup.js
   import '@testing-library/jest-dom';
   ```

4. **Add Test Script**: Update your `package.json` to include a test script.

   ```json
   "scripts": {
     "dev": "next dev",
     "build": "next build",
     "start": "next start",
     "test": "jest"
   }
   ```

### Writing Your First Unit Test

Now that your testing environment is set up, you can write your first unit test.

#### Example: Testing a Simple Component
```typescript
// components/Hello.tsx
import React from 'react';

const Hello: React.FC<{ name: string }> = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

export default Hello;
```

#### Example: Writing the Test
```typescript
// __tests__/Hello.test.tsx
import { render, screen } from '@testing-library/react';
import Hello from '../components/Hello';

describe('Hello Component', () => {
  it('renders the correct message', () => {
    render(<Hello name="John" />);
    const headingElement = screen.getByText(/hello, john!/i);
    expect(headingElement).toBeInTheDocument();
  });
});
```

### Running Your Tests

Run your tests using the following command:

```bash
npm test
```

This command will execute all tests in your project and display the results in the console.

## Integration Testing of Next.js Components and Pages

Integration testing involves testing how different components work together. In Next.js, this often means testing how pages interact with components and API routes.

### Example of Integration Testing

Suppose you have a component that fetches data from an API and displays it.

#### Example: Fetching Data Component
```typescript
// components/UserList.tsx
import React, { useEffect, useState } from 'react';

const UserList: React.FC = () => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    const fetchUsers = async () => {
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    };
    fetchUsers();
  }, []);

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};

export default UserList;
```

#### Example: Writing the Integration Test
```typescript
// __tests__/UserList.test.tsx
import { render, screen } from '@testing-library/react';
import UserList from '../components/UserList';

jest.mock('next/router', () => ({
  useRouter: () => ({
    query: {},
  }),
}));

describe('UserList Component', () => {
  it('fetches and displays users', async () => {
    // Mock the API response
    global.fetch = jest.fn(() =>
      Promise.resolve({
        json: () => Promise.resolve([{ id: 1, name: 'John Doe' }]),
      })
    ) as jest.Mock;

    render(<UserList />);
    
    // Wait for the users to be displayed
    const userElement = await screen.findByText(/john doe/i);
    expect(userElement).toBeInTheDocument();
  });
});
```

## End-to-End Testing with Cypress

End-to-end (E2E) testing involves testing the entire application flow, simulating user interactions to ensure everything works as expected. Cypress is a popular testing framework for E2E testing in web applications.

### Setting Up Cypress

1. **Install Cypress**: Install Cypress in your Next.js project.

   ```bash
   npm install --save-dev cypress
   ```

2. **Open Cypress**: You can open the Cypress test runner using the following command:

   ```bash
   npx cypress open
   ```

3. **Creating a Test**: Create a new test file in the `cypress/integration` directory.

#### Example: Writing an E2E Test
```javascript
// cypress/integration/userList.spec.js
describe('User List', () => {
  it('displays users from the API', () => {
    cy.intercept('GET', '/api/users', {
      statusCode: 200,
      body: [{ id: 1, name: 'John Doe' }],
    }).as('getUsers');

    cy.visit('/'); // Visit the home page
    cy.wait('@getUsers'); // Wait for the API call to finish
    cy.contains('John Doe').should('be.visible'); // Assert that the user is displayed
  });
});
```

### Running Cypress Tests

You can run your Cypress tests in the test runner or from the command line using:

```bash
npx cypress run
```

## Performance Testing and Lighthouse CI

Performance testing is essential for ensuring that your application loads quickly and provides a good user experience. Lighthouse is a tool that analyzes web applications and provides insights into performance, accessibility, and SEO.

### Using Lighthouse

1. **Running Lighthouse**: You can run Lighthouse directly from Chrome DevTools by opening the Performance tab and clicking on "Lighthouse". Alternatively, you can use the Lighthouse CI to automate performance testing.

2. **Setting Up Lighthouse CI**: Install Lighthouse CI in your project.

   ```bash
   npm install --save-dev lighthouse-ci
   ```

3. **Configuring Lighthouse CI**: Create a configuration file for Lighthouse CI.

#### Example: Creating a Lighthouse CI Configuration
```javascript
// lighthouse-ci.config.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000'],
      numberOfRuns: 5,
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['warn', { minScore: 0.8 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

### Running Lighthouse CI

You can run Lighthouse CI using the following command:

```bash
npx lhci autorun
```

This command will collect performance data and provide a report based on your configuration.

## Accessibility Testing and Compliance

Accessibility testing ensures that your application is usable by people with disabilities. This includes testing for keyboard navigation, screen reader compatibility, and adherence to accessibility standards such as WCAG.

### Using Jest and React Testing Library for Accessibility Testing

You can use `@testing-library/jest-dom` to assert accessibility features in your components.

#### Example: Accessibility Test
```typescript
// __tests__/Accessibility.test.tsx
import { render, screen } from '@testing-library/react';
import MyButton from '../components/MyButton';

test('button should have accessible name', () => {
  render(<MyButton label="Submit" />);
  const button = screen.getByRole('button', { name: /submit/i });
  expect(button).toBeInTheDocument();
});
```

### Using axe-core for Automated Accessibility Testing

You can integrate `axe-core` with your testing framework to perform automated accessibility checks.

1. **Install axe-core**:
   ```bash
   npm install --save-dev @axe-core/react
   ```

2. **Integrate axe-core in Your Tests**:
```typescript
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import MyComponent from '../components/MyComponent';

expect.extend(toHaveNoViolations);

test('should have no accessibility violations', async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## Conclusion

In this lesson, you learned about various testing methodologies and quality assurance practices for your Next.js applications. We covered unit testing with Jest and React Testing Library, integration testing of components and pages, end-to-end testing with Cypress, performance testing with Lighthouse CI, and accessibility testing. By implementing these testing strategies, you can ensure the reliability, performance, and accessibility of your applications. In the next lesson, we will explore advanced topics in Next.js, including server-side rendering and static site generation.

[Next Lesson](./10_deployment_and_devops.md)