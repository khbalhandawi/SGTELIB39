# ----------------------------------------------------------------------------
#  Root CMake file for sgtelib library
#
#    From the off-tree build directory, invoke:
#      $ cmake <PATH_TO_OPENCV_ROOT>
#
# ----------------------------------------------------------------------------
# cmake version : the latest one
cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

set(SGTELIB_VERSION_MAJOR 2)
set(SGTELIB_VERSION_MINOR 0)
set(SGTELIB_VERSION_PATCH 2)
set(SGTELIB_VERSION ${SGTELIB_VERSION_MAJOR}.${SGTELIB_VERSION_MINOR}.${SGTELIB_VERSION_PATCH})

# name of the project
project (sgtelib LANGUAGES CXX VERSION ${SGTELIB_VERSION})

# Disable in-source builds to prevent source tree corruption.
if(" ${CMAKE_SOURCE_DIR}" STREQUAL " ${CMAKE_BINARY_DIR}")
    message(FATAL_ERROR "FATAL: In-source builds are not allowed.
    You should create a separate directory for build files.")
endif()

# check compiler version
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    # require at least gcc 4
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4)
        message(FATAL_ERROR "GCC version < 4 has not been tested for Nomad")
    endif()
elseif (("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang") 
    OR ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang"))
    # require at least clang 5
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 5)
        message(FATAL_ERROR "Clang version has not been tested for Nomad")
    endif()
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    # require at least 15.0 (MSVC 2017) for C++14 support
    if (MSVC_TOOLSET_VERSION VERSION_LESS 141)
        message(FATAL_ERROR "MSVC version ${CMAKE_CXX_COMPILER_VERSION} has not been tested for Nomad")
    endif()
else()
    message(WARNING "You are using an unsupported compiler. Compilation has only been tested with Clang, GCC, and MSVC.")
endif()

# require c++14
set(CXX_STANDARD_REQUIRED 14)


# header files
set(SGTELIB_HEADERS
    include/Defines.hpp
    include/Exception.hpp
    include/Kernel.hpp
    include/Matrix.hpp
    include/Metrics.hpp
    include/Surrogate.hpp
    include/Surrogate_CN.hpp
    include/Surrogate_Ensemble.hpp
    include/Surrogate_Factory.hpp
    include/Surrogate_KS.hpp
    include/Surrogate_Kriging.hpp
    include/Surrogate_LOWESS.hpp
    include/Surrogate_PRS.hpp
    include/Surrogate_PRS_CAT.hpp
    include/Surrogate_PRS_EDGE.hpp
    include/Surrogate_Parameters.hpp
    include/Surrogate_RBF.hpp
    include/Surrogate_Utils.hpp
    include/Tests.hpp
    include/TrainingSet.hpp
    include/sgtelib_help.hpp
    )

# source files
set(SGTELIB_SOURCES
    src/Kernel.cpp
    src/Matrix.cpp
    src/Metrics.cpp
    src/Surrogate.cpp
    src/Surrogate_CN.cpp
    src/Surrogate_Ensemble.cpp
    src/Surrogate_Factory.cpp
    src/Surrogate_KS.cpp
    src/Surrogate_Kriging.cpp
    src/Surrogate_LOWESS.cpp
    src/Surrogate_PRS.cpp
    src/Surrogate_PRS_CAT.cpp
    src/Surrogate_PRS_EDGE.cpp
    src/Surrogate_Parameters.cpp
    src/Surrogate_RBF.cpp
    src/Surrogate_Utils.cpp
    src/Tests.cpp
    src/TrainingSet.cpp
    src/sgtelib_help.cpp
    )

set(SGTELIB_MAIN_SOURCE src/sgtelib.cpp)

# load classic directories for unix systems
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# build shared library sgtelib
add_library (sgtelib SHARED ${SGTELIB_SOURCES})
target_include_directories(sgtelib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/sgtelib>
    )

# export symbols on Windows
if (WIN32)
    target_compile_definitions(sgtelib PRIVATE -DDLL_EXPORTS)
endif()

#add_executable (sgtelib_main ${SGTELIB_MAIN_SOURCE})
#set_target_properties(sgtelib_main PROPERTIES INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")
#target_include_directories(sgtelib_main PUBLIC
#    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
#    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/sgtelib>
#)
#target_link_libraries(sgtelib_main PUBLIC sgtelib)

set_target_properties(sgtelib  PROPERTIES 
                               VERSION ${SGTELIB_VERSION}
                               SOVERSION ${SGTELIB_VERSION_MAJOR}.${SGTELIB_VERSION_MINOR})

# installing executables and libraries
install(TARGETS sgtelib 
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}  
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/
    )

if(MSVC)  
    install(FILES $<TARGET_PDB_FILE:sgtelib> DESTINATION ${CMAKE_INSTALL_LIBDIR} OPTIONAL)  
endif() 

# install header files
install(FILES ${SGTELIB_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/sgtelib
)