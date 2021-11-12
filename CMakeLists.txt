# swatbotics/apriltags-cpp/CMakeLists.txt

cmake_minimum_required(VERSION 2.6)

project(apriltags_cpp)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

#set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
#set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})

if(APPLE)
  include_directories(/opt/local/include) # MacPorts
  link_directories(/opt/local/lib)
  find_library(OPENGL_LIBRARY OpenGL)
else()
  find_library(OPENGL_LIBRARY GL)
  find_library(GLU_LIBRARY GLU)
  set(OPENGL_LIBRARY ${OPENGL_LIBRARY} ${GLU_LIBRARY})
endif()

find_library(GLUT_LIBRARY glut)
find_package(OpenCV)
find_package(catkin REQUIRED COMPONENTS roscpp rosbag tf std_msgs geometry_msgs sensor_msgs nav_msgs visualization_msgs cv_bridge)
find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED COMPONENTS system filesystem thread date_time)

# Describe catkin project
catkin_package(
        CATKIN_DEPENDS roscpp rosbag tf std_msgs geometry_msgs sensor_msgs nav_msgs visualization_msgs cv_bridge
        INCLUDE_DIRS src
        LIBRARIES april_lib
)
include_directories(
        ${OpenCV_INCLUDE_DIRS}
        src
        ${EIGEN3_INCLUDE_DIR}
        ${Boost_INCLUDE_DIRS}
        ${catkin_INCLUDE_DIRS})

include(FindPkgConfig)
pkg_check_modules(CAIRO cairo)
if (${CAIRO_FOUND})
add_definitions(-DMZ_HAVE_CAIRO)
endif ()

find_package( CGAL QUIET COMPONENTS  )
if (CGAL_FOUND)
  include( ${CGAL_USE_FILE} )
  add_definitions(-DHAVE_CGAL)
  find_package( Boost REQUIRED )
else()
  message("CGAL not found, can not use new quad detector")
endif()

set(CMAKE_C_FLAGS "-Wall -g")
set(CMAKE_CXX_FLAGS "-Wall -g")

if (APPLE OR UNIX)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wsign-compare -frounding-math")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wsign-compare -frounding-math")
endif()

# Set link libraries used by all binaries
list(APPEND thirdparty_libraries
        ${Boost_LIBRARIES}
        ${OpenCV_LIBRARIES}
        ${catkin_LIBRARIES}
        )


set(CMAKE_C_FLAGS_DEBUG "-O")
set(CMAKE_CXX_FLAGS_DEBUG "-O")
set(CMAKE_C_FLAGS_RELEASE "-O2")
set(CMAKE_CXX_FLAGS_RELEASE "-O2")

##################################################
# Make the core library
##################################################
add_library(april_lib
        src/CameraUtil.cpp
        src/DebugImage.cpp
        src/Geometry.cpp
        src/GrayModel.cpp
        src/MathUtil.cpp
        src/Refine.cpp
        src/TagFamily.cpp
        src/TagDetector.cpp
        src/TagFamilies.cpp
        src/UnionFindSimple.cpp
        )
target_link_libraries(april_lib  ${thirdparty_libraries})


##################################################
# Make binary files!
##################################################
add_executable(camtest src/camtest.cpp)
target_link_libraries(camtest april_lib ${thirdparty_libraries})

add_executable(quadtest src/quadtest.cpp)
target_link_libraries(quadtest april_lib ${thirdparty_libraries})
#
add_executable(tagtest src/tagtest.cpp)
target_link_libraries(tagtest april_lib ${thirdparty_libraries})

if (GLUT_LIBRARY)

  add_executable(gltest src/gltest.cpp)
  target_link_libraries(gltest april_lib  ${thirdparty_libraries} ${GLUT_LIBRARY} ${OPENGL_LIBRARY})

endif()

if (CAIRO_FOUND)

  add_executable(maketags src/maketags.cpp)
  target_link_libraries(maketags april_lib ${CAIRO_LIBRARIES} ${CAIRO_LIBS} ${thirdparty_libraries})

endif()




#install(DIRECTORY src/ DESTINATION include/apriltags_cpp FILES_MATCHING PATTERN "*.h" PATTERN ".svn" EXCLUDE)
#install(FILES package.xml DESTINATION share/apriltags_cpp)
#configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/Config.cmake.in "${PROJECT_BINARY_DIR}/apriltags_cppConfig.cmake" @ONLY)
#install(FILES "${PROJECT_BINARY_DIR}/apriltags_cppConfig.cmake" DESTINATION share/apriltags_cpp/cmake)
