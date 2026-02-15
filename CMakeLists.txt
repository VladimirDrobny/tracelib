cmake_minimum_required(VERSION 3.22)
project(tracelib VERSION 1.0 LANGUAGES CXX DESCRIPTION "Library for representation and manipulation of performance traces")
include(GNUInstallDirs)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(INCLUDE_EXAMPLE_PROGRAMS "Include the example programs in building process." OFF)

set(ABSL_ENABLE_INSTALL ON CACHE BOOL "" FORCE)
set(ABSL_PROPAGATE_CXX_STD ON CACHE BOOL "" FORCE)
add_subdirectory(abseil-cpp)

# Directory structure
set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Find required packages
find_package(PkgConfig REQUIRED)
pkg_check_modules(zstd REQUIRED libzstd)
include_directories(${ZSTD_INCLUDE_DIR})

find_package(Boost REQUIRED NO_MODULE COMPONENTS serialization)
#find_package(zstd REQUIRED)
find_package(nlohmann_json REQUIRED)


# Add library target
add_library(${PROJECT_NAME} SHARED)
target_compile_options(${PROJECT_NAME} PRIVATE -Wno-unused-parameter -flto=auto -g -Wall -Wextra -O2 -fno-omit-frame-pointer -fno-optimize-sibling-calls)
target_sources(${PROJECT_NAME} PRIVATE
    ${SRC_DIR}/event.cpp
    ${SRC_DIR}/parser.cpp
    ${SRC_DIR}/serialization.cpp
    ${SRC_DIR}/perfFolded.cpp
    ${SRC_DIR}/perunPin.cpp
    ${SRC_DIR}/perunSystemTap.cpp
)
target_include_directories(${PROJECT_NAME} PUBLIC
        $<BUILD_INTERFACE:${INCLUDE_DIR}>
        $<INSTALL_INTERFACE:include>
)
set_target_properties(${PROJECT_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
)
target_link_libraries(${PROJECT_NAME}
        PUBLIC
        absl::flat_hash_map
        PRIVATE
        Boost::serialization
        nlohmann_json::nlohmann_json
        zstd
        ${ZSTD_LIBRARY}
)

install(TARGETS ${PROJECT_NAME} # EXPORT tracelibConfig
    ARCHIVE  DESTINATION lib
    LIBRARY  DESTINATION lib
    RUNTIME  DESTINATION bin
    INCLUDES DESTINATION include
)
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

if(INCLUDE_EXAMPLE_PROGRAMS)
    add_subdirectory(examples)
endif()
