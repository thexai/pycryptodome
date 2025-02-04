cmake_minimum_required(VERSION 3.3)

project(pycryptodome VERSION 3.4.9 LANGUAGES C)

option(SEPARATE_NAMESPACE "Install as Cryptodome instead of Crypto" OFF)

set(NAMESPACE Crypto)
if(SEPARATE_NAMESPACE)
  set(NAMESPACE Cryptodome)
endif()

find_package(python REQUIRED NO_MODULE)

include(CheckIncludeFile)
include(CheckSymbolExists)

check_symbol_exists(_X86_ "Windows.h" _X86_)
check_symbol_exists(_AMD64_ "Windows.h" _AMD64_)
check_symbol_exists(_ARM_ "Windows.h" _ARM_)
check_symbol_exists(_ARM64_ "Windows.h" _ARM64_)

message("_X86 ${_X86_} _AMD64_ ${_AMD64_} _ARM_ ${_ARM_} _ARM64_ ${_ARM64_}")
if(CMAKE_SIZEOF_VOID_P EQUAL 4)
  set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} /arch:sse2")
endif()
CHECK_INCLUDE_FILE(intrin.h HAVE_INTRIN_H)
CHECK_INCLUDE_FILE(cpuid.h HAVE_CPUID_H)

add_library(tomcrypt STATIC
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_hash.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_mac.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_macros.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_math.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_misc.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_pk.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_pkcs.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_prng.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_argchk.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_cfg.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_cipher.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_custom.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libtom/tomcrypt_des.c
)

target_include_directories(tomcrypt
  PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/libtom>
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/libtom>
)

# Hash functions
add_library(_MD2
  ${CMAKE_CURRENT_SOURCE_DIR}/src/MD2.c
)

add_library(_MD4
  ${CMAKE_CURRENT_SOURCE_DIR}/src/MD4.c
)

add_library(_MD5
  ${CMAKE_CURRENT_SOURCE_DIR}/src/MD5.c
)

add_library(_SHA1
  ${CMAKE_CURRENT_SOURCE_DIR}/src/SHA1.c
)

add_library(_SHA256
  ${CMAKE_CURRENT_SOURCE_DIR}/src/SHA256.c
)

add_library(_SHA224
  ${CMAKE_CURRENT_SOURCE_DIR}/src/SHA224.c
)

add_library(_SHA384
  ${CMAKE_CURRENT_SOURCE_DIR}/src/SHA384.c
)

add_library(_SHA512
  ${CMAKE_CURRENT_SOURCE_DIR}/src/SHA512.c
)

add_library(_RIPEMD160
  ${CMAKE_CURRENT_SOURCE_DIR}/src/RIPEMD160.c
)

add_library(_keccak
  ${CMAKE_CURRENT_SOURCE_DIR}/src/keccak.c
)

add_library(_BLAKE2b
  ${CMAKE_CURRENT_SOURCE_DIR}/src/blake2b.c
)

add_library(_BLAKE2s
  ${CMAKE_CURRENT_SOURCE_DIR}/src/blake2s.c
)

add_library(_ghash_portable
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ghash_portable.c
)

if(_X86_ OR _AMD64_)
add_library(_ghash_clmul
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ghash_clmul.c
)
endif()

# MACs

add_library(_poly1305
  ${CMAKE_CURRENT_SOURCE_DIR}/src/poly1305.c
)

# Block encryption algorithms
add_library(_raw_aes
  ${CMAKE_CURRENT_SOURCE_DIR}/src/AES.c
)

if(_X86_ OR _AMD64_)
add_library(_raw_aesni
  ${CMAKE_CURRENT_SOURCE_DIR}/src/AESNI.c
)
endif()

add_library(_raw_arc2
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ARC2.c
)

add_library(_raw_blowfish
  ${CMAKE_CURRENT_SOURCE_DIR}/src/Blowfish.c
)

add_library(_raw_eksblowfish
  ${CMAKE_CURRENT_SOURCE_DIR}/src/Blowfish.c
)
target_compile_definitions(_raw_eksblowfish
  PRIVATE EKS
)

add_library(_raw_cast
  ${CMAKE_CURRENT_SOURCE_DIR}/src/CAST.c
)

add_library(_raw_des
  ${CMAKE_CURRENT_SOURCE_DIR}/src/DES.c
)

