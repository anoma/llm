---
name: elixir-conventions
description: Elixir-specific coding patterns and conventions.
---

# Elixir Conventions

## Control Flow

- Prefer `with` chains over nested `case`.
- Use `Enum.reduce_while` or `Enum.flat_map` over `Enum.reduce`
  with internal conditionals.
- Pattern match in function heads rather than branching in the body.

## Types & Structs

- Use `typedstruct` for struct definitions.
- Express startup/init arguments as a union of tagged tuples so
  callers can pass options in any order:
  ```elixir
  @type startup_options ::
          {:name, atom()}
          | {:timeout, pos_integer()}

  @spec start_link(list(startup_options())) :: GenServer.on_start()
  ```
- Keep type definitions near the top, after `use`/`alias` blocks.

## GenServer / CQRS

- `call` for reads or synchronization only. Writes use `cast`.
- `handle_call` body: `{:reply, do_foo(...), state}` — core logic
  in the `do_` function.
- Never call a callback from another callback.
- Organize GenServer modules into clear sections (public API,
  callbacks, private helpers) using banner comments:
  ```elixir
  ############################################################
  #                      Public API                          #
  ############################################################

  ############################################################
  #                    GenServer Callbacks                   #
  ############################################################

  ############################################################
  #                    Private Helpers                       #
  ############################################################
  ```

## Documentation

- First-person voice: "I am the X module." / "I return the Y."
- `@moduledoc` starts with one-sentence purpose.
- `### Public API` section listing all public functions.
- All public functions need `@doc` and `@spec`.

## Examples

- Keep runnable examples alongside production code (not just in
  `test/`). A common pattern is a dedicated examples directory.
- Use `import ExUnit.Assertions` for `assert` in examples.

## Interactive Testing

- Run one-off expressions: `timeout 60 mix run -e 'code'`
  (never use `--no-halt`, it hangs the VM).
- Inspect process state: `:sys.get_state(pid)`
- Query persistent stores (ETS, Mnesia, databases) to verify data
  is actually written and read.

## Formatting

- 78 character line length.
- Run `mix format` before finalizing.
- Section headers use banner comments:
  ```elixir
  ############################################################
  #                      Section Name                        #
  ############################################################
  ```
