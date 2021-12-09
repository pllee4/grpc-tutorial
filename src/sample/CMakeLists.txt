# Find Protobuf installation
# Looks for protobuf-config.cmake file installed by Protobuf's cmake installation.
set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(Protobuf CONFIG REQUIRED)
message(STATUS "Using protobuf ${Protobuf_VERSION}")

set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
set(_REFLECTION gRPC::grpc++_reflection)
if(CMAKE_CROSSCOMPILING)
    find_program(_PROTOBUF_PROTOC protoc)
else()
    set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)
endif()

# Find gRPC installation
# Looks for gRPCConfig.cmake file installed by gRPC's cmake installation.
find_package(gRPC CONFIG REQUIRED)
message(STATUS "Using gRPC ${gRPC_VERSION}")

set(_GRPC_GRPCPP gRPC::grpc++)
if(CMAKE_CROSSCOMPILING)
    find_program(_GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)
else()
    set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:gRPC::grpc_cpp_plugin>)
endif()

file(TO_NATIVE_PATH ${CMAKE_CURRENT_LIST_DIR}/protos PROTO_SRC_DIR)
file(TO_NATIVE_PATH ${CMAKE_CURRENT_LIST_DIR}/include PROTO_GEN_DIR)

file(GLOB_RECURSE PROTO_FILES "${PROTO_SRC_DIR}/*.proto")
foreach(proto ${PROTO_FILES})
  file(TO_NATIVE_PATH ${proto} proto_file)
  message(STATUS "Processing ${proto_file}")

  get_filename_component(PROTO_FILE_NAME ${proto_file} NAME_WE)
  set(gen_proto_srcs "${PROTO_GEN_DIR}/${PROTO_FILE_NAME}.pb.cc")
  set(gen_proto_hdrs "${PROTO_GEN_DIR}/${PROTO_FILE_NAME}.pb.h")
  set(gen_grpc_srcs "${PROTO_GEN_DIR}/${PROTO_FILE_NAME}.grpc.pb.cc")
  set(gen_grpc_hdrs "${PROTO_GEN_DIR}/${PROTO_FILE_NAME}.grpc.pb.h")

  list(APPEND ALL_PROTO_SRCS ${gen_proto_srcs})
  list(APPEND ALL_GRPC_SRCS ${gen_grpc_srcs})

  # Generated sources
  add_custom_command(
      OUTPUT "${gen_proto_srcs}" "${gen_proto_hdrs}" "${gen_grpc_srcs}" "${gen_grpc_hdrs}"
      COMMAND ${_PROTOBUF_PROTOC}
      ARGS --grpc_out "${PROTO_GEN_DIR}"
        --cpp_out "${PROTO_GEN_DIR}"
        -I "${PROTO_SRC_DIR}"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${proto_file}"
      DEPENDS "${proto_file}")

endforeach(proto)

add_executable(greeter_server src/greeter_server.cc ${ALL_PROTO_SRCS} ${ALL_GRPC_SRCS})
target_include_directories(greeter_server PUBLIC
  $<BUILD_INTERFACE:${PROTO_GEN_DIR}>
	$<INSTALL_INTERFACE:include>	
    PRIVATE src)
target_link_libraries(greeter_server ${_REFLECTION} ${_GRPC_GRPCPP} ${_PROTOBUF_LIBPROTOBUF})

add_executable(greeter_client src/greeter_client.cc ${ALL_PROTO_SRCS} ${ALL_GRPC_SRCS})
target_include_directories(greeter_client PUBLIC
  $<BUILD_INTERFACE:${PROTO_GEN_DIR}>
	$<INSTALL_INTERFACE:include>	
    PRIVATE src)
target_link_libraries(greeter_client ${_REFLECTION} ${_GRPC_GRPCPP} ${_PROTOBUF_LIBPROTOBUF})
