project(incbin)
add_library(incbin STATIC incbin.h incbin.c)
target_include_directories(incbin PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

project(incbin-tool)
add_executable(incbin-tool incbin.c)
target_include_directories(incbin-tool PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

# Wrapper for incbin
# target - name of the target that requires incbin embedding
# sources - list of sources to that contain incbin calls
# (optional) prefix - prefix to be added to the embedded data (default: embedded_)
# (optional) style - snake_case or camelCase (default: snake_case)
function(incbin_include target sources)

  foreach(arg IN LISTS ARGN)
    if(EXISTS ${arg} OR IS_ABSOLUTE ${arg})
      list(APPEND sources ${arg})
    elseif (arg STREQUAL camelCase)
      set(style INCBIN_STYLE_CAMEL)
      set(style_tool camelcase)
    elseif (arg STREQUAL snake_case)
      set(style INCBIN_STYLE_SNAKE)
      set(style_tool snakecase)
    else()
      set(prefix ${arg})
    endif()
  endforeach()
  
  if (NOT DEFINED prefix)
    set(prefix embedded_)
  endif()

  if (NOT DEFINED style)
    set(style INCBIN_STYLE_SNAKE)
    set(style_tool snakecase)
  endif()

  target_link_libraries(${target} PRIVATE incbin)
  target_compile_definitions(${target} PUBLIC -DINCBIN_PREFIX=${prefix} -DINCBIN_STYLE=${style})

  if(NOT MSVC)
    return()
  endif()

  set(i 0)

  foreach(source ${sources})
    
    get_filename_component(source_dir ${source} DIRECTORY)
    get_filename_component(source_name ${source} NAME_WE)
    get_filename_component(source_ext ${source} EXT)

    set(destination "${CMAKE_BINARY_DIR}/incbin/${source_name}.${i}${source_ext}")

    add_custom_command(
      OUTPUT ${destination}
      COMMAND incbin-tool ${source} -o ${destination} -p ${prefix} -S${style_tool}
      WORKING_DIRECTORY ${source_dir}
      VERBATIM
    )

    target_sources(${target} PRIVATE ${destination})

    math(EXPR i "${i}+1")

  endforeach()

endfunction()
