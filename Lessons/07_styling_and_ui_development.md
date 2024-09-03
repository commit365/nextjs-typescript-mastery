# Lesson 07: Styling and UI Development

In this lesson, we will explore various methods for styling your Next.js applications, focusing on CSS Modules, Sass integration, CSS-in-JS approaches, and utility-first frameworks like Tailwind CSS. We will also discuss creating a consistent design system and implementing responsive and adaptive design techniques. By the end of this lesson, you will have a comprehensive understanding of how to style your Next.js applications effectively.

## CSS Modules and Sass Integration

### CSS Modules

CSS Modules provide a way to locally scope CSS styles to specific components, preventing class name collisions and improving maintainability. In Next.js, you can use CSS Modules by creating files with the `.module.css` extension.

#### Setting Up CSS Modules

1. **Creating a CSS Module**: Create a CSS file with the `.module.css` extension in your component directory.

   **Example**: Create `Button.module.css`:
   ```css
   .primary {
     background-color: blue;
     color: white;
     padding: 10px 20px;
     border: none;
     border-radius: 5px;
     cursor: pointer;
   }
   ```

2. **Importing and Using the CSS Module**: Import the CSS module in your component and use the styles.

   **Example**: Create `Button.tsx`:
   ```typescript
   import styles from './Button.module.css';

   const Button = ({ label }: { label: string }) => {
     return <button className={styles.primary}>{label}</button>;
   };

   export default Button;
   ```

### Sass Integration

Sass is a popular CSS preprocessor that extends CSS with features like variables, nested rules, and mixins. Next.js has built-in support for Sass, allowing you to use `.scss` files in your projects.

#### Setting Up Sass

1. **Installing Sass**: Install the Sass package in your Next.js project.

   ```bash
   npm install sass
   ```

2. **Creating a Sass File**: Create a Sass file with the `.scss` extension.

   **Example**: Create `styles.scss`:
   ```scss
   $primary-color: blue;

   .button {
     background-color: $primary-color;
     color: white;
     padding: 10px 20px;
     border: none;
     border-radius: 5px;
     cursor: pointer;

     &:hover {
       background-color: darken($primary-color, 10%);
     }
   }
   ```

3. **Importing the Sass File**: Import the Sass file in your component or in `_app.tsx` for global styles.

   **Example**: Import in `_app.tsx`:
   ```typescript
   import '../styles/styles.scss';

   const MyApp = ({ Component, pageProps }) => {
     return <Component {...pageProps} />;
   };

   export default MyApp;
   ```

## Styled-Components and CSS-in-JS Approaches

Styled-components is a popular library for styling React components using tagged template literals. It allows you to write CSS directly within your JavaScript, enabling dynamic styling based on props and state.

### Setting Up Styled-Components

1. **Installing Styled-Components**: Install the styled-components package.

   ```bash
   npm install styled-components
   npm install --save-dev @types/styled-components
   ```

2. **Creating a Styled Component**: Use the `styled` function to create a styled component.

   **Example**: Create `StyledButton.tsx`:
   ```typescript
   import styled from 'styled-components';

   const StyledButton = styled.button`
     background-color: blue;
     color: white;
     padding: 10px 20px;
     border: none;
     border-radius: 5px;
     cursor: pointer;

     &:hover {
       background-color: darkblue;
     }
   `;

   const Button = ({ label }: { label: string }) => {
     return <StyledButton>{label}</StyledButton>;
   };

   export default Button;
   ```

### Advantages of CSS-in-JS

- **Dynamic Styling**: Easily apply styles based on component props or state.
- **Scoped Styles**: Styles are scoped to the component, preventing global namespace collisions.
- **Theming**: Simplifies the implementation of themes across your application.

## Tailwind CSS Configuration and Best Practices

Tailwind CSS is a utility-first CSS framework that allows you to build custom designs quickly by composing utility classes. It promotes a different approach to styling compared to traditional CSS.

### Setting Up Tailwind CSS

1. **Installing Tailwind CSS**: Install Tailwind CSS and its dependencies.

   ```bash
   npm install -D tailwindcss postcss autoprefixer
   npx tailwindcss init -p
   ```

