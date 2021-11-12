add_library(apriltags SHARED
  CameraUtil.cpp
  DebugImage.cpp
  Geometry.cpp 
  GrayModel.cpp
  MathUtil.cpp 
  Refine.cpp
  TagDetector.cpp 
  TagFamily.cpp
  TagFamilies.cpp
  UnionFindSimple.cpp
)

target_link_libraries(apriltags ${OpenCV_LIBRARIES})

add_executable(camtest camtest.cpp)
target_link_libraries(camtest apriltags ${OpenCV_LIBRARIES})

add_executable(tagtest tagtest.cpp)
target_link_libraries(tagtest apriltags ${OpenCV_LIBRARIES})

add_executable(quadtest quadtest.cpp)
target_link_libraries(quadtest apriltags ${OpenCV_LIBRARIES})

if (GLUT_LIBRARY)

  add_executable(gltest gltest.cpp)
  target_link_libraries(gltest apriltags ${GLUT_LIBRARY} ${OPENGL_LIBRARY} ${OpenCV_LIBRARIES})

endif()

if (CAIRO_FOUND)

  add_executable(maketags maketags.cpp)
  target_link_libraries(maketags apriltags ${CAIRO_LIBRARIES} ${CAIRO_LIBS} ${OpenCV_LIBRARIES})

endif() 

install(TARGETS apriltags DESTINATION lib/)
