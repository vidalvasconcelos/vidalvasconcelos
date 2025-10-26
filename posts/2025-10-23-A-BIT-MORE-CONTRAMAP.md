A bit more contramap
---

The goal of this post is to take a closer look at `contramap` without going into the formal world of Contravariant
Functors [^1]. We’ll keep things grounded in real code and focus on how this small helper can make everyday work
easier to reason about.

Imagine you’re traveling abroad. You’ve got your phone charger, the wall has its socket, and the only problem is
that the plug doesn’t fit. Luckily, you packed a power adapter. It doesn’t change your charger or the wall. It
simply helps one connect to the other.

That is the same idea behind `contramap`. It sits in the middle of two shapes that don’t naturally match and helps
them work together without changing anything on either side. If you come from an OOP background, it might remind
you of the Open/Closed Principle, where you extend behavior without modifying what already works.

To see this in practice, let’s revisit the earlier post about composing sorting rules [^2]. This time we’ll focus
on the `contramap` step and set aside the details about Ord. In this setup, the socket is an Ord<string> that knows
how to compare strings, while your plug is a Category, which is clearly not a string. `contramap` becomes the adapter
that allows the two to connect perfectly.

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
module [^3] is a good example because it helps you model rules, filters, and guards that capture your business logic.
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

`contramap` works like that universal adapter you carry when you travel. It bridges shapes that don’t naturally fit,
keeping what already works untouched. The trick is simple but powerful: instead of changing your logic, you adapt
the data to match it. Once you start thinking this way, composition stops being an abstract principle and becomes
a habit. You spend less time rewriting and more time connecting the pieces that are already there.

[^1]: https://blog.ploeh.dk/2021/09/02/contravariant-functors/

[^2]: https://github.com/vidalvasconcelos/vidalvasconcelos/blob/main/posts/2025-10-10-COMPOSE-SORTING-RULES.md

[^3]: https://gcanti.github.io/fp-ts/modules/Predicate.ts.html
