cmake_minimum_required(VERSION 4.2.3)
project(GlimmerWorksProto)
include("../../../versions.cmake")
include(FetchContent)
file(GLOB_RECURSE PROTO_FILES
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.proto"
)

set(PROTO_OUTPUT_DIR "${CMAKE_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${PROTO_OUTPUT_DIR})

FetchContent_Declare(
    protobuf
    GIT_REPOSITORY "https://github.com/protocolbuffers/protobuf"
    GIT_TAG ${PROTOCOLBUFFERS_PROTOBUF_VERSION}
    GIT_PROGRESS TRUE
)

FetchContent_MakeAvailable(protobuf)

add_library(GlimmerWorksProto)
include("${CMAKE_BINARY_DIR}/_deps/protobuf-src/cmake/protobuf-generate.cmake")
protobuf_generate(
        TARGET GlimmerWorksProto
        LANGUAGE cpp
        PROTOS ${PROTO_FILES}
        IMPORT_DIRS "${CMAKE_CURRENT_SOURCE_DIR}"
        PROTOC_OUT_DIR "${PROTO_OUTPUT_DIR}"
)

target_link_libraries(GlimmerWorksProto
        PUBLIC protobuf::libprotobuf-lite
)

target_include_directories(GlimmerWorksProto
        PUBLIC ${PROTO_OUTPUT_DIR}
)

add_library(glimmerworks::proto ALIAS GlimmerWorksProto)
