# Copyright 2026 The ODML Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# [Google-internal load of `cc_library`]
# [Google-internal load of `cc_test`]

package(
    default_hdrs_check = "strict",
    default_visibility = [
        "//visibility:public",
    ],
)

licenses(["notice"])

ENGINE_COMMON_DEPS = [
    "@com_google_absl//absl/base:core_headers",
    "@com_google_absl//absl/base:log_severity",
    "@com_google_absl//absl/functional:any_invocable",
    "@com_google_absl//absl/log:absl_log",
    "@com_google_absl//absl/log:globals",
    "@com_google_absl//absl/log:initialize",
    "@com_google_absl//absl/log:log_sink_registry",
    "@com_google_absl//absl/status",
    "@com_google_absl//absl/status:statusor",
    "@com_google_absl//absl/strings:str_format",
    "@com_google_absl//absl/strings:string_view",
    "@com_google_absl//absl/time",
    "@nlohmann_json//:json",
    "@litert//litert/c/internal:litert_logging",
    "//runtime/conversation",
    "//runtime/conversation:io_types",
    "//runtime/engine:engine_factory",
    "//runtime/engine:engine_interface",
    "//runtime/engine:engine_settings",
    "//runtime/engine:io_types",
    "//runtime/executor:executor_settings_base",
    "//runtime/executor:llm_executor_settings",
    "//runtime/proto:engine_cc_proto",
    "//runtime/proto:sampler_params_cc_proto",
]

cc_library(
    name = "engine",
    srcs = [
        "engine.cc",
        "litert_lm_logging.cc",
    ],
    hdrs = [
        "engine.h",
        "litert_lm_logging.h",
    ],
    visibility = ["//visibility:public"],
    deps = ENGINE_COMMON_DEPS + [
        "//runtime/core:engine_impl",
    ],
)

cc_library(
    name = "engine_cpu",
    srcs = [
        "engine.cc",
        "litert_lm_logging.cc",
    ],
    hdrs = [
        "engine.h",
        "litert_lm_logging.h",
    ],
    visibility = ["//visibility:public"],
    deps = ENGINE_COMMON_DEPS + [
        "//runtime/core:engine_impl_cpu_only",
    ],
)

cc_test(
    name = "engine_test",
    srcs = ["engine_test.cc"],
    data = [
        "//runtime/testdata",
    ],
    tags = ["requires-mac-inputs:hard"],
    target_compatible_with = select({
        "@platforms//os:ios": [],
        "@platforms//os:linux": [],
        "@platforms//os:osx": [],
        "@platforms//os:windows": [],
        "//conditions:default": ["@platforms//:incompatible"],
    }),
    deps = [
        ":engine",
        "@com_google_googletest//:gtest_main",
        "@com_google_absl//absl/status",
        "@com_google_absl//absl/status:status_matchers",
        "@com_google_absl//absl/synchronization",
        "@nlohmann_json//:json",
        "//runtime/conversation",
        "//runtime/conversation:io_types",
        "//runtime/engine:engine_settings",
        "//runtime/executor:executor_settings_base",
        "//runtime/executor:llm_executor_settings",
    ],
)

# ============================================================
# Lillup -- deps manquantes pour le xcframework iOS
# Root cause : apple_static_xcframework ne compile pas les archives
#   absl/status, absl/synchronization, absl/strings, absl/time,
#   protobuf_lite, protobuf, sentencepiece_processor
#   car ces targets ont alwayslink=False dans le graphe Bazel.
# Prouve : absents des 526 archives bazel-out/ -- diag E2.1
# Solution [BZ-02 + rules_ios#1938] : alwayslink=True force
#   l inclusion de toutes les deps dans l archive xcframework.
# Sources :
#   @com_google_absl : WORKSPACE global [G-05]
#   @com_google_protobuf : tf_workspace2() WORKSPACE l.182
#   @sentencepiece : BUILD.sentencepiece l.24
# ============================================================
cc_library(
    name = "lillup_xcfw_deps",
    deps = [
        "@com_google_absl//absl/base",
        "@com_google_absl//absl/debugging:leak_check",
        "@com_google_absl//absl/status",
        "@com_google_absl//absl/status:statusor",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/strings:cord",
        "@com_google_absl//absl/strings:str_format",
        "@com_google_absl//absl/synchronization",
        "@com_google_absl//absl/time",
        "@com_google_protobuf//:protobuf_lite",
        "@com_google_protobuf//:protobuf",
        "@sentencepiece//:sentencepiece_processor",
        "@zlib//:zlib",
        "@flatbuffers//:runtime_cc",
    ],
    alwayslink = True,
    visibility = ["//visibility:public"],
)

# ============================================================
# Lillup -- xcframework iOS natif Bazel
# Source : rules_apple 3.22.0 [WORKSPACE ligne 49]
# apple_support 1.23.1 [WORKSPACE ligne 70]
# Approuve panel auditeurs -- Architecture v2.0 -- 13 avril 2026
# ============================================================
load(
    "@build_bazel_rules_apple//apple:apple.bzl",
    "apple_static_xcframework",
)

apple_static_xcframework(
    name = "LiteRTLM_xcframework",
    bundle_name = "LiteRTLM",
    public_hdrs = ["engine.h"],
    minimum_os_versions = {"ios": "15.0"},
    ios = {
        "device": ["arm64"],
        "simulator": ["arm64"],
    },
    deps = [":engine_cpu", ":lillup_xcfw_deps"],
    # avoid_deps retire (PATCH-L4-XCFW-RUN24500353459-V1) :
    # Le stub cmake/patches/stubs/BUILD (session 1 commit c951d50) emet
    # les symboles gemma_model_constraint_provider_* en object code via
    # runtime/components/constrained_decoding/BUILD srcs iOS selects.
    # Ces symboles sont compatibles libtool (pas des dylibs LFS).
    visibility = ["//visibility:public"],
)
