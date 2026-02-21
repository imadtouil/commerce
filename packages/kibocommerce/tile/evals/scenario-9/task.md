# Product Display from Kibo Commerce Data

Implement a Next.js server-side data fetching function that retrieves raw product data from the Kibo Commerce API and normalizes it to the standard commerce product type for use in a product detail page.

## Requirements

1. Use the package's `normalizeProduct` utility function to convert a raw Kibo Commerce product object into the standard commerce `Product` type.
2. The function receives a raw Kibo product object (as returned by the Kibo GraphQL API) and returns a normalized `Product` object.
3. The normalized product must include:
   - `id` mapped from the Kibo product's `productCode`
   - `name` mapped from the product's `content.productName`
   - `slug` derived from the product's `productCode` (lowercased)
   - `price` object with `value` and `currencyCode`
   - `images` array with at least the main image
4. Also implement a `normalizeCart` call: given a raw Kibo cart object, return a normalized cart using the package's `normalizeCart` function.
5. Export both functions.

## Notes

- The `normalizeProduct` and `normalizeCart` functions are exported from the package's `lib/normalize` module (or the package's main entry point on the server side).
- These functions accept the raw API shapes and produce the standard @vercel/commerce types.
- You do not need to implement the normalization logic yourself—use the package's utilities.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides normalizeProduct, normalizeCart, normalizeCustomer, normalizeCategory, and normalizeLineItem for converting Kibo API responses to standard commerce types.

## Test Cases

- [@test](./tests/normalize-product-id.test.ts) normalizeProduct maps the productCode to the id field of the result.
- [@test](./tests/normalize-product-name.test.ts) normalizeProduct maps content.productName to the name field of the result.
- [@test](./tests/normalize-cart-lineitems.test.ts) normalizeCart converts raw Kibo cart lineItems to the standard LineItem shape.
- [@test](./tests/normalize-returns-correct-types.test.ts) Both functions return objects conforming to the standard @vercel/commerce types, not raw Kibo types.
