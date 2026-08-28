# carp-ecs-lite

A lightweight, modular Entity Component System (ECS) for the [Carp language](https://github.com/carp-lang/Carp).

Built on top of the `SparseSet` and `Handle` modules from
[carp-collections](https://github.com/carpentry-org/carp-collections).

> **Status: does not build.** The API is written against `defprotocol` and
> `impl-for`, which are not forms carp has. Loading `src/ecs.carp` stops at
> `Can't find symbol 'defprotocol'` before anything else is checked, so both
> test suites are unrunnable. This needs a port onto interfaces (or the
> feature needs to land in the compiler) before the library is usable.

## Features

- **Generic Protocol API**: Use `add-component!`, `get-component`, and `query` without passing field names or symbols.
- **Automated World Definition**: The `ECS.defworld` macro handles all storage and implementation boilerplate.
- **Sparse Set Storage**: High-performance contiguous iteration via `carp-sparseset`.
- **Generational Entities**: Stable, versioned handles via `carp-handle` to prevent stale references.

## Installation

```clojure
(load "carp-ecs-lite/src/ecs.carp")
(use ECS)
```

## Basic Usage

```clojure
(use ECS)

;; 1. Define your component types
(deftype Position [x Float y Float])
(defmodule Position
  (defn zero [] (Position.init 0.0f 0.0f))
  (implements zero Position.zero)
)

(deftype Velocity [dx Float dy Float])
(defmodule Velocity
  (defn zero [] (Velocity.init 0.0f 0.0f))
  (implements zero Velocity.zero)
)

;; 2. Define your world with registered components
(ECS.defworld GameWorld Position Velocity)

(defn main []
  (let [world (GameWorld.new)
        ent (GameWorld.create-entity! &world)]
    (do
      ;; 3. Add components (type is inferred from the value!)
      (add-component! &world &ent (Position.init 10.0f 20.0f))
      (add-component! &world &ent (Velocity.init 1.0f 1.0f))

      ;; 4. Query entities with specific component sets
      (query &world [e Position Velocity]
        (let [pos (Maybe.unsafe-from (get-component &world &e))
              vel (Maybe.unsafe-from (get-component &world &e))]
          (add-component! &world &e (Position.init (+ @(Position.x &pos) @(Velocity.dx &vel))
                                                    (+ @(Position.y &pos) @(Velocity.dy &vel))))))
      0)))
```

## Examples

See [examples.md](examples.md) for usage examples.

## License

MIT
