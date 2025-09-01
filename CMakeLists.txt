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
# languages are required (oomph-lib is written entirely in C++, 
# so we suspect your driver code is too).
#--------------------------------------------------------------
project(my_test_project CXX C Fortran)

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
# additional information. By default we expect this file to sit somewhere 
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
#     (Check that it exists!) 
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
# While this saves you a bit of typing we discourage this as it reduces 
# portability. Having hard-coded paths in CMakeLists.txt files
# is generally considered bad practice.
#--------------------------------------------------------------
find_package(oomphlib CONFIG REQUIRED)


#####################################################################################
# Normal build of driver code without any customisation (by default!). 
#####################################################################################

#--------------------------------------------------------------
# Define the first executable: Specify the name of the
# executable, the sources and the oomph-lib libraries
# required. (This is a Poisson problem so we need
# oomph-lib's generic library, the meshes library and
# the Poisson library).
#--------------------------------------------------------------
oomph_add_executable(
  NAME one_d_poisson
  SOURCES one_d_poisson.cc
  LIBRARIES oomph::poisson oomph::meshes oomph::generic)

#--------------------------------------------------------------
# Note that when providing additional compiler flags using the native
# CMake target_compile_options function (or using other native
# CMake functions that affect a given target) we can't
# use the name of the executable directly because
# our wrapper oomph_add_executable(...) creates a different
# (hashed) name for it to avoid internal clashes (don't ask!).
# You therefore have to obtain the actual target name from the 
# function oomph_get_target_name(...) whose first argument is the NAME 
# specified in the oomph_add_executable(...) function above; 
# the second argument returns the hashed name that can then 
# used for native CMake function calls.
#--------------------------------------------------------------

#---------------------------------------------------------------
# Get hashed target name
#---------------------------------------------------------------
oomph_get_target_name(one_d_poisson HASHED_TARGET_NAME)

#-------------------------------------------------------------
# Option to add -DMY_FLAG to C++ compilation ; activate
# by configuring project with
#
#   cmake [...] -DMY_FLAG=ON
#
# Do not brute force such flags into the build by configuring
# with -DCMAKE_CXX_FLAGS. It would overwrite flags defined
# in the oomph-lib build!
#-------------------------------------------------------------
if(MY_FLAG) 
    target_compile_definitions(${HASHED_TARGET_NAME} PRIVATE MY_FLAG)
endif()



#####################################################################################
# Three options to customise build by passing compiler options and
# (macro) definitions
#####################################################################################


#####################################################################################
# Option 1: Use additional arguments in oomph_add_executable():
#####################################################################################

#---------------------------------------------------------------
# The driver code contains the macro
#
# #ifdef MY_FLAG
#   [...]
# #endif
# 
# Such flags can be passed to most compilers using the compiler 
# flag "-DMY_FLAG".
# 
# To achieve this from within the CMakeFiles.txt file, such macros
# can be passed to the compiler (or rather the preprocessor) by
# adding it (without the -D prefix!) to the CXX_DEFINITIONS
# argument.
#
# We also provide the option to specify compiler options
# such as -Wall. 
#--------------------------------------------------------------
oomph_add_executable(
  NAME one_d_poisson_with_flags1
  SOURCES one_d_poisson.cc
  LIBRARIES oomph::poisson oomph::meshes oomph::generic
  CXX_OPTIONS -Wall 
  CXX_DEFINITIONS MY_FLAG)


#####################################################################################
# Option 2: Use native CMake commands to add the relevant flags afterwards
#####################################################################################

#------------------------------------------------------------
# You can also use native CMake commands to achieve this.
# To demonstrate this, here's another target, defined without
# the CXX_OPTIONS and CXX_DEFINITIONS arguments:
#--------------------------------------------------------------
oomph_add_executable(
  NAME one_d_poisson_with_flags2
  SOURCES one_d_poisson.cc
  LIBRARIES oomph::poisson oomph::meshes oomph::generic)

