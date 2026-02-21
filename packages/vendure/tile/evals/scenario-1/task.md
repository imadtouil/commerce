# Vendure GraphQL Fetcher Behavior

## Overview

Implement a test harness that exercises the Vendure commerce package's built-in GraphQL fetcher to verify its behavior under various conditions.

## Capabilities

### Environment-based API URL resolution

Write a function `getVendureFetcherUrl()` that demonstrates how the Vendure fetcher resolves the Shop API URL. The fetcher checks `NEXT_PUBLIC_VENDURE_LOCAL_URL` first, falling back to `NEXT_PUBLIC_VENDURE_SHOP_API_URL`. If neither is set, any call to the fetcher should reject.

- When `NEXT_PUBLIC_VENDURE_LOCAL_URL` is set, the fetcher targets that URL
- When only `NEXT_PUBLIC_VENDURE_SHOP_API_URL` is set, the fetcher uses that URL
- When neither is set, calling the fetcher rejects with an error about the missing environment variable

[@test](./tests/fetcher-url.test.ts)

### GraphQL request format

Implement a `callVendureFetcher(query, variables)` wrapper that uses the package's fetcher to execute a GraphQL request. The fetcher should:

- Send a POST request with `Content-Type: application/json`
- Include `credentials: 'include'` for session cookie support
- JSON-encode `{ query, variables }` as the request body when both are provided
- Throw a structured error (not a generic Error) on non-2xx responses or when GraphQL errors are present in the response

[@test](./tests/fetcher-request.test.ts)
[@test](./tests/fetcher-error.test.ts)

## Implementation

[@generates](./src/fetcher-harness.ts)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Exports a `fetcher` function pre-configured to communicate with the Vendure Shop API via GraphQL.

[@satisfied-by](@vercel/commerce-vendure)
