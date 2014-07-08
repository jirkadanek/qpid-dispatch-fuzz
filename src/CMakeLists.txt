##
## Licensed to the Apache Software Foundation (ASF) under one
## or more contributor license agreements.  See the NOTICE file
## distributed with this work for additional information
## regarding copyright ownership.  The ASF licenses this file
## to you under the Apache License, Version 2.0 (the
## "License"); you may not use this file except in compliance
## with the License.  You may obtain a copy of the License at
##
##   http://www.apache.org/licenses/LICENSE-2.0
##
## Unless required by applicable law or agreed to in writing,
## software distributed under the License is distributed on an
## "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
## KIND, either express or implied.  See the License for the
## specific language governing permissions and limitations
## under the License.
##

# Generate code from the management schema.
set(GENERATED_SOURCES
  schema_enum.h
  schema_enum.c
  )

file (GLOB_RECURSE GENERATOR_DEPENDS
  ${CMAKE_CURRENT_SOURCE_DIR}/schema_c.py
  ${CMAKE_SOURCE_DIR}/python/qpid_router_internal/management/*.py
  ${CMAKE_SOURCE_DIR}/python/qpid_router_internal/management/qdrouterd.json)

add_custom_command (
  OUTPUT ${GENERATED_SOURCES}
  COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_BINARY_DIR}/tests/run.py -s ${CMAKE_CURRENT_SOURCE_DIR}/schema_c.py
  DEPENDS ${GENERATOR_DEPENDS})

add_custom_target(generate ALL DEPENDS ${GENERATED_SOURCES})

# Build the qpid-dispatch library.
set(qpid_dispatch_SOURCES
  agent.c
  alloc.c
  amqp.c
  bitmask.c
  buffer.c
  error.c
  compose.c
  connection_manager.c
  container.c
  dispatch.c
  entity.c
  hash.c
  iovec.c
  iterator.c
  log.c
  message.c
  parse.c
  posix/threading.c
  python_embedded.c
  router_agent.c
  router_config.c
  router_node.c
  router_pynode.c
  schema_enum.c
  server.c
  timer.c
  waypoint.c
  )

set_property(
  SOURCE python_embedded.c router_pynode.c
  PROPERTY COMPILE_FLAGS -Wno-strict-aliasing
  )

add_library(qpid-dispatch SHARED ${qpid_dispatch_SOURCES})
target_link_libraries(qpid-dispatch ${proton_lib} ${pthread_lib} ${rt_lib} ${PYTHON_LIBRARIES})
set_target_properties(qpid-dispatch PROPERTIES
  VERSION "${SO_VERSION}"
  SOVERSION "${SO_VERSION_MAJOR}"
  LINK_FLAGS "${CATCH_UNDEFINED}"
  )
install(TARGETS qpid-dispatch
  LIBRARY DESTINATION ${LIB_INSTALL_DIR})