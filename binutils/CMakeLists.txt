
set(TARGET_DIR "${PROJECT_BINARY_DIR}/binutils")

add_custom_command(
    OUTPUT ${TARGET_DIR}/.mkdir.done

    COMMAND mkdir -p ${TARGET_DIR}
    COMMAND touch ${TARGET_DIR}/.mkdir.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.download.done

    COMMAND wget "https://ftp.gnu.org/gnu/binutils/${BINUTILS_VERSION}.tar.gz" -O ${TARGET_DIR}/${BINUTILS_VERSION}.tar.gz
    COMMAND touch ${TARGET_DIR}/.download.done

    DEPENDS ${TARGET_DIR}/.mkdir.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.untar.done

    COMMAND tar xpvf ${TARGET_DIR}/${BINUTILS_VERSION}.tar.gz -C ${TARGET_DIR}
    COMMAND touch ${TARGET_DIR}/.untar.done

    DEPENDS ${TARGET_DIR}/.download.done
    WORKING_DIRECTORY ${TARGET_DIR}
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.configure.done

    COMMAND ${TARGET_DIR}/${BINUTILS_VERSION}/configure --prefix=${INSTALL_PATH} --target=${CC_TARGET} --disable-multilib
    COMMAND touch ${TARGET_DIR}/.configure.done

    WORKING_DIRECTORY ${TARGET_DIR}/${BINUTILS_VERSION}
    DEPENDS ${TARGET_DIR}/.untar.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.build.done

    COMMAND make
    COMMAND touch ${TARGET_DIR}/.build.done

    WORKING_DIRECTORY ${TARGET_DIR}/${BINUTILS_VERSION}
    DEPENDS ${TARGET_DIR}/.configure.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.done

    COMMAND make install
    COMMAND touch ${TARGET_DIR}/.install.done

    WORKING_DIRECTORY ${TARGET_DIR}/${BINUTILS_VERSION}
    DEPENDS ${TARGET_DIR}/.build.done
)

add_custom_target(
    make_binutils ALL
    DEPENDS ${TARGET_DIR}/.install.done
    WORKING_DIRECTORY ${TARGET_DIR}
)
