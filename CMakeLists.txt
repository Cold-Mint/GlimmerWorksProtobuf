cmake_minimum_required(VERSION 4.2.2)
project(GlimmerWorksProto)

find_package(Protobuf REQUIRED)

file(GLOB_RECURSE PROTO_FILES
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.proto"
)

set(PROTO_OUTPUT_DIR "${CMAKE_BINARY_DIR}/cpp")

add_library(GlimmerWorksProto)

protobuf_generate(
        TARGET GlimmerWorksProto
        LANGUAGE cpp
        PROTOS ${PROTO_FILES}
        IMPORT_DIRS "${CMAKE_CURRENT_SOURCE_DIR}"
        PROTOC_OUT_DIR ${PROTO_OUTPUT_DIR}
)

target_link_libraries(GlimmerWorksProto
        PUBLIC protobuf::libprotobuf-lite
)

target_include_directories(GlimmerWorksProto
        PUBLIC ${PROTO_OUTPUT_DIR}
)

add_library(glimmerworks::proto ALIAS GlimmerWorksProto)