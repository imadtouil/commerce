# Kibo Commerce API Client Configuration

Set up the server-side Kibo Commerce API configuration for a Next.js Commerce application that uses OAuth client credentials for authentication.

## Requirements

1. Create a module that exports a fully configured Kibo Commerce API provider object using the package's `getCommerceApi` (or equivalent configuration factory) with all required authentication parameters.
2. The configuration must supply the following values read from environment variables:
   - `commerceUrl` from `KIBO_API_URL`
   - `clientId` from `KIBO_CLIENT_ID`
   - `sharedSecret` from `KIBO_SHARED_SECRET`
   - `authUrl` from `KIBO_AUTH_URL`
   - `cartCookie` from `KIBO_CART_COOKIE` (with a default fallback value of `'kibo_cart'`)
   - `customerCookie` from `KIBO_CUSTOMER_COOKIE`
3. The configuration module must also wire up the package's GraphQL fetcher so that API calls use the OAuth client_credentials token from the authentication helper.
4. Export the configured API object as the default export.

## Notes

- The Kibo Commerce package provides a configuration-based API factory rather than requiring manual HTTP configuration.
- Authentication is handled internally by the package's `APIAuthenticationHelper` class using the `clientId` and `sharedSecret`.
- The `cartCookieMaxAge` can be left at the default (2592000 seconds = 30 days).

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides the API configuration factory, GraphQL fetcher utilities, and APIAuthenticationHelper for server-side OAuth token management.

## Test Cases

- [@test](./tests/exports-api-object.test.ts) The module exports a non-null API object as its default export.
- [@test](./tests/reads-env-vars.test.ts) The commerceUrl, clientId, sharedSecret, and authUrl values are sourced from the corresponding environment variables.
- [@test](./tests/cart-cookie-default.test.ts) When KIBO_CART_COOKIE is not set, the cartCookie defaults to 'kibo_cart'.
- [@test](./tests/fetcher-configured.test.ts) The exported API object has a fetch function that uses the package's GraphQL fetcher (not a plain fetch or axios).