add_library(_raw_des3
  ${CMAKE_CURRENT_SOURCE_DIR}/src/DES3.c
)

if(_X86_ OR _AMD64_)
add_library(_cpuid_c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/cpuid.c
)
endif()

# Chaining modes
add_library(_raw_ecb
  ${CMAKE_CURRENT_SOURCE_DIR}/src/raw_ecb.c
)

add_library(_raw_cbc
  ${CMAKE_CURRENT_SOURCE_DIR}/src/raw_cbc.c
)

add_library(_raw_cfb
  ${CMAKE_CURRENT_SOURCE_DIR}/src/raw_cfb.c
)

add_library(_raw_ofb
  ${CMAKE_CURRENT_SOURCE_DIR}/src/raw_ofb.c
)

add_library(_raw_ctr
  ${CMAKE_CURRENT_SOURCE_DIR}/src/raw_ctr.c
)

add_library(_raw_ocb
  ${CMAKE_CURRENT_SOURCE_DIR}/src/raw_ocb.c
)

# Stream ciphers
add_library(_ARC4
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ARC4.c
)

add_library(_Salsa20
  ${CMAKE_CURRENT_SOURCE_DIR}/src/Salsa20.c
)

add_library(_chacha20
  ${CMAKE_CURRENT_SOURCE_DIR}/src/chacha20.c
)

# Others
add_library(_scrypt
  ${CMAKE_CURRENT_SOURCE_DIR}/src/scrypt.c
)

# Utility modules
add_library(_strxor
  ${CMAKE_CURRENT_SOURCE_DIR}/src/strxor.c
)

# ECC
add_library(_ec_ws
  ${CMAKE_CURRENT_SOURCE_DIR}/src/modexp_utils.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/siphash.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ec_ws.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/mont.c
)
target_compile_definitions(_ec_ws
  PRIVATE SYS_BITS=${CMAKE_SIZEOF_VOID_P}*8
)
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  target_sources(_ec_ws
    PRIVATE 
      ${CMAKE_CURRENT_SOURCE_DIR}/src/multiply_64.c
  )
else()
  target_sources(_ec_ws
    PRIVATE 
      ${CMAKE_CURRENT_SOURCE_DIR}/src/multiply_32.c
  )
endif()

# Math
add_library(_modexp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/modexp_utils.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/siphash.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/modexp.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/mont.c
)
target_compile_definitions(_modexp
  PRIVATE SYS_BITS=${CMAKE_SIZEOF_VOID_P}*8
)
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  target_sources(_modexp
    PRIVATE 
      ${CMAKE_CURRENT_SOURCE_DIR}/src/multiply_64.c
  )
else()
  target_sources(_modexp
    PRIVATE 
      ${CMAKE_CURRENT_SOURCE_DIR}/src/multiply_32.c
  )
endif()

set(CRYPTO_HASH
  _MD2
  _MD4
  _MD5
  _SHA1
  _SHA256
  _SHA224
  _SHA384
  _SHA512
  _RIPEMD160
  _keccak
  _BLAKE2b
  _BLAKE2s
  _poly1305
  _ghash_portable
)
set(CRYPTO_CIPHER
  _raw_aes
  _raw_arc2
  _raw_blowfish
  _raw_eksblowfish
  _raw_cast
  _raw_des
  _raw_des3
  _raw_ecb
  _raw_cbc
  _raw_cfb
  _raw_ofb
  _raw_ctr
  _raw_ocb
  _ARC4
  _Salsa20
  _chacha20
)
set(CRYPTO_UTIL _strxor)
set(CRYPTO_PROTOCOL _scrypt)
set(CRYPTO_PUBLICKEY _ec_ws)
set(CRYPTO_MATH _modexp)

if(_X86_ OR _AMD64_)
  list(APPEND CRYPTO_HASH _ghash_clmul)
  list(APPEND CRYPTO_CIPHER _raw_aesni)
  list(APPEND CRYPTO_UTIL _cpuid_c)
  message("CRYPTO_UTIL: ${CRYPTO_UTIL}")
endif()

