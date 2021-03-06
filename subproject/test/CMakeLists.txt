cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

project(jkj_fp_tests LANGUAGES CXX)

# ---- Add dependencies ----

# We want warnings in tests
set(jkj_fp_INCLUDE_WITHOUT_SYSTEM ON CACHE INTERNAL "")

include(FetchContent)
if (NOT TARGET jkj_fp)
    FetchContent_Declare(jkj_fp SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/../..")
    FetchContent_MakeAvailable(jkj_fp)
endif()
if (NOT TARGET common)
    FetchContent_Declare(common SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/../common")
    FetchContent_MakeAvailable(common)
endif()
if (NOT TARGET ryu)
    FetchContent_Declare(ryu SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/../3rdparty/ryu")
    FetchContent_MakeAvailable(ryu)
endif()

# ---- Tests ----

enable_testing()

function(add_test NAME)
  cmake_parse_arguments(TEST "CHARCONV;RYU" "NAME" "" ${ARGN})
  if(TEST_CHARCONV)
    set(jkj_fp jkj_fp::charconv)
  else()
    set(jkj_fp jkj_fp::fp)
  endif()

  add_executable(${NAME} source/${NAME}.cpp)

  target_link_libraries(${NAME} PRIVATE ${jkj_fp} jkj_fp::common)

  if(TEST_RYU)
    target_link_libraries(${NAME} PRIVATE ryu::ryu)
  endif()

  target_compile_features(${NAME} PRIVATE cxx_std_17)

  _add_test(NAME ${NAME} COMMAND ${NAME})

  # ---- MSVC Specifics ----
  if (MSVC)
      # No need to not generate PDB
      # /permissive- should be the default
      # The compilation will fail without /experimental:newLambdaProcessor
      # See also https://gitlab.kitware.com/cmake/cmake/-/issues/16478
      target_compile_options(${NAME} PUBLIC
              /Zi /permissive-
              $<$<NOT:$<CXX_COMPILER_ID:Clang>>:/experimental:newLambdaProcessor>
              $<$<CONFIG:Release>:/GL>)
      target_link_options(${NAME} PUBLIC $<$<CONFIG:Release>:/LTCG> /DEBUG:FASTLINK)
      set_target_properties(${NAME} PROPERTIES 
            VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}")
  endif()
endfunction()

add_test(test_bigint)
add_test(test_minmax_euclid)
add_test(test_policy_holder)
add_test(verify_log_computation)
add_test(dragonbox_test_all_shorter_interval_cases CHARCONV RYU)
add_test(dragonbox_uniform_random_test CHARCONV RYU)
add_test(dragonbox_verify_fast_multiplication)
add_test(dragonbox_verify_magic_division)
add_test(dragonbox_dooly_binary32_exhaustive_joint_test RYU)
add_test(dragonbox_dooly_binary64_uniform_random_joint_test RYU)
add_test(ryu_printf_uniform_random_test CHARCONV RYU)
add_test(ryu_printf_dooly_uniform_random_joint_test CHARCONV RYU)