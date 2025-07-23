# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

load("@rules_foreign_cc//foreign_cc:defs.bzl", "configure_make")

filegroup(
    name = "all_srcs",
    srcs = glob(["**"]),
)

# Use system libffi on macOS ARM64 to avoid CFI assembly issues
cc_library(
    name = "libffi_system",
    linkopts = ["-lffi"],
    visibility = ["//visibility:public"],
)

configure_make(
    name = "libffi_built",
    configure_options = [
        "--disable-multi-os-directory",
        "--disable-dependency-tracking",
        "--disable-docs",
        "AR=ar",
    ] + select({
        "@bazel_tools//src/conditions:darwin_arm64": [
            "--build=aarch64-apple-darwin",
            "--host=aarch64-apple-darwin",
            "--disable-builddir",
        ],
        "//conditions:default": [],
    }),
    configure_env_vars = select({
        "@bazel_tools//src/conditions:darwin_arm64": {
            "CFLAGS": "-Wno-error",
            "CPPFLAGS": "-DHAVE_AS_CFI_PSEUDO_OP=0",
            "CCASFLAGS": "-Wa,--noexecstack",
        },
        "//conditions:default": {},
    }),
    lib_source = ":all_srcs",
    visibility = ["//visibility:public"],
    alwayslink = True,
)

# Use system libffi on macOS ARM64, build from source elsewhere
alias(
    name = "libffi",
    actual = select({
        "@bazel_tools//src/conditions:darwin_arm64": ":libffi_system",
        "//conditions:default": ":libffi_built",
    }),
    visibility = ["//visibility:public"],
)
