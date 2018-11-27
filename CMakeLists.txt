cmake_minimum_required(VERSION 2.8.3)
project(qpoases)

find_package(catkin REQUIRED)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME})

##############
## Building ##
##############

include_directories(include)

#
# suppress any printline output from qpOASES
#
add_definitions(-D__SUPPRESSANYOUTPUT__)

#
# building qpOASES LIBRARY
#
set(SRCS
  src/BLASReplacement.cpp
  src/Bounds.cpp
  src/Constraints.cpp
  src/Flipper.cpp
  src/Indexlist.cpp
  src/LAPACKReplacement.cpp
  src/Matrices.cpp
  src/MessageHandling.cpp
  src/Options.cpp
  src/OQPinterface.cpp 
  src/QProblem.cpp
  src/QProblemB.cpp
  src/SolutionAnalysis.cpp
  src/SparseSolver.cpp
  src/SubjectTo.cpp
  src/SQProblem.cpp
  src/SQProblemSchur.cpp
  src/Utils.cpp)
add_library(${PROJECT_NAME} ${SRCS})

set(qpOASES_PYTHON_SRC
  interfaces/python/qpoases.pxd
  interfaces/python/qpoases.pyx
)

#
# building example applications
#
set(EXAMPLES
  example1
  example1a
  example1b
  example2
  example3
  example3b
  example4
  example5
  exampleLP
  qrecipe
  qrecipeSchur)
 
foreach(EXAMPLE ${EXAMPLES})
  add_executable(${EXAMPLE} examples/${EXAMPLE}.cpp)
  target_link_libraries(${EXAMPLE} ${PROJECT_NAME})
endforeach(EXAMPLE)

################
## Installing ##
################

install(TARGETS ${PROJECT_NAME} ${EXAMPLES}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY include/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.hpp"
  PATTERN ".svn" EXCLUDE)

# the files listed above are present when qpoases_svn (i.e. getting source files) is completed
add_custom_command(OUTPUT ${qpOASES_SRC} ${qpOASES_PYTHON_SRC}
  DEPENDS ${PROJECT_NAME})

# Custom build Python wrappers
add_custom_target(qpOASES_Python ALL
  COMMAND python setup.py build_ext --include-dirs include ${INCLUDE_DIRECTORIES} --build-lib ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_PYTHON_DESTINATION} --build-temp ${CMAKE_CURRENT_BINARY_DIR} --sources ${qpOASES_PYTHON_SRC} ${SRCS}
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  DEPENDS ${PROJECT_NAME}
  COMMENT "Building Python wrappers."
)

# Tell CMake about the build output
set(qpOASES_PYTHON_LIB
  ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_PYTHON_DESTINATION}/qpoases.so
)

add_custom_command(OUTPUT ${qpOASES_PYTHON_LIB}
  DEPENDS qpOASES_Python)

# Install
install(FILES ${qpOASES_PYTHON_LIB}
  DESTINATION ${CATKIN_GLOBAL_PYTHON_DESTINATION}
)