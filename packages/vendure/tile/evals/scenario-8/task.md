# Fetch Site Categories as Hierarchical Tree

## Overview

Implement a server-side function that fetches the Vendure product collections and returns them as a hierarchical category tree for use in navigation menus.

## Capabilities

### Fetch collections and build category hierarchy

Implement a `fetchSiteCategories()` function that:

- Initializes the Vendure commerce API using `getCommerceApi`
- Calls the `getSiteInfo` operation to retrieve collections
- Returns a `categories` array that is a hierarchical tree (not a flat array)
- Returns an empty `brands` array (Vendure does not provide brand data)

[@test](./tests/fetch-categories.test.ts)

### Flat collections converted to tree structure

Show that the `getSiteInfo` operation uses the package's tree-building utility to convert the flat Vendure collection array (where each item has an optional `parent.id`) into a nested hierarchy. Given:

- A root collection with id `"1"` and no parent
- A child collection with id `"2"` and `parent.id = "1"`
- A grandchild collection with id `"3"` and `parent.id = "2"`

The resulting tree should have the root's children containing the grandchild at the correct nesting level.

[@test](./tests/collections-tree.test.ts)

### Collection path format

Show that each category in the result has a `path` property formatted as `/${id}` (using the collection's id, not its slug).

[@test](./tests/category-path.test.ts)

## Implementation

[@generates](./src/fetch-categories.ts)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Exports `getCommerceApi` for server-side operations. The `getSiteInfo` operation fetches Vendure collections and converts them to a hierarchical tree using the `arrayToTree` utility.

[@satisfied-by](@vercel/commerce-vendure)
