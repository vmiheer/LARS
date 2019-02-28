cmake_minimum_required (VERSION 2.6)
project (lars)
# enable_testing()
set (CMAKE_CXX_STANDARD 11)

find_package(Boost REQUIRED)

find_package(BISON 3.0.4)
find_package(FLEX)
if (WIN32)
  FLEX_TARGET(MyScanner scanner.l ${CMAKE_CURRENT_BINARY_DIR}/scanner.cpp COMPILE_FLAGS --wincompat)
else()
  FLEX_TARGET(MyScanner scanner.l ${CMAKE_CURRENT_BINARY_DIR}/scanner.cpp)
endif()
BISON_TARGET(MyParser grammar.y ${CMAKE_CURRENT_BINARY_DIR}/parser.cpp)
ADD_FLEX_BISON_DEPENDENCY(MyScanner MyParser)
add_definitions(-DDEBUG=true -DDROP_COEFS=true -DASSOC_MULT=false
  -DAVAIL_EXPR_OPT=false -DRETAIN_SIMPLE_OPS=false -DSPLICE_EQUALITY=false
  -DPAR_LOADS=1 -DEXPLICIT_LOADS=false -DRESTRICT_INTER_OPT=true
  -DSPLICE_TEMP_LABELS=false -DINTRA_TYPE_INTER_OPT=false -DFIRST_LEVEL=false
  -DSECOND_LEVEL=true -DOPERATION_VIEW=false)
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${lars_SOURCE_DIR})

add_executable(lars
  exprnode.cpp
  funcdefn.cpp
  vardecl.cpp
  codegen.cpp
  main.cpp
  codegen.hpp
  datatypes.hpp
  exprnode.hpp
  funcdefn.hpp
  grammar.hpp
  parser.hpp
  sort.hpp
  symtab.hpp
  utils.hpp
  vardecl.hpp
  ${BISON_MyParser_OUTPUTS}
  ${FLEX_MyScanner_OUTPUTS}
  )
if (NOT WIN32)
  add_custom_target(dummy ALL
    COMMAND ${CMAKE_COMMAND} -E create_symlink $<TARGET_FILE_DIR:lars>/$<TARGET_FILE_NAME:lars> "${lars_SOURCE_DIR}/test"
    DEPENDS lars)
  endif()
target_link_libraries(lars PRIVATE Boost::boost)
