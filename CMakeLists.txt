cmake_minimum_required(VERSION 4.2.3)
project(GlimmerWorksProto)
include("versions.cmake")
include(FetchContent)
file(GLOB_RECURSE PROTO_FILES
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.proto"
)

set(PROTO_OUTPUT_DIR "${CMAKE_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${PROTO_OUTPUT_DIR})

if (ANDROID)
    FetchContent_Declare(
            pb
            GIT_REPOSITORY "https://github.com/protocolbuffers/protobuf"
            GIT_TAG ${PROTOCOLBUFFERS_PROTOBUF_VERSION}
            GIT_PROGRESS TRUE
    )
endif ()
set(protobuf_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_CSHARP OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_PYTHON OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_PROTOBUF_BINARIES ON CACHE BOOL "" FORCE)
set(protobuf_BUILD_LIBPROTOPBUF OFF CACHE BOOL "" FORCE)  # 关闭完整版
set(protobuf_BUILD_LIBPROTOBUF_LITE ON CACHE BOOL "" FORCE)  # 启用 Lite
FetchContent_MakeAvailable(pb)

add_library(GlimmerWorksProto)
if (ANDROID)
    include("${CMAKE_BINARY_DIR}/_deps/pb-src/cmake/protobuf-generate.cmake")
endif ()
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
