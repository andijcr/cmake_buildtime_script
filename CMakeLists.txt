cmake_minimum_required(VERSION 3.16)

project(scripts_with_cmake C)


# step 0: define externally the seed file (useful for step 2)
set(SIGNATURE_SEED_FILE ${CMAKE_CURRENT_SOURCE_DIR}/signature_user.c)

# step 1: configure the script that will generate the signature at build time
configure_file(gen_signature.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/gen_signature.cmake @ONLY)

# step 2: add a command to generate the signature (via a cmake script)
# thanks to DEPENDS, it will be re-run when the seed file changes
add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/signature.c
    COMMAND ${CMAKE_COMMAND} -P
        ${CMAKE_CURRENT_BINARY_DIR}/gen_signature.cmake
    DEPENDS
        ${CMAKE_CURRENT_BINARY_DIR}/gen_signature.cmake
        ${CMAKE_CURRENT_SOURCE_DIR}/signature.c.in
        ${SIGNATURE_SEED_FILE}
    COMMENT "generating the private signature"
    VERBATIM
    )

# step 3: use the file to get the signature
add_executable(self_signature
	signature_user.c
	signature.h
	${CMAKE_CURRENT_BINARY_DIR}/signature.c
	)
target_include_directories(self_signature PUBLIC ${CMAKE_CURRENT_LIST_DIR})
