INCLUDE_DIRECTORIES(".")

# list sources files
FILE(GLOB LIBRARY_FILES_C "*.c" "*.tc")
FILE(GLOB LIBRARY_FILES_H "*.h")
FILE(GLOB LIBRARY_FILES_TH "*.th")

c_library_with_type_no_pch(vlfeat "" "" "${c_default}" ${LIBRARY_FILES_C} ${LIBRARY_FILES_H} ${LIBRARY_FILES_TH})

INSTALL(FILES ${LIBRARY_FILES_H} DESTINATION "include/vlfeat")
INSTALL(TARGETS vlfeat DESTINATION "lib")
