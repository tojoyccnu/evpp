
file(GLOB evpp_SRCS *.cc */*.cc *.h */*.h)
file(GLOB evpp_lite_SRCS *.cc *.h)
file(GLOB evpp_PUBLIC_HEADERS *.h)
file(GLOB evpp_HTTP_PUBLIC_HEADERS http/*.h)
file(GLOB evpp_UDP_PUBLIC_HEADERS udp/*.h)

message(STATUS "evpp_SRCS : " ${evpp_SRCS})

set (MACOSX_RPATH ${LIBRARY_OUTPUT_PATH})

include_directories(${PROJECT_SOURCE_DIR})

add_library(evpp_static STATIC ${evpp_SRCS})
target_link_libraries(evpp_static ${DEPENDENT_LIBRARIES})

add_library(evpp_lite_static STATIC ${evpp_lite_SRCS})
target_link_libraries(evpp_lite_static ${DEPENDENT_LIBRARIES})

if (UNIX)
    add_library(evpp SHARED ${evpp_SRCS})
    target_link_libraries(evpp ${DEPENDENT_LIBRARIES})

    # boost lockfree queue
    add_library(evpp_boost SHARED ${evpp_SRCS})
    add_library(evpp_boost_static STATIC ${evpp_SRCS})
    target_compile_definitions(evpp_boost PRIVATE -DH_HAVE_BOOST=1)
    target_compile_definitions(evpp_boost_static PRIVATE -DH_HAVE_BOOST=1)
    target_link_libraries(evpp_boost        ${DEPENDENT_LIBRARIES})
    target_link_libraries(evpp_boost_static ${DEPENDENT_LIBRARIES})

    # https://github.com/cameron314/concurrentqueue
    add_library(evpp_concurrentqueue SHARED ${evpp_SRCS})
    add_library(evpp_concurrentqueue_static STATIC ${evpp_SRCS})
    target_compile_definitions(evpp_concurrentqueue PRIVATE -DH_HAVE_CAMERON314_CONCURRENTQUEUE=1)
    target_compile_definitions(evpp_concurrentqueue_static PRIVATE -DH_HAVE_CAMERON314_CONCURRENTQUEUE=1)
    target_link_libraries(evpp_concurrentqueue        ${DEPENDENT_LIBRARIES})
    target_link_libraries(evpp_concurrentqueue_static ${DEPENDENT_LIBRARIES})

    set (CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
    include (utils)
    include (packages)
    set_target_properties (
        evpp evpp_boost evpp_concurrentqueue PROPERTIES
        VERSION     "${PACKAGE_VERSION}"
        SOVERSION   "${PACKAGE_SOVERSION}"
    )

    install (
      TARGETS evpp evpp_static evpp_lite_static evpp_boost evpp_boost_static evpp_concurrentqueue evpp_concurrentqueue_static
      EXPORT ${PACKAGE_NAME}
      RUNTIME DESTINATION bin
      LIBRARY DESTINATION lib
      ARCHIVE DESTINATION lib)

    install (FILES ${evpp_PUBLIC_HEADERS} DESTINATION "include/evpp")
    install (FILES ${evpp_HTTP_PUBLIC_HEADERS} DESTINATION "include/evpp/http")
    install (FILES ${evpp_UDP_PUBLIC_HEADERS} DESTINATION "include/evpp/udp")

else (UNIX)
	link_directories(${PROJECT_SOURCE_DIR}/vsprojects/bin/${CMAKE_BUILD_TYPE})
    
    install (
      TARGETS evpp_static evpp_lite_static
      EXPORT ${PACKAGE_NAME}
      RUNTIME DESTINATION bin
      LIBRARY DESTINATION lib
      ARCHIVE DESTINATION lib)

    install (FILES ${evpp_PUBLIC_HEADERS} DESTINATION "include/evpp")
    install (FILES ${evpp_HTTP_PUBLIC_HEADERS} DESTINATION "include/evpp/http")
    install (FILES ${evpp_UDP_PUBLIC_HEADERS} DESTINATION "include/evpp/udp")
endif (UNIX)

