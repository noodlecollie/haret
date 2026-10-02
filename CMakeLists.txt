cmake_minimum_required(VERSION 3.24)

project(haret LANGUAGES C CXX)

set(VERSION "" CACHE STRING "Build version override")

if(NOT CMAKE_CROSSCOMPILING)
	message(WARNING "Not cross-compiling, assuming we are building the CeGCC toolchain for this host")

	include(ExternalProject)
	set(INSTALL_STAGING "${CMAKE_BINARY_DIR}/cegcc-prefix/install/cegcc")
	ExternalProject_Add(cegcc
		GIT_REPOSITORY https://github.com/enlyze/cegcc-build.git
		GIT_TAG de5adf6f941275b405870f295e69fa7a29f59891 # Latest at the time (2026-10-02)
		GIT_SHALLOW YES
		GIT_PROGRESS YES

		# The repo isn't very good with separating out its configure, build and install commands,
		# so we just run everything as build and install to a local directory. The user can
		# then copy that directory wherever it needs to go, and this also relegates the possible
		# need for sudo to the install step only.
		CONFIGURE_COMMAND ""
		BUILD_COMMAND mkdir -p "${INSTALL_STAGING}" && cd "${CMAKE_BINARY_DIR}/cegcc-prefix/src/cegcc" && "./build_cf.sh" "${INSTALL_STAGING}" arm-mingw32ce
		INSTALL_COMMAND ""
		EXCLUDE_FROM_ALL
	)

	return()
endif()
