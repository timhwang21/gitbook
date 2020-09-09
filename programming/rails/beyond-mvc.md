---
description: Patterns for code organization in Rails beyond plain MVC.
---

# Beyond MVC

## Concerns

Built into Rails past v4. Ultimately just mixins with a little bit of sugar (provides less weird ways do things, e.g. `#included` vs. `self.included`, `#class_methods` vs. `ClassMethods`). The purpose is to allow modularizing of shared code across models and controllers.

While concerns have their place, they are generally not the right solution for business logic. Concerns merely repackage logic, they don't better encapsulate logic into distinct objects. When choosing between concerns and an object-oriented approach, think to yourself: does this bag of functionality model some distinct "thing" in the world? A good use case for concerns would be for something like handling hashing (`use_secure_password` for instance).

## Service objects (or just "services")

POROs that contain highly domain-specific logic that doesn't strictly belong in a model or controller. Examples involve objects containing configuration and handling invocation of a service (as in SOA), or objects orchestrating a transaction involving multiple models.

Some argue that service objects are merely transaction scripts masquerading as OO. This can be seen from service objects that have no state, such that the module is less of a module and more of just a namespace for grouping related logic. These service objects are again, merely mixins, this time with a thin object-oriented veneer.

When service objects do store and consume state, consider if that state is important to persist. If it is, and this service object really does model some real-world object, why not just make it a table? After all, service objects are frequently used to orchestrate logic involving multiple models. Explicitly defining these inter-model relationships can be beneficial.
