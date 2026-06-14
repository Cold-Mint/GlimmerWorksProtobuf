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

# CMAKE_SOURCE_DIR and CMAKE_BINARY_DIR always point to the root project directories
# regardless of which subproject is currently being configured
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
    # CMAKE_BINARY_DIR points to the root project's build directory
    # For multi-config generators (like Visual Studio), protoc is in bin/<Config>/ subdirectory
    # For single-config generators (like Ninja), protoc is in _deps/protobuf-build/
    if(WIN32)
        set(PROTOC_NAME "protoc.exe")
    else()
        set(PROTOC_NAME "protoc")
    endif()
    
    # Try multi-config path first (for Visual Studio)
    set(PROTOC_EXECUTABLE "${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/${PROTOC_NAME}")
    if(NOT EXISTS "${PROTOC_EXECUTABLE}")
        # Fall back to single-config path
        set(PROTOC_EXECUTABLE "${CMAKE_BINARY_DIR}/_deps/protobuf-build/${PROTOC_NAME}")
    endif()
    
    # Get absolute path
    get_filename_component(PROTOC_EXECUTABLE "${PROTOC_EXECUTABLE}" ABSOLUTE)
endif ()
message(STATUS "PROTOC_EXECUTABLE = ${PROTOC_EXECUTABLE}")

add_library(GlimmerWorksProto)

# Use custom command to generate protobuf source files
# 使用自定义命令生成 protobuf 源文件
set(_generated_srcs_all)
foreach(_proto ${PROTO_FILES})
    get_filename_component(_abs_file ${_proto} ABSOLUTE)
    get_filename_component(_abs_dir ${_abs_file} DIRECTORY)
    get_filename_component(_file_full_name ${_proto} NAME)
    string(FIND "${_file_full_name}" "." _file_last_ext_pos REVERSE)
    string(SUBSTRING "${_file_full_name}" 0 ${_file_last_ext_pos} _basename)
    file(RELATIVE_PATH _rel_dir "${CMAKE_CURRENT_SOURCE_DIR}/src" ${_abs_dir})
    set(_generated_srcs
        "${PROTO_OUTPUT_DIR}/${_rel_dir}/${_basename}.pb.cc"
        "${PROTO_OUTPUT_DIR}/${_rel_dir}/${_basename}.pb.h"
    )
    list(APPEND _generated_srcs_all ${_generated_srcs})
    add_custom_command(
        OUTPUT ${_generated_srcs}
        COMMAND ${PROTOC_EXECUTABLE}
        ARGS --cpp_out=${PROTO_OUTPUT_DIR} -I${CMAKE_CURRENT_SOURCE_DIR} ${_abs_file}
        DEPENDS ${_abs_file} protobuf::protoc
        COMMENT "Running C++ protocol buffer compiler on ${_proto}"
        VERBATIM
    )
endforeach()
set_source_files_properties(${_generated_srcs_all} PROPERTIES GENERATED TRUE)
target_sources(GlimmerWorksProto PRIVATE ${_generated_srcs_all})

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
