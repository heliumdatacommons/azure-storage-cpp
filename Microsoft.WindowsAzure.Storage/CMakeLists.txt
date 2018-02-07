set(CMAKE_LEGACY_CYGWIN_WIN32 0)
cmake_minimum_required(VERSION 2.6)
project(azurestorage)

enable_testing()

set(WARNINGS)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

option(BUILD_TESTS "Build test codes" OFF)
option(BUILD_SAMPLES "Build sample codes" OFF)

# Platform (not compiler) specific settings
if(UNIX)
if(CMAKE_COMPILE_FOR_IRODS)
   find_package(IRODS 4.2.2 EXACT REQUIRED)

   if (NOT IRODS_EXTERNALS_PACKAGE_ROOT)
     set(IRODS_EXTERNALS_PACKAGE_ROOT "/opt/irods-externals" CACHE STRING "Choose the location of iRODS external packages." FORCE)
     message(STATUS "Setting unspecified IRODS_EXTERNALS_PACKAGE_ROOT to '${IRODS_EXTERNALS_PACKAGE_ROOT}'. This is the correct setting for normal builds.")
   endif()
   macro(IRODS_MACRO_CHECK_DEPENDENCY_SET_FULLPATH_ADD_TO_IRODS_PACKAGE_DEPENDENCIES_LIST DEPENDENCY_NAME DEPENDENCY_SUBDIRECTORY)
  IRODS_MACRO_CHECK_DEPENDENCY_SET_FULLPATH(${DEPENDENCY_NAME} ${DEPENDENCY_SUBDIRECTORY})
  list(APPEND IRODS_PACKAGE_DEPENDENCIES_LIST irods-externals-${DEPENDENCY_SUBDIRECTORY})
  endmacro()
  string(REPLACE ";" ", " IRODS_PACKAGE_DEPENDENCIES_STRING "${IRODS_PACKAGE_DEPENDENCIES_LIST}")

  set(CMAKE_C_COMPILER ${IRODS_EXTERNALS_FULLPATH_CLANG}/bin/clang)
  set(CMAKE_CXX_COMPILER ${IRODS_EXTERNALS_FULLPATH_CLANG}/bin/clang++)
  set(CMAKE_EXE_LINKER_FLAGS_INIT "${CMAKE_EXE_LINKER_FLAGS} -stdlib=libc++")
  include(${IRODS_TARGETS_PATH})
# Build up the Boost_LIBRARIES string
  set(Boost_LIBRARIES  "${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_log.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_log_setup.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_random.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_system.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_thread.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_locale.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_regex.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_filesystem.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_chrono.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_date_time.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_atomic.so")
  set(Boost_LIBRARIES  "${Boost_LIBRARIES};${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_thread.so")
  set(Boost_INCLUDE_DIR "${IRODS_EXTERNALS_FULLPATH_BOOST}/include")
  find_package(Threads REQUIRED)
  
else()
  find_package(Boost REQUIRED COMPONENTS log log_setup random system thread locale regex filesystem chrono date_time)
  find_package(Threads REQUIRED)
