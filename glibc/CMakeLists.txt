
set(TARGET_DIR "${PROJECT_BINARY_DIR}/glibc")
set(TARGET_NAME "${GLIBC_VERSION}")
set(TARGET_NAME_ARC "${GLIBC_VERSION}.tar.xz")
set(BUILD_GLIBC_DIR "${TARGET_DIR}/build-glibc")

add_custom_command(
    OUTPUT ${TARGET_DIR}/.mkdir.done

    COMMAND mkdir -p ${BUILD_GLIBC_DIR}
    COMMAND mkdir -p ${TARGET_DIR}

    COMMAND touch ${TARGET_DIR}/.mkdir.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.download.done

    COMMAND wget "https://ftp.gnu.org/gnu/glibc/${TARGET_NAME_ARC}" -O ${TARGET_DIR}/${TARGET_NAME_ARC}
    COMMAND touch ${TARGET_DIR}/.download.done

    DEPENDS ${TARGET_DIR}/.mkdir.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.untar.done

    COMMAND mkdir -p ${TARGET_DIR}/${TARGET_NAME}
    COMMAND tar xpvf ${TARGET_DIR}/${TARGET_NAME_ARC} -C ${TARGET_DIR}/${TARGET_NAME} --strip-components=1
    COMMAND touch ${TARGET_DIR}/.untar.done

    DEPENDS ${TARGET_DIR}/.download.done
    WORKING_DIRECTORY ${TARGET_DIR}
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.configure.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} ${TARGET_DIR}/${TARGET_NAME}/configure --prefix=${INSTALL_PATH}/${CC_TARGET} --enable-kernel=2.6.32 --build=${CC_MACHTYPE} --host=${CC_TARGET} --target=${CC_TARGET} --with-headers=${INSTALL_PATH}/${CC_TARGET}/include --disable-multilib libc_cv_forced_unwind=yes
    COMMAND touch ${TARGET_DIR}/.configure.done

    WORKING_DIRECTORY ${BUILD_GLIBC_DIR}
    DEPENDS ${TARGET_DIR}/.untar.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.boot-strap.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make install-bootstrap-headers=yes install-headers
    COMMAND touch ${TARGET_DIR}/.boot-strap.done

    WORKING_DIRECTORY ${BUILD_GLIBC_DIR}
    DEPENDS ${TARGET_DIR}/.configure.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.build.csu.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make csu/subdir_lib
    COMMAND touch ${TARGET_DIR}/.build.csu.done

    WORKING_DIRECTORY ${BUILD_GLIBC_DIR}
    DEPENDS ${TARGET_DIR}/.boot-strap.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.csu.done
    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make csu/install-lib
    COMMAND touch ${TARGET_DIR}/.install.csu.done
    WORKING_DIRECTORY ${BUILD_GLIBC_DIR}
    DEPENDS ${TARGET_DIR}/.build.csu.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.libc.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} ${CC_TARGET}-gcc -nostdlib -nostartfiles -shared -x c /dev/null -o ${INSTALL_PATH}/${CC_TARGET}/lib/libc.so
    COMMAND touch ${TARGET_DIR}/.install.libc.done

    WORKING_DIRECTORY ${BUILD_GLIBC_DIR}
    DEPENDS ${TARGET_DIR}/.install.csu.done
)


add_custom_command(
    OUTPUT ${TARGET_DIR}/.make.stubs.done
    COMMAND touch ${INSTALL_PATH}/${CC_TARGET}/include/gnu/stubs.h
    COMMAND touch ${TARGET_DIR}/.make.stubs.done
    DEPENDS ${TARGET_DIR}/.install.libc.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.compiler.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make all-target-libgcc
    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make install-target-libgcc
    COMMAND touch ${TARGET_DIR}/.install.compiler.done
    DEPENDS ${TARGET_DIR}/.make.stubs.done

    WORKING_DIRECTORY ${PROJECT_BINARY_DIR}/gcc/build-gcc
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.glibc.done

    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make
    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make install
    COMMAND touch ${TARGET_DIR}/.install.glibc.done

    DEPENDS ${TARGET_DIR}/.install.compiler.done
    WORKING_DIRECTORY ${BUILD_GLIBC_DIR}
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.gcc-all.done
    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make
    COMMAND PATH=${INSTALL_PATH}/bin:$ENV{PATH} make install
    COMMAND touch ${TARGET_DIR}/.install.gcc-all.done
    DEPENDS ${TARGET_DIR}/.install.glibc.done
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR}/gcc/build-gcc
)

add_custom_target(
    make_glibc ALL
    DEPENDS make_binutils make_linux_kernel_headers make_gcc make_gcc_dirs ${TARGET_DIR}/.install.gcc-all.done
    WORKING_DIRECTORY ${TARGET_DIR}
)
