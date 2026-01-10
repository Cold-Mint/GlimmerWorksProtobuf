cmake_minimum_required(VERSION 3.28)
project(GlimmerWorksProto)
include("versions.cmake")
#Recursively scan all proto files.
#递归扫描所有的proto文件。
file(GLOB_RECURSE PROTO_FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.proto"
)

set(PROTO_OUTPUT_DIR "${CMAKE_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${PROTO_OUTPUT_DIR})

include(FetchContent)
FetchContent_Declare(
        pb
        GIT_REPOSITORY "https://github.com/protocolbuffers/protobuf"
        GIT_TAG ${PROTOCOLBUFFERS_PROTOBUF_VERSION}
        GIT_PROGRESS TRUE
)
set(protobuf_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_CSHARP OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_PYTHON OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_PROTOBUF_BINARIES ON CACHE BOOL "" FORCE)
set(protobuf_INSTALL OFF CACHE BOOL "" FORCE)
set(utf8_range_INSTALL OFF CACHE BOOL "" FORCE)
set(absl_INSTALL OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(pb)

add_library(GlimmerWorksProto)
include("${CMAKE_BINARY_DIR}/_deps/pb-build/cmake/protobuf/protobuf-generate.cmake")
protobuf_generate(
    TARGET GlimmerWorksProto
    LANGUAGE cpp
    PROTOS ${PROTO_FILES}
    IMPORT_DIRS "${CMAKE_CURRENT_SOURCE_DIR}"
    PROTOC_OUT_DIR "${PROTO_OUTPUT_DIR}"
)

target_link_libraries(GlimmerWorksProto
    PUBLIC protobuf::libprotobuf
)

target_include_directories(GlimmerWorksProto
    PUBLIC ${PROTO_OUTPUT_DIR}
)

#Export aliases externally.
#对外导出别名。
add_library(glimmerworks::proto ALIAS GlimmerWorksProto)