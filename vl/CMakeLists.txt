INCLUDE_DIRECTORIES(".")

# list sources files
FILE(GLOB LIBRARY_FILES_C "*.c" "*.tc" "*.cpp")
FILE(GLOB LIBRARY_FILES_H "*.h" "*.th" "*.inl")

cxx_library_with_type(VLFeatLib "" "" "${cxx_default}" ${LIBRARY_FILES_C} ${LIBRARY_FILES_H})
SET_TARGET_PROPERTIES(VLFeatLib PROPERTIES LINKER_LANGUAGE C)
