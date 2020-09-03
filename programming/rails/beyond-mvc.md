---
description: Patterns for code organization in Rails beyond plain MVC.
---

# Beyond MVC

## Concerns

Built into Rails past v4. Ultimately just modules with a little bit of sugar (provides less weird ways do things, e.g. `#included` vs. `self.included`, `#class_methods` vs. `ClassMethods`). The purpose is to allow modularizing of shared code across models and controllers.

## Service objects (or just "services")

POROs that contain highly domain-specific logic that doesn't strictly belong in a model or controller. Examples involve objects containing configuration and handling invocation of a service (as in SOA), or objects orchestrating a transaction involving multiple models.

Some argue that service objects are merely transaction scripts masquerading as OO.

## Qeury objects

Similar to service objects, but for complex ActiveRecord / SQL calls.
