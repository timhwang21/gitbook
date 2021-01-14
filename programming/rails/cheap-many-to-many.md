# Cheap many-to-many

There are two common ways to establish a many-to-many relationship:

1. `has_and_belongs_to_many`
2. `has_many, :through`

Both require a join table, and the second requires a little more setup than the first bit is more future-proof.

A third alternative is so simple it sounds dumb: simply add a JSON column to the parent that contains an array of child IDs. If you understand the constraints of this approach and your use case doesn't need them, it can be a flexible and noncommittal way to model relationships.

## Downsides

- No guarantee of data integrity (can have orphaned IDs, no `dependent: destroy`, etc.).
- Cannot access list of parents from child (can't see who a model belongs to).
- Worse performance (or, at least, unmanaged performance).

## Advantages

- Very noncommittal. When working with uncertainty, it's good to be as flexible as possible until you are sure of requirements. Creating a table is _very_ committal.
- Automatic transactions. Simply update the parent record without needing to worry about data inconsistency.
- Can easily migrate to a true join-table based relationship.

