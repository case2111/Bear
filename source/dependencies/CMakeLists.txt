set(JSON_BuildTests OFF CACHE INTERNAL "")
set(JSON_Install OFF CACHE INTERNAL "")
add_subdirectory(json)

set(INSTALL_GTEST OFF CACHE INTERNAL "")
add_subdirectory(googletest)
