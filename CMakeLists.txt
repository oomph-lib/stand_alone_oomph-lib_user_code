#============================================================
# Sample CMakeLists.txt file to illustrate how to
# link against an existing oomph-lib installation.
# The driver code is the same as in
#
#     demo_drivers/poisson/one_d_poisson
#
#============================================================


# hierher Puneet: can this be imported from oomph-lib's install;
# as a naive user I have no idea which version I need.

# Specify minimum cmake version; die if you can't find it
cmake_minimum_required(VERSION 3.24 FATAL_ERROR)

# Name of the project, followed by languages used
project(one_d_poisson C CXX Fortran)

# The find_package statement declares that:
# -- the project needs oomphlib (obviously!)
# -- we get the configuration (e.g. definition of
#    oomph_add_executable(...), used below) from the
#    oomph-lib project
# -- oomph-lib is required, so if you can't find
#    it die!
#
# Furthermore, we assume here that we've installed oomph-lib
# either in a standard location (/usr/local/) or that the
# install directory is available via the PATH
# environment variable.
#
# Assuming oomph-lib is installed in
#
#    /home/matthias-heil/oomph-lib_install_playground/oomph_lib_serial
#
# you can either provide this path on the command line when
# configuring the current directory:
#
#   cmake -G Ninja -B build -DCMAKE_PREFIX_PATH=/home/matthias-heil/oomph-lib_install_playground/oomph_lib_serial
# 
# or you can hard-code it here by uncommenting the following line
# set(CMAKE_PREFIX_PATH /home/matthias-heil/oomph-lib_install_playground/oomph_lib_serial ${CMAKE_PREFIX_PATH})

# Find oomph-lib or die:
find_package(oomphlib CONFIG REQUIRED)




# Now add the first executable: Specify the name of the
# executable, the sources and the oomph-lib libraries
# required. (This is a Poisson problem so we need
# oomph-lib's generic library, the meshes library and
# the Poisson library)
oomph_add_executable(
  NAME one_d_poisson
  SOURCES one_d_poisson.cc
  LIBRARIES oomph::poisson oomph::meshes oomph::generic)


# Now add the second executable: Specify the name of the
# executable, the sources and the oomph-lib libraries
# required. (This is a Poisson problem so we need
# oomph-lib's generic library, the meshes library and
# the Poisson library). The only difference is that
# we're specifying an additional compiler flag.
oomph_add_executable(
  NAME one_d_poisson_with_flag
  SOURCES one_d_poisson.cc
  LIBRARIES oomph::poisson oomph::meshes oomph::generic)


# Add additional flags for this version.
# NOTE: We can't use the name of exeuctable directly because
# our wrapper oomph_add_excutable(...) creates a different
# (hashed) name for it to avoid internal clashes (don't ask!).
# You therefore have to obtain it from the oomph_get_target_name(...)
# function whose first argument is the NAME specified in the
# oomph_add_executable(...) function above; the second argument
# returns the hashed name that can used for "raw" cmake function
# calls
oomph_get_target_name(one_d_poisson_with_flag HASHED_TARGET_NAME)

# Now set the additional properties
target_compile_options(${HASHED_TARGET_NAME} PRIVATE -DMY_FLAG)


