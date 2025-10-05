# Bzlmod Migration Guide

This repository has been migrated to use Bazel's bzlmod system with Bazel version 7.6.1.

## What Changed

### Bazel Version
- Updated `.bazelversion` files to 7.6.1 for both root and stablehlo workspaces

### Module Configuration
- Added `MODULE.bazel` files for both root (mlir-hlo) and stablehlo workspaces
- Configured module dependencies from Bazel Central Registry:
  - `bazel_skylib` (1.3.0)
  - `rules_cc` (0.0.9)
  - `rules_python` (0.30.0)
  - `rules_shell` (0.2.0)
  - `apple_support` (1.5.0)

### LLVM Dependencies (Hybrid Approach)
Since LLVM doesn't support bzlmod natively, we use `WORKSPACE.bzlmod` files to manage these dependencies:
- `llvm-raw`: The raw LLVM source archive
- `llvm-project`: Configured LLVM project via `llvm_configure`
- `llvm_zstd`: zstd compression library for LLVM
- `llvm_zlib`: zlib compression library for LLVM

### File Structure
```
mlir-hlo/
├── .bazelversion (7.6.1)
├── MODULE.bazel (bzlmod dependencies)
├── WORKSPACE.bzlmod (LLVM dependencies)
├── WORKSPACE (kept for compatibility)
└── stablehlo/
    ├── .bazelversion (7.6.1)
    ├── MODULE.bazel (bzlmod dependencies)
    ├── WORKSPACE.bzlmod (LLVM dependencies)
    └── WORKSPACE.bazel (kept for compatibility)
```

## Building

The migration is transparent to users. Build commands remain the same:

```bash
# From root workspace
bazel build //path/to:target

# From stablehlo workspace
cd stablehlo && bazel build //:target
```

## Benefits of Bzlmod

1. **Dependency Management**: Automatic version resolution through Bazel Central Registry
2. **Better Isolation**: Each workspace can have its own module configuration
3. **Reproducibility**: MODULE.bazel.lock files ensure reproducible builds
4. **Future-proof**: Bzlmod is the default in Bazel 7.x and beyond

## Hybrid Approach

We use a hybrid approach (bzlmod + WORKSPACE.bzlmod) because:
- LLVM doesn't have a MODULE.bazel file
- WORKSPACE.bzlmod is the recommended way to handle non-bzlmod dependencies in Bazel 7.x
- This allows gradual migration as more dependencies adopt bzlmod

## Verification

To verify the migration:

```bash
# Check Bazel version
bazel --version  # Should show 7.6.1

# View module dependency graph
bazel mod graph

# Test LLVM dependency resolution
bazel build --nobuild @llvm-project//llvm:Support
```
