# SPDX-License-Identifier: Apache-2.0

# Point to this project's root directory.
set(SC16IS75_DIR ${CMAKE_CURRENT_LIST_DIR} CACHE PATH "SC16IS75XX module root directory")

# Subdirectories
add_subdirectory(drivers)

# Include headers
zephyr_include_directories(drivers)