# ----------------------------------------------------------------------------
#  ��Ŀ¼��CMake�ļ�
#
#  fengbing <fengbing123@gmail.com>
#
#  ע��CMake�������� http://www.cmake.org/
# ----------------------------------------------------------------------------

CMAKE_MINIMUM_REQUIRED (VERSION 2.8)#�趨��С��CMake�汾��

PROJECT (OpenSLAM)#�趨��������

LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake") # ���ӱ���CMake�ű�Ŀ¼

INCLUDE(cmake/script_version_number.cmake REQUIRED)#������Ŀ�汾��

#����·��
SET(OPENSLAM_LIBS_ROOT "${CMAKE_SOURCE_DIR}/libs" CACHE INTERNAL "") 
SET(OPENSLAM_SOURCE_DIR  ${CMAKE_CURRENT_SOURCE_DIR})
SET(OPENSLAM_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
SET(OPENSLAM_BINARY_DIR  ${PROJECT_BINARY_DIR})

#�������õĺ�
INCLUDE(cmake/UtilsMacros.cmake REQUIRED)# ����һЩ�й��ߺ�
INCLUDE(cmake/FilterFileLists.cmake REQUIRED)#�����ļ�ɸѡ�����ں��ڽ��п�ƽ̨
INCLUDE(cmake/DeclareOPENSLAMLib.cmake REQUIRED)#�����Զ�������Ŀ

set(CMAKE_DEBUG_POSTFIX  "d")#����Debug��lib�����ƺ�����d����openslam_XXX100d.lib

#������Ŀ������ɸѡ��
IF (CMAKE_VERSION VERSION_GREATER 2.8.0)
	SET(ENABLE_SOLUTION_FOLDERS ON CACHE BOOL "Group projects under virtual folders of the compiler IDE (e.g. VisualStudio)")
ENDIF (CMAKE_VERSION VERSION_GREATER 2.8.0)

if (ENABLE_SOLUTION_FOLDERS)
  set_property(GLOBAL PROPERTY USE_FOLDERS ON)
  set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMakeTargets")
endif (ENABLE_SOLUTION_FOLDERS)

#����libs
SET(OpenCV_STATIC OFF)
FIND_PACKAGE(OpenCV REQUIRED) #ͨ��CMakeϵͳ����OpenCV
#FIND_PACKAGE(g2o REQUIRED) #ͨ��CMakeϵͳ����g2o
#FIND_PACKAGE(Pangolin REQUIRED)

#include(cmake/script_eigen.cmake REQUIRED)  #����Eigen


if(MSVC)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")  # ���ж�˱���
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /O1")
endif()

# ����c++ 11��׼֧��
if(MSVC AND (MSVC10 OR MSVC11 OR MSVC12))
   set(COMPILER_SUPPORTS_CXX11 1)
endif()

#������̬�⻹�Ƕ�̬�⣬Ŀǰ������Ϊ��̬�⣬���ڸ���ΪĬ�϶�̬��
SET(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared libraries (.dll/.so) instead of static ones (.lib/.a)")
IF(BUILD_SHARED_LIBS)
	SET(CMAKE_OPENSLAM_BUILD_SHARED_LIB "#define OPENSLAM_BUILT_AS_DLL")
	SET(CMAKE_OPENSLAM_BUILD_SHARED_LIB_ONOFF 1)
ELSE(BUILD_SHARED_LIBS)
	SET(CMAKE_OPENSLAM_BUILD_SHARED_LIB "/* #define OPENSLAM_BUILT_AS_DLL */")
	SET(CMAKE_OPENSLAM_BUILD_SHARED_LIB_ONOFF 0)
ENDIF(BUILD_SHARED_LIBS)

include(cmake/script_create_config_h.cmake REQUIRED)   # Build config.h
include(cmake/script_create_version_h.cmake REQUIRED)  # Build version.h

#����include��link·��
INCLUDE_DIRECTORIES(".")#���ð�����ǰĿ¼
INCLUDE_DIRECTORIES(${OPENSLAM_CONFIG_FILE_INCLUDE_DIR})#����ȫ�������ļ�config.h
INCLUDE_DIRECTORIES(${Pangolin_INCLUDE_DIRS})
LINK_DIRECTORIES("${CMAKE_BINARY_DIR}/lib")

#����·��������lib��bin��ͬһĿ¼
SET( LIBRARY_OUTPUT_PATH ${OPENSLAM_BINARY_DIR}/lib CACHE PATH "Output directory for libraries" )
SET( EXECUTABLE_OUTPUT_PATH ${OPENSLAM_BINARY_DIR}/bin CACHE PATH "Output directory for applications" )

# ----------------------------------------------------------------------------
#      					������Ŀ¼
# ----------------------------------------------------------------------------
ADD_SUBDIRECTORY(3rdparty)   # ���������·��
ADD_SUBDIRECTORY(libs)        # ����C++ lib��·��

SET( BUILD_APPLICATIONS ON CACHE BOOL "If you only want the OPENSLAM libraries, disable this.")
IF(BUILD_APPLICATIONS)
	ADD_SUBDIRECTORY(apps)    # ����СӦ�ó���Ŀ¼
ENDIF(BUILD_APPLICATIONS)

ADD_SUBDIRECTORY(doc) #�����ĵ�Ŀ¼

#   ���ӵ�Ԫ����
# ----------------------------------------------------------------------------
SET( BUILD_TESTING ON CACHE BOOL "Build OPENSLAM tests")
IF( BUILD_TESTING)
	ADD_SUBDIRECTORY(tests)  # ���ӵ�Ԫ����
ENDIF( BUILD_TESTING)