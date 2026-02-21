# Vendure Commerce Context Setup

## Overview

Implement a Next.js page component that sets up the Vendure commerce context so that child components can access commerce functionality.

## Capabilities

### Wrap application with commerce context provider

Using the Vendure commerce package, create a root layout component that wraps its children with the appropriate commerce context provider. The provider should be initialized with the pre-built Vendure configuration object exported from the package.

- The component accepts a `children` prop of type `React.ReactNode`
- The provider must use the Vendure-specific provider configuration (not a generic one)
- Child components should be able to call `useCommerce()` and receive a non-null commerce context

### Access commerce context in child components

Implement a `useVendureCommerce` hook that wraps the package's `useCommerce` export so consumers can retrieve the current commerce context.

- Returns the result of calling the package's commerce context hook
- Must be callable within a component tree wrapped by the provider from the previous capability

[@test](./tests/provider.test.tsx)
[@test](./tests/use-commerce.test.tsx)

## Implementation

[@generates](./src/index.tsx)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Exports a pre-configured provider object and a `CommerceProvider` component for wrapping applications.

[@satisfied-by](@vercel/commerce-vendure)
