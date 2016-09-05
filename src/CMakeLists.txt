# list sources files
FILE(GLOB LIBRARY_FILES_C "sift.c")
FILE(GLOB LIBRARY_FILES_H "")

cxx_executable_with_flags(sift "Apps" "${cxx_default}" "vlfeat" ${LIBRARY_FILES_C} ${LIBRARY_FILES_H})

if(UNIX OR MINGW)
	target_link_libraries(sift m)
endif()

# Install
INSTALL(TARGETS sift
	EXPORT vlfeatTargets
	RUNTIME DESTINATION "${INSTALL_BIN_DIR}" COMPONENT bin)
