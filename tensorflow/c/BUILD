# Description:
# C API for TensorFlow, for use by client language bindings.

licenses(["notice"])  # Apache 2.0

load(
    "//tensorflow:tensorflow.bzl",
    "tf_cc_test",
    "tf_cuda_cc_test",
    "tf_copts",
    "tf_cuda_library",
    "tf_custom_op_library",
)

# -----------------------------------------------------------------------------
# Public targets

filegroup(
    name = "headers",
    srcs = [
        "c_api.h",
        "c_api_experimental.h",
    ],
    visibility = ["//tensorflow:__subpackages__"],
)

filegroup(
    name = "srcs",
    srcs = glob(
        [
            "*.cc",
            "*.h",
        ],
        exclude = [
            "c_api_experimental.cc",
            "c_api_experimental.h",
            "*test*",
        ],
    ),
    visibility = ["//visibility:public"],
)

tf_cuda_library(
    name = "c_api_internal",
    srcs = ["c_api.h"],
    hdrs = ["c_api_internal.h"],
    visibility = [
        "//tensorflow:internal",
        "//tensorflow/c:__subpackages__",
    ],
    deps = select({
        "//tensorflow:android": [
            "//tensorflow/core:android_tensorflow_lib_lite",
        ],
        "//conditions:default": [
            "//tensorflow/core:core_cpu",
            "//tensorflow/core:framework",
            "//tensorflow/core:lib",
            "//tensorflow/core:op_gen_lib",
        ],
    }),
)

tf_cuda_library(
    name = "c_api",
    srcs = [
        "c_api.cc",
        "c_api_function.cc",
    ],
    hdrs = [
        "c_api.h",
    ],
    copts = tf_copts(),
    visibility = ["//visibility:public"],
    deps = select({
        "//tensorflow:android": [
            ":c_api_internal",
            "//tensorflow/core:android_tensorflow_lib_lite",
        ],
        "//conditions:default": [
            ":c_api_internal",
            "//tensorflow/cc/saved_model:loader",
            "//tensorflow/cc:gradients",
            "//tensorflow/cc:ops",
            "//tensorflow/cc:grad_ops",
            "//tensorflow/cc:scope_internal",
            "//tensorflow/cc:while_loop",
            "//tensorflow/core:core_cpu",
            "//tensorflow/core:core_cpu_internal",
            "//tensorflow/core:framework",
            "//tensorflow/core:op_gen_lib",
            "//tensorflow/core:protos_all_cc",
            "//tensorflow/core:lib",
            "//tensorflow/core:lib_internal",
        ],
    }) + select({
        "//tensorflow:with_xla_support": [
            "//tensorflow/compiler/tf2xla:xla_compiler",
            "//tensorflow/compiler/jit",
        ],
        "//conditions:default": [],
    }),
)

tf_cuda_library(
    name = "c_api_experimental",
    srcs = [
        "c_api_experimental.cc",
    ],
    hdrs = [
        "c_api_experimental.h",
    ],
    copts = tf_copts(),
    visibility = ["//visibility:public"],
    deps = [
        ":c_api",
        ":c_api_internal",
        "//tensorflow/compiler/jit/legacy_flags:mark_for_compilation_pass_flags",
        "//tensorflow/core:protos_all_cc",
    ],
)

exports_files(
    [
        "version_script.lds",
        "exported_symbols.lds",
    ],
    visibility = ["//visibility:public"],
)

tf_cuda_library(
    name = "tf_status_helper",
    srcs = ["tf_status_helper.cc"],
    hdrs = ["tf_status_helper.h"],
    visibility = ["//visibility:public"],
    deps = [
        ":c_api",
        ":c_api_internal",
        "//tensorflow/core:lib",
    ],
)

tf_cuda_library(
    name = "checkpoint_reader",
    srcs = ["checkpoint_reader.cc"],
    hdrs = ["checkpoint_reader.h"],
    visibility = ["//visibility:public"],
    deps = [
        ":tf_status_helper",
        "//tensorflow/core:framework",
        "//tensorflow/core:lib",
        "//tensorflow/core/util/tensor_bundle",
    ],
)

# -----------------------------------------------------------------------------
# Tests

tf_cuda_library(
    name = "c_test_util",
    testonly = 1,
    srcs = ["c_test_util.cc"],
    hdrs = ["c_test_util.h"],
    visibility = [
        "//learning/brain:__subpackages__",
        "//tensorflow:__subpackages__",
    ],
    deps = [
        ":c_api",
        ":c_api_experimental",
        "//tensorflow/core:lib",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core:session_options",
        "//tensorflow/core:test",
    ],
)

tf_cuda_cc_test(
    name = "c_api_test",
    size = "small",
    srcs = ["c_api_test.cc"],
    data = [
        ":test_op.so",
        "//tensorflow/cc/saved_model:saved_model_half_plus_two",
    ],
    linkopts = select({
        "//tensorflow:darwin": ["-headerpad_max_install_names"],
        "//conditions:default": [],
    }),
    # We must ensure that the dependencies can be dynamically linked since
    # the shared library must be able to use core:framework.
    # linkstatic = tf_kernel_tests_linkstatic(),
    deps = [
        ":c_api",
        ":c_test_util",
        "//tensorflow/cc:cc_ops",
        "//tensorflow/cc:grad_ops",
        "//tensorflow/cc/saved_model:signature_constants",
        "//tensorflow/cc/saved_model:tag_constants",
        "//tensorflow/core:core_cpu_internal",
        "//tensorflow/core:direct_session",
        "//tensorflow/core:framework",
        "//tensorflow/core:framework_internal",
        "//tensorflow/core:lib",
        "//tensorflow/core:proto_text",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core/kernels:array",
        "//tensorflow/core/kernels:control_flow_ops",
        "//tensorflow/core/kernels:math",
    ],
)

tf_cc_test(
    name = "c_api_function_test",
    size = "small",
    srcs = ["c_api_function_test.cc"],
    deps = [
        ":c_api",
        ":c_test_util",
        "//tensorflow/core:lib",
        "//tensorflow/core:lib_internal",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
    ],
)

tf_cc_test(
    name = "while_loop_test",
    size = "small",
    srcs = ["while_loop_test.cc"],
    deps = [
        ":c_api",
        ":c_test_util",
        "//tensorflow/core:lib",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
    ],
)

tf_custom_op_library(
    name = "test_op.so",
    srcs = ["test_op.cc"],
)

# -----------------------------------------------------------------------------
# Python API target

tf_cuda_library(
    name = "python_api",
    srcs = ["python_api.cc"],
    hdrs = ["python_api.h"],
    visibility = ["//tensorflow/python:__pkg__"],
    deps = [
        ":c_api",
        ":c_api_internal",
    ],
)

# -----------------------------------------------------------------------------
# Google-internal targets.

filegroup(
    name = "all_files",
    srcs = glob(
        ["**/*"],
        exclude = [
            "**/METADATA",
            "**/OWNERS",
        ],
    ),
    visibility = ["//tensorflow:__subpackages__"],
)
