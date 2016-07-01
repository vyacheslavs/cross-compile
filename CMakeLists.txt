cmake_minimum_required(VERSION 2.6)

set(INSTALL_PATH "/opt/cross-compile" CACHE STRING "Cross compile install path")
set(BINUTILS_VERSION "binutils-2.26" CACHE STRING "BINUTILS VERSION")
set(CC_TARGET "arm-none-linux-gnueabi" CACHE STRING "Cross compile target")
set(KERNEL_VERSION "linux-3.10.20" CACHE STRING "linux kernel version")
set(LINUX_ARCH "arm" CACHE STRING "linux kernel architecture")
set(GCC_VERSION "gcc-4.9.2" CACHE STRING "gcc version")
set(MPFR_VERSION "mpfr-3.1.2" CACHE STRING "mpfr version")
set(GMP_VERSION "gmp-6.0.0a" CACHE STRING "gmp version")
set(MPC_VERSION "mpc-1.0.2" CACHE STRING "mpc version")
set(ISL_VERSION "isl-0.12.2" CACHE STRING "isl version")

add_subdirectory(binutils)
add_subdirectory(linux-kernel)
add_subdirectory(gcc)

add_custom_target(cross_compile ALL
    DEPENDS
        make_binutils
        make_linux_kernel_headers
        make_gcc
)
