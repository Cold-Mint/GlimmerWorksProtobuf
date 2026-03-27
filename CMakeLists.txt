cmake_minimum_required(VERSION 4.2.3)
project(GlimmerWorksProto)
include("versions.cmake")
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
set(protobuf_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_CONFORMANCE OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_LIBUPB OFF CACHE BOOL "" FORCE) 
set(protobuf_BUILD_LIBPROTOBUF OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_LIBPROTOBUF_LITE ON CACHE BOOL "" FORCE)
set(protobuf_BUILD_PROTOC_BINARIES ON CACHE BOOL "" FORCE)
set(protobuf_BUILD_LIBPROTOC OFF CACHE BOOL "" FORCE)
#language
set(protobuf_BUILD_CSHARP OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_PYTHON OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_JAVA OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_GO OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_RUST OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_OBJECTIVE_C OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_PHP OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_LUA OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_RUBY OFF CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(protobuf)

add_library(GlimmerWorksProto)
include("${CMAKE_BINARY_DIR}/_deps/protobuf-src/cmake/protobuf-generate.cmake")
protobuf_generate(
        TARGET GlimmerWorksProto
        LANGUAGE cpp
        PROTOS ${PROTO_FILES}
        IMPORT_DIRS "${CMAKE_CURRENT_SOURCE_DIR}"
        PROTOC_OUT_DIR "${PROTO_OUTPUT_DIR}"
        LITE_MODE ON
)

target_link_libraries(GlimmerWorksProto
        PUBLIC protobuf::libprotobuf-lite
)

target_include_directories(GlimmerWorksProto
        PUBLIC ${PROTO_OUTPUT_DIR}
)

add_library(glimmerworks::proto ALIAS GlimmerWorksProto)
