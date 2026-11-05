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

Make the source tree copy stale by adding a field to `schema.txt`:

```sh
echo "traceId:string" >> logger/schema/schema.txt
```

Run the test again. It fails because `logger.pb.go` no longer matches the generated output:

```sh
bazel test //logger/schema:write_logger_pb_test --test_output=errors
```

```
==================== Test output for //logger/schema:write_logger_pb_test:
9d8
< 	TraceId string
FAIL: files "logger/schema/_logger.pb.go" and "logger/schema/logger.pb.go" differ.

@@//logger/schema:logger.pb.go is out of date. To update this file, run:

    bazel run //logger/schema:write_logger_pb
```

Update the source tree copy:

```sh
bazel run //logger/schema:write_logger_pb
```

Run the test again to confirm it passes:

```sh
bazel test //logger/schema:write_logger_pb_test
```

## Requirements

- Bazel 9.1.0 (via [Bazelisk](https://github.com/bazelbuild/bazelisk))
