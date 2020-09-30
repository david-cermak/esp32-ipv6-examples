# The following lines of boilerplate have to be in your project's CMakeLists
# in this exact order for cmake to work correctly
cmake_minimum_required(VERSION 3.5)

# (Not part of the boilerplate)
# This example uses an extra component for common functions such as Wi-Fi and Ethernet connection.
set(EXTRA_COMPONENT_DIRS $ENV{IDF_PATH}/examples/common_components/protocol_examples_common)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(ipv6-test)

idf_component_get_property(lwip lwip COMPONENT_LIB)

if(CONFIG_IPV6_DHCP6_OTHER_FLAG)
    target_compile_definitions(${lwip} PRIVATE "-DLWIP_IPV6_DHCP6=1")
endif()

if(CONFIG_IPV6_RA_RDNSS)
    target_compile_definitions(${lwip} PRIVATE "-DLWIP_ND6_RDNSS_MAX_DNS_SERVERS=1")
endif()
