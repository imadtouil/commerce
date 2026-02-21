# Normalize a Commerce.js Product

Implement a utility function that converts a raw Commerce.js product object into the normalized product format used by the @vercel/commerce framework. The function should handle product images, price, slug/path construction, and variant option group mapping.

## Capabilities

### Product data normalization

- The normalized product's slug is derived from the product's permalink field [@test](./tests/normalize-slug.test.ts)
- The normalized product's path is constructed as `/<permalink>` [@test](./tests/normalize-path.test.ts)
- Each product image in the raw data is mapped to an object with url and alt fields [@test](./tests/normalize-images.test.ts)
- Variant groups from the raw product are mapped to options with displayName and values, and individual variants are normalized with their SKU and selected options [@test](./tests/normalize-variants.test.ts)

## Implementation

[@generates](./src/utils/normalize-product.ts)

## API

```typescript { #api }
interface NormalizedProduct {
  id: string;
  name: string;
  description: string;
  descriptionHtml: string;
  slug: string;
  path: string;
  images: { url: string; alt: string }[];
  price: { value: number; currencyCode: string };
  variants: { id: string; sku: string; options: { __typename: string; id: string; displayName: string; values: { label: string }[] }[] }[];
  options: { id: string; displayName: string; values: { label: string }[] }[];
}
export function normalizeProduct(
  commercejsProduct: Record<string, any>,
  variants?: Record<string, any>[]
): NormalizedProduct;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides a product normalization utility that transforms raw Commerce.js product objects (including variant groups) into the standard @vercel/commerce product format.

[@satisfied-by](@vercel/commerce-commercejs)
