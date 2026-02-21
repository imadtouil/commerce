# Checkout

Custom checkout flow using Commerce.js checkout token system. The checkout hooks integrate with a checkout context component (`@components/checkout/context`) that manages card and address form fields.

## Import

```typescript
import { useCheckout, useSubmitCheckout } from '@vercel/commerce-commercejs/checkout'
```

## Checkout Flow

1. User fills in card fields and address fields (via checkout form UI using `@components/checkout/context`)
2. `useCheckout()` reflects validation state (`hasPayment`, `hasShipping`) and exposes `submit`
3. `submit()` (or `useSubmitCheckout()`) POSTs to `/api/commerce/checkout` with the card and address data
4. The server endpoint generates a checkout token from the cart, fetches shipping methods, and captures the order

## Types

```typescript { .api }
interface CheckoutState {
  data: {
    hasPayment: boolean   // true if any card field has a value
    hasShipping: boolean  // true if any address field has a value
  }
  submit: () => Promise<any>
}

interface CardFields {
  firstName?: string
  lastName?: string
  zipCode?: string
  streetNumber?: string
  city?: string
  // (any card form field values)
}

interface AddressFields {
  firstName?: string
  lastName?: string
  zipCode?: string
  streetNumber?: string
  city?: string
  // (any address form field values)
}
```

## Capabilities

### useCheckout

Returns the current checkout state indicating whether payment and shipping information has been entered. Also exposes the `submit` function to complete the checkout.

```typescript { .api }
/**
 * Reads checkout context to determine payment/shipping readiness.
 * Requires checkout context provider (@components/checkout/context) to be mounted.
 * @returns Checkout state with hasPayment, hasShipping flags and submit function
 */
function useCheckout(): {
  data: {
    hasPayment: boolean   // true if any card field has a non-empty value
    hasShipping: boolean  // true if any address field has a non-empty value
  }
  submit: () => Promise<any>
}
```

**Note:** Requires `useCheckoutContext()` from `@components/checkout/context` to be available in the component tree.

**Usage:**

```typescript
import { useCheckout } from '@vercel/commerce-commercejs/checkout'

function CheckoutSummary() {
  const { data, submit } = useCheckout()

  const canSubmit = data?.hasPayment && data?.hasShipping

  return (
    <div>
      <p>Payment info: {data?.hasPayment ? '✓' : '✗'}</p>
      <p>Shipping info: {data?.hasShipping ? '✓' : '✗'}</p>
      <button disabled={!canSubmit} onClick={submit}>
        Place Order
      </button>
    </div>
  )
}
```

### useSubmitCheckout

Returns a function to submit the checkout. Reads card and address fields from checkout context and POSTs them to the checkout API endpoint.

```typescript { .api }
/**
 * @returns Async function to submit the checkout
 */
function useSubmitCheckout(): (input?: any) => Promise<any>
```

**Behavior:**
- Reads `cardFields` and `addressFields` from `useCheckoutContext()`
- POSTs `{ item: { card: cardFields, address: addressFields } }` to `/api/commerce/checkout` (POST)
- Returns the API response data

**Usage:**

```typescript
import { useSubmitCheckout } from '@vercel/commerce-commercejs/checkout'

function PlaceOrderButton() {
  const submitCheckout = useSubmitCheckout()

  const handleSubmit = async () => {
    try {
      const result = await submitCheckout()
      console.log('Order placed:', result)
    } catch (error) {
      console.error('Checkout failed:', error)
    }
  }

  return <button onClick={handleSubmit}>Place Order</button>
}
```

## API Endpoint: Checkout Submission

The checkout API endpoint at `/api/commerce/checkout` (POST) performs the server-side order capture.

**Request body:**
```typescript
{
  item: {
    card: CardFields    // Card form field values
    address: AddressFields  // Address form field values
  },
  cartId: string  // Cart ID from cookie (injected by framework)
}
```

**Server-side process:**
1. Generate checkout token: `commerce.checkout.generateTokenFrom('cart', cartId)`
2. Fetch shipping options: `commerce.checkout.getShippingOptions(checkoutToken, { country: 'US' })`
3. Select first available shipping method
4. Normalize checkout data via `normalizeTestCheckout()` (uses Commerce.js test gateway)
5. Capture order: `commerce.checkout.capture(checkoutToken, checkoutData)`

**Important:** The current implementation uses Commerce.js **test gateway** with hardcoded test card values (`4242 4242 4242 4242`). The card fields from the form are only used for billing/shipping name and address fields, not actual payment processing.

**Response:** `{ data: null }` on success

**Registration:**

```typescript
// pages/api/commerce/[...commerce].ts
import commercejsAPI from '@vercel/commerce-commercejs/api/endpoints'
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()
export default commercejsAPI(commerce)
```