endif()
  if(APPLE AND NOT OPENSSL_ROOT_DIR)
    # Prefer a homebrew version of OpenSSL over the one in /usr/lib
    file(GLOB OPENSSL_ROOT_DIR /usr/local/Cellar/openssl/*)

	# Prefer the latest (make the latest one first)
    list(REVERSE OPENSSL_ROOT_DIR)

	# There is a dependency chain Libxml++ -> glibmm -> gobject -> glib -> lintl, however, for some reason,
	# with homebrew at least, the -L for where lintl resides is left out. So, we try to find it where homebrew
	# would put it, or allow the user to specify it
	if(NOT GETTEXT_LIB_DIR)
		message(WARNING "No GETTEXT_LIB_DIR specified, assuming: /usr/local/opt/gettext/lib")
		set(GETTEXT_LIB_DIR "/usr/local/opt/gettext/lib")
	endif()
	# If we didn't find it where homebrew would put it, and it hasn't been specified, then we have to throw an error
	if(NOT IS_DIRECTORY "${GETTEXT_LIB_DIR}")
		message(ERROR "We couldn't find your gettext lib directory (${GETTEXT_LIB_DIR}). Please re-run cmake with -DGETTEXT_LIB_DIR=<your gettext lib dir>. This is usually where libintl.a and libintl.dylib reside.")
	endif()

	# if we actually have a GETTEXT_LIB_DIR we add the linker flag for it
	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -L${GETTEXT_LIB_DIR}")
  endif()

  set(_OPENSSL_VERSION "")
  find_package(OpenSSL 1.0.0 REQUIRED)


  find_package(Glibmm REQUIRED)
  find_package(LibXML++ REQUIRED)
  find_package(UUID REQUIRED)
  find_package(Casablanca REQUIRED)

  if(BUILD_TESTS)
    find_package(UnitTest++ REQUIRED)
  endif()

  option(BUILD_SHARED_LIBS "Build shared Libraries." ON)

  file(GLOB WAS_HEADERS includes/was/*.h)
  install(FILES ${WAS_HEADERS} DESTINATION include/was)
  file(GLOB WASCORE_HEADERS includes/wascore/*.h)
  install(FILES ${WASCORE_HEADERS} DESTINATION include/wascore)
  file(GLOB WASCORE_DATA includes/wascore/*.dat)
  install(FILES ${WASCORE_DATA} DESTINATION include/wascore)
else()
  message("-- Unsupported Build Platform.")
endif()

# Compiler (not platform) specific settings
if(CMAKE_COMPILE_FOR_IRODS)
   find_package(IRODS 4.2.2 EXACT REQUIRED)

   if (NOT IRODS_EXTERNALS_PACKAGE_ROOT)
     set(IRODS_EXTERNALS_PACKAGE_ROOT "/opt/irods-externals" CACHE STRING "Choose the location of iRODS external packages." FORCE)
     message(STATUS "Setting unspecified IRODS_EXTERNALS_PACKAGE_ROOT to '${IRODS_EXTERNALS_PACKAGE_ROOT}'. This is the correct setting for normal builds.")
   endif()
   macro(IRODS_MACRO_CHECK_DEPENDENCY_SET_FULLPATH_ADD_TO_IRODS_PACKAGE_DEPENDENCIES_LIST DEPENDENCY_NAME DEPENDENCY_SUBDIRECTORY)
  IRODS_MACRO_CHECK_DEPENDENCY_SET_FULLPATH(${DEPENDENCY_NAME} ${DEPENDENCY_SUBDIRECTORY})
  list(APPEND IRODS_PACKAGE_DEPENDENCIES_LIST irods-externals-${DEPENDENCY_SUBDIRECTORY})
endmacro()
string(REPLACE ";" ", " IRODS_PACKAGE_DEPENDENCIES_STRING "${IRODS_PACKAGE_DEPENDENCIES_LIST}")

set(CMAKE_C_COMPILER ${IRODS_EXTERNALS_FULLPATH_CLANG}/bin/clang)
set(CMAKE_CXX_COMPILER ${IRODS_EXTERNALS_FULLPATH_CLANG}/bin/clang++)
set(CMAKE_EXE_LINKER_FLAGS_INIT "${CMAKE_EXE_LINKER_FLAGS} -stdlib=libc++")
   include(${IRODS_TARGETS_PATH})
  message("-- Setting clang options using the iRods external lib")
  if (NOT IRODS_EXTERNALS_CLANG)
    set(IRODS_EXTERNALS_CLANG "/opt/irods-externals/clang3.8-0" CACHE STRING "Choose the location of the iRODS clang compiler." FORCE)
    message(STATUS "Setting unspecified IRODS_EXTERNALS_CLANG to '${IRODS_EXTERNALS_CLANG}'.")
endif()
    message("-- Setting clang options")

    set(WARNINGS "-Wall -Wextra -Wcast-qual -Wconversion -Wformat=2 -Winit-self -Winvalid-pch -Wmissing-format-attribute -Wmissing-include-dirs -Wpacked -Wredundant-decls")
    set(OSX_SUPPRESSIONS "-Wno-overloaded-virtual -Wno-sign-conversion -Wno-deprecated -Wno-unknown-pragmas -Wno-reorder -Wno-char-subscripts -Wno-switch -Wno-unused-parameter -Wno-unused-variable -Wno-deprecated -Wno-unused-value -Wno-unknown-warning-option -Wno-return-type-c-linkage -Wno-unused-function -Wno-sign-compare -Wno-shorten-64-to-32 -Wno-reorder -Wno-unused-local-typedefs")
    set(WARNINGS "${WARNINGS} ${OSX_SUPPRESSIONS}")

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -Wno-return-type-c-linkage -Wno-unneeded-internal-declaration")
    set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++")
    set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++11")

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fno-strict-aliasing")
    if (BUILD_SHARED_LIBS)
        add_definitions(-DBOOST_LOG_DYN_LINK)
    endif()
    add_definitions(-D_TURN_OFF_PLATFORM_STRING)

elseif("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU")
  message("-- Setting gcc options")

  set(WARNINGS "-Wall -Wextra -Wunused-parameter -Wcast-align -Wcast-qual -Wconversion -Wformat=2 -Winit-self -Winvalid-pch -Wmissing-format-attribute -Wmissing-include-dirs -Wpacked -Wredundant-decls -Wunreachable-code")
  set(LINUX_SUPPRESSIONS "-Wno-deprecated -Wno-unknown-pragmas -Wno-reorder -Wno-unused-function -Wno-char-subscripts -Wno-switch -Wno-unused-but-set-parameter -Wno-unused-value -Wno-unused-local-typedefs -Wno-unused-parameter")
  set(WARNINGS "${WARNINGS} ${LINUX_SUPPRESSIONS}")

  set(LD_FLAGS "${LD_FLAGS} -Wl,-z,defs")

  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fno-strict-aliasing")

  set(STRICT_CXX_FLAGS ${WARNINGS} "-Werror -pedantic")

  if (BUILD_SHARED_LIBS)
    add_definitions(-DBOOST_LOG_DYN_LINK)
  endif()
  add_definitions(-D_TURN_OFF_PLATFORM_STRING)
elseif((CMAKE_CXX_COMPILER_ID MATCHES "Clang"))
	message("-- Setting clang options")

	set(WARNINGS "-Wall -Wextra -Wcast-qual -Wconversion -Wformat=2 -Winit-self -Winvalid-pch -Wmissing-format-attribute -Wmissing-include-dirs -Wpacked -Wredundant-decls")
	set(OSX_SUPPRESSIONS "-Wno-overloaded-virtual -Wno-sign-conversion -Wno-deprecated -Wno-unknown-pragmas -Wno-reorder -Wno-char-subscripts -Wno-switch -Wno-unused-parameter -Wno-unused-variable -Wno-deprecated -Wno-unused-value -Wno-unknown-warning-option -Wno-return-type-c-linkage -Wno-unused-function -Wno-sign-compare -Wno-shorten-64-to-32 -Wno-reorder -Wno-unused-local-typedefs")
	set(WARNINGS "${WARNINGS} ${OSX_SUPPRESSIONS}")

	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -Wno-return-type-c-linkage -Wno-unneeded-internal-declaration")
	set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++")
	set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++11")

	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fno-strict-aliasing")
	if (BUILD_SHARED_LIBS)
		add_definitions(-DBOOST_LOG_DYN_LINK)
	endif()
	add_definitions(-D_TURN_OFF_PLATFORM_STRING)
else()
  message("-- Unknown compiler, success is doubtful.")
endif()

# Reconfigure final output directory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/Binaries)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/Binaries)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/Binaries)

set(AZURESTORAGE_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/includes)
set(AZURESTORAGE_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/includes ${CASABLANCA_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS} ${OPENSSL_INCLUDE_DIRS} ${LibXML++_INCLUDE_DIRS} ${UUID_INCLUDE_DIRS} ${Glibmm_INCLUDE_DIRS})

message("Boost_LIBRARIES: ${Boost_LIBRARIES}")
message("Boost_FRAMEWORK: ${Boost_FRAMEWORK}")
set(AZURESTORAGE_LIBRARY azurestorage)
set(AZURESTORAGE_LIBRARIES ${AZURESTORAGE_LIBRARY} ${CASABLANCA_LIBRARIES} ${Boost_LIBRARIES} ${Boost_FRAMEWORK} ${OPENSSL_LIBRARIES} ${LibXML++_LIBRARIES} ${UUID_LIBRARIES} ${Glibmm_LIBRARIES})

# Set version numbers centralized
set (AZURESTORAGE_VERSION_MAJOR 3)
set (AZURESTORAGE_VERSION_MINOR 1)
set (AZURESTORAGE_VERSION_REVISION 0)

# Add sources per configuration
add_subdirectory(src)

if(BUILD_TESTS)
  set(AZURESTORAGE_LIBRARY_TEST azurestoragetest)
  add_subdirectory(tests)
endif()

if(BUILD_SAMPLES)
  add_subdirectory(samples)
endif()
