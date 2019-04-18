cmake_minimum_required(VERSION 2.6)
project("glew")

set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(RC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/build)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

option (glew-cmake_BUILD_SHARED "Build the shared glew library" ON)
option (glew-cmake_BUILD_STATIC "Build the static glew library" ON)
option (glew-cmake_BUILD_SINGLE_CONTEXT "Build the single context glew library" ON)
option (glew-cmake_BUILD_MULTI_CONTEXT "Build the multi context glew library" ON)


set(LIBGLEW_SRCS ${SRC_DIR}/glew.c)

if (WIN32)
	add_definitions(-DWIN32_MEAN_AND_LEAN -DVC_EXTRALEAN -DGLEW_BUILD -D_CRT_SECURE_NO_WARNINGS)
endif ()

if (MSVC)
	list(APPEND LIBGLEW_SRCS ${RC_DIR}/glew.rc)
endif ()

if (NOT USE_GLU)
	add_definitions(-DGLEW_NO_GLU)
endif ()

if (WIN32)
	set(GL_LIBRARY opengl32)
elseif (APPLE)
	find_library(GL_LIBRARY OpenGL)
	find_library(AGL_LIBRARY AGL)
	set(PLATFORM_SPECIFIC_LIBRARIES ${AGL_LIBRARY})
else()
	set(GL_LIBRARY GL)
	set(PLATFORM_SPECIFIC_LIBRARIES X11 dl Xext)
endif ()

if (glew-cmake_BUILD_STATIC)
	if (glew-cmake_BUILD_SINGLE_CONTEXT)
		add_library(glew-static STATIC ${LIBGLEW_SRCS})

		set_target_properties(glew-static PROPERTIES
		                       ARCHIVE_OUTPUT_NAME "glew")
		target_include_directories(glew-static PRIVATE
                                                        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                                  PUBLIC
						  	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                                        $<INSTALL_INTERFACE:include/GL>)
		target_link_libraries(glew-static ${GL_LIBRARY} ${PLATFORM_SPECIFIC_LIBRARIES})
        install(TARGETS glew-static
                EXPORT glew-static-targets
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
               )
        install(EXPORT glew-static-targets
                NAMESPACE glew::
                DESTINATION lib/cmake/glew)
    endif ()

    if (glew-cmake_BUILD_MULTI_CONTEXT)
        add_library(glewmx-static STATIC ${LIBGLEW_SRCS})

        set_target_properties(glewmx-static PROPERTIES
                               ARCHIVE_OUTPUT_NAME "glewmx")
        target_compile_definitions(glewmx-static INTERFACE GLEW_MX)
        target_include_directories(glewmx-static PRIVATE
                                                        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                                    PUBLIC
						  	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                                        $<INSTALL_INTERFACE:include/GL>)
		target_link_libraries(glewmx-static ${GL_LIBRARY} ${PLATFORM_SPECIFIC_LIBRARIES})
		install(TARGETS glewmx-static
                EXPORT glewmx-static-targets
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
               )
		install(EXPORT glewmx-static-targets
                NAMESPACE glew::
                DESTINATION lib/cmake/glew)
	endif ()
endif ()

if (glew-cmake_BUILD_SHARED)
	if (glew-cmake_BUILD_SINGLE_CONTEXT)
		add_library(glew SHARED ${LIBGLEW_SRCS})

		set_target_properties(glew PROPERTIES
		                      RUNTIME_OUTPUT_NAME "glew"
		                      LIBRARY_OUTPUT_NAME "glew")

		target_include_directories(glew PRIVATE
                                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                        PUBLIC
						  	                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                            $<INSTALL_INTERFACE:include/GL>)
		target_link_libraries(glew ${GL_LIBRARY} ${PLATFORM_SPECIFIC_LIBRARIES})
		install(TARGETS glew
                EXPORT glew-targets
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
                LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
                RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
               )
		install(EXPORT glew-targets
                NAMESPACE glew::
                DESTINATION lib/cmake/glew)
	endif ()

	if (glew-cmake_BUILD_MULTI_CONTEXT)
		add_library(glewmx SHARED ${LIBGLEW_SRCS})

		set_target_properties(glewmx PROPERTIES
		                       RUNTIME_OUTPUT_NAME "glewmx"
		                       LIBRARY_OUTPUT_NAME "glewmx")

		target_compile_definitions(glewmx INTERFACE GLEW_MX)
		target_include_directories(glewmx PRIVATE
                                                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                          PUBLIC
						  	                    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                                $<INSTALL_INTERFACE:include/GL>)
		target_link_libraries(glewmx ${GL_LIBRARY} ${PLATFORM_SPECIFIC_LIBRARIES})
		install(TARGETS glewmx
                EXPORT glewmx-targets
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
                LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
                RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
               )
		install(EXPORT glewmx-targets
                NAMESPACE glew::
                DESTINATION lib/cmake/glew)
    endif ()
endif ()

file(GLOB PUBLIC_HEADERS "include/GL/*.h")
install(FILES ${PUBLIC_HEADERS} DESTINATION include/GL/)

if (${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${CMAKE_SOURCE_DIR} AND NOT ONLY_LIBS)
    set(GLEWINFO_SRCS ${SRC_DIR}/glewinfo.c)
    set(VISUALINFO_SRCS ${SRC_DIR}/visualinfo.c)
    if (MSVS)
        list(APPEND GLEWINFO_SRCS ${RC_DIR}/glewinfo.rc)
        list(APPEND VISUALINFO_SRCS ${RC_DIR}/visualinfo.rc)
    endif ()

    add_executable(glewinfo ${GLEWINFO_SRCS})
    add_executable(visualinfo ${VISUALINFO_SRCS})

    if (glew-cmake_BUILD_STATIC)
        target_link_libraries(glewinfo glew_static)
        target_link_libraries(visualinfo glew_static)
    else ()
        target_link_libraries(glewinfo glew_shared)
        target_link_libraries(visualinfo glew_shared)
    endif ()

    install(TARGETS glewinfo visualinfo DESTINATION bin)
endif ()