#--------------------------------------------------------------
# Note that when providing additional compiler flags using the native
# CMake target_compile_options function (or using other native
# CMake functions that affect a given target) we can't
# use the name of the executable directly because
# our wrapper oomph_add_executable(...) creates a different
# (hashed) name for it to avoid internal clashes (don't ask!).
# You therefore have to obtain the actual target name from the 
# function oomph_get_target_name(...) whose first argument is the NAME 
# specified in the oomph_add_executable(...) function above; 
# the second argument returns the hashed name that can then 
# used for native CMake function calls.
#--------------------------------------------------------------

#---------------------------------------------------------------
# Get hashed target name
#---------------------------------------------------------------
oomph_get_target_name(one_d_poisson_with_flags2 HASHED_TARGET_NAME)


#---------------------------------------------------------------------
# Now use the hashed name when using the native CMake functions:
#---------------------------------------------------------------------
target_compile_definitions(${HASHED_TARGET_NAME} PRIVATE MY_FLAG)
target_compile_options(${HASHED_TARGET_NAME} PRIVATE -Wall)




#####################################################################################
# Option 3: Do it all with native CMake commands
#####################################################################################

# Specify the target name and source(s)
add_executable(one_d_poisson_with_flags3 one_d_poisson.cc)

# Link against oomph-lib's poisson library
target_link_libraries(one_d_poisson_with_flags3 PRIVATE oomph::poisson)

# Compiler options
target_compile_options(one_d_poisson_with_flags3 PRIVATE -Wall) 

# Preprocessor macros
target_compile_definitions(one_d_poisson_with_flags3 PRIVATE MY_FLAG)

# Pull in the information about oomph-lib build
target_compile_definitions(one_d_poisson_with_flags3 PRIVATE ${OOMPH_COMPILE_DEFINITIONS})




#####################################################################################
# Option 4: Allow enabling the compiler macro via the cmake command line when
# configuring the project
#####################################################################################

#------------------------------------------------------------
#  Here's the general setup, without any flags
#--------------------------------------------------------------
oomph_add_executable(
  NAME one_d_poisson_with_flags4
  SOURCES one_d_poisson.cc
  LIBRARIES oomph::poisson oomph::meshes oomph::generic)

#--------------------------------------------------------------
# Note that when providing additional compiler flags using the native
# CMake target_compile_options function (or using other native
# CMake functions that affect a given target) we can't
# use the name of the executable directly because
# our wrapper oomph_add_executable(...) creates a different
# (hashed) name for it to avoid internal clashes (don't ask!).
# You therefore have to obtain the actual target name from the 
# function oomph_get_target_name(...) whose first argument is the NAME 
# specified in the oomph_add_executable(...) function above; 
# the second argument returns the hashed name that can then 
# used for native CMake function calls.
#--------------------------------------------------------------

#---------------------------------------------------------------
# Get hashed target name
#---------------------------------------------------------------
oomph_get_target_name(one_d_poisson_with_flags4 HASHED_TARGET_NAME)


#---------------------------------------------------------------
# Conditional application of the MY_FLAG macro. Enable this
# by configuring the project with
#
#   cmake -G Ninja -B build -DCMAKE_PREFIX_PATH=/home/joe_cool/oomph-lib/install -DMY_FLAG=ON
#
# say. Configuring with
#
#   cmake -G Ninja -B build -DCMAKE_PREFIX_PATH=/home/joe_cool/oomph-lib/install
#
# is equivalent to
#
#   cmake -G Ninja -B build -DCMAKE_PREFIX_PATH=/home/joe_cool/oomph-lib/install -DMY_FLAG=OFF
#
#---------------------------------------------------------------
if(MY_FLAG)
    target_compile_definitions(${HASHED_TARGET_NAME} PRIVATE MY_FLAG)
endif()



#####################################################################################

#--------------------------------------------------------------------------
# Final comment: The oomphlibConfig.cmake file contains information about
# all the flags used during the oomph-lib build process. This means
# in particular that any compiler flags used during this process (PARANOID,
# RANGE_CHECKING, OOMPH_HAS_MPI, etc.) are available to any driver
# codes that use this installation. This is demonstrated by the final #ifdef 
# at the bottom of the driver code where we announce if oomph-lib was 
# compiled with MPI support or not.
#--------------------------------------------------------------------------
