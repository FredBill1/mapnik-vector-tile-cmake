cmake_minimum_required(VERSION 3.15)
project(mapnik-vector-tile)
option(ENABLE_SSE_MATH "Enable SSE math optimization" OFF)
message(STATUS "ENABLE_SSE_MATH: ${ENABLE_SSE_MATH}")

if(ENABLE_SSE_MATH)
    add_definitions(-DSSE_MATH=)
endif()

find_package(mapnik CONFIG REQUIRED)
find_path(PROTOZERO_INCLUDE_DIRS "protozero/basic_pbf_builder.hpp")
find_package(Protobuf CONFIG REQUIRED)
find_path(MAPBOX_WAGYU_INCLUDE_DIRS "mapbox/geometry/wagyu/active_bound_list.hpp")

protobuf_generate_cpp(PROTOSRCS PROTOHDRS
    proto/vector_tile.proto
)
add_library(mapnik-vector-tile-proto STATIC ${PROTOSRCS} ${PROTOSHDRS})
target_link_libraries(mapnik-vector-tile-proto PUBLIC ${Protobuf_LIBRARIES})
target_include_directories(mapnik-vector-tile-proto PUBLIC ${CMAKE_CURRENT_BINARY_DIR} ${Protobuf_INCLUDE_DIRS})

file(GLOB SRC_FILES src/*.cpp)
add_library(mapnik-vector-tile STATIC
    ${SRC_FILES}
    ${PROTOSRCS}
    ${PROTOSHDRS}
)

target_link_libraries(mapnik-vector-tile PUBLIC
    mapnik-vector-tile-proto
    mapnik::mapnik mapnik::json mapnik::wkt
    ${Protobuf_LIBRARIES}
)
target_include_directories(mapnik-vector-tile
    PUBLIC
    ${PROTOZERO_INCLUDE_DIRS}
    ${MAPBOX_WAGYU_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)
target_compile_definitions(mapnik-vector-tile
    PUBLIC
    _USE_MATH_DEFINES
    MAPNIK_VECTOR_TILE_LIBRARY=1
)