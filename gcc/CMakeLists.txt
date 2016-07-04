
set(TARGET_DIR "${PROJECT_BINARY_DIR}/gcc")
set(BUILD_TARGET_DIR "${PROJECT_BINARY_DIR}/gcc/build-gcc")
set(TARGET_NAME "${GCC_VERSION}")
set(TARGET_NAME_ARC "${GCC_VERSION}.tar.gz")

add_subdirectory(dirs)
add_subdirectory(mpfr)
add_subdirectory(gmp)
add_subdirectory(mpc)
add_subdirectory(isl)
add_subdirectory(cloog)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.download.done

    COMMAND wget "https://ftp.gnu.org/gnu/gcc/${TARGET_NAME}/${TARGET_NAME_ARC}" -O ${TARGET_DIR}/${TARGET_NAME_ARC}
    COMMAND touch ${TARGET_DIR}/.download.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.untar.done

    COMMAND tar xpvf ${TARGET_DIR}/${TARGET_NAME_ARC} -C ${TARGET_DIR}
    COMMAND touch ${TARGET_DIR}/.untar.done

    DEPENDS ${TARGET_DIR}/.download.done
    WORKING_DIRECTORY ${TARGET_DIR}
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.configure.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} ${TARGET_DIR}/${TARGET_NAME}/configure --prefix=${INSTALL_PATH} --target=${CC_TARGET} --enable-languages=c,c++ --disable-multilib
    COMMAND touch ${TARGET_DIR}/.configure.done

    WORKING_DIRECTORY ${BUILD_TARGET_DIR}
    DEPENDS ${TARGET_DIR}/.untar.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.build.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make all-gcc
    COMMAND touch ${TARGET_DIR}/.build.done

    WORKING_DIRECTORY ${BUILD_TARGET_DIR}
    DEPENDS ${TARGET_DIR}/.configure.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make install-gcc
    COMMAND touch ${TARGET_DIR}/.install.done

    WORKING_DIRECTORY ${BUILD_TARGET_DIR}
    DEPENDS ${TARGET_DIR}/.build.done
)

add_custom_target(
    make_gcc ALL
    DEPENDS make_binutils make_linux_kernel_headers make_mpfr make_gmp make_mpc make_isl make_cloog ${TARGET_DIR}/.install.done
    WORKING_DIRECTORY ${TARGET_DIR}
)
