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
    srcs = ["//cmake/patches/stubs:gemma_model_constraint_provider.cc"],
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
        # ============================================================
        # Lillup -- POC D-6 (a8v3) — sous-packages absl manquants
        # v3 : validation empirique upstream absl/BUILD.bazel master
        # Retrait : str_format_internal, log_internal, low_level_alloc (inexistants)
        # Réajout : base_internal, raw_logging_internal (existants, retirés par erreur v2)
        # H-6 empirique : lillup_xcfw_deps racine ne ramène pas
        #   base_internal, container_internal, cord_internal, log_internal,
        #   str_format_internal, stacktrace, symbolize, crc32c
        # Mesure diff undefined D-6 : 205 absl + 526 protobuf + 47 spm
        # ============================================================
        "@com_google_absl//absl/base:base_internal",
        "@com_google_absl//absl/base:raw_logging_internal",
        "@com_google_absl//absl/strings:internal",
        "@com_google_absl//absl/strings:cord_internal",
        "@com_google_absl//absl/log",
        "@com_google_absl//absl/log:initialize",
        "@com_google_absl//absl/debugging:stacktrace",
        "@com_google_absl//absl/debugging:symbolize",
        "@com_google_absl//absl/container:hashtablez_sampler",
        "@com_google_absl//absl/container:raw_hash_set",
        "@com_google_absl//absl/crc:crc32c",
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
    avoid_deps = [
        "//runtime/components/constrained_decoding:gemma_model_constraint_provider_shared_lib",
        "//runtime/components/constrained_decoding:gemma_model_constraint_provider_lib",
    ],
    visibility = ["//visibility:public"],
)

# ============================================================
# Lillup -- POC (a11) DIAGNOSTIC : 2e xcframework deps-only
# D-6 Jalon 5d, Voie 1 pattern MediaPipe (2 xcframeworks)
# OBJECTIF : valider empirique que apple_static_xcframework
#   accepte deps = [lillup_xcfw_deps] et produit un xcframework
#   contenant les .o absl/protobuf/sentencepiece/zlib dans le slice
#   ios_arm64. NE REMPLACE PAS LiteRTLM_xcframework (co-existe).
# PATTERN : équivalent MediaPipeTaskGraphs_library de Google MediaPipe
#   mediapipe_static_xcframework(
#     name = "MediaPipeTaskGraphs_library",
#     bundle_name = "MediaPipeTaskGraphs_library",
#     deps = [":MediaPipeTaskGraphs"],
#   )
# Volontairement minimal : pas de public_hdrs, pas d'avoid_deps.
# ============================================================
apple_static_xcframework(
    name = "LiteRTLM_deps_xcframework",
    bundle_name = "LiteRTLM_deps",
    minimum_os_versions = {"ios": "15.0"},
    ios = {
        "device": ["arm64"],
        "simulator": ["arm64"],
    },
    deps = [":lillup_xcfw_deps"],
    visibility = ["//visibility:public"],
)
