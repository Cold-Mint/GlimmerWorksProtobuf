cmake_minimum_required(VERSION 3.28)
project(GlimmerWorksProtobuf)

find_package(Protobuf REQUIRED)

#Recursively scan all proto files.
#递归扫描所有的proto文件。
file(GLOB_RECURSE PROTO_FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/*.proto"
)

set(PROTO_OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${PROTO_OUTPUT_DIR})

add_library(glimmerworks_proto)

foreach(proto_file IN LISTS PROTO_FILES)
    file(RELATIVE_PATH rel_path "${CMAKE_CURRENT_SOURCE_DIR}" "${proto_file}")
    get_filename_component(proto_dir "${rel_path}" DIRECTORY)

    protobuf_generate(
        TARGET glimmerworks_proto
        LANGUAGE cpp
        PROTOS "${proto_file}"
        IMPORT_DIRS "${PROTO_OUTPUT_DIR}"
        PROTOC_OUT_DIR "${PROTO_OUTPUT_DIR}"
    )
endforeach()

target_link_libraries(glimmerworks_proto
    PUBLIC protobuf::libprotobuf
)

target_include_directories(glimmerworks_proto
    PUBLIC ${PROTO_OUTPUT_DIR}
)
