cmake_minimum_required(VERSION 3.5.1)

project(gRPC_tutorial C CXX)

## Additional cmake prefix path
list(APPEND CMAKE_PREFIX_PATH "/opt/pllee4/lib/cmake")

## Put all binary files into /bin
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}/bin)

add_subdirectory(src)