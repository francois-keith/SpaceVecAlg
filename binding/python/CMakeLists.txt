pkg_check_modules(EIGEN3 REQUIRED eigen3)
find_package(PythonInterp REQUIRED)
find_package(PythonLibs REQUIRED)

# create the package in build dir for testing purpose
set(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/spacevecalg)
configure_file(__init__.py ${CMAKE_CURRENT_BINARY_DIR}/spacevecalg/__init__.py COPYONLY)

set(OUTPUT_BINDING ${CMAKE_CURRENT_BINARY_DIR}/spacevecalg.cpp)

# generate python binding code
add_custom_command (
  OUTPUT ${OUTPUT_BINDING}
  COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/generate.py ${OUTPUT_BINDING}
  DEPENDS generate.py
)

# build the library
set(SOURCES ${OUTPUT_BINDING})
include_directories(.)
include_directories(../../src)
include_directories(${EIGEN3_INCLUDE_DIRS})
include_directories(${PYTHON_INCLUDE_DIRS})
add_library(_spacevecalg SHARED ${SOURCES})

set_target_properties(_spacevecalg PROPERTIES PREFIX "")

# install rules
# find python version (not implemented in cmake 2.8.5)
execute_process(COMMAND "${PYTHON_EXECUTABLE}" -V ERROR_VARIABLE _VERSION
                OUTPUT_QUIET ERROR_STRIP_TRAILING_WHITESPACE)
string(REPLACE "Python " "" PYTHON_VERSION_STRING "${_VERSION}")
string(REGEX REPLACE "^([0-9]+)\\.[0-9]+\\.[0-9]+.*" "\\1"
       PYTHON_VERSION_MAJOR "${PYTHON_VERSION_STRING}")
string(REGEX REPLACE "^[0-9]+\\.([0-9])+\\.[0-9]+.*" "\\1"
       PYTHON_VERSION_MINOR "${PYTHON_VERSION_STRING}")
 
set(INSTALL_PATH "lib/python${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}/dist-packages/spacevecalg/")
install(TARGETS _spacevecalg DESTINATION ${INSTALL_PATH})
install(FILES __init__.py DESTINATION ${INSTALL_PATH})
