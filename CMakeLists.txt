#=============================================================================
# Setup project
#=============================================================================
project(dbg CXX)
cmake_minimum_required(VERSION 2.8)


#=============================================================================
# Setup CMake options
#=============================================================================
include(CMakeDependentOption)
option(DBG_ENABLE_PEDANTIC   "Compile with pedantic enabled." ON)
option(DBG_ENABLE_WERROR     "Fail and stop if a warning is triggered." OFF)
option(DBG_ENABLE_CXX0X      "Enable -std=c++0x and use of c++0x language features if the compiler supports it." OFF)
option(DBG_ENABLE_LIBCXX     "Enable -stdlib=libc++ if the compiler supports it." OFF)


#=============================================================================
# Check available compiler flags
#=============================================================================
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag(-pedantic                  DBG_HAS_PEDANTIC_FLAG)
check_cxx_compiler_flag(-std=c++0x                 DBG_HAS_STDCXX0X_FLAG)
check_cxx_compiler_flag(-stdlib=libc++             DBG_HAS_STDLIB_LIBCXX_FLAG)
check_cxx_compiler_flag(-W                         DBG_HAS_W_FLAG)
check_cxx_compiler_flag(-Wall                      DBG_HAS_WALL_FLAG)
check_cxx_compiler_flag(-Werror                    DBG_HAS_WERROR_FLAG)
check_cxx_compiler_flag(-Wextra                    DBG_HAS_WEXTRA_FLAG)
check_cxx_compiler_flag(-Wno-long-long             DBG_HAS_WNO_LONG_LONG_FLAG)
check_cxx_compiler_flag(-Wno-unused-local-typedefs DBG_HAS_WNO_UNUSED_LOCAL_TYPEDEFS_FLAG)
check_cxx_compiler_flag(-Wno-unused-parameter      DBG_HAS_WNO_UNUSED_PARAMETER_FLAG)
check_cxx_compiler_flag(-Wwrite-strings            DBG_HAS_WWRITE_STRINGS_FLAG)


#=============================================================================
# Setup warning flags
#=============================================================================
macro(append_if condition lst var)
    if (${condition})
        list(APPEND ${lst} ${var})
    endif()
endmacro()

append_if(DBG_HAS_WALL_FLAG                      DBG_CXX_WARNING_FLAGS -Wall)
append_if(DBG_HAS_WEXTRA_FLAG                    DBG_CXX_WARNING_FLAGS -Wextra)
append_if(DBG_HAS_W_FLAG                         DBG_CXX_WARNING_FLAGS -W)
append_if(DBG_HAS_WNO_UNUSED_LOCAL_TYPEDEF_FLAG  DBG_CXX_WARNING_FLAGS -Wno-unused-local-typedefs)
append_if(DBG_HAS_WNO_UNUSED_PARAMETER_FLAG      DBG_CXX_WARNING_FLAGS -Wno-unused-parameter)
append_if(DBG_HAS_WWRITE_STRINGS_FLAG            DBG_CXX_WARNING_FLAGS -Wwrite-strings)
append_if(DBG_HAS_WNO_LONG_LONG_FLAG             DBG_CXX_WARNING_FLAGS -Wno-long-long)

if (DBG_ENABLE_WERROR)
    append_if(DBG_HAS_WERROR_FLAG                DBG_CXX_WARNING_FLAGS -Werror)
    append_if(DBG_HAS_WX_FLAG                    DBG_CXX_WARNING_FLAGS -WX)
endif()

if (DBG_ENABLE_PEDANTIC)
    append_if(DBG_HAS_PEDANTIC_FLAG              DBG_CXX_WARNING_FLAGS -pedantic)
endif()


#=============================================================================
# Setup feature flags
#=============================================================================
# Compile with c++0x or not
if (DBG_ENABLE_CXX0X AND DBG_HAS_STDCXX0X_FLAG)
    list(APPEND DBG_CXX_FEATURE_FLAGS -std=c++0x)
endif()

# Whether to use libc++
if (DBG_ENABLE_LIBCXX AND DBG_HAS_STDLIB_LIBCXX_FLAG)
    list(APPEND DBG_CXX_FEATURE_FLAGS -stdlib=libc++)
    # the flag has to be passed to the linker too.
    set(CMAKE_EXE_LINKER_FLAGS -stdlib=libc++)
    set(CMAKE_SHARED_LINKER_FLAGS -stdlib=libc++)
    set(CMAKE_MODULE_LINKER_FLAGS -stdlib=libc++)
endif()

# This is the only place where add_definitions is called.
add_definitions(
    ${DBG_CXX_WARNING_FLAGS}
    ${DBG_CXX_FEATURE_FLAGS}
    -DDBG_DONT_USE_FUNGO
)


#=============================================================================
# Setup the dbg library
#=============================================================================
include_directories(include)
add_subdirectory(src)
