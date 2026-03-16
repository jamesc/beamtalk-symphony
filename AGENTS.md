# symphony — Agent Guide

## Project Structure

```
symphony/
├── beamtalk.toml    # Package manifest
├── src/             # Source files (.bt)
│   └── main.bt      # Entry point
├── test/            # BUnit test files
├── _build/          # Build output (generated)
├── AGENTS.md        # This file
├── .github/
│   └── copilot-instructions.md
├── .mcp.json        # MCP server config
├── README.md
└── .gitignore
```

## Build & Run

```bash
beamtalk build       # Compile to BEAM bytecode
beamtalk repl        # Interactive development (auto-loads package)
beamtalk test        # Run BUnit tests
```

## Beamtalk Syntax Basics

```beamtalk
// Variables
x := 42
name := "hello"

// Message sends
x factorial              // unary
3 + 4                    // binary
list at: 1 put: "value"  // keyword

// Blocks (closures)
square := [:x | x * x]
square value: 5          // => 25

// Classes
Object subclass: Counter
  state: count = 0

  increment => self.count := self.count + 1
  count => self.count
```

## Live Workspace (MCP)

The `.mcp.json` in this project configures the `beamtalk` MCP server, which gives
you live access to a running REPL. Claude Code starts it automatically via
`beamtalk-mcp --start` — no manual `beamtalk repl` required.

**Prefer MCP tools over guessing.** If you're uncertain what a method returns or
whether code is correct, evaluate it directly rather than inferring from source.

| Tool | When to use |
|------|-------------|
| `evaluate` | Test expressions, explore values, prototype code snippets |
| `complete` | Get autocompletion suggestions for partial input |
| `load_file` | Load a `.bt` file into the workspace before evaluating it |
| `reload_module` | Hot-reload a module after editing — migrates live actors |
| `unload` | Remove a module and its classes from the workspace |
| `list_modules` | Check what's currently loaded |
| `list_actors` | See running actors and their classes |
| `inspect` | Examine a live actor's state by PID |
| `test` | Run BUnit tests — pass a class name or omit to run all |
| `docs` | Look up stdlib class or method docs — primary stdlib reference |
| `show_codegen` | Inspect generated Core Erlang to debug compilation |
| `get_bindings` | See current REPL variable bindings |
| `clear` | Reset the REPL — clears all bindings |
| `interrupt` | Cancel a stuck or long-running evaluation |
| `describe` | List available MCP ops and protocol version |

**Stdlib reference:** use `docs` instead of guessing. The stdlib source lives
inside the beamtalk installation, not in this project. Ask the live workspace:
- `docs: "Integer"` — all Integer methods with docs
- `docs: "List" selector: "select:"` — specific method docs

**Typical workflow:**
1. Edit a `.bt` source file
2. `load_file` (new module) or `reload_module` (existing) to apply changes
3. `evaluate` to verify behaviour interactively
4. `run_tests` to confirm correctness

## Not Smalltalk — Common Pitfalls

Beamtalk looks like Smalltalk but has important differences. The compiler will
catch most of these, but they waste time:

| Smalltalk habit | Beamtalk equivalent | Notes |
|---|---|---|
| `\| temp \|` temp var declarations | Just use `:=` directly | No declaration syntax |
| Trailing `.` on every statement | Newline is the separator | `.` is optional; use it only to disambiguate cascades |
| `"this is a comment"` | `// this is a comment` | Double-quoted strings are data, not comments |
| `^value` on last expression | Just write `value` | `^` is early-return only; last expr is implicitly returned |
| Left-to-right binary (`2+3*4=20`) | Standard math precedence (`2+3*4=14`) | `*` binds tighter than `+` |
| `'hello', name` concatenation | `"hello {name}"` interpolation | `++` also works: `"hello" ++ name` |
| `[:x \| \|temp\| temp := x]` block locals | `[:x \| temp := x]` | No block-local declarations |
| `:` for type annotations | `::` (double-colon) | `state: x :: Integer = 0`, `param :: Type -> ReturnType =>` |
| Unknown message raises error | Same — DNU raises `does_not_understand` error | Use `respondsTo:` to check before sending |

**`^` in blocks is a non-local return (exits the enclosing method):**

```beamtalk
// ^ inside a block exits the METHOD, not just the block:
firstPositive: items =>
  items do: [:x | x > 0 ifTrue: [^x]].   // ^ returns from firstPositive:
  nil   // reached only if no positive element found
```

**DNU raises a `does_not_understand` error.** Sending a message a class
doesn't implement raises a structured error — not a silent `false`. Use
`respondsTo:` or `docs` in the live workspace to confirm a method exists
before calling it.

**Implicit return rule:** the last expression of a method body is always its
return value. Never write `^` on the last line — only use it for early exits
inside the method:

```beamtalk
// Wrong — redundant ^
max: other =>
  ^(self > other ifTrue: [self] ifFalse: [other])

// Correct
max: other =>
  self > other ifTrue: [self] ifFalse: [other]

// Correct use of ^ for early return
safeDiv: other =>
  other = 0 ifTrue: [^0].
  self / other
```

**Binary precedence:**

```beamtalk
2 + 3 * 4      // => 14  (standard: * before +)
2 + (3 * 4)    // => 14  (same)
(2 + 3) * 4    // => 20  (use parens to override)
```

## Language Documentation

- Language features: https://jamesc.github.io/beamtalk/docs/language-features.html
- Syntax rationale: https://jamesc.github.io/beamtalk/docs/syntax-rationale.html
- Examples: see `src/` directory
