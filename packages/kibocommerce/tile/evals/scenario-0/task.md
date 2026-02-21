# Commerce Provider Initialization

You are building a Next.js e-commerce application using the Kibo Commerce provider package. Your task is to correctly initialize and configure the commerce provider so that the rest of the application can access the commerce context.

## Requirements

1. Create a React component that wraps its children with the commerce provider, passing the Kibo Commerce provider configuration object.
2. The provider must be initialized using the Kibo-specific provider configuration exported by the package.
3. The wrapping component must accept and render a `children` prop.
4. Export a hook from the same file that returns the current commerce context using the hook provided by the package.

## Notes

- The Kibo Commerce package exports a specific provider object pre-configured for Kibo.
- The generic `CommerceProvider` component must receive this object via its `provider` prop.
- No manual configuration of individual endpoints is needed—use the pre-configured provider object.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for the Next.js Commerce framework. Provides the provider configuration object, the CommerceProvider component, and the useCommerce hook.

## Test Cases

- [@test](./tests/provider-wraps-children.test.tsx) When the component wraps a child element, the child is rendered.
- [@test](./tests/use-commerce-returns-context.test.tsx) Calling the exported hook inside the provider returns a non-null commerce context object.
- [@test](./tests/provider-uses-kibo-config.test.tsx) The provider is initialized with the Kibo-specific provider configuration (not a generic or empty config).
- [@test](./tests/missing-provider-throws.test.tsx) Calling the exported hook outside of the provider throws or returns an error state.
