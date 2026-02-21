Overview
Set up the project structure for the Storage Patterns example, which will demonstrate the three types of storage available in Soroban contracts.

Context
Storage is a fundamental concept in smart contracts. This example will showcase:

Persistent storage (survives contract upgrades)
Instance storage (contract-specific, survives across invocations)
Temporary storage (single ledger lifetime)
Understanding when to use each type is crucial for efficient contract design.

Objectives
Create organized project structure
Configure dependencies appropriately
Set up basic contract shell ready for storage implementation
Integrate with workspace
Acceptance Criteria

Project directory created at examples/basics/02-storage-patterns/

Cargo.toml configured with:
Package name: storage-patterns
Soroban SDK dependencies
cdylib crate-type
Test utilities feature

src/lib.rs created with basic contract struct

Project added to workspace members

Initial build succeeds
Technical Details
Project will demonstrate:

Storage trait usage
get() and set() operations
Storage key management
TTL (Time To Live) concepts
Directory structure:

examples/basics/02-storage-patterns/
├── Cargo.toml
├── README.md
├── src/
│   └── lib.rs
└── test_snapshots/
Dependencies
soroban-sdk (latest)
soroban-sdk with testutils for testing
Priority
🔴 High - Foundational storage concepts