cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(spreadsheet)

set(CMAKE_CXX_STANDARD 17)
if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    set(
        CMAKE_CXX_FLAGS_DEBUG
        "${CMAKE_CXX_FLAGS_DEBUG} /JMC"
    )
else()
    set(
        CMAKE_CXX_FLAGS
        "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic -Wno-unused-parameter -Wno-implicit-fallthrough"
    )
endif()

if(DEFINED ENV{CLASSPATH})
    set(ANTLR_EXECUTABLE "$ENV{CLASSPATH}")
else()
    message(FATAL_ERROR "CLASSPATH environment variable is not set. Please set it to point to your antlr-*.jar file for this configuration.")
endif()

include(${CMAKE_CURRENT_SOURCE_DIR}/FindANTLR.cmake)

add_definitions(
    -DANTLR4CPP_STATIC
    -D_SILENCE_ALL_CXX17_DEPRECATION_WARNINGS
)

set(WITH_STATIC_CRT OFF CACHE BOOL "Visual C++ static CRT for ANTLR" FORCE)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../antlr4_runtime ${CMAKE_CURRENT_BINARY_DIR}/antlr4_runtime_build)

antlr_target(FormulaParser grammar/Formula.g4 LEXER PARSER LISTENER)

include_directories(
    ${ANTLR4_INCLUDE_DIRS}
    ${ANTLR_FormulaParser_OUTPUT_DIR}
#    ${CMAKE_CURRENT_SOURCE_DIR}/antlr4_runtime/runtime/src
)

file(GLOB SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
)

add_executable(
    spreadsheet
    ${ANTLR_FormulaParser_CXX_OUTPUTS}
    ${SOURCES}
)

target_include_directories(spreadsheet PRIVATE
    ${ANTLR4_INCLUDE_DIRS} # Для заголовочных файлов ANTLR runtime (например, antlr4-runtime.h)
    ${ANTLR_FormulaParser_OUTPUT_DIR} # Для сгенерированных заголовочных файлов (например, FormulaLexer.h)
)
target_link_libraries(spreadsheet PRIVATE antlr4_static)

if(MSVC)
    target_compile_options(antlr4_static PRIVATE /W0)
endif()

install(
    TARGETS spreadsheet
    DESTINATION bin
    EXPORT spreadsheet
)

set_directory_properties(PROPERTIES VS_STARTUP_PROJECT spreadsheet)

# D:\Artem\Cpp-Course\cpp-spreadsheet\build>cmake ../spreadsheet/ -DCMAKE_BUILD_TYPE=Debug -G "MinGW Makefiles" -DCMAKE_POLICY_VERSION_MINIMUM=3.5
# D:\Artem\Cpp-Course\cpp-spreadsheet\build>cmake --build .