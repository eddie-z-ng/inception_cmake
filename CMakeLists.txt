cmake_minimum_required (VERSION 3.6.2)

SET(CMAKE_VERBOSE_MAKEFILE ON)
# EN: Build the Tensorflow Serving stubs. We build from here
#     instead of using add_subdirectory so that we don't need a
#     modified fork of the official tensorflow/serving repository.
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/serving_cmake ${CMAKE_MODULE_PATH})
set(SERVING_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/serving)

# EN: By default the protobuf compiler cmake submodule only works from within
#     the same directory. We get past this by providing an override to specify
#     the actual base directory to compile from.
set(BASE_PROTO_SOURCE_DIR_OVERRIDE ${SERVING_SOURCE_DIR})

find_package(Protobuf REQUIRED)
find_package(GRPC REQUIRED)
find_package(TensorflowCC REQUIRED COMPONENTS Shared)

set(TFS_PROTOS
  ${SERVING_SOURCE_DIR}/tensorflow_serving/apis/regression.proto
  ${SERVING_SOURCE_DIR}/tensorflow_serving/apis/inference.proto
  ${SERVING_SOURCE_DIR}/tensorflow_serving/apis/input.proto
  ${SERVING_SOURCE_DIR}/tensorflow_serving/apis/model.proto
  ${SERVING_SOURCE_DIR}/tensorflow_serving/apis/get_model_metadata.proto
  ${SERVING_SOURCE_DIR}/tensorflow_serving/apis/classification.proto
  ${SERVING_SOURCE_DIR}/tensorflow_serving/apis/predict.proto
  ${SERVING_SOURCE_DIR}/tensorflow_serving/apis/prediction_service.proto
)

set(PROTOBUF_IMPORT_DIRS
  ${SERVING_SOURCE_DIR}/tensorflow/
)

set(PROTO_SRC_DIR ${CMAKE_CURRENT_BINARY_DIR}/proto-src)
file(MAKE_DIRECTORY ${PROTO_SRC_DIR})
include_directories(${PROTO_SRC_DIR})

set(PROTOBUF_GENERATE_CPP_APPEND_PATH OFF)
set(GRPC_GENERATE_CPP_APPEND_PATH OFF)
PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS ${PROTO_SRC_DIR} ${TFS_PROTOS})
GRPC_GENERATE_CPP(GRPC_SRCS GRPC_HDRS ${PROTO_SRC_DIR} ${TFS_PROTOS})

add_library(tfserving SHARED ${PROTO_SRCS} ${GRPC_SRCS})
add_custom_target(tfserving_headers DEPENDS ${PROTO_HDRS} ${GRPC_HDRS})
add_dependencies(tfserving tfserving_headers)

target_include_directories(tfserving PUBLIC ${PROTO_SRC_DIR})
target_link_libraries(tfserving PUBLIC
  TensorflowCC::Shared
  gRPC::grpc++_reflection
  protobuf::libprotobuf
)

add_executable(inception_client
    ${CMAKE_CURRENT_SOURCE_DIR}/inception_client.cc
)
add_dependencies(inception_client
    tfserving_headers
)
target_link_libraries(inception_client
    tfserving
)
