project(incbin)
add_library(incbin STATIC incbin.h incbin.c)
target_include_directories(incbin PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

project(incbin-tool)
add_executable(incbin-tool incbin.c)
target_include_directories(incbin-tool PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})