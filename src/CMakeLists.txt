# list sources files
FILE(GLOB LIBRARY_FILES_C "sift.c")
FILE(GLOB LIBRARY_FILES_H "")

cxx_executable_with_flags(SIFT "Apps" "${cxx_default}" "VLFeatLib" WIN32 ${LIBRARY_FILES_C} ${LIBRARY_FILES_H})