2. **Configuring Tailwind**: Update the `tailwind.config.js` file to specify the paths to your template files.

   **Example**: Update `tailwind.config.js`:
   ```javascript
   module.exports = {
     content: [
       './pages/**/*.{js,ts,jsx,tsx}',
       './components/**/*.{js,ts,jsx,tsx}',
     ],
     theme: {
       extend: {},
     },
     plugins: [],
   };
   ```

3. **Adding Tailwind to Your CSS**: Include the Tailwind directives in your global CSS file (e.g., `globals.css`).

   **Example**: Update `globals.css`:
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```

### Best Practices for Using Tailwind CSS

- **Utility-First Approach**: Use utility classes to build your components directly in your JSX, promoting rapid development.
- **Extracting Components**: If a component has many utility classes, consider extracting it into a separate component for better readability.
- **Responsive Design**: Use Tailwind's responsive utilities to create mobile-first designs easily.

#### Example of a Tailwind CSS Button Component
```typescript
// components/TailwindButton.tsx
const TailwindButton = ({ label }: { label: string }) => {
  return (
    <button className="bg-blue-500 text-white py-2 px-4 rounded hover:bg-blue-700">
      {label}
    </button>
  );
};

export default TailwindButton;
```

## Creating a Consistent Design System

A consistent design system helps maintain a cohesive look and feel across your application. It includes design tokens (colors, typography, spacing), reusable components, and guidelines for usage.

### Defining Design Tokens

Design tokens are the visual design elements that can be reused throughout your application. They can be defined in CSS variables, Sass variables, or directly in your component styles.

#### Example of Defining Design Tokens
```css
/* styles/tokens.css */
:root {
  --color-primary: #1d4ed8; /* Blue */
  --color-secondary: #9333ea; /* Purple */
  --font-family: 'Arial, sans-serif';
}
```

### Creating Reusable Components

Build reusable components that adhere to your design system. Ensure that they accept props for customization while maintaining consistent styling.

#### Example of a Reusable Card Component
```typescript
// components/Card.tsx
import React from 'react';

interface CardProps {
  title: string;
  content: string;
}

const Card: React.FC<CardProps> = ({ title, content }) => {
  return (
    <div className="border rounded-lg p-4 shadow-md">
      <h2 className="font-bold text-lg">{title}</h2>
      <p>{content}</p>
    </div>
  );
};

export default Card;
```

## Responsive and Adaptive Design Techniques

Responsive design ensures that your application looks good on various screen sizes, while adaptive design adjusts the layout based on the specific device being used.

### Responsive Design with CSS

Use media queries to apply different styles based on the viewport size.

#### Example of Responsive Styling
```css
/* styles/global.css */
@media (max-width: 768px) {
  .container {
    padding: 10px;
  }
}

@media (min-width: 769px) {
  .container {
    padding: 20px;
  }
}
```

### Responsive Design with Tailwind CSS

Tailwind CSS provides responsive utilities that allow you to apply styles at specific breakpoints easily.

#### Example of Responsive Utilities
```typescript
// components/ResponsiveCard.tsx
const ResponsiveCard = () => {
  return (
    <div className="bg-white p-4 md:p-6 lg:p-8">
      <h2 className="text-lg md:text-xl">Responsive Card</h2>
      <p>This card adjusts its padding based on the screen size.</p>
    </div>
  );
};

export default ResponsiveCard;
```

### Adaptive Design Techniques

Adaptive design involves creating distinct layouts for different devices. You can achieve this by using media queries or Tailwind's responsive utilities to conditionally render different components based on screen size.

#### Example of Adaptive Design
```typescript
// components/AdaptiveComponent.tsx
const AdaptiveComponent = () => {
  return (
    <>
      <div className="block md:hidden">Mobile View</div>
      <div className="hidden md:block">Desktop View</div>
    </>
  );
};

export default AdaptiveComponent;
```

## Conclusion

In this lesson, you learned various methods for styling your Next.js applications, including CSS Modules, Sass integration, and CSS-in-JS with styled-components. You also explored utility-first styling with Tailwind CSS, creating a consistent design system, and implementing responsive and adaptive design techniques. With these skills, you can create visually appealing and user-friendly applications. In the next lesson, we will cover data fetching and state management techniques to enhance the functionality of your applications.

[Next Lesson](./07_styling_and_ui_development.md)