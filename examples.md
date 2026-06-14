# carp-ecs-lite Examples

This document showcases more advanced usage and patterns for `carp-ecs-lite`.

## Complex Queries

You can query for any number of components. The macro will efficiently iterate over the smallest storage and perform existence checks on the others.

```clojure
(query &world [e Position Velocity Sprite Health]
  (do
    ;; This code only runs for entities that have ALL four components
    (update-position! &world &e)
    (render-sprite! &world &e)))
```

## Manual Protocol Implementation

If you don't want to use `ECS.defworld` (e.g., if you have a custom struct), you can implement the `HasComponent` protocol manually:

```clojure
(deftype MyCustomWorld [
  registry ECSRegistry
  pos (SparseSet Position)
])

(impl-for (MyCustomWorld Position) HasComponent
  (defn get-storage [w] (MyCustomWorld.pos w)))
```

## Generational Safety

Entity handles are stable. Even if an entity is destroyed and its index is reused, old handles will be recognized as invalid.

```clojure
(let [e1 (GameWorld.create-entity! &world)]
  (do
    (GameWorld.destroy-entity! &world e1)
    (assert (not (Registry.valid? (GameWorld.registry &world) e1)))))
```
