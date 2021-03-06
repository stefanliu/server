# mangos-zero is a full featured server for World of Warcraft in its vanilla
# version, supporting clients for patch 1.12.x.
#
# Copyright (C) 2005-2013  MaNGOS project <http://getmangos.com>
#
# ***** BEGIN GPL LICENSE BLOCK *****
#
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
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
#
# ***** END GPL LICENSE BLOCK *****
#
# World of Warcraft, and all World of Warcraft or Warcraft art, images,
# and lore are copyrighted by Blizzard Entertainment, Inc.

include(MacroMangosSourceGroup)

#-----------------------------------------------------------------------------
# Define the shared library
set(sources
    Auth/AuthCrypt.cpp
    Auth/BigNumber.cpp
    Auth/Hmac.cpp
    Auth/Sha1.cpp
    Auth/md5.c

    Config/Config.cpp

    Database/Database.cpp
    Database/DBCFileLoader.cpp
    Database/Field.cpp
    Database/SqlDelayThread.cpp
    Database/SqlOperations.cpp
    Database/SqlPreparedStatement.cpp
    Database/SQLStorage.cpp

    ByteBuffer.cpp
    Common.cpp
    Log.cpp
    ProgressBar.cpp
    Threading.cpp
    Util.cpp
)
set(headers
    Auth/AuthCrypt.h
    Auth/BigNumber.h
    Auth/Hmac.h
    Auth/md5.h
    Auth/Sha1.h

    Config/Config.h

    Database/Database.h
    Database/DatabaseEnv.h
    Database/DatabaseImpl.h
    Database/DBCFileLoader.h
    Database/DBCStore.h
    Database/Field.h
    Database/QueryResult.h
    Database/SqlDelayThread.h
    Database/SqlOperations.h
    Database/SqlPreparedStatement.h
    Database/SQLStorage.h
    Database/SQLStorageImpl.h

    ByteBuffer.h
    Common.h
    Errors.h
    LockedQueue.h
    Log.h
    ProgressBar.h
    Threading.h
    Timer.h
    Util.h
    WorldPacket.h

    revision_nr.h
    revision_sql.h
)

#-----------------------------------------------------------------------------
# Add platform specific sources
if(WIN32)
    list(APPEND sources
        WheatyExceptionReport.cpp
        ServiceWin32.cpp
     )
    list(APPEND headers
        WheatyExceptionReport.h
        ServiceWin32.h
    )
else()
    list(APPEND sources
        PosixDaemon.cpp
    )
    list(APPEND headers
        PosixDaemon.h
    )
endif()

if(WITH_MYSQL AND MYSQL_FOUND)
    include_directories(
        ${MYSQL_INCLUDE_DIR}
    )
    list(APPEND sources
        Database/DatabaseMysql.cpp
        Database/QueryResultMysql.cpp
    )
    list(APPEND headers
        Database/DatabaseMysql.h
        Database/QueryResultMysql.h
    )
elseif(WITH_POSTGRESQL AND PostgreSQL_FOUND)
    include_directories(
        ${PostgreSQL_INCLUDE_DIRS}
    )
    list(APPEND sources
        Database/DatabasePostgre.cpp
        Database/QueryResultPostgre.cpp
    )
    list(APPEND headers
        Database/DatabasePostgre.h
        Database/PGSQLDelayThread.h
        Database/QueryResultPostgre.h
    )
endif()

set(shared_LIB_SRCS ${sources} ${headers})

mangos_source_group(${shared_LIB_SRCS})

include_directories(
    ${ACE_INCLUDE_DIRS}
    ${OPENSSL_INCLUDE_DIR}
    ${CMAKE_SOURCE_DIR}/dep/include/mersennetwister/
    ${CMAKE_SOURCE_DIR}/dep/include/utf8cpp/
    ${CMAKE_SOURCE_DIR}/src/framework/
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_BINARY_DIR}
    ${CMAKE_BINARY_DIR}/src/shared/
)

#-----------------------------------------------------------------------------
# Generate SystemConfig.h
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/SystemConfig.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/SystemConfig.h)

#-----------------------------------------------------------------------------
# Build the shared library
add_library(mangos-shared STATIC ${shared_LIB_SRCS})
target_link_libraries(mangos-shared mangos-framework ${ACE_LIBRARIES} ${OPENSSL_LIBRARIES})

set(BUILD_PROPERTIES "-DHAVE_CONFIG_H")
if(${CMAKE_BUILD_TYPE} MATCHES Debug)
    set(BUILD_PROPERTIES "${BUILD_PROPERTIES} -DMANGOS_DEBUG")
endif()
if(WITH_MYSQL AND MYSQL_FOUND)
    set(BUILD_PROPERTIES "${BUILD_PROPERTIES} -DDO_MYSQL")
    target_link_libraries(mangos-shared ${MYSQL_LIBRARIES})
elseif(WITH_POSTGRESQL AND PostgreSQL_FOUND)
    set(BUILD_PROPERTIES "${BUILD_PROPERTIES} -DDO_POSTGRESQL")
    target_link_libraries(mangos-shared ${PostgreSQL_LIBRARIES})
endif()
if(${CMAKE_BUILD_TYPE} MATCHES Debug)
    set(BUILD_PROPERTIES "${BUILD_PROPERTIES} -DMANGOS_DEBUG")
endif()
set_target_properties(mangos-shared PROPERTIES COMPILE_FLAGS ${BUILD_PROPERTIES})

add_dependencies(mangos-shared revision.h)
