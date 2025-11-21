cmake_minimum_required(VERSION 3.20)
project(packer CXX)

# Runtime statique pour éviter les dépendances DLL
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

# Collecter les fichiers sources du packer
file(GLOB_RECURSE SRC_FILES ${PROJECT_SOURCE_DIR}/src/*.cpp)
file(GLOB_RECURSE HDR_FILES ${PROJECT_SOURCE_DIR}/src/*.hpp)

# Ajouter les sous-projets
add_subdirectory(${PROJECT_SOURCE_DIR}/stub)
add_subdirectory(${PROJECT_SOURCE_DIR}/dummy)

# Générer les fichiers de ressources pour inclure le stub
file(GENERATE OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/$<CONFIG>/stub.hpp"
     CONTENT "#pragma once\n#define IDB_STUB 1000\n")

file(GENERATE OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/$<CONFIG>/stub.rc"
     CONTENT "#include <winresrc.h>\n#include \"stub.hpp\"\nIDB_STUB STUB \"$<TARGET_FILE:stub>\"\n")

# Créer l'exécutable packer
add_executable(packer ${HDR_FILES} ${SRC_FILES})
target_sources(packer PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/$<CONFIG>/stub.rc")

# Includes
target_include_directories(packer PUBLIC
    "${PROJECT_SOURCE_DIR}/src"
    "${CMAKE_CURRENT_BINARY_DIR}/$<CONFIG>"
)

# Dépendances : stub doit être compilé avant packer
add_dependencies(packer stub)
add_dependencies(dummy packer)

# Tests
enable_testing()
add_test(NAME test_pack COMMAND "$<TARGET_FILE:packer>" "$<TARGET_FILE:dummy>")
add_test(NAME test_unpack COMMAND "packed.exe")
