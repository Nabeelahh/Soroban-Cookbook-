# Hello World Contract

Your first Soroban smart contract! This example demonstrates the basic structure of a Soroban contract and how to interact with it. Perfect for developers new to Soroban who already know Rust basics.

## 📖 What You'll Learn

- Basic contract structure using `#[contract]` and `#[contractimpl]`
- How to define public contract functions
- Working with Soroban's type system (`Symbol`, `Vec`)
- Using `symbol_short!` for compact symbol literals
- Writing and running tests with the Soroban test environment
- Building a WASM binary for deployment

## ⚙️ Prerequisites

Before you begin, ensure you have:

- **Rust** (1.75 or later recommended)
- **Soroban SDK** (installed via the project's `Cargo.toml` dependencies)
- **WASM target** for building deployable contracts

### Verify Your Setup

```bash
# Check Rust version
rustc --version
# Expected: rustc 1.75+ (or similar)

# Add the wasm32 target (required for building contracts)
rustup target add wasm32-unknown-unknown
# Expected: info: component 'rust-std' for target 'wasm32-unknown-unknown' is up to date
```

From this example's directory, verify the project resolves:

```bash
cd examples/basics/01-hello-world
cargo check
# Expected: Finished dev [unoptimized] target(s)
```

## 🔍 Overview

This contract demonstrates the minimal Soroban pattern. It has a single function `hello()` that:

1. Takes a `Symbol` as input (a name to greet)
2. Returns a `Vec<Symbol>` containing `["Hello", name]`

```rust
pub fn hello(env: Env, to: Symbol) -> Vec<Symbol>
```

No storage, no authentication—just the core mechanics of defining and calling a contract function. Understanding this structure is the foundation for every Soroban contract you'll write.

## 🏗️ Key Concepts

### Contract Macro

```rust
#[contract]
pub struct HelloContract;
```

Defines your contract struct. The `#[contract]` macro marks it as a Soroban contract. The struct can be empty (as here) or contain state. The macro generates the boilerplate needed for the Stellar network to load and execute your WASM.

### Contract Implementation

```rust
#[contractimpl]
impl HelloContract {
    pub fn hello(env: Env, to: Symbol) -> Vec<Symbol> {
        vec![&env, symbol_short!("Hello"), to]
    }
}
```

The `#[contractimpl]` macro makes every public function in this `impl` block a callable contract method. It also generates a client type (`HelloContractClient`) for invoking these methods from tests or off-chain code.

### Environment (`Env`)

```rust
env: Env
```

The first parameter of every contract function. `Env` provides access to blockchain context: storage, events, cryptographic primitives, and ledger information. Even when you don't use it directly, it must be present—the `vec!` macro needs `&env` because Soroban values are allocated in the host environment.

### Types: `Symbol` and `Vec`

- **`Symbol`** — A short identifier (up to 9 characters with `symbol_short!`). Used for function names, storage keys, and compact string-like values. More efficient than raw strings on-chain.
- **`Vec<T>`** — Soroban's owned vector type. Created with the `vec![&env, ...]` macro. Different from Rust's `std::vec!`—Soroban's `Vec` lives in the host environment.

## 📝 Code Walkthrough

A line-group walkthrough of the implementation:

### Imports and Configuration

```rust
#![no_std]

use soroban_sdk::{contract, contractimpl, symbol_short, vec, Env, Symbol, Vec};
```

- `#![no_std]` — Required for all Soroban contracts. Prevents linking the standard library; contracts run in a constrained host environment.
- Imports bring in the contract macros, helpers (`symbol_short!`, `vec!`), and core types.

### Contract Definition

```rust
#[contract]
pub struct HelloContract;
```

An empty struct marked as a contract. The macro generates the contract interface and client.

### Function Implementation

```rust
#[contractimpl]
impl HelloContract {
    pub fn hello(env: Env, to: Symbol) -> Vec<Symbol> {
        vec![&env, symbol_short!("Hello"), to]
    }
}
```

- **`env: Env`** — Always first. Provides host environment access.
- **`to: Symbol`** — The name passed in by the caller.
- **`vec![&env, ...]`** — Builds a Soroban `Vec` in the host. The `&env` is required for allocation.
- **`symbol_short!("Hello")`** — A literal symbol (≤9 chars). For longer symbols, use `symbol!()`.
- Returns a `Vec<Symbol>` containing `"Hello"` and the provided name.

### Why This Design?

Soroban uses a host/guest model: your contract runs as a guest, and the host (the ledger) provides storage, crypto, and allocation. Values like `Vec` and `Symbol` are hosted values—hence `vec![&env, ...]` and the need to pass `env` through.

## 🔨 Building Instructions

Build the WASM binary from the example directory:

```bash
cd examples/basics/01-hello-world
cargo build --target wasm32-unknown-unknown --release
```

**Expected output:**

```
   Compiling hello_world v0.1.0 (...)
    Finished release [optimized] target(s) in X.XXs
```

The compiled contract will be at:

```
target/wasm32-unknown-unknown/release/hello_world.wasm
```

> **Tip:** Release builds are optimized and smaller. Always use `--release` for deployment.

## 🧪 Testing Instructions

Run the test suite:

```bash
cargo test
```

**Expected output:**

```
running 3 tests
test hello_world::test::test_hello ... ok
test hello_world::test::test_hello_with_different_names ... ok
test hello_world::test::test_hello_response_length ... ok

test result: ok. 3 passed; 0 failed
```

### What the Tests Cover

| Test | Purpose |
|------|---------|
| `test_hello` | Core behavior: call `hello("World")`, assert result is `["Hello", "World"]` |
| `test_hello_with_different_names` | Verifies output for multiple inputs (Alice, Bob, Stellar) |
| `test_hello_response_length` | Ensures the response always has exactly 2 elements |

### Test Flow

1. **Create environment** — `Env::default()` simulates the blockchain for unit tests.
2. **Register contract** — `env.register_contract(None, HelloContract)` deploys the contract into the test env; `None` uses an auto-generated ID.
3. **Create client** — `HelloContractClient::new(&env, &contract_id)` lets you call contract methods.
4. **Invoke and assert** — Call `client.hello(&symbol_short!("World"))` and assert the returned `Vec`.

## 📦 Deploying

Deployment is optional for learning. When you're ready, you can:

1. **Use Soroban CLI** — See the [Soroban CLI Reference](https://developers.stellar.org/docs/tools/developer-tools/cli) for setup.
2. **Deploy to testnet** — Follow the [Getting Started Guide](https://developers.stellar.org/docs/smart-contracts/getting-started) for network config, identity creation, and funding.
3. **Invoke** — Use `soroban contract invoke` with your contract ID and the `hello` function.

Example invoke (after deploy):

```bash
soroban contract invoke \
  --id <CONTRACT_ID> \
  --source alice \
  --network testnet \
  -- hello \
  --to World
```

## 🎯 Key Takeaways

- **`#[contract]` + `#[contractimpl]`** — The standard pattern for defining Soroban contracts.
- **`Env` is always first** — Every contract function receives `env` for host access.
- **Soroban types ≠ Rust std** — Use `Symbol` and Soroban's `Vec`; they are host-allocated.
- **`symbol_short!` for literals** — Efficient for short identifiers (≤9 chars).
- **Tests use `Env::default()`** — No real network; fast, deterministic unit tests.

## 🎓 Next Steps

Once you're comfortable with this example, explore:

- [Storage Patterns](../02-storage-patterns/) — Learn to persist data with persistent, instance, and temporary storage
- [Basics Index](../README.md) — Browse the full basics learning path and upcoming examples
- [Intermediate Examples](../../intermediate/) — Continue with multi-contract and more advanced patterns

## ⚠️ Troubleshooting

### `rustup: command not found` or outdated Rust

Install or update Rust from [rustup.rs](https://rustup.rs/). Soroban typically requires Rust 1.75+.

### `error[E0463]: can't find crate for 'std'` with `#![no_std]`

You're likely building for the wrong target. Use `--target wasm32-unknown-unknown` for the contract.

### `wasm32-unknown-unknown` target not found

```bash
rustup target add wasm32-unknown-unknown
```

### Tests fail with "contract not found" or client errors

Ensure you're in `examples/basics/01-hello-world` and run `cargo test` from there. The workspace layout can affect paths.

### `symbol_short!` panics or errors with long strings

`symbol_short!` supports up to 9 characters. For longer identifiers, use `symbol!()`.

## 📚 References

- [Soroban SDK Documentation](https://docs.rs/soroban-sdk) — API reference for types, macros, and `Env`
- [Getting Started with Soroban](https://developers.stellar.org/docs/smart-contracts/getting-started) — End-to-end setup and first deploy
- [Soroban CLI Reference](https://developers.stellar.org/docs/tools/developer-tools/cli) — Deploy and invoke from the command line
