#============================================================
# Sample CMakeLists.txt file to illustrate how to
# link against an existing oomph-lib installation.
# The driver code is essentially the same as in
#
#     demo_drivers/poisson/one_d_poisson
#
#============================================================

#--------------------------------------------------------------
# Specify minimum CMake version; die if you can't find it...
#
# Note to inexperienced users: CMake requires this flag. If you
# have no preferences yourself, simply use the value specified
# in the CMakeLists.txt file in the oomph-lib root directory.
#
#--------------------------------------------------------------
cmake_minimum_required(VERSION 3.24 FATAL_ERROR)

#--------------------------------------------------------------
# Name of the project, followed by languages used; the name doesn't
# really matter, so my_test_project will do. If you don't specify this
# you'll get a warning. We also use this function to specify which 
# languages are required (oomph-lib is written entirely in C++, so we suspect
# your driver code is too).
#--------------------------------------------------------------
project(my_test_project C CXX Fortran)

#--------------------------------------------------------------
# Locate an installed version of oomph-lib.
# 
# The find_package statement declares that:
# -- the project needs oomphlib (obviously!)
# -- we get the configuration (e.g. definition of
#    oomph_add_executable(...), used below) from the
#    oomph-lib project
# -- oomph-lib is required, so if you can't find
#    it die!
#
# During oomph-lib's installation, a file
# oomphlibConfig.cmake is created. It explains to CMake how to
# locate the oomph-lib header/library files and provides some
# additional information. We expect this file to sit somewhere 
# under a standard prefix (/usr/local, /opt/...). 
# 
# If you have installed oomph-lib elsewhere you will have to remember the 
# directory where oomph-lib was installed. Here are some 
# typical situations:
# 
# -- Assume you built oomph-lib by running the oomph_build.py 
#    script in the /home/joe_cool/oomph-lib/ directory, say. Then by default
#    the library will be installed in /home/joe_cool/oomph-lib/install
#    and the required file will be 
#
#      /home/joe_cool/oomph-lib/install/lib/cmake/oomphlib/oomphlibConfig.cmake 
#
#     You'll have to pass the path to the oomph-lib install directory to CMake when building
#     your current project, using the -Doomphlib_ROOT or -DCMAKE_PREFIX_PATH 
#     arguments. So, assuming you're using Ninja as the generator and want to build
#     your project in a local directory called build, use
#
#        cmake -G Ninja -B build -Doomphlib_ROOT=/home/joe_cool/oomph-lib/install
#
# or
#
#        cmake -G Ninja -B build -DCMAKE_PREFIX_PATH=/home/joe_cool/oomph-lib/install
#
# -- If you have specified a different install directory when building/installing
#    oomph-lib, then specify that directory instead. So, if you ran oomph_build.py with the
#    --root-CMAKE_INSTALL_PREFIX PATH flag, or used -DCMAKE_INSTALL_PREFIX when installing
#    oomph-lib directly using CMake, then specify the directory you provided as an argument
#    for these flags. 
#
# If you're working by yourself and always want to link against the same oomph-lib
# installation you can also hardcode the information here, by uncommenting the following
# line (and updating the reference to joe_cool, obviously)
# 
#   set(oomphlib_ROOT "/home/joe_cool/oomph-lib/install" CACHE PATH "oomph-lib install prefix")
# 
# while this saves you a bit of typing we discourage this as it reduces 
# portability. Having hard-coded paths in CMakeLists.txt files
# is generally considered bad practice.
#--------------------------------------------------------------
find_package(oomphlib CONFIG REQUIRED)


#--------------------------------------------------------------
# Define the first executable: Specify the name of the
# executable, the sources and the oomph-lib libraries
# required. (This is a Poisson problem so we need
# oomph-lib's generic library, the meshes library and
# the Poisson library). The 
#--------------------------------------------------------------
oomph_add_executable(
  NAME one_d_poisson
  SOURCES one_d_poisson.cc
  LIBRARIES oomph::poisson oomph::meshes oomph::generic)


#------------------------------------------------------------
# Define the second executable: Specify the name of the
# executable, the sources and the oomph-lib libraries
# required. (This is a Poisson problem so we need
# oomph-lib's generic library, the meshes library and
# the Poisson library). (Note: this is exactly the same
# as for the previous executable, but we demonstrate below
# how to specify additional compiler flags for this executable.
#--------------------------------------------------------------
oomph_add_executable(
  NAME one_d_poisson_with_flag
  SOURCES one_d_poisson.cc
  LIBRARIES oomph::poisson oomph::meshes oomph::generic)


#--------------------------------------------------------------
# We wish to provide additional compiler flags for this executable.
# NOTE: We can't use the name of the executable directly because
# our wrapper oomph_add_executable(...) creates a different
# (hashed) name for it to avoid internal clashes (don't ask!).
# You therefore have to obtain the actual target name from the 
# function oomph_get_target_name(...) whose first argument is the NAME 
# specified in the oomph_add_executable(...) function above; 
# the second argument returns the hashed name that can then 
# used for "raw" CMake function calls.
#--------------------------------------------------------------

#---------------------------------------------------------------
# Get hashed target name
#---------------------------------------------------------------
oomph_get_target_name(one_d_poisson_with_flag HASHED_TARGET_NAME)


#--------------------------------------------------------------------
# The driver code contains the macro
#
# #ifdef MY_FLAG
# [...]
# #endif
# 
# Such flags can be passed to most compilers using the compiler 
# flag "-DMY_FLAG".
# 
# If you're working on your own and only ever use a compiler that
# handles macros this way, you could uncomment the following line to 
# achieve this
#
#    target_compile_options(${HASHED_TARGET_NAME} PRIVATE -DMY_FLAG) 
#
# This function passes the flag "-DMY_FLAG" verbatim to the compiler.
# The function can therefore also be used to specify other compiler-specific
# flags like -O3 or -Wall, say.
#
# However, CMake provides a compiler-independent way of specifying 
# compiler macros: the target_compile_definitions function which simply 
# takes the flag (without any compiler-specific prefixes like "-D"):
#--------------------------------------------------------------------------
target_compile_definitions(${HASHED_TARGET_NAME} PRIVATE MY_FLAG)



