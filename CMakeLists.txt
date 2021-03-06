cmake_minimum_required(VERSION 2.6)

project(nodewatcher-agent C)
add_definitions(-Os -Wall -Werror --std=gnu99 -Wmissing-declarations)

option(GENERAL_MODULE "General module support" ON)
option(RESOURCES_MODULE "Resources module support" ON)
option(INTERFACES_MODULE "Interfaces module support" ON)
option(WIRELESS_MODULE "Wireless module support" ON)
option(KEYS_SSH_MODULE "SSH keys module support" ON)
option(PACKAGES_MODULE "Packages module support" ON)
option(CLIENTS_MODULE "Clients module support" ON)
option(HTTP_PUSH_MODULE "HTTP push module support" ON)
option(ROUTING_BABEL_MODULE "Babel routing module support" ON)
option(ROUTING_OLSR_MODULE "OLSR routing module support" ON)

set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")

include(FindPkgConfig)

pkg_search_module(JSONC json-c)
if(JSONC_FOUND)
  include_directories(${JSONC_INCLUDE_DIRS})
endif()

find_path(ubus_include_dir libubus.h)
include_directories(${ubus_include_dir})

find_path(ubox_include_dir libubox/blobmsg_json.h)
include_directories(${ubox_include_dir})

find_path(uci_include_dir uci.h)
include_directories(${uci_include_dir})

find_library(json_library NAMES json-c json)
find_library(ubus_library NAMES ubus)
find_library(ubox_library NAMES ubox)
find_library(uci_library NAMES uci)
find_library(blobmsg_json_library NAMES blobmsg_json)

set(SOURCES
  main.c
)

set(LIBS
  ${ubox_library}
  ${ubus_library}
  ${uci_library}
  ${json_library}
  ${blobmsg_json_library}
  dl
  m
)

include_directories(include)

# Nodewatcher agent common library
set(COMMON_SOURCES
  common/module.c
  common/scheduler.c
  common/json.c
  common/utils.c
  common/output.c
)
add_library(nodewatcher-agent-common SHARED ${COMMON_SOURCES})
target_link_libraries(nodewatcher-agent-common ${LIBS})

# Nodewatcher agent binary
add_executable(nodewatcher-agent ${SOURCES})
target_link_libraries(nodewatcher-agent ${LIBS} nodewatcher-agent-common)

# Modules
set(MODULES "")

# General module
if(GENERAL_MODULE)
  set(MODULES ${MODULES} general_module)
  add_library(general_module MODULE modules/general.c)
  target_link_libraries(general_module ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(general_module PROPERTIES OUTPUT_NAME general PREFIX "")
endif()

# Resources module
if(RESOURCES_MODULE)
  set(MODULES ${MODULES} resources_module)
  add_library(resources_module MODULE modules/resources.c)
  target_link_libraries(resources_module ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(resources_module PROPERTIES OUTPUT_NAME resources PREFIX "")
endif()

# Interfaces module
if(INTERFACES_MODULE)
  set(MODULES ${MODULES} interfaces_module)
  add_library(interfaces_module MODULE modules/interfaces.c)
  target_link_libraries(interfaces_module ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(interfaces_module PROPERTIES OUTPUT_NAME interfaces PREFIX "")
endif()

# Wireless module
if(WIRELESS_MODULE)
  set(MODULES ${MODULES} wireless_module)
  add_library(wireless_module MODULE modules/wireless.c)
  target_link_libraries(wireless_module ${ubox_library} ${uci_library} iwinfo nodewatcher-agent-common)
  set_target_properties(wireless_module PROPERTIES OUTPUT_NAME wireless PREFIX "")
endif()

# SSH keys module
if(KEYS_SSH_MODULE)
  set(MODULES ${MODULES} keys_ssh_module)
  add_library(keys_ssh_module MODULE modules/keys_ssh.c)
  target_link_libraries(keys_ssh_module ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(keys_ssh_module PROPERTIES OUTPUT_NAME keys_ssh PREFIX "")
endif()

# Packages module
if(PACKAGES_MODULE)
  set(MODULES ${MODULES} packages_module)
  add_library(packages_module MODULE modules/packages.c)
  target_link_libraries(packages_module ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(packages_module PROPERTIES OUTPUT_NAME packages PREFIX "")
endif()

# Clients module
if(CLIENTS_MODULE)
  set(MODULES ${MODULES} clients_module)
  add_library(clients_module MODULE modules/clients.c)
  target_link_libraries(clients_module ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(clients_module PROPERTIES OUTPUT_NAME clients PREFIX "")
endif()

# HTTP push module
if(HTTP_PUSH_MODULE)
  find_path(curl_include_dir curl.h PATH_SUFFIXES curl)
  include_directories(${curl_include_dir})
  find_library(curl_library NAMES curl)

  set(MODULES ${MODULES} http_push_module)
  add_library(http_push_module MODULE modules/http_push.c)
  target_link_libraries(http_push_module ${curl_library} ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(http_push_module PROPERTIES OUTPUT_NAME http_push PREFIX "")
endif()

# Babel routing module
if(ROUTING_BABEL_MODULE)
  set(MODULES ${MODULES} routing_babel_module)
  add_library(routing_babel_module MODULE modules/routing_babel.c)
  target_link_libraries(routing_babel_module ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(routing_babel_module PROPERTIES OUTPUT_NAME routing_babel PREFIX "")
endif()

# OLSR routing module
if(ROUTING_OLSR_MODULE)
  set(MODULES ${MODULES} routing_olsr_module)
  add_library(routing_olsr_module MODULE modules/routing_olsr.c)
  target_link_libraries(routing_olsr_module ${ubox_library} ${uci_library} nodewatcher-agent-common)
  set_target_properties(routing_olsr_module PROPERTIES OUTPUT_NAME routing_olsr PREFIX "")
endif()

install(TARGETS nodewatcher-agent nodewatcher-agent-common
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
)

install(TARGETS ${MODULES} LIBRARY DESTINATION lib/nodewatcher-agent)

file(GLOB headers include/nodewatcher-agent/*.h)
install(FILES ${headers} DESTINATION include/nodewatcher-agent)
