cmake_minimum_required(VERSION 3.15.0)
project(advancing_front_mesher VERSION 0.1.0 LANGUAGES C CXX)

find_package(ros_industrial_cmake_boilerplate REQUIRED)
find_package(
  PCL REQUIRED
  COMPONENTS common
             surface
             visualization
             io
             console)

# Advancing Front Library
add_library(${PROJECT_NAME} SHARED src/advancing_front.cpp)
target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra)
target_include_directories(${PROJECT_NAME} PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
                                                  "$<INSTALL_INTERFACE:include>")
target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC ${PCL_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PUBLIC ${PCL_LIBRARIES})
target_cxx_version(${PROJECT_NAME} PRIVATE VERSION 14)

# Add OpenMP support, if available
find_package(OpenMP)
if(OPENMP_FOUND)
  # Link the library against the OpenMP target
  target_link_libraries(${PROJECT_NAME} PUBLIC OpenMP::OpenMP_CXX)
else(OPENMP_FOUND)
  message(STATUS "OpenMP not found")
endif()

# Advancing Front Tool
add_executable(${PROJECT_NAME}_tool src/advancing_front_reconstruction.cpp)
target_link_libraries(${PROJECT_NAME}_tool PUBLIC ${PROJECT_NAME})
target_cxx_version(${PROJECT_NAME} PRIVATE VERSION 14)

configure_package(NAMESPACE advancing_front_mesher TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_tool
                  DEPENDENCIES "PCL COMPONENTS common surface visualization io console" OpenMP)

install(DIRECTORY include/${PROJECT_NAME} DESTINATION include)

# ######################################################################################################################
# Testing ##
# ######################################################################################################################
if(BUILD_TESTING)
  enable_testing()
  add_subdirectory(test)
endif()
