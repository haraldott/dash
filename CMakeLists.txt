check_include_file_cxx(stdint.h HAVE_STDINT_H)
if(HAVE_STDINT_H)
     add_definitions(-DHAVE_STDINT_H)
endif()

set(examples_as_tests_sources)
if(${ENABLE_EXAMPLES})
     set(examples_as_tests_sources
         #test/dash-examples-test-suite.cc
         )
endif()

build_lib(
    LIBNAME dash
    SOURCE_FILES 
        model/tcp-stream-client.cc
        model/tcp-stream-server.cc
        model/tcp-stream-adaptation-algorithm.cc
        model/festive.cc
        model/panda.cc
        model/tobasco2.cc
        helper/tcp-stream-helper.cc
    HEADER_FILES 
        model/tcp-stream-client.h
        model/tcp-stream-server.h
        model/tcp-stream-interface.h
        model/tcp-stream-adaptation-algorithm.h
        model/festive.h
        model/panda.h
        model/tobasco2.h
        helper/tcp-stream-helper.h
    LIBRARIES_TO_LINK 
        ${libinternet}
        ${libconfig-store}
        ${libstats}
    # TEST_SOURCES test/dash-test-suite.cc
    #              ${examples_as_tests_sources}
)

