cmake_minimum_required(VERSION 2.8)
project(tf)

find_package(catkin REQUIRED)

find_package(Boost REQUIRED thread signals)
find_package(catkin REQUIRED angles geometry_msgs message_filters message_generation roscpp rostime sensor_msgs std_msgs tf2_ros)


include_directories(SYSTEM ${Boost_INCLUDE_DIR}
                           ${catkin_INCLUDE_DIRS}
)
include_directories(include)
link_directories(${catkin_LIBRARY_DIRS})

add_message_files(DIRECTORY msg FILES tfMessage.msg)
add_service_files(DIRECTORY srv FILES FrameGraph.srv)

generate_messages(DEPENDENCIES geometry_msgs sensor_msgs std_msgs)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS geometry_msgs message_filters message_runtime roscpp sensor_msgs std_msgs tf2_ros
)

add_library(${PROJECT_NAME} src/tf.cpp src/transform_listener.cpp src/cache.cpp src/transform_broadcaster.cpp)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES} ${Boost_LIBRARIES})
add_dependencies(${PROJECT_NAME} ${PROJECT_NAME}_gencpp)

# Debug
add_executable(tf_empty_listener src/empty_listener.cpp)
target_link_libraries(tf_empty_listener ${PROJECT_NAME})

add_executable(tf_echo src/tf_echo.cpp)
target_link_libraries(tf_echo ${PROJECT_NAME})

add_executable(tf_change_notifier src/change_notifier.cpp)
target_link_libraries(tf_change_notifier ${PROJECT_NAME})

add_executable(tf_monitor src/tf_monitor.cpp)
target_link_libraries(tf_monitor ${PROJECT_NAME})

add_executable(static_transform_publisher src/static_transform_publisher.cpp)
target_link_libraries(static_transform_publisher ${PROJECT_NAME})

# Dynamic linking with tf worked OK, except for exception propagation, which failed in the unit test.
# so build with the objects directly instead.

add_library(src/tf.cpp src/transform_listener.cpp src/cache.cpp)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

install(TARGETS ${PROJECT_NAME} tf_echo tf_empty_listener tf_change_notifier tf_monitor static_transform_publisher
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})


# Need to install python msg and srv directories manually because genmsg
# specifically avoids installing python message code if catkin_python_setup()
# has been used.  See https://github.com/ros/genmsg/issues/10
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/${PYTHON_INSTALL_DIR}/tf/msg
  DESTINATION ${CMAKE_INSTALL_PREFIX}/${PYTHON_INSTALL_DIR}/tf)
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/${PYTHON_INSTALL_DIR}/tf/srv
  DESTINATION ${CMAKE_INSTALL_PREFIX}/${PYTHON_INSTALL_DIR}/tf)
