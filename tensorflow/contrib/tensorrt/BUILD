# Description:
#   Wrap NVIDIA TensorRT (http://developer.nvidia.com/tensorrt) with tensorflow
#   and provide TensorRT operators and converter package.
#   APIs are meant to change over time.

package(default_visibility = ["//visibility:public"])

licenses(["notice"])  # Apache 2.0

exports_files(["LICENSE"])

load(
    "//tensorflow:tensorflow.bzl",
    "tf_copts",
    "tf_cuda_library",
    "tf_custom_op_library_additional_deps",
)
load("//tensorflow:tensorflow.bzl", "cuda_py_test")
load("//tensorflow:tensorflow.bzl", "cuda_py_tests")
load("//tensorflow:tensorflow.bzl", "tf_py_wrap_cc")
load(
    "@local_config_tensorrt//:build_defs.bzl",
    "if_tensorrt",
)

exports_files(glob([
    "test/testdata/*",
]))

tf_cuda_library(
    name = "trt_shape_function",
    srcs = ["shape_fn/trt_shfn.cc"],
    hdrs = ["shape_fn/trt_shfn.h"],
    visibility = ["//visibility:public"],
    deps = [
        "//tensorflow/compiler/tf2tensorrt:trt_logging",
        "//tensorflow/compiler/tf2tensorrt:trt_plugins",
    ] + if_tensorrt([
        "@local_config_tensorrt//:tensorrt",
    ]) + tf_custom_op_library_additional_deps(),
)

py_library(
    name = "init_py",
    srcs = [
        "__init__.py",
        "python/__init__.py",
    ],
    srcs_version = "PY2AND3",
    deps = [
        ":tf_trt_integration_test_base",
        ":trt_convert_py",
        ":trt_ops_py",
        "//tensorflow/python:errors",
    ],
)

py_library(
    name = "trt_ops_py",
    srcs_version = "PY2AND3",
    deps = [
        "//tensorflow/compiler/tf2tensorrt:trt_ops",
        "//tensorflow/compiler/tf2tensorrt:trt_ops_loader",
    ],
)

py_library(
    name = "trt_convert_py",
    srcs = ["python/trt_convert.py"],
    srcs_version = "PY2AND3",
    deps = [
        ":wrap_conversion",
        "//tensorflow/python:graph_util",
        "//tensorflow/python:session",
        "//tensorflow/python:tf_optimizer",
        "//tensorflow/python/saved_model:builder",
        "//tensorflow/python/saved_model:loader",
        "//tensorflow/python/saved_model:tag_constants",
    ],
)

# TODO(aaroey): this wrapper has been causing troubles of double linking, so
# either get rid of it, or split to make it contain minimum dependencies.
tf_py_wrap_cc(
    name = "wrap_conversion",
    srcs = ["trt_conversion.i"],
    copts = tf_copts(),
    swig_includes = [
        "//tensorflow/python:platform/base.i",
    ],
    deps = [
        "//tensorflow/compiler/tf2tensorrt:test_utils",
        "//tensorflow/compiler/tf2tensorrt:trt_conversion",
        "//tensorflow/compiler/tf2tensorrt:trt_op_kernels",
        "//third_party/python_runtime:headers",
    ],
)

py_library(
    name = "tf_trt_integration_test_base",
    srcs = ["test/tf_trt_integration_test_base.py"],
    deps = [
        ":trt_convert_py",
        ":trt_ops_py",
        "//tensorflow/python:client_testlib",
        "//tensorflow/python:framework_test_lib",
    ],
)

cuda_py_test(
    name = "trt_convert_test",
    srcs = ["python/trt_convert_test.py"],
    additional_deps = [
        ":trt_convert_py",
        ":trt_ops_py",
        "//tensorflow/python:client_testlib",
        "//tensorflow/python:framework_test_lib",
        "//tensorflow/python:graph_util",
        "//tensorflow/python/saved_model:builder",
        "//tensorflow/python/saved_model:loader",
        "//tensorflow/python/saved_model:signature_constants",
        "//tensorflow/python/saved_model:signature_def_utils",
        "//tensorflow/python/saved_model:tag_constants",
        "//tensorflow/python/saved_model:utils",
        "//tensorflow/python/tools:freeze_graph_lib",
        "//tensorflow/python/tools:saved_model_utils",
    ],
    tags = [
        "no_cuda_on_cpu_tap",
        "no_windows",
        "nomac",
    ],
)

cuda_py_tests(
    name = "tf_trt_integration_test",
    srcs = [
        "test/base_test.py",
        "test/batch_matmul_test.py",
        "test/biasadd_matmul_test.py",
        "test/binary_tensor_weight_broadcast_test.py",
        "test/concatenation_test.py",
        "test/const_broadcast_test.py",
        "test/conv2d_test.py",
        "test/dynamic_input_shapes_test.py",
        "test/identity_output_test.py",
        "test/int32_test.py",
        "test/lru_cache_test.py",
        "test/memory_alignment_test.py",
        "test/multi_connection_neighbor_engine_test.py",
        "test/neighboring_engine_test.py",
        "test/quantization_test.py",
        "test/rank_two_test.py",
        "test/reshape_transpose_test.py",
        "test/topk_test.py",
        "test/unary_test.py",
        "test/vgg_block_nchw_test.py",
        "test/vgg_block_test.py",
    ],
    additional_deps = [
        ":tf_trt_integration_test_base",
        "//tensorflow/python:client_testlib",
        "//tensorflow/python:framework_test_lib",
    ],
    tags = [
        "no_cuda_on_cpu_tap",
        "no_windows",
        "nomac",
    ],
)

cuda_py_test(
    name = "quantization_mnist_test",
    srcs = ["test/quantization_mnist_test.py"],
    additional_deps = [
        ":tf_trt_integration_test_base",
        "//tensorflow/python:client_testlib",
        "//tensorflow/python:framework_test_lib",
        "//tensorflow/python/keras:keras",
        "//tensorflow/python/estimator:estimator",
    ],
    data = [
        "test/testdata/checkpoint",
        "test/testdata/model.ckpt-46900.data-00000-of-00001",
        "test/testdata/model.ckpt-46900.index",
    ],
    tags = [
        "no_cuda_on_cpu_tap",
        "no_pip",
        "no_tap",  # It is not able to download the mnist data.
        "no_windows",
        "nomac",
    ],
)

# The following rules forward the libraries that were moved in order to not
# break other internal targets.

alias(
    name = "trt_conversion",
    actual = "//tensorflow/compiler/tf2tensorrt:trt_conversion",
)

alias(
    name = "trt_op_kernels",
    actual = "//tensorflow/compiler/tf2tensorrt:trt_op_kernels",
)

alias(
    name = "trt_engine_op_op_lib",
    actual = "//tensorflow/compiler/tf2tensorrt:trt_engine_op_op_lib",
)
