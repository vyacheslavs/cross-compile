
set(TARGET_DIR "${PROJECT_BINARY_DIR}/isl")
set(TARGET_NAME "${ISL_VERSION}")
set(TARGET_NAME_ARC "${ISL_VERSION}.tar.bz2")

add_custom_command(
    OUTPUT ${TARGET_DIR}/.mkdir.done

    COMMAND mkdir -p ${TARGET_DIR}
    COMMAND touch ${TARGET_DIR}/.mkdir.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.download.done

    COMMAND wget "ftp://gcc.gnu.org/pub/gcc/infrastructure/${TARGET_NAME_ARC}" -O ${TARGET_DIR}/${TARGET_NAME_ARC}
    COMMAND touch ${TARGET_DIR}/.download.done

    DEPENDS ${TARGET_DIR}/.mkdir.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.untar.done

    COMMAND mkdir -p ${TARGET_DIR}/${TARGET_NAME}
    COMMAND tar xpvjf ${TARGET_DIR}/${TARGET_NAME_ARC} -C ${TARGET_DIR}/${TARGET_NAME} --strip-components=1
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

    COMMAND ln -sf ${TARGET_DIR}/${TARGET_NAME} ${PROJECT_BINARY_DIR}/gcc/${GCC_VERSION}/isl
    COMMAND touch ${TARGET_DIR}/.install.done

    WORKING_DIRECTORY ${TARGET_DIR}
    DEPENDS ${TARGET_DIR}/.build.done
)

add_custom_target(
    make_isl ALL
    DEPENDS make_gcc_dirs ${TARGET_DIR}/.install.done
    WORKING_DIRECTORY ${TARGET_DIR}
)
