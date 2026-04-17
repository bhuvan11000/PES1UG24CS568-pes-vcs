# PES-VCS: A Version Control System from Scratch

## Overview
PES-VCS is a local version control system implemented entirely in C. It tracks file changes, stores full project snapshots efficiently, and maintains a linked commit history. The project mirrors the underlying architecture of Git, demonstrating core operating system and filesystem concepts such as content-addressable storage, directory sharding, and atomic file operations.

## Architecture
PES-VCS operates on a content-addressable filesystem. Every file, directory snapshot, and commit is stored as an "object" named by the SHA-256 hash of its contents.

When initialized, the system creates a hidden `.pes/` directory with the following structure:
* **`objects/`**: The content-addressable store. Objects are sharded into subdirectories based on the first two hex characters of their SHA-256 hash to prevent massive flat directories.
* **`refs/heads/`**: Contains branch reference files. Each file holds the hash of the latest commit on that branch.
* **`HEAD`**: A pointer to the current active branch (e.g., `ref: refs/heads/main`).
* **`index`**: A text file representing the staging area, tracking which files are ready for the next commit.

## Supported Commands
PES-VCS supports the fundamental workflows of a version control system:

* `pes init` : Initializes an empty repository and creates the `.pes/` directory structure.
* `pes add <file>...` : Hashes the file contents, stores them as binary blobs in the object store, and stages the files in the index.
* `pes status` : Compares the working directory to the index to show staged changes, unstaged modifications, and untracked files.
* `pes commit -m "<message>"` : Builds a directory tree from the staging area, creates a commit object pointing to the tree and the parent commit, and updates the branch head.
* `pes log` : Walks backward through the commit history starting from `HEAD`, printing authors, timestamps, and messages.

## Implementation Phases
The project is built in four primary phases:

1. **Object Storage Foundation (`object.c`)**: Implements `object_write` and `object_read`. It prepends headers (e.g., `blob <size>\0`), calculates SHA-256 hashes using OpenSSL's EVP API, and ensures safe atomic writes using temporary files and the `rename()` syscall.
2. **Tree Objects (`tree.c`)**: Generates recursive directory structures. It groups files sharing identical paths into subdirectories and serializes them into tree objects representing a complete project snapshot.
3. **The Index / Staging Area (`index.c`)**: Manages the `.pes/index` file. Features robust parsing to load staged files, dynamic metadata tracking (file size, `mtime`), and heap allocation for atomic saves to prevent stack overflow on large repositories.
4. **Commits and History (`commit.c`)**: Ties the snapshots together. It retrieves the parent commit hash from `HEAD`, injects author and timestamp metadata, writes the final `OBJ_COMMIT` to storage, and safely advances the branch pointer.

## Building and Running

### Prerequisites
Ensure you have `gcc`, `make`, and the OpenSSL development libraries installed on your Linux system:

    sudo apt update && sudo apt install -y gcc build-essential libssl-dev

### Compilation
The project uses a `Makefile` for streamlined compilation:

    make          # Builds the main 'pes' executable
    make clean    # Removes all compiled binaries and the .pes/ directory

### Testing
PES-VCS includes both unit tests for internal logic and a full bash-scripted integration test to verify end-to-end functionality:

    make test-unit        # Runs Phase 1 (test_objects) and Phase 2 (test_tree) binaries
    make test-integration # Runs the end-to-end sequence in test_sequence.sh
    make test             # Runs all tests

### Author Configuration
PES-VCS pulls the author name from the environment variable `PES_AUTHOR`. You can configure it in your terminal as follows:

    export PES_AUTHOR="Bhuvan <PES1UG24CS568>"
