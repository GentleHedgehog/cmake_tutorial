# description:
# policies are used to control over whether old or new behavior should be used

# --- Policy control ---
# cmake_minimum_required(VERSION x.x) - ensure to use policies that are specific for x.x version

# more fine-grained control:
# cmake_policy(VERSION major[.minor[.patch[.tweak]]])
#   cmake_minimum_required implicitely call it
#   example:
#       cmake_minimum_required(VERSION 3.7)
#       project(WithLegacy)
#       # uses recent cmake features:
#       add_subdirectory(modernDir)
#       # import from another project, relies on old behavior:
#       cmake_policy(VERSION 2.8.11)
#       add_subdirectory(legacyDir)

# version range (from CMake 3.12):
# cmake_policy(VERSION 3.7...3.12) # cmake ver < 3.12 will ignore '...3.12' part

# set individual policy (to stop any warning too):
# cmake_policy(SET CMPxxxx NEW) # if you need a new feature in the project with an old ver in cmake_minimum_required
# cmake_policy(SET CMPxxxx OLD) # if your code rely on old behavior

# special if:
# if (POLICY CMPxxxx)
#   cmake_policy(SET CMPxxxx NEW)
# endif()

# get policy val: cmake_policy(GET CMPxxxx outVar) # -> OLD, NEW or empty

# cmake_minimum_required and cmake_policy:
#   reset the state of all policies
#   policies for pointed ver(or earlier) - reset to NEW
#   policies that were added after the pointed ver - reset to empty


# set as a cache vars to suppress warnings:
# CMAKE_POLICY_DEFAULT_CMPxxxx
# CMAKE_POLICY_WARNING_CMPxxxx



# --- Policy scope ---

# policy stack (often used in modules):
# cmake_policy(PUSH)
# cmake_policy(SET CMPxxxx OLD)
# cmake_policy(POP)

# add_subdirectory() uses PUSH (at entering) and POP (at return)
# include() and find_package() do the same thing but have NO_POLICY_SCOPE option (for backward compatibility)


# --- Recommended practices ---
# avoid using of cmake_policy (harder to track)
# use if(POLICY..) to check it
# use PUSH and POP to set individual policies (pay attention to return())
# avoid NO_POLICY_SCOPE
# avoid set policy in functions (without push/pop) - change it in scope where func is defined
# avoid using CMAKE_POLICY_DEFAULT_CMPxxxx and CMAKE_POLICY_WARNING_CMPxxxx - update to new behavior



