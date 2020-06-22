# Counter cache

Counter caches serve a very specific purpose: they track the amount of entities associated with another entity.

It can be tempting to lean on them for broader use cases. For example, it can be unperformant to query something like "select all foos with at least one bar", where foos `has_many` bars. However, in this case, rather than using a counter cache as a proxy for what you want, you can model what you want better. For example, oftentimes the question you're really trying to answer is "select all foos that have transitioned to a certain state", e.g. "created" to "started". Then, you can simply select by that status, which is much closer to the business logic.

One correct usage of counter cache might be if you wanted to show a list of entities in a table, and also show the count of child entities in the list. Basically, use cases where:

1. You really do want a numerical count (rather than using count as a proxy for some state).
2. You'd be N+1-ing without some sort of cache.

Problems with counter cache:

1. Brittle to maintain. If your business logic ever changes where you need to track more than 1 thing, you will need to add more counters.
2. Often a proxy for what you want to select by, vs. the thing you want to select by itself.

