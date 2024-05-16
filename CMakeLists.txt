cmake_minimum_required(VERSION 3.10)
project(ubloxcfg)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Debug)

#### config.h generation 

add_custom_target(config_h DEPENDS ${CMAKE_BINARY_DIR}/config.h)
add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/config.h
    COMMAND ${CMAKE_COMMAND} -E echo_append "Generating config.h ..."
    COMMAND ${PERL} ${PROJECT_SOURCE_DIR}/config.h.pl < ${PROJECT_SOURCE_DIR}/config.h.in > ${CMAKE_BINARY_DIR}/config.h.tmp
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/config.h.tmp ${CMAKE_BINARY_DIR}/config.h
    COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_BINARY_DIR}/config.h.tmp
    MAIN_DEPENDENCY config.h.in
    DEPENDS ${PROJECT_SOURCE_DIR}/config.h.pl
    COMMENT "Generating config.h"
)

#### ubloxcfg_gen.{h,c} generation

add_custom_target(ubloxcfg_gen DEPENDS ${CMAKE_BINARY_DIR}/ubloxcfg_gen.h ${CMAKE_BINARY_DIR}/ubloxcfg_gen.c)

file(GLOB_RECURSE UBLOXCFG_JSONC "ubloxcfg/ubloxcfg-*.jsonc")

add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/ubloxcfg_gen.h ${CMAKE_BINARY_DIR}/ubloxcfg_gen.c
    COMMAND ${CMAKE_COMMAND} -E echo_append "Generating ubloxcfg/ubloxcfg_gen.h and ubloxcfg/ubloxcfg_gen.c ..."
    COMMAND ${PERL} ${PROJECT_SOURCE_DIR}/ubloxcfg/ubloxcfg_gen.pl ${CMAKE_BINARY_DIR}/ubloxcfg_gen ${UBLOXCFG_JSONC}
    MAIN_DEPENDENCY ${UBLOXCFG_JSONC}
    DEPENDS ${PROJECT_SOURCE_DIR}/ubloxcfg/ubloxcfg_gen.pl
    COMMENT "Generating ubloxcfg/ubloxcfg_gen.h and ubloxcfg/ubloxcfg_gen.c"
)

#### ubloxcfg library

add_library(ubloxcfg STATIC)
add_dependencies(ubloxcfg config_h ubloxcfg_gen)
target_sources(ubloxcfg
    PRIVATE
    ubloxcfg/ubloxcfg.c
    ${CMAKE_BINARY_DIR}/ubloxcfg_gen.c
)
target_compile_options(ubloxcfg PRIVATE -fPIC)
target_link_options(ubloxcfg PUBLIC -lm)
target_include_directories(ubloxcfg PRIVATE ubloxcfg ${CMAKE_BINARY_DIR})

#### FF library

add_library(ff STATIC)
add_dependencies(ubloxcfg config_h)
target_sources(ff
    PRIVATE
    ff/ff_crc.c
    ff/ff_debug.c
    ff/ff_epoch.c
    ff/ff_nmea.c
    ff/ff_novatel.c
    ff/ff_parser.c
    ff/ff_port.c
    ff/ff_rtcm3.c
    ff/ff_rx.c
    ff/ff_spartn.c
    ff/ff_stuff.c
    ff/ff_time.c
    ff/ff_trafo.c
    ff/ff_ubx.c
)
target_link_libraries(ff PRIVATE ubloxcfg)
target_link_options(ff PUBLIC -lm)
target_include_directories(ff PRIVATE ${CMAKE_BINARY_DIR} ubloxcfg)

#### cfgtool application

add_executable(cfgtool)
add_dependencies(cfgtool config_h)
target_sources(cfgtool
    PRIVATE
    cfgtool/cfgtool.c
    cfgtool/cfgtool_bin2hex.c
    cfgtool/cfgtool_cfg2rx.c
    cfgtool/cfgtool_cfg2ubx.c
    cfgtool/cfgtool_cfginfo.c
    cfgtool/cfgtool_dump.c
    cfgtool/cfgtool_parse.c
    cfgtool/cfgtool_reset.c
    cfgtool/cfgtool_rx2cfg.c
    cfgtool/cfgtool_status.c
    cfgtool/cfgtool_uc2cfg.c
    cfgtool/cfgtool_util.c
)
target_link_libraries(cfgtool PRIVATE ubloxcfg ff)
target_include_directories(cfgtool PRIVATE ${CMAKE_BINARY_DIR} ubloxcfg ff)
