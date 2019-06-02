# Author: Sébastien Gonzalve (2019)
# 
# LICENSE:
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA 02111-1307, USA.
# 

cmake_minimum_required(VERSION 3.11)

project(libcanardbc)

FIND_PACKAGE(BISON REQUIRED)
find_package(FLEX REQUIRED)
find_package (PkgConfig REQUIRED)
pkg_check_modules(GLIB2 REQUIRED glib-2.0)
pkg_check_modules(JSON-GLIB REQUIRED json-glib-1.0)

SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wmissing-declarations -Wmissing-prototypes \
         -Wnested-externs -Wpointer-arith -Wpointer-arith -Wsign-compare -Wchar-subscripts \
         -Wstrict-prototypes -Wshadow -Wformat-security -Wtype-limits")

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/ ${CMAKE_CURRENT_BINARY_DIR})

bison_target(parser src/candbc-parser.y ${CMAKE_CURRENT_BINARY_DIR}/candbc-parser.c)
flex_target(scanner src/candbc-lexer.l ${CMAKE_CURRENT_BINARY_DIR}/candbc-lexer.c)

add_flex_bison_dependency(scanner parser)

add_library(canardbc STATIC src/candbc-reader.c
                            src/candbc-model.c
                            ${BISON_parser_OUTPUTS}
                            ${FLEX_MyScanner_OUTPUTS}
                            ${CMAKE_CURRENT_BINARY_DIR}/candbc-lexer.c)


add_executable(dbc2json tools/dbc2json.c)
target_link_libraries(dbc2json canardbc ${JSON-GLIB_LIBRARIES} -lm)
target_include_directories (dbc2json PRIVATE ${GLIB2_INCLUDE_DIRS} ${JSON-GLIB_INCLUDE_DIRS})
install(TARGETS dbc2json DESTINATION bin)

install(PROGRAMS tools/dbcjson2html.py tools/dbc2html.sh DESTINATION bin)
