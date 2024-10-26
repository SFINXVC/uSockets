cmake_minimum_required(VERSION 3.15)

file(GLOB_RECURSE EXAMPLES
    *.c
)

foreach(EXAMPLE IN LISTS EXAMPLES)
    get_filename_component(EXAMPLE_NAME "${EXAMPLE}" NAME_WE)
    
    add_executable(${EXAMPLE_NAME} ${EXAMPLE})

    target_link_libraries(${EXAMPLE_NAME} PRIVATE uSockets)
endforeach()