# Bazel Code Generation with Source Tree Output

This project demonstrates how to write Bazel-generated files back to the source tree using [`write_source_files`](https://github.com/bazel-contrib/bazel-lib/blob/main/lib/write_source_files.bzl) from [bazel-lib](https://github.com/bazel-contrib/bazel-lib).

## Problem

Bazel always writes build outputs to `bazel-out`, but editors and IDEs expect generated files (like protobuf definitions) to exist in the source tree for features like autocompletion and navigation to work.

## Solution

The `write_source_files` rule bridges this gap with the following pattern:

1. A `genrule` generates code (e.g. `logger.pb.go`) into `bazel-out`
2. A diff test verifies the source tree copy matches the generated output
3. A runnable target copies the generated file back to the source tree when it becomes stale

## Project Structure

```
logger/schema/
├── schema.txt       # Schema definition (input for code generation)
├── logger.pb.go     # Generated file committed to the source tree
└── BUILD.bazel      # genrule + write_source_files
```

## Usage

Build the generated file:

```sh
bazel build //logger/schema:gen_logger_pb
```

Verify the source tree copy is up-to-date:

```sh
bazel test //logger/schema:write_logger_pb_test
```

Update the source tree copy after changing `schema.txt`:

```sh
bazel run //logger/schema:write_logger_pb
```

## Requirements

- Bazel 9.1.0 (via [Bazelisk](https://github.com/bazelbuild/bazelisk))
