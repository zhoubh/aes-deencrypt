# Copyright 2012 Cloudera Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# where to put generated libraries
set(LIBRARY_OUTPUT_PATH "${BUILD_OUTPUT_ROOT_DIRECTORY}/udf_samples")
# where to put generated binaries
set(EXECUTABLE_OUTPUT_PATH "${BUILD_OUTPUT_ROOT_DIRECTORY}/udf_samples")

# Function to generate rule to cross compile a source file to an IR module.
# This should be called with the .cc src file and it will generate a
# src-file-ir target that can be built.
# e.g. COMPILE_TO_IR(test.cc) generates the "test-ir" make target.
set(IR_COMPILE_FLAGS "-emit-llvm" "-O3" "-c" "-I../" ${CLANG_BASE_FLAGS})
function(COMPILE_TO_IR SRC_FILE)
  get_filename_component(BASE_NAME ${SRC_FILE} NAME_WE)
  set(OUTPUT_FILE "${LIBRARY_OUTPUT_PATH}/${BASE_NAME}.ll")
  add_custom_command(
    OUTPUT ${OUTPUT_FILE}
    COMMAND ${LLVM_CLANG_EXECUTABLE} ${IR_COMPILE_FLAGS} ${CLANG_INCLUDE_FLAGS} ${SRC_FILE} -o ${OUTPUT_FILE}
    DEPENDS ${SRC_FILE})
  add_custom_target(${BASE_NAME}-ir ALL DEPENDS ${OUTPUT_FILE})
endfunction(COMPILE_TO_IR)

# Build the UDA/UDFs into a shared library.
add_library(udfsample SHARED udf-sample.cc)
add_dependencies(udfsample thrift-deps)
add_library(udasample SHARED uda-sample.cc hyperloglog-uda.cc)
add_dependencies(udasample thrift-deps)
set(CRYPTOPP_INCLUDE_DIR "/home/dev/Impala/cryptopp")
set(CRYPTOPP_LIBS "/home/dev/Impala/cryptopp/libcryptopp.a")
include_directories(${CRYPTOPP_INCLUDE_DIR})
add_library(aes-deencrypt SHARED aes-deencrypt.cc)
add_dependencies(aes-deencrypt thrift-deps)
target_link_libraries(aes-deencrypt ${CRYPTOPP_LIBS})
#add_library(decrypt SHARED decrypt.cc)
#add_dependencies(decrypt thrift-deps)
#target_link_libraries(decrypt ${IMPALA_LINK_LIBS} ImpalaUdf ${CRYPTOPP_LIBS})
#set(LIBGFSHARE_INCLUDE_DIR "/home/dev/Impala/libgfshare/include")
#set(LIBGFSHARE_LIBS "/home/dev/Impala/libgfshare/libgfshare.a")
#include_directories(${LIBGFSHARE_INCLUDE_DIR})
#add_library(secret-sharing SHARED secret-sharing.cc)
#add_dependencies(secret-sharing thrift-deps)
#target_link_libraries(secret-sharing ${LIBGFSHARE_LIBS})

# Custom targest to cross compile UDA/UDF to ir
COMPILE_TO_IR(udf-sample.cc )
add_dependencies(udf-sample-ir thrift-deps)
COMPILE_TO_IR(uda-sample.cc )
add_dependencies(uda-sample-ir thrift-deps)

# This is an example of how to use the test harness to help develop UDF and UDAs.
add_executable(udf-sample-test udf-sample-test.cc)
target_link_libraries(udf-sample-test ImpalaUdf udfsample)
add_executable(uda-sample-test uda-sample-test.cc)
target_link_libraries(uda-sample-test ImpalaUdf udasample)
#add_executable(decrypt-test decrypt-test.cc)
#target_link_libraries(decrypt-test ImpalaUdf decrypt)
