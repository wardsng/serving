cmake_minimum_required (VERSION 3.6.2)
project(tfserving)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

find_package(Protobuf REQUIRED)
find_package(GRPC REQUIRED)
find_package(TensorflowCC REQUIRED COMPONENTS Shared)

SET(CMAKE_VERBOSE_MAKEFILE ON)

set(TFS_PROTOS
  tensorflow_serving/apis/regression.proto
  tensorflow_serving/apis/inference.proto
  tensorflow_serving/apis/input.proto
  tensorflow_serving/apis/model.proto
  tensorflow_serving/apis/get_model_metadata.proto
  tensorflow_serving/apis/classification.proto
  tensorflow_serving/apis/predict.proto
  tensorflow_serving/apis/prediction_service.proto
)

set(PROTOBUF_IMPORT_DIRS
  tensorflow/
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