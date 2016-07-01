
set(TARGET_DIR "${PROJECT_BINARY_DIR}/linux-headers")
set(TARGET_NAME "${KERNEL_VERSION}")
set(TARGET_NAME_ARC "${KERNEL_VERSION}.tar.xz")

add_custom_command(
    OUTPUT ${TARGET_DIR}/.mkdir.done

    COMMAND mkdir -p ${TARGET_DIR}
    COMMAND touch ${TARGET_DIR}/.mkdir.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.download.done

    COMMAND wget "https://www.kernel.org/pub/linux/kernel/v3.x/${TARGET_NAME_ARC}" -O ${TARGET_DIR}/${TARGET_NAME_ARC}
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

    COMMAND make ARCH=${LINUX_ARCH} INSTALL_HDR_PATH=${INSTALL_PATH}/${CC_TARGET} headers_install

    COMMAND touch ${TARGET_DIR}/.configure.done

    WORKING_DIRECTORY ${TARGET_DIR}/${TARGET_NAME}
    DEPENDS ${TARGET_DIR}/.untar.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.build.done

    COMMAND touch ${TARGET_DIR}/.build.done

    WORKING_DIRECTORY ${TARGET_DIR}/${TARGET_NAME}
    DEPENDS ${TARGET_DIR}/.configure.done
)

add_custom_command(
    OUTPUT ${TARGET_DIR}/.install.done

    COMMAND touch ${TARGET_DIR}/.install.done

    WORKING_DIRECTORY ${TARGET_DIR}/${TARGET_NAME}
    DEPENDS ${TARGET_DIR}/.build.done
)

add_custom_target(
    make_linux_kernel_headers ALL
    DEPENDS ${TARGET_DIR}/.install.done
    WORKING_DIRECTORY ${TARGET_DIR}
)
