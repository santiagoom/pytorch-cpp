cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(example-app)

#-DCMAKE_PREFIX_PATH=D:\\lib\\libtorch-win-shared-with-deps-1.6.0+cu101\\libtorch
#D:\\lib\\libtorch-win-shared-with-deps-debug-1.6.0+cu101\\libtorch
set (BOOST_ROOT "D:\\lib\\libtorch-win-shared-with-deps-debug-1.6.0+cu101\\libtorch")
set (Boost_LIBRARY_DIR "D:\\lib\\libtorch-win-shared-with-deps-debug-1.6.0+cu101\\libtorch\\lib")

list(APPEND CMAKE_PREFIX_PATH "D:\\lib\\libtorch-win-shared-with-deps-debug-1.6.0+cu101\\libtorch")

find_package(Torch REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")




add_executable(MNIST-cpp MNIST-cpp.cpp)
target_link_libraries(MNIST-cpp "${TORCH_LIBRARIES}")
set_property(TARGET MNIST-cpp PROPERTY CXX_STANDARD 14)


add_executable(example-app example-app.cpp)
target_link_libraries(example-app "${TORCH_LIBRARIES}")
set_property(TARGET example-app PROPERTY CXX_STANDARD 14)

# The following code block is suggested to be used on Windows.
# According to https://github.com/pytorch/pytorch/issues/25457,
# the DLLs need to be copied to avoid memory errors.
if (MSVC)
    file(GLOB TORCH_DLLS "${TORCH_INSTALL_PREFIX}/lib/*.dll")
    add_custom_command(TARGET example-app
            POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${TORCH_DLLS}
            $<TARGET_FILE_DIR:example-app>)
endif (MSVC)

if (MSVC)
    file(GLOB TORCH_DLLS "${TORCH_INSTALL_PREFIX}/lib/*.dll")
    add_custom_command(TARGET MNIST-cpp
            POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${TORCH_DLLS}
            $<TARGET_FILE_DIR:MNIST-cpp>)
endif (MSVC)


option(DOWNLOAD_MNIST "Download the MNIST dataset from the internet" ON)
if (DOWNLOAD_MNIST)
    message(STATUS "Downloading MNIST dataset")
    execute_process(
#            COMMAND python ${CMAKE_CURRENT_LIST_DIR}/../tools/download_mnist.py
            COMMAND python ${CMAKE_CURRENT_LIST_DIR}/tools/download_mnist.py
            -d ${CMAKE_BINARY_DIR}/data
            ERROR_VARIABLE DOWNLOAD_ERROR)
    if (DOWNLOAD_ERROR)
        message(FATAL_ERROR "Error downloading MNIST dataset: ${DOWNLOAD_ERROR}")
    endif()
endif()

add_executable(mnist mnist.cpp)
target_compile_features(mnist PUBLIC cxx_range_for)
target_link_libraries(mnist ${TORCH_LIBRARIES})

if (MSVC)
    file(GLOB TORCH_DLLS "${TORCH_INSTALL_PREFIX}/lib/*.dll")
    add_custom_command(TARGET mnist
            POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${TORCH_DLLS}
            $<TARGET_FILE_DIR:mnist>)
endif (MSVC)

message(STATUS "print_all_variables------------------------------------------{")
get_cmake_property(_variableNames VARIABLES)
foreach (_variableName ${_variableNames})
    message(STATUS "${_variableName}=${${_variableName}}")
endforeach()
message(STATUS "print_all_variables------------------------------------------}")
