set(TARGET_DIR "${PROJECT_BINARY_DIR}/gcc")
set(BUILD_TARGET_DIR "${PROJECT_BINARY_DIR}/gcc/build-gcc")
set(TARGET_NAME "${GCC_VERSION}")
set(TARGET_NAME_ARC "${GCC_VERSION}.tar.gz")

add_custom_command(
    OUTPUT ${TARGET_DIR}/.mkdir.done

    COMMAND mkdir -p ${TARGET_DIR}
    COMMAND mkdir -p ${TARGET_DIR}/${TARGET_NAME}
    COMMAND mkdir -p ${BUILD_TARGET_DIR}
    COMMAND touch ${TARGET_DIR}/.mkdir.done
)

add_custom_target(
    make_gcc_dirs ALL
    DEPENDS ${TARGET_DIR}/.mkdir.done
    WORKING_DIRECTORY ${TARGET_DIR}
)
