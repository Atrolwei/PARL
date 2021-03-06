#   Copyright (c) 2018 PaddlePaddle Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 3.0)

enable_testing()

option(WITH_TESTING "Include unit testing" ON)
option(IS_TESTING_IMPORT "Whether is testing import parl" OFF)

set(PADDLE_PYTHON_PATH "" CACHE STRING "Python path to PaddlePaddle Fluid")

function(py_test TARGET_NAME)
    set(options "")
    set(oneValueArgs "")
    set(multiValueArgs SRCS DEPS ARGS ENVS)
    cmake_parse_arguments(py_test "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})
    add_test(NAME ${TARGET_NAME}_with_python2
        COMMAND env PYTHONPATH=.:${py_test_ENVS}
        python -u ${py_test_SRCS} ${py_test_ARGS}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
    add_test(NAME ${TARGET_NAME}_with_python3
        COMMAND env PYTHONPATH=.:${py_test_ENVS}
        python3.6 -u ${py_test_SRCS} ${py_test_ARGS}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
endfunction()

function(import_test TARGET_NAME)
    set(options "")
    set(oneValueArgs "")
    set(multiValueArgs SRCS DEPS ARGS ENVS)
    cmake_parse_arguments(py_test "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})
    add_test(NAME ${TARGET_NAME}_with_empty_env
        COMMAND env PYTHONPATH=.:${py_test_ENVS}
        /root/miniconda3/envs/empty_env/bin/python -u ${py_test_SRCS} ${py_test_ARGS}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
    add_test(NAME ${TARGET_NAME}_with_paddle1.4.0
        COMMAND env PYTHONPATH=.:${py_test_ENVS}
        /root/miniconda3/envs/paddle1.4.0/bin/python -u ${py_test_SRCS} ${py_test_ARGS}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
endfunction()

if (WITH_TESTING)
    if (IS_TESTING_IMPORT)
        set(src "parl/tests/import_test")
        import_test(${src} SRCS ${src}.py ENVS ${PADDLE_PYTHON_PATH})
    else ()
        file(GLOB_RECURSE TEST_OPS RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" "*_test.py")
        string(REPLACE ".py" "" TEST_OPS "${TEST_OPS}")
        foreach(src ${TEST_OPS})
            py_test(${src} SRCS ${src}.py ENVS ${PADDLE_PYTHON_PATH})
        endforeach()
    endif()
endif()
