# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.

cmake_minimum_required(VERSION 2.8.3)
project(roscpp_azure_iothub)

if(MSVC)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  topic_tools
  ros_type_introspection
  dynamic_reconfigure
)

if (WIN32)
  find_package(azure_c_shared_utility REQUIRED)
  find_package(azure_iot_sdks REQUIRED)
  find_package(umqtt REQUIRED)
  find_package(uamqp REQUIRED)  
else (WIN32)
  set(IOTHUB_CLIENT_INCLUDES ".." "/usr/include/azureiot" "/usr/include/azureiot/inc" "./parson")
  add_library(parson STATIC
    parson/parson.c
    parson/parson.h
  )
endif()

###################################
## catkin specific configuration ##
###################################

catkin_package(
  CATKIN_DEPENDS roscpp std_msgs topic_tools ros_type_introspection
)

###########
## Build ##
###########

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${IOTHUB_CLIENT_INCLUDES}
)

add_executable(${PROJECT_NAME}_node src/ros_azure_iothub_cpp_node.cpp)

link_directories(${catkin_LIBRARY_DIRS})

if (WIN32)
  target_link_libraries(${PROJECT_NAME}_node
    ${catkin_LIBRARIES}
    iothub_client
    iothub_client_mqtt_ws_transport
  )
else (WIN32)
  target_link_libraries(${PROJECT_NAME}_node
    ${catkin_LIBRARIES}
    iothub_client
    iothub_client_mqtt_transport
    umqtt
    aziotsharedutil
    pthread
    curl
    ssl
    crypto
    m
    uuid
    parson
    prov_auth_client
    hsm_security_client
    uhttp
  )
endif()

#############
## Install ##
#############

install(TARGETS ${PROJECT_NAME}_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
)

#############
##  Test   ##
#############

if(WIN32)
  if(CATKIN_ENABLE_TESTING)
    find_package(rostest REQUIRED)
    add_rostest_gtest(${PROJECT_NAME}_testnode test/testiothub.launch test/testiothub.cpp)
    target_link_libraries(${PROJECT_NAME}_testnode iothub_service_client ${catkin_LIBRARIES} )
  endif()
endif()