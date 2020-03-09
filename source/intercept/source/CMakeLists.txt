# Create a header file with the relevant configuration values.
configure_file(config.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)

# Create a static library, which is used for unit tests and the final shared library.
add_library(intercept_a STATIC
        SystemCalls.h
        SystemCalls.cc
        )

target_include_directories(intercept_a PRIVATE ${EXE_CFG_DIR})
target_include_directories(intercept_a PRIVATE ${EXE_API_DIR})
target_link_libraries(intercept_a PRIVATE nlohmann_json::nlohmann_json)
target_compile_features(intercept_a PUBLIC cxx_std_14)
target_compile_options(intercept_a PUBLIC -fno-exceptions -fno-rtti)
set_target_properties(intercept_a PROPERTIES LINKER_LANGUAGE "C")

# Proto file
get_filename_component(supervise_proto "../proto/supervise.proto" ABSOLUTE)
get_filename_component(supervise_proto_path "${supervise_proto}" PATH)
get_filename_component(hw_proto "../proto/helloworld.proto" ABSOLUTE)
get_filename_component(hw_proto_path "${hw_proto}" PATH)

# Generated sources
set(hw_proto_srcs "${CMAKE_CURRENT_BINARY_DIR}/helloworld.pb.cc")
set(hw_proto_hdrs "${CMAKE_CURRENT_BINARY_DIR}/helloworld.pb.h")
set(hw_grpc_srcs "${CMAKE_CURRENT_BINARY_DIR}/helloworld.grpc.pb.cc")
set(hw_grpc_hdrs "${CMAKE_CURRENT_BINARY_DIR}/helloworld.grpc.pb.h")
add_custom_command(
        OUTPUT "${hw_proto_srcs}" "${hw_proto_hdrs}" "${hw_grpc_srcs}" "${hw_grpc_hdrs}"
        COMMAND ${_PROTOBUF_PROTOC}
        ARGS --grpc_out "${CMAKE_CURRENT_BINARY_DIR}"
        --cpp_out "${CMAKE_CURRENT_BINARY_DIR}"
        -I "${hw_proto_path}"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${hw_proto}"
        DEPENDS "${hw_proto}")

# Include generated *.pb.h files
include_directories("${CMAKE_CURRENT_BINARY_DIR}")

# Targets greeter_[async_](client|server)
foreach(_target
        greeter_client greeter_server
        greeter_async_client greeter_async_server)
    add_executable(${_target} "${_target}.cc"
            ${hw_proto_srcs}
            ${hw_grpc_srcs})
    target_link_libraries(${_target}
            ${_GRPC_GRPCPP_UNSECURE}
            ${_PROTOBUF_LIBPROTOBUF})
endforeach()