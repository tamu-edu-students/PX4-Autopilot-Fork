############################################################################
#
#   Copyright (c) 2025 PX4 Development Team.
#
############################################################################

project(OpticalFlowSystem)

# Include the OpticalFlow external dependency
include(${CMAKE_CURRENT_SOURCE_DIR}/optical_flow.cmake)

# --- Include OpticalFlow and KLT headers
set(OpticalFlow_INCLUDE_DIRS
    ${CMAKE_BINARY_DIR}/OpticalFlow/src/OpticalFlow/include
    ${CMAKE_BINARY_DIR}/OpticalFlow/src/OpticalFlow/external/klt_feature_tracker/include
)
include_directories(${OpticalFlow_INCLUDE_DIRS})

# --- Find OpenCV
find_package(OpenCV REQUIRED COMPONENTS core imgproc)

# --- Find OpticalFlow shared library
find_library(OpticalFlow_LIB OpticalFlow PATHS
    ${CMAKE_BINARY_DIR}/OpticalFlow/src/OpticalFlow-build
    ${CMAKE_BINARY_DIR}/OpticalFlow/src/OpticalFlow
    $ENV{HOME}/.local/usr/local/lib
)

if (NOT OpticalFlow_LIB)
    message(FATAL_ERROR "OpticalFlow library not found — please build OpticalFlow first.")
endif()

# --- Build the plugin
add_library(${PROJECT_NAME} SHARED
    OpticalFlowSensor.cpp
    OpticalFlowSystem.cpp
)

# --- Ensure OpticalFlow builds first
add_dependencies(${PROJECT_NAME} OpticalFlow)

# --- Link libraries
target_link_libraries(${PROJECT_NAME}
    PUBLIC px4_gz_msgs
    PUBLIC ${GZ_SENSORS_TARGET}
    PUBLIC ${GZ_PLUGIN_TARGET}
    PUBLIC ${GZ_SIM_TARGET}
    PUBLIC ${GZ_TRANSPORT_TARGET}
    PUBLIC ${OpenCV_LIBS}
    PUBLIC ${OpticalFlow_LIB}
)

# --- Include directories
target_include_directories(${PROJECT_NAME}
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}
    PUBLIC ${CMAKE_CURRENT_BINARY_DIR}
    PUBLIC ${OpenCV_INCLUDE_DIRS}
    PUBLIC ${OpticalFlow_INCLUDE_DIRS}
)

# --- Install if needed
if (NOT CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    install(TARGETS ${PROJECT_NAME} LIBRARY DESTINATION lib/px4_gz_plugins)
endif()
