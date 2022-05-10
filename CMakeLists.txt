# stop if CMake version is below 3.0
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)

# project name and supported languages
project(calculator VERSION 1.0.0 LANGUAGES CXX Fortran)

# we do not need it for this exercise but it is good to be
# able to use C++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# specify where to place binaries and libraries
include(GNUInstallDirs)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})

# process src/CMakeLists.txt
add_subdirectory(src)

# enable testing
enable_testing()        # <- we added this

# process test/CMakeLists.txt
add_subdirectory(test)  # <- we added this

