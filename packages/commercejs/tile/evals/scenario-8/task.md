# Submit a Checkout Order

Implement a Next.js API route handler at `POST /api/commerce/checkout` that completes a purchase. The handler should generate a checkout token from the current cart, retrieve available shipping options, and capture the order with the provided payment card and shipping address data.

## Capabilities

### Server-side order capture

- The handler generates a checkout token from the cart by providing the cart ID [@test](./tests/checkout-token-generation.test.ts)
- The handler fetches available shipping options for the US country code using the generated token [@test](./tests/checkout-shipping-options.test.ts)
- The handler captures the order using the checkout token, the first available shipping option, and the provided card/address data [@test](./tests/checkout-capture.test.ts)
- Payment card data is normalized for the Commerce.js API, including card number, expiry month, expiry year, and CVC [@test](./tests/checkout-card-normalization.test.ts)

## Implementation

[@generates](./src/pages/api/commerce/checkout.ts)

## API

```typescript { #api }
import type { NextApiRequest, NextApiResponse } from 'next';
export default async function checkoutHandler(
  req: NextApiRequest,
  res: NextApiResponse
): Promise<void>;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides checkout API operations: token generation from cart, shipping options retrieval by country code, and order capture with normalized payment and shipping data.

[@satisfied-by](@vercel/commerce-commercejs)
