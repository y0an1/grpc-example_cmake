cmake_minimum_required(VERSION 3.15)
#工程名,可自行修改
project(grpc-example_cmake CXX)

#以下三个find_package需要添加,否则找不到对应的target会报错
find_package(Threads REQUIRED)#注意不要加CONFIG
find_package(protobuf CONFIG REQUIRED)
find_package(gRPC CONFIG REQUIRED)

##添加共享的静态库,包含helloworld.proto中定义的RPC协议代码
add_library(mytest)
target_sources(mytest
        PRIVATE "proto/mathtest.proto"
        )

##生成helloworld.proto对应的C++代码
protobuf_generate(TARGET mytest LANGUAGE cpp)

##获取proto的grpc插件
get_target_property(grpc_cpp_plugin_location gRPC::grpc_cpp_plugin LOCATION)

##生成helloworld.proto对应的gRPC-C++代码,以此来支持gRPC协议
protobuf_generate(TARGET mytest LANGUAGE grpc GENERATE_EXTENSIONS .grpc.pb.h .grpc.pb.cc PLUGIN "protoc-gen-grpc=${grpc_cpp_plugin_location}")

target_link_libraries(mytest
        PRIVATE gRPC::grpc++
        )

##上述protobuf_generate将自动生成的代码存放于该位置,需要添加到include路径
target_include_directories(mytest
        PUBLIC  ${CMAKE_CURRENT_BINARY_DIR}
        )

##遍历列表创建应用程序
foreach(_target
        client server)
    add_executable(${_target} "src/${_target}.cpp")
    target_link_libraries(${_target}
            PRIVATE mytest
            gRPC::grpc++ gRPC::grpc++_reflection
            )
endforeach()