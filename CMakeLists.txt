#*******************************************************************************
#*   (c) 2018 ZondaX GmbH
#*
#*  Licensed under the Apache License, Version 2.0 (the "License");
#*  you may not use this file except in compliance with the License.
#*  You may obtain a copy of the License at
#*
#*      http://www.apache.org/licenses/LICENSE-2.0
#*
#*  Unless required by applicable law or agreed to in writing, software
#*  distributed under the License is distributed on an "AS IS" BASIS,
#*  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#*  See the License for the specific language governing permissions and
#*  limitations under the License.
#********************************************************************************
cmake_minimum_required(VERSION 3.0)
project(ledger-cosmos VERSION 0.0.0)
enable_testing()

cmake_policy(SET CMP0025 NEW)
set(CMAKE_CXX_STANDARD 11)

include(cmake/conan/CMakeLists.txt)
add_subdirectory(cmake/gtest)

string(APPEND CMAKE_CXX_FLAGS " -fsanitize=address -fno-omit-frame-pointer")
string(APPEND CMAKE_LINKER_FLAGS " -fsanitize=address -fno-omit-frame-pointer")

##############################################################
##############################################################
#  static libs
file(GLOB_RECURSE JSMN_SRC
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/deps/jsmn/src/jsmn.c
        )

file(GLOB_RECURSE LIB_SRC
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src/lib/json/json_parser.c
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src/lib/json/tx_parser.c
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src/lib/json/tx_display.c
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src/lib/json/tx_validate.c
        #########
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src/lib/parser.c
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src/lib/parser_impl.c
        )

file(GLOB_RECURSE TESTS_SRC
        ${CMAKE_CURRENT_SOURCE_DIR}/tests/*.cpp)

file(GLOB_RECURSE FUZZING_SRC
        ${CMAKE_CURRENT_SOURCE_DIR}/fuzzing/fuzzingMain.cpp
        tests/util/common.cpp
        )

add_library(app_lib STATIC ${LIB_SRC} ${JSMN_SRC})
target_include_directories(app_lib PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/deps/jsmn/src
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src/lib
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/deps/ledger-zxlib/include
        )

set(JSON_BuildTests OFF CACHE INTERNAL "")

add_executable(unittests ${TESTS_SRC})
target_include_directories(unittests PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src/lib
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/deps/jsmn/src
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/deps/ledger-zxlib/include
        ${gtest_SOURCE_DIR}/include
        ${gmock_SOURCE_DIR}/include
        ${CONAN_INCLUDE_DIRS_FMT}
        ${CONAN_INCLUDE_DIRS_JSONCPP}
        )

target_link_libraries(unittests PRIVATE
        gtest_main
        app_lib
        CONAN_PKG::fmt
        CONAN_PKG::jsoncpp)

add_test(unittests ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/unittests)
set_tests_properties(unittests PROPERTIES WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/tests)

add_executable(fuzzing_stub ${FUZZING_SRC})
target_include_directories(fuzzing_stub PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/src
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ledger/deps/jsmn/src
        )
target_link_libraries(fuzzing_stub app_lib)

###############################################################
# Force tests to depend from app compiling
###############################################################

#set(DISABLE_DOCKER_BUILDS OFF CACHE BOOL "Disables Docker Builds")
#
#if (NOT DISABLE_DOCKER_BUILDS)
#    add_custom_target(ledger_app
#            COMMAND make build
#            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
#            )
#    add_dependencies(unittests ledger_app)
#endif ()
