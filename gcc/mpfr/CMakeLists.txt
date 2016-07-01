
set(TARGET_DIR "${PROJECT_BINARY_DIR}/mpfr")
set(TARGET_NAME "${MPFR_VERSION}")
set(TARGET_NAME_ARC "${MPFR_VERSION}.tar.xz")

add_custom_command(
    OUTPUT ${TARGET_DIR}/.mkdir.done

    COMMAND mkdir -p ${TARGET_DIR}
    COMMAND touch ${TARGET_DIR}/.mkdir.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.download.done

    COMMAND wget "https://ftp.gnu.org/gnu/mpfr/${TARGET_NAME_ARC}" -O ${TARGET_DIR}/${TARGET_NAME_ARC}
    COMMAND touch ${TARGET_DIR}/.download.done

    DEPENDS ${TARGET_DIR}/.mkdir.done
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

    COMMAND touch ${TARGET_DIR}/.configure.done

    WORKING_DIRECTORY ${TARGET_DIR}
    DEPENDS ${TARGET_DIR}/.untar.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.build.done

    COMMAND touch ${TARGET_DIR}/.build.done

    WORKING_DIRECTORY ${TARGET_DIR}
    DEPENDS ${TARGET_DIR}/.configure.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.done

    COMMAND ln -sf ${TARGET_DIR}/${TARGET_NAME} ${PROJECT_BINARY_DIR}/gcc/${GCC_VERSION}/mpfr
    COMMAND touch ${TARGET_DIR}/.install.done

    WORKING_DIRECTORY ${TARGET_DIR}
    DEPENDS ${TARGET_DIR}/.build.done
)

add_custom_target(
    make_mpfr ALL
    DEPENDS make_gcc_dirs ${TARGET_DIR}/.install.done
    WORKING_DIRECTORY ${TARGET_DIR}
)
