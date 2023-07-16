#
# Copyright(c) 2023 gnufas.se and gnusfaas.com
#
# This program and the accompanying materials are made available under the
# terms of the Apache License Version 2.0 http://www.apache.org/licenses/
#
# SPDX-License-Identifier: Apache-2.0
#
if (BUILD_HALLIB)
  add_subdirectory(hal)
endif()
if (BUILD_OSALLIB)
  add_subdirectory(osal)
endif()
