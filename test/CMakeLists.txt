#
# Compiling/running tests

if(CN_CBOR_USE_CONTEXT)
  add_definitions(-DUSE_CBOR_CONTEXT)
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${dist_dir}/test)

function(create_test name)
  add_executable(${name}_test ${name}_test.c)
  target_link_libraries(${name}_test PRIVATE cn-cbor)
  target_include_directories(${name}_test PRIVATE ../include)
  add_test(NAME ${name} COMMAND ${name}_test)
endfunction()

create_test(cbor)
# create_test(file)
create_test(memory)
target_sources(cbor_test PRIVATE context.c)
target_sources(memory_test PRIVATE context.c)
include(CTest)

if(APPLE)
  # difftest uses Apple-specific memory tests
  add_executable(cn-test test.c)
  target_link_libraries(cn-test PRIVATE cn-cbor)

  configure_file(cases.cbor cases.cbor COPYONLY)
  configure_file(expected.out expected.out COPYONLY)

  add_custom_target(
    difftest
    COMMAND env MallocStackLogging=true ./cn-test >new.out
    COMMAND diff new.out expected.out
    DEPENDS cn-test
    WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
    COMMENT "generate differences between actual and expected output")

endif()
