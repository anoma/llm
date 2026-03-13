---
name: elixir-conventions
description: Elixir-specific coding patterns and conventions.
---

# Elixir Conventions

## Control Flow

- Pattern match in function heads rather than branching in the body.
- Prefer `with` chains over nested `case`.
- Keep iteration flat: `Enum.flat_map`, `Enum.filter`, or split
  into sequential steps (gather then process). Never nest
  `Enum.reduce_while` or `Enum.reduce` — if you need two levels,
  decompose into a flat_map + single reduce, or use recursion.
- `Enum.reduce_while` is fine for a single-level early-exit loop,
  but reach for simpler constructs first (`flat_map`, `Enum.find`,
  `Enum.any?`, multi-clause recursion).
- **Map-then-combine over stateful reduce** (Guy Steele pattern):
  instead of threading an accumulator through `Enum.reduce`, map
  each element into monoid space then combine with a monoidal op:
  ```elixir
  # Instead of:
  Enum.reduce(items, %{}, fn {k, v}, acc -> Map.put(acc, k, v) end)
  # Prefer:
  Map.new(items, fn {k, v} -> {k, v} end)
  # Or when Map.new doesn't fit:
  items |> Enum.map(&to_map/1) |> Enum.reduce(%{}, &Map.merge/2)
  ```
  Common monoids: `Map.merge/2`, `MapSet.union/2`, `++`, `+`.
  Shorthand: `Map.new/2`, `MapSet.new/2`, `Enum.flat_map/2`.

## Types & Structs

- Use `typedstruct` for struct definitions.
- Prefer union types for option lists over keyword defaults:
  ```elixir
  @type startup_options ::
          {:name, atom()}
          | {:timeout, pos_integer()}

  @spec start_link(list(startup_options())) :: GenServer.on_start()
  ```
- Keep type definitions near the top, after `use`/`alias` blocks.

## GenServer

- `handle_call` body: `{:reply, do_foo(...), state}` — core logic
  in the `do_` function.
- Never call a callback from another callback.
- Organize GenServer modules into clear sections:
  ```elixir
  ############################################################
  #                        Public API                        #
  ############################################################

  ############################################################
  #                    GenServer Callbacks                   #
  ############################################################

  ############################################################
  #                   Private Implementation                 #
  ############################################################
  ```

## State Boundaries

Choose where state lives based on two questions:

1. **Should users or external code be able to query it?**
   If yes → persistent storage (DB on disk).
2. **Can we tolerate losing it on crash or shutdown?**
   If yes → process state is fine for pure implementation details.
   If we can tolerate loss on shutdown but not on crash →
   an ETS/Mnesia table (survives process crashes, lost on VM stop)
   may be the right middle ground.

## Documentation

- First-person voice: "I am the X module." / "I return the Y."
- `@moduledoc` starts with one-sentence purpose.
- `### Public API` section listing all public functions.
- All public functions need `@doc` and `@spec`.

## Examples

Examples use the `ExExample` library for caching and composition.

- `use ExExample` in the module, `import ExUnit.Assertions` for
  assertions.
- Define examples with the `example` macro:
  ```elixir
  @spec create_upload() :: Upload.t()
  example create_upload do
    {:ok, upload} = Uploads.store_new_file(...)
    assert upload.stored_name != nil
    upload  # return a useful object
  end
  ```
- Examples are cached — calling one from another reuses the
  result instead of re-running. This enables composition:
  ```elixir
  example add_comment do
    upload = EUpload.create_upload()  # cached, not re-run
    {:ok, comment} = Comments.create(...)
    comment
  end
  ```
- Override `rerun?/1` to control caching per example:
  `def rerun?(_), do: true` to always re-run (e.g., examples
  that spawn processes or depend on runtime state). Pattern
  match on specific examples for selective control.
- Tests: `use ExExample.ExUnit, for: Examples.EModule` to
  auto-generate one test per example.
- Live in `lib/examples/e_<module>.ex`.
- Module name: `E<Module>` (e.g., `EUpload`, `EEventStore`).
- Every example gets `@spec` — they are typed public functions.
- Run with: `mix run -e 'EModule.example_name()'`
- Build examples incrementally: verify each layer in IEX before
  writing the next. The examples ARE the verification.

## Interactive Testing

- Run one-off expressions: `timeout 60 mix run -e 'code'`
  (never use `--no-halt`, it hangs the VM).
- Inspect process state: `:sys.get_state(pid)`

## Formatting

- 98 character line length.
- Run `mix format` before finalizing.
- Section headers use banner comments:
  ```elixir
  ############################################################
  #                      Section Name                        #
  ############################################################
  ```
