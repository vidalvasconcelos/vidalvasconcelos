Composing sort rules using fp-ts
---

In this example I want to double down on the power of composition: the power of assembling complex rules from smaller,
well-understood pieces and how that helps us define better boundaries between our modules. In this walkthrough we are
using plain files, but in a more realistic setup each piece could happily live inside its own module in different
directories.

So let's start with a very common scenario that shows up across different kinds of applications: categories and
products, and the rules that govern how we sort them for display.

Our first step is to list the tools we have on hand so we keep our code cohesive without reinventing the wheel. For that
we reach for the `Ord` module from fp-ts[^1] along with a couple of essentials such as Monoids.

This module lets us express ordering rules declaratively. Contrast that with what we usually find in applications:
ad-hoc sorting logic scattered all over the place, reproduced multiple times, full of ternaries and `if`/`else`
statements. Things get gnarly the moment complex, nested rules arrive, and maintaining or extending that code quickly
becomes a nightmare.

Enough "lero-lero", let's see how this works in practice. First, let's lock down our acceptance criteria:

```markdown
**0001: Products must be ordered according to these rules:**

1. Products must be sorted alphabetically by their `name`.
2. Products that belong to the same category must stay together.
3. Product categories must be ordered alphabetically by their `name`.
```

With that in mind we can start modeling the problem we want to solve. We face two entities: Product and Category. In
this example the relationship is 1:1. We'll introduce them with interfaces, which lets us reuse them freely while
abstracting away any concrete implementation details that do not matter right now.

```ts
interface Category {
  id: string
  name: string
}

interface Product {
  id: string
  name: string
  category: Category
  unavailable: boolean
}
```

Now we can tackle each criterion in isolation.

"Product categories must be ordered alphabetically by their `name` attribute."

We can read that as: a product entity has to follow the exact same ordering rule as its `name` property, which is a
`string`. Great, we already know how to sort strings. Take a look at this piece of documentation from the fp-ts `string`
module[^2]:

```ts
import * as s from 'fp-ts/string'

assert.deepStrictEqual(s.Ord.compare('a', 'a'), 0)
assert.deepStrictEqual(s.Ord.compare('a', 'b'), -1)
assert.deepStrictEqual(s.Ord.compare('b', 'a'), 1)
```

We can leverage `contramap` to hand that same behavior to both Category and Product. To keep things short, let's focus
on Category first.

```ts
import * as s from 'fp-ts/string'

const ordCategoriesAlphabetically: Ord.Ord<Category> = pipe(
  s.Ord,
  Ord.contramap((category: Category): string => category.name),
)
```

Now we have an `Ord<Category>` (and, by extension, an `Ord<Product>`) that partially solves criteria 1 and 3. Next we
have to handle criteria 2, which we can rephrase as: "Products must be ordered by category." Once again we face the same
shape of problem, we want a type to follow the ordering of one of its internal attributes. So we can reuse the exact
same
strategy:

```ts
import * as s from 'fp-ts/string'

const ordProductsAlphabeticallyByCategory: Ord.Ord<Product> = pipe(
  s.Ord,
  Ord.contramap((product: Product) => product.category.name),
)
```

Notice that while this works, we can simplify further because we already defined what it means to order categories by
`name`. That gives us a healthier coupling between the two modules:

```ts
const ordProductsAlphabeticallyByCategory: Ord.Ord<Product> = pipe(
  ordCategoriesAlphabetically,
  Ord.contramap((product: Product) => product.category),
)
```

Although we've addressed each requirement individually, and in a reusable way that extends beyond the original problem.
we
still need a mechanism to combine these rules so they deliver the result we actually want. This is where a remarkably
powerful tool steps in: the Monoid[^3].

Yes, the name sounds odd, but there's no reason to panic. Think of Monoids as pressure cookers: surprisingly easy to
operate and capable of making your life a lot simpler.

```ts
import * as M from 'fp-ts/Monoid'

const ordProducts: Ord.Ord<Product> = M.concatAll(Ord.getMonoid())([
  ordProductsAlphabeticallyByCategory,
  ordProductsAlphabeticallyByName,
])
```

If currying is not your daily driver, we can rewrite the solution like this:

```ts
import * as M from 'fp-ts/Monoid'

const combineMultipleOrds = M.concatAll(Ord.getMonoid())

const ordProducts: Ord.Ord<Product> = combineMultipleOrds([
  ordProductsAlphabeticallyByCategory,
  ordProductsAlphabeticallyByName,
])
```

The result is still an `Ord<Product>`, but now each criteria lives in isolation, making maintenance equally isolated.
The implementation also becomes a semantic entry point for understanding the sorting rules. By combining them in a list
we get a ranking view where we can quickly see which rule takes precedence. We can toggle them on or off with nothing
more than a comment.

Let's push this system a bit further by introducing a hypothetical feature. Imagine Products now have an
`unavailable` boolean flag and a new acceptance criteria:

```markdown
**0002: Products must be ordered according to these rules:**

1. They must follow the rules defined in #0001.
2. Products marked as unavailable must appear at the end of their category list.
```

Following the same approach, we add a new `Ord<Product>` and include it alongside the existing rules:

```ts
import * as b from "fp-ts/boolean";

const ordProductsByUnavailability: Ord.Ord<Product> = pipe(
  b.Ord,
  Ord.contramap((product: Product): boolean => product.unavailable),
)

// ...

const ordProducts: Ord.Ord<Product> = M.concatAll(Ord.getMonoid())([
  ordProductsAlphabeticallyByCategory,
  ordProductsByUnavailability,
  ordProductsAlphabeticallyByName,
])
```

Stop the machines! The users started complaining, the unavailable products buried the items they actually wanted to see.
A new requirement pops
up:

```markdown
**0003: Products must be ordered according to these rules:**

1. This requirement deprecates #0002.
```

Done. We just disable the availability rule:

```ts
const ordProducts: Ord.Ord<Product> = M.concatAll(Ord.getMonoid())([
  ordProductsAlphabeticallyByCategory,
  // ordProductsByUnavailability,
  ordProductsAlphabeticallyByName,
])
```

## Conclusion

Composition keeps our sorting logic friendly and ready for change. Each rule stays readable, the intent is obvious, and
we can reuse behavior or toggle features with a quick comment. Monoids are still that pressure cooker from earlier:
grab the handle, let them do the heavy lifting, and serve the result with zero stress. The next time someone asks for a
fresh sorting tweak, just add it to the list and count on the combined rule set to keep everything tidy.

[^1]: https://github.com/gcanti/fp-ts

[^2]: https://gcanti.github.io/fp-ts/modules/string.ts.html#ord

[^3]: https://gcanti.github.io/fp-ts/modules/Monoid.ts.html#monoid-overview
