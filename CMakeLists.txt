CMAKE_MINIMUM_REQUIRED(VERSION 3.5)
SET(Python_INCLUDE_DIR "/usr/include/python3.12")
set (CMAKE_CXX_STANDARD 11)
set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/ext)
PROJECT(PythonModule LANGUAGES CXX)
    SET (pycades_LIB_SRCS
            ${SRC_DIR}/PyCades.cpp
            ${SRC_DIR}/PyCadesAbout.cpp
            ${SRC_DIR}/PyCadesAlgorithm.cpp
            ${SRC_DIR}/PyCadesAttribute.cpp
            ${SRC_DIR}/PyCadesAttributes.cpp
            ${SRC_DIR}/PyCadesBasicConstraints.cpp
            ${SRC_DIR}/PyCadesBlobs.cpp
            ${SRC_DIR}/PyCadesCertificate.cpp
            ${SRC_DIR}/PyCadesCertificates.cpp
            ${SRC_DIR}/PyCadesCertificateStatus.cpp
            ${SRC_DIR}/PyCadesCRL.cpp
            ${SRC_DIR}/PyCadesEncodedData.cpp
            ${SRC_DIR}/PyCadesEnvelopedData.cpp
            ${SRC_DIR}/PyCadesEKU.cpp
            ${SRC_DIR}/PyCadesEKUs.cpp
            ${SRC_DIR}/PyCadesExtension.cpp
            ${SRC_DIR}/PyCadesExtensions.cpp
            ${SRC_DIR}/PyCadesExtendedKeyUsage.cpp
            ${SRC_DIR}/PyCadesHashedData.cpp
            ${SRC_DIR}/PyCadesKeyUsage.cpp
            ${SRC_DIR}/PyCadesOID.cpp
            ${SRC_DIR}/PyCadesPrivateKey.cpp
            ${SRC_DIR}/PyCadesPublicKey.cpp
            ${SRC_DIR}/PyCadesRawSignature.cpp
            ${SRC_DIR}/PyCadesRecipients.cpp
            ${SRC_DIR}/PyCadesSignatureStatus.cpp
            ${SRC_DIR}/PyCadesSignedData.cpp
            ${SRC_DIR}/PyCadesSignedXML.cpp
            ${SRC_DIR}/PyCadesSigner.cpp
            ${SRC_DIR}/PyCadesSigners.cpp
            ${SRC_DIR}/PyCadesStore.cpp
            ${SRC_DIR}/PyCadesSymmetricAlgorithm.cpp
            ${SRC_DIR}/PyCadesVersion.cpp
            ${SRC_DIR}/PyEnrollContainer.cpp
            ${SRC_DIR}/PyEnrollContainers.cpp
            ${SRC_DIR}/PyEnrollContainerKey.cpp
            ${SRC_DIR}/PyEnrollContainerKeys.cpp
            ${SRC_DIR}/PyEnrollCspInformation.cpp
        )
FIND_PACKAGE( Boost REQUIRED )

ADD_DEFINITIONS(-DUNIX -DSIZEOF_VOID_P=${CMAKE_SIZEOF_VOID_P})

EXECUTE_PROCESS(
    COMMAND bash -c "if type dpkg > /dev/null 2>&1; then dpkg -l|grep cprocsp-pki-cades|awk ' { print $3 } '; else rpm -qa|grep cprocsp-pki-cades; fi|cut -d. -f1|cut -d- -f1"
    OUTPUT_VARIABLE CPRO_CADES_VERSION_MAJOR
    OUTPUT_STRIP_TRAILING_WHITESPACE
    )
EXECUTE_PROCESS(
    COMMAND bash -c "if type dpkg > /dev/null 2>&1; then dpkg -l|grep cprocsp-pki-cades|awk ' { print $3 } '; else rpm -qa|grep cprocsp-pki-cades; fi|cut -d. -f2|cut -d- -f1"
    OUTPUT_VARIABLE CPRO_CADES_VERSION_MINOR
    OUTPUT_STRIP_TRAILING_WHITESPACE
    )
EXECUTE_PROCESS(
    COMMAND bash -c "if type dpkg > /dev/null 2>&1; then dpkg -l|grep cprocsp-pki-cades|awk ' { print $3 } '; else rpm -qa|grep cprocsp-pki-cades; fi|cut -d. -f3|cut -d- -f1"
    OUTPUT_VARIABLE CPRO_CADES_VERSION_BUILD
    OUTPUT_STRIP_TRAILING_WHITESPACE
    )
ADD_DEFINITIONS(-DCPRO_CADES_VERSION_MAJOR=${CPRO_CADES_VERSION_MAJOR})
ADD_DEFINITIONS(-DCPRO_CADES_VERSION_MINOR=${CPRO_CADES_VERSION_MINOR})
ADD_DEFINITIONS(-DCPRO_CADES_VERSION_BUILD=${CPRO_CADES_VERSION_BUILD})
MESSAGE("-- cprocsp-pki-cades: ${CPRO_CADES_VERSION_MAJOR}.${CPRO_CADES_VERSION_MINOR}.${CPRO_CADES_VERSION_BUILD}")

#SET(CMAKE_BUILD_TYPE Debug)
SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wno-write-strings")
SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations")
SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wno-narrowing")
#SET(CMAKE_INCLUDE_CURRENT_DIR ON)
SET(pycades_DIRS
     /opt/cprocsp/include
     /opt/cprocsp/include/cpcsp
     /opt/cprocsp/include/pki/atl
     /opt/cprocsp/include/pki/cppcades
     /opt/cprocsp/include/pki/cplib
     /opt/cprocsp/include/pki
     ${SRC_DIR}
     ${SRC_DIR}/cplib
)    

INCLUDE_DIRECTORIES( ${Boost_INCLUDE_DIRS} )
INCLUDE_DIRECTORIES( ${Python_INCLUDE_DIR})
INCLUDE_DIRECTORIES( ${pycades_DIRS})

ADD_LIBRARY(pycades SHARED ${pycades_LIB_SRCS})
SET_TARGET_PROPERTIES(pycades PROPERTIES PREFIX "")

FIND_LIBRARY(FOUND_LIB_CPPCADES
    NAMES cppcades
    HINTS /opt/cprocsp/lib/amd64 /opt/cprocsp/lib/ia32 /opt/cprocsp/lib/aarch64)
TARGET_LINK_LIBRARIES(pycades ${FOUND_LIB_CPPCADES})

FIND_LIBRARY(FOUND_LIB_CPLIB
    NAMES cplib
    HINTS /opt/cprocsp/lib/amd64 /opt/cprocsp/lib/ia32 /opt/cprocsp/lib/aarch64)
TARGET_LINK_LIBRARIES(pycades ${FOUND_LIB_CPLIB})

ADD_CUSTOM_COMMAND(TARGET pycades
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:pycades> ../samples)
