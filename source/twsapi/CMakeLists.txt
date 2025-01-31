file(GLOB SOURCES "*.cpp")
file(GLOB HEADERS "*.h")
set(twsapi_VERSION 9.79.02)
add_library(twsapi STATIC ${SOURCES})
# On Windows (including MinGW), link against Winsock 2

set(LIBS LibBid)

if( WIN32 )
    set(LIBS ${LIBS} ws2_32)
endif()
target_link_libraries(twsapi ${LIBS})
if( MSVC )
    target_compile_options( twsapi PRIVATE /W3 )
else()
    target_compile_options( twsapi PRIVATE -Wall -Wextra -pedantic -Wno-unused-parameter )
endif()
set_target_properties(twsapi
    PROPERTIES
    PREFIX "lib"
    DEBUG_POSTFIX "d"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    PUBLIC_HEADER "${HEADERS}"
)

target_include_directories(twsapi PUBLIC ${PROJECT_SOURCE_DIR}/source)
install(TARGETS twsapi EXPORT twsapi-targets PUBLIC_HEADER DESTINATION include/twsapi LIBRARY DESTINATION lib)