A bit more contramap
---

The main aim of this post is to explore `contramap` without touching Variant or Contravariant Functors [^1]. We
stick to the practical corners of the fp-ts ecosystem [^2] and spotlight a scenario where this helper can be very
useful.

Think about your last trip abroad. You had your phone charger, the wall had its socket, and the only mismatch lived in
the plug shape. With luck you brought a power adapter, so you could connect your charger to the wall socket without
problems. The adapter is the small piece that converts the shape of the plug into the shape of the socket.

That is exactly how `contramap` behaves. It stands between two shapes that would not normally fit without requiring any
change. If you carry an OOP mindset, it should feel similar to the Open/Closed Principle: extend behavior without
cracking open the original piece.

Let's return to the earlier article on composing sorting rules [^4], but now we zoom in on the `contramap` step and
leave the `Ord` discussion aside. In this setup, the socket is the `Ord<string>` that helps us compare strings and your
plug is an instance of `Category` that clearly is not a string. The `contramap` becomes the adapter that makes them fit.

```ts
import {pipe} from 'fp-ts/function'
import * as s from 'fp-ts/string'

const ordCategoriesAlphabetically: Ord.Ord<Category> = pipe(
  s.Ord,
  Ord.contramap((category: Category): string => category.name),
)
```

The adapter here is the function that turns a `Category` into a `string`. That string is not random; it follows the
rules of the domain. Once the adapter is in place, much like a power adapter, the `Categories` can now be ordered
alphabetically without changing the original `Ord<string>`. The same pattern shows up in other modules. The `Predicate`
module [^5] is a good example because it helps you model rules, filters, and guards that capture your business logic.
These tiny pieces feel simple, but they can be used to build complex logic.

Let us turn that into requirements and code.

```markdown
0001: As a user, I want to see products as unavailable when the category is disabled.

1. The categories can be enabled or disabled.
2. The products with disabled categories must be considered unavailable.
```

Let us model these requirements with types:

```ts
interface Category {
  readonly id: string
  readonly name: string
  readonly disabled: boolean
}

interface Product {
  readonly id: string
  readonly name: string
  readonly category: Category
}
```

We can translate the two rules as follows:

```ts
// Category module
const isCategoryDisabled = (category: Category): boolean => category.disabled
```

The second rule tells us that a product with a disabled category must count as unavailable. We already know how to
identify disabled categories, so we would like to reuse that knowledge for products. The `isCategoryDisabled` function
expects a `Category`, while the new predicate receives a `Product`. A small adapter that extracts the category is all we
need, and `contramap` provides it:

```ts
import * as P from 'fp-ts/Predicate'
import {pipe} from 'fp-ts/function'
import {isCategoryDisabled} from './category'

// Product module
const isProductUnavailable: P.Predicate<Product> = pipe(
  isCategoryDisabled,
  P.contramap((product: Product): Category => product.category)
)
```

Now imagine a bug report: reviews should be hidden when the related product is unavailable.

```markdown
0002: As a user, I should not see reviews for unavailable products.

1. The reviews with unavailable products must be marked inactive.
```

The same pattern repeats. We already have a predicate that tells us if a product is unavailable, and we need to adapt
the argument from a `Review` to a `Product`:

```ts
// Review module
import * as P from 'fp-ts/Predicate'
import {pipe} from 'fp-ts/function'
import {isProductUnavailable} from './product'

interface Review {
  readonly product: Product
}

const isReviewInactive: P.Predicate<Review> = pipe(
  isProductUnavailable,
  P.contramap((review: Review): Product => review.product)
)
```

For teaching purposes, we can inline every step so this bridge's behavior becomes crystal clear:

```ts
// Review module
import * as P from 'fp-ts/Predicate'
import {pipe} from 'fp-ts/function'
import {isCategoryDisabled} from './category'

interface Review {
  readonly product: Product
}

const isReviewInactive: P.Predicate<Review> = pipe(
  isCategoryDisabled,
  P.contramap((product: Product): Category => product.category),
  P.contramap((review: Review): Product => review.product),
)
```

Conclusion

Contramap is that travel adapter you toss in your bag. You pack it because you know the outlet will not bend to your
charger. Treat features the same way. Look for spots where an adapter keeps logic reusable instead of hacking together
another one-off, and suddenly those tiny rules start shaping your business logic. That is composable thinking, and once
it lands, it is tough to ship software any other way.

[^1]: https://blog.ploeh.dk/2021/09/02/contravariant-functors/

[^2]: https://gcanti.github.io/fp-ts

[^3]: https://gcanti.github.io/fp-ts/modules/Option.ts.html

[^4]: 2025-10-10-COMPOSE-SORTING-RULES.md

[^5]: https://gcanti.github.io/fp-ts/modules/Predicate.ts.html