foreach(CRYPTO_LIB IN LISTS CRYPTO_HASH CRYPTO_CIPHER CRYPTO_UTIL CRYPTO_PROTOCOL CRYPTO_PUBLICKEY CRYPTO_MATH)
  target_compile_definitions(${CRYPTO_LIB}
    PRIVATE
      ARGTYPE=3
      HAVE_STDINT_H
      PYCRYPTO_LITTLE_ENDIAN
  )
  if(_X86_ OR _AMD64_)
    target_compile_definitions(${CRYPTO_LIB}
      PRIVATE
        USE_SSE2
        HAVE_WMMINTRIN_H
        HAVE_TMMINTRIN_H
    )
  endif()

  if (HAVE_INTRIN_H)
    target_compile_definitions(${CRYPTO_LIB}
      PRIVATE
        HAVE_INTRIN_H
    )
  endif()

  if (HAVE_CPUID_H)
    target_compile_definitions(${CRYPTO_LIB}
      PRIVATE
        HAVE_CPUID_H
        USE_SSE2
    )
  endif()

  target_compile_options(${CRYPTO_LIB}
    PRIVATE
      /sdl-
  )

  target_sources(${CRYPTO_LIB}
    PRIVATE
      ${CMAKE_CURRENT_SOURCE_DIR}/src/common.h
      ${CMAKE_CURRENT_SOURCE_DIR}/src/errors.h
  )

  target_link_libraries(${CRYPTO_LIB}
    PRIVATE
      python::python
      tomcrypt
      $<$<CONFIG:Debug>:vccorlibd.lib>
      $<$<CONFIG:RelWithDebInfo>:vccorlib.lib>
      $<$<CONFIG:Debug>:msvcrtd.lib>
      $<$<CONFIG:RelWithDebInfo>:msvcrt.lib>
  )

  target_include_directories(${CRYPTO_LIB}
    PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
  )

  set_target_properties(${CRYPTO_LIB}
    PROPERTIES
      SUFFIX ".pyd"
      PDB_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
      PDB_NAME ${CRYPTO_LIB}
  )

  if(${CRYPTO_LIB} IN_LIST CRYPTO_HASH)
    set(CRYPTO_INSTALL bin/Python/Lib/site-packages/${NAMESPACE}/Hash/)
  elseif(${CRYPTO_LIB} IN_LIST CRYPTO_CIPHER)
    set(CRYPTO_INSTALL bin/Python/Lib/site-packages/${NAMESPACE}/Cipher/)
  elseif(${CRYPTO_LIB} IN_LIST CRYPTO_UTIL)
    set(CRYPTO_INSTALL bin/Python/Lib/site-packages/${NAMESPACE}/Util/)
  elseif(${CRYPTO_LIB} IN_LIST CRYPTO_PUBLICKEY)
    set(CRYPTO_INSTALL bin/Python/Lib/site-packages/${NAMESPACE}/PublicKey/)
  elseif(${CRYPTO_LIB} IN_LIST CRYPTO_MATH)
    set(CRYPTO_INSTALL bin/Python/Lib/site-packages/${NAMESPACE}/Math/)
  else()
    set(CRYPTO_INSTALL bin/Python/Lib/site-packages/${NAMESPACE}/Protocol/)
  endif()

  target_link_options(${CRYPTO_LIB}
    PRIVATE
      /NODEFAULTLIB
      /INCREMENTAL:NO
      /debug
  )

  install(TARGETS ${CRYPTO_LIB}
    RUNTIME DESTINATION ${CRYPTO_INSTALL}
  )
  install(FILES
    $<TARGET_PDB_FILE:${CRYPTO_LIB}>
    DESTINATION lib
  )
endforeach()

install(DIRECTORY
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/Crypto/
  DESTINATION bin/Python/Lib/site-packages/${NAMESPACE}
  PATTERN SelfTest EXCLUDE
)

if(SEPARATE_NAMESPACE)
  install(CODE "file(GLOB_RECURSE PYMODULES ${CMAKE_INSTALL_PREFIX}/bin/Python/Lib/site-packages/${NAMESPACE}/*.py)")
  install(CODE "foreach(PYMODULE \${PYMODULES})
    file(READ \${PYMODULE} content)
    string(REPLACE \"Crypto.\" \"Cryptodome.\" content \"\${content}\")
    string(REPLACE \"Crypto \" \"Cryptodome \" content \"\${content}\")
    string(REPLACE \"'Crypto'\" \"'Cryptodome'\" content \"\${content}\")
    string(REPLACE \"\\\"Crypto\\\"\" \"\\\"Cryptodome\\\"\" content \"\${content}\")
    file(WRITE \${PYMODULE} \"\${content}\")
  endforeach()")
endif()
