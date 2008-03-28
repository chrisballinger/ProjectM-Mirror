# the next line sets up include and link directories and defines some variables that we will use.
# you can modify the behavior by setting some variables, e.g.
project(projectM-qt) # the name of your project

cmake_minimum_required(VERSION 2.4.0)

include(FindPkgConfig.cmake)

if (${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")
	set(LIBPROJECTM_FOUND true)
else(${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")
	pkg_search_module(LIBPROJECTM REQUIRED libprojectM)
endif(${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")


find_package(Qt4 REQUIRED) # find and setup Qt4 for this project
include(${QT_USE_FILE})

if (QT4_FOUND)
MESSAGE(STATUS "Qt4 Detected fine.")
else (QT4_FOUND)
MESSAGE(FATAL_ERROR "Qt4 not detected. Please install Qt4 before continuing (visit www.trolltech.org for more info)")
endif(QT4_FOUND)
set(QT_USE_OPENGL TRUE)

if (LIBPROJECTM_FOUND)
	MESSAGE(STATUS "projectM detected.")
else (LIBPROJECTM_FOUND)
	MESSAGE(FATAL_ERROR "projectM not detected! Please install the projectM module or build from top level projectM source directory.")
endif(LIBPROJECTM_FOUND)


CONFIGURE_FILE("libprojectM-qt.pc.in" "libprojectM-qt.pc" @ONLY)

ADD_DEFINITIONS(-DLINUX -DPROJECTM_PREFIX='"${PROJECTM_PREFIX}"')
ADD_DEFINITIONS(${QT_DEFINITIONS})
ADD_DEFINITIONS(-DQT_PLUGIN)
ADD_DEFINITIONS(-DQT_NO_DEBUG)
ADD_DEFINITIONS(-DQT_SHARED)

# the variable "projectM-qt_SRCS" contains all .cpp files of this project
set(projectM-qt_SRCS
    qprojectm_mainwindow.cpp
    qprojectm_mainwindow.hpp
    configfile.hpp configfile.cpp
    qpresetfiledialog.hpp
    qplaylistfiledialog.cpp
    qplaylistfiledialog.hpp
    qplaylistmodel.cpp
    qplaylistmodel.hpp
    qxmlplaylisthandler.hpp
    qprojectmconfigdialog.cpp
    qprojectmconfigdialog.hpp
    qplaylisttableview.hpp
    qpresettextedit.cpp
    qpresettextedit.hpp
    qpreseteditordialog.cpp
    qpreseteditordialog.hpp
    qprojectm.hpp
    qprojectmwidget.hpp
)


set(projectM-qt_MOC_HDRS
    qprojectm_mainwindow.hpp
    qpresetfiledialog.hpp
    qplaylistfiledialog.hpp
    qplaylistmodel.hpp
    qprojectmconfigdialog.hpp
    qplaylisttableview.hpp
    qpresettextedit.hpp
    qpreseteditordialog.hpp
    qprojectm.hpp
    qprojectmwidget.hpp
)

set(projectM-qt_UIS
    qprojectm_mainwindow.ui
    qprojectmconfigdialog.ui
    qpreseteditordialog.ui
)

# Qt resource file
set( projectM-qt_RESOURCES
    application.qrc
)

# compile resources
qt4_add_resources( projectM-qt_RC_SRCS ${projectM-qt_RESOURCES} )

qt4_wrap_ui(projectM-qt_UIS_H ${projectM-qt_UIS})

# After this call, foo_MOC_SRCS = moc_Class1.cxx moc_Class2.cxx moc_Class3.cxx.
qt4_wrap_cpp(projectM-qt_MOC_SRCS ${projectM-qt_MOC_HDRS})

ADD_LIBRARY(projectM-qt SHARED ${projectM-qt_SRCS} ${projectM-qt_RC_SRCS} ${projectM-qt_MOC_SRCS} ${projectM-qt_UIS_H})

SET_TARGET_PROPERTIES(projectM-qt PROPERTIES VERSION 1.10 SOVERSION 1)

if (${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")
   set(PROJECTM_INCLUDE ${PROJECTM_ROOT_SOURCE_DIR}/projectM-engine)
   set(PROJECTM_LINK ${PROJECTM_ROOT_BINARY_DIR}/projectM-engine)
elseif (${CMAKE_PROJECT_NAME} MATCHES ${PROJECT_NAME})
   set(PROJECTM_INCLUDE ${LIBPROJECTM_INCLUDEDIR}/libprojectM)
   set(PROJECTM_LINK ${PROJECTM_LDFLAGS})
endif (${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")

LINK_DIRECTORIES(${PROJECTM_LINK})

include_directories(${PROJECTM_INCLUDE} ${QT_INCLUDE_DIR} ${QT_QTGUI_INCLUDE_DIR} ${QT_QTXML_INCLUDE_DIR} ${QT_QTOPENGL_INCLUDE_DIR} ${CMAKE_CURRENT_LIBRARY_DIR})

# link the "qprojectM" target against the Qt libraries. which libraries exactly, is defined by the "include(${QT_USE_FILE})" line above, which sets up this variable.
target_link_libraries(projectM-qt projectM ${QT_QTGUI_LIBRARIES} ${QT_QTOPENGL_LIBRARIES} ${QT_QTXML_LIBRARIES} ${QT_LIBRARIES})

install(FILES qprojectm_mainwindow.hpp DESTINATION include/libprojectM-qt)
install(TARGETS projectM-qt DESTINATION lib)
install(FILES "${CMAKE_BINARY_DIR}/libprojectM-qt.pc" DESTINATION ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}/pkgconfig)