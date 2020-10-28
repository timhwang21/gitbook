# Advice

Very WIP. Find a better home for this.

## How to make decisions and avoid analysis paralysis

- Do not strive to do the best thing. You likely don't have complete information about the requirements, and even if you did, the extra effort likely won't be worth it. When faced with indecision, do **the least risky thing**.
  - A good rule-of-thumb for less risk is less complex. Oftentimes (especially earlier in a product's lifecycle), less complex and less correct is better than more complex and more correct (with "correct" meaning "comprehensive coverage of use cases," "correct handling of all edge cases," "perfect performance," etc.). Eschew obfuscation.
- What does this mean? Do the thing that defers the most decisions and is easy to change. In other words, leave as many doors open to yourself as possible.
- Focus on small versions. "You don't understand a problem until after your first attempt to solve it." You _need_ to know how something will be used before properly designing anything (from a function to an interface to a product). No one's smart enough to always get it right. Build in a way that allows you to swap out implementation with as little pain as possible. (It will always be painful! Just aim to minimize it.)
- This also opens the door for you to not be afraid of choosing the wrong abstractions. While choosing the wrong abstractions is expensive and not ideal, if you leave doors open to yourself, you can quickly react to mistakes, missed use cases, etc. (and they _will_ exist). Additionally, less of a focus on perfection makes you less attached to any particular implementation. "Don't patch bad code, rewrite it." -K&P

tl;dr: de-risk over perfection; build code to be discardable; leave doors open.

## How to produce good code

Note the choice of words: "produce" over "write." You can't be responsible for writing all the code, and writing it well -- you will eventually need to delegate without hand-holding.

### Hands-on

- Code reviews are probably the single most efficient method for knowledge transfer. Aim for a mix of high-level design feedback and low-level tactics feedback.
  - Aim for (but do not overoptimize for) for small, bite-size patches. They do make code reviews easier and more likely to be done in a timely manner; however, they can make high-level feedback harder to give. However, this can be remedied with good design docs, etc.

### Hands-off

- Aim for "the pit of success." That is, build a foundation where extending existing patterns without thought leads to the right thing being done.
- Engineers copy-paste a lot. Thus, it is worth cleaning up "canonical" reference implementations, such that copy-pasting doesn't lead to a proliferation of patterns and results in the same level of code quality.
- When modeling, consider how code can be extended by someone with less domain knowledge as you. (As you're the one building the initial version you probably have _the most_ domain knowledge.) If there is some obvious future use case that's out of scope, it can be worth it to make a stub class, etc., that provides a natural avenue for future expansion. Leave breadcrumbs for engineers who will work on your code in the future.
- Write [hard to misuse interfaces](https://ozlabs.org/~rusty/ols-2003-keynote/img39.html).
  - Good types to make incorrect code unmergable. Static tools become more and more valuable once codebase no longer fits in one person's head. As long as it does, the benefit of types is lesser (but still existent); given the cost of static types, the right inflection point is up to the individual's preference.
  - Few, well-named, easy-to-understand arguments with good defaults. Well-named public interfaces.
  - Strict, fail-fast implementations that always (or almost always) break obviously if used incorrectly.
  - Adhere to common conventions.
  - Bad: reliance on reading docs, reliance on reading implementation.
  - Very bad: reliance on reading Slack threads. Outdated documentation.
