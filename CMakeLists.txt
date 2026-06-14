cmake_minimum_required(VERSION 3.31.6)
project(GlimmerWorksProto)
include("${CMAKE_SOURCE_DIR}/versions.cmake")
include(FetchContent)
FetchContent_Declare(
        protobuf
        GIT_REPOSITORY "https://github.com/protocolbuffers/protobuf"
        GIT_TAG ${PROTOCOLBUFFERS_PROTOBUF_VERSION}
        GIT_PROGRESS TRUE
        OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(protobuf)

file(GLOB_RECURSE PROTO_FILES
        "${CMAKE_CURRENT_SOURCE_DIR}/src/*.proto"
)

set(PROTO_OUTPUT_DIR "${CMAKE_BINARY_DIR}/_deps/glimmerworksprotobuf-build/cpp")

set(PROTOC_EXECUTABLE_PATH "" CACHE FILEPATH
        "Full path to the host protoc executable (required for cross-compilation)"
)

set(PROTOC_EXECUTABLE "")
if (CMAKE_CROSSCOMPILING)
    # If it is a cross-compilation, then the user must specify the protoc directory.
    # 如果是交叉编译，那么必须得由用户指定protoc
    message(STATUS "Cross-compilation mode Host protoc must be specified manually.")
    if (PROTOC_EXECUTABLE_PATH STREQUAL "")
        message(FATAL_ERROR
                "Manual specification of protoc is required for cross-compilation!\n"
                "Please use the argument: -DPROTOC_EXECUTABLE_PATH=/path/to/your/host/protoc"
        )
    endif ()
    set(PROTOC_EXECUTABLE ${PROTOC_EXECUTABLE_PATH})
else ()
    if(WIN32)
        set(PROTOC_EXECUTABLE "${CMAKE_BINARY_DIR}/bin/$<CONFIG>/protoc.exe")
    else()
        set(PROTOC_EXECUTABLE "${CMAKE_BINARY_DIR}/_deps/protobuf-build/protoc")
    endif()
endif ()
message(STATUS "PROTOC_EXECUTABLE = ${PROTOC_EXECUTABLE}")

add_library(GlimmerWorksProto)
include("${CMAKE_BINARY_DIR}/_deps/protobuf-src/cmake/protobuf-generate.cmake")
protobuf_generate(
        TARGET GlimmerWorksProto
        LANGUAGE cpp
        PROTOS ${PROTO_FILES}
        IMPORT_DIRS "${CMAKE_CURRENT_SOURCE_DIR}"
        PROTOC_OUT_DIR ${PROTO_OUTPUT_DIR}
        PROTOC_EXE ${PROTOC_EXECUTABLE}
)

add_dependencies(GlimmerWorksProto protobuf::protoc)

# Link our compiled Protobuf as a dependency library
# 链接我们编译的Protobuf作为依赖库
target_link_libraries(GlimmerWorksProto
        PUBLIC protobuf::libprotobuf-lite
)

target_include_directories(GlimmerWorksProto
        PUBLIC ${PROTO_OUTPUT_DIR}
)

add_library(glimmerworks::proto ALIAS GlimmerWorksProto)
