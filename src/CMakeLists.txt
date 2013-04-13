# Gather the right source files depending on the platform.
file(GLOB DBG_COMMON_SOURCES ${dbg_SOURCE_DIR}/src/*.cpp)
file(GLOB DBG_OSX_SOURCES    ${dbg_SOURCE_DIR}/src/osx/*.cpp)
file(GLOB DBG_MSVC_SOURCES   ${dbg_SOURCE_DIR}/src/windows/*.cpp
                             ${dbg_SOURCE_DIR}/src/windows/msvc/*.cpp)
file(GLOB DBG_MINGW_SOURCES  ${dbg_SOURCE_DIR}/src/windows/*.cpp
                             ${dbg_SOURCE_DIR}/src/windows/mingw/*.cpp)

if (MSVC)
    set(DBG_SOURCES ${DBG_COMMON_SOURCES} ${DBG_MSVC_SOURCES})
elseif (MINGW)
    set(DBG_SOURCES ${DBG_COMMON_SOURCES} ${DBG_MINGW_SOURCES})
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
   set(DBG_SOURCES ${DBG_COMMON_SOURCES} ${DBG_OSX_SOURCES})
else()
    message(FATAL_ERROR "Building on your system is not supported right now.")
endif()


# Add all the headers to the project for IDEs.
if (MSVC_IDE OR XCODE)
    file(GLOB_RECURSE DBG_HEADERS ${dbg_SOURCE_DIR}/include/*.hpp)
endif()

add_library(dbg STATIC ${DBG_SOURCES} ${DBG_HEADERS})


#=============================================================================
# Setup the install target
#=============================================================================
install(TARGETS dbg
    COMPONENT dbg
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

install(DIRECTORY ${dbg_SOURCE_DIR}/include/
    DESTINATION include
    COMPONENT dbg
    FILES_MATCHING PATTERN *.hpp
)
