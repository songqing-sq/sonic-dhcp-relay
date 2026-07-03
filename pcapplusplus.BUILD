"""Bazel BUILD file for PcapPlusPlus v24.09.

Replaces the upstream CMake build. Produces Common++ and Packet++ static
libraries. Pcap++ is compiled from the main dhcprelay BUILD.bazel because it
needs libpcap headers from @dhcprelay_deps.

3rdParty deps:
  - EndianPortable: header-only
  - hash-library: 1 .cpp file
  - LightPcapNg: 12 .c files (used privately by Pcap++)
  - json: header-only (nlohmann/json)

Feature detection: libpcap in trixie supports PCAP_D_INOUT, so we hard-code
-DHAS_PCAP_IMMEDIATE_MODE and -DHAS_SET_DIRECTION_ENABLED.
"""
load("@rules_cc//cc:cc_library.bzl", "cc_library")

package(default_visibility = ["//visibility:public"])

# 3rdParty: EndianPortable (header-only)
cc_library(
    name = "EndianPortable",
    hdrs = ["3rdParty/EndianPortable/include/EndianPortable.h"],
    includes = ["3rdParty/EndianPortable/include"],
)

# 3rdParty: json (header-only nlohmann/json)
cc_library(
    name = "json",
    hdrs = glob(["3rdParty/json/include/**/*.hpp"]),
    includes = ["3rdParty/json/include"],
)

# 3rdParty: hash-library (md5)
cc_library(
    name = "hash-library",
    srcs = ["3rdParty/hash-library/md5.cpp"],
    hdrs = ["3rdParty/hash-library/md5.h"],
    includes = ["3rdParty/hash-library"],
    deps = [":EndianPortable"],
)

# 3rdParty: LightPcapNg
cc_library(
    name = "light_pcapng",
    srcs = glob(["3rdParty/LightPcapNg/LightPcapNg/src/*.c"]),
    hdrs = glob(["3rdParty/LightPcapNg/LightPcapNg/include/*.h"]),
    includes = ["3rdParty/LightPcapNg/LightPcapNg/include"],
    copts = ["-w"],  # suppress warnings from 3rdParty code
)

# ---------------------------------------------------------------------------
# Common++ and Packet++ libraries.
# Pcap++ is compiled from the main dhcprelay BUILD (needs libpcap headers).
# ---------------------------------------------------------------------------

COMMON_COPTS = [
    "-Wall",
    "-std=c++17",
    "-fPIC",
    '-DGIT_COMMIT=\\"24.09\\"',
    '-DGIT_BRANCH=\\"main\\"',
    "-DHAS_PCAP_IMMEDIATE_MODE",
    "-DHAS_SET_DIRECTION_ENABLED",
]

cc_library(
    name = "Common++",
    srcs = glob(["Common++/src/*.cpp"]),
    hdrs = glob(["Common++/header/*.h"]),
    copts = COMMON_COPTS,
    includes = ["Common++/header"],
    deps = [
        ":EndianPortable",
        ":json",
    ],
)

cc_library(
    name = "Packet++",
    srcs = glob(["Packet++/src/*.cpp"]),
    hdrs = glob(["Packet++/header/*.h"]),
    copts = COMMON_COPTS,
    includes = ["Packet++/header"],
    deps = [
        ":Common++",
        ":EndianPortable",
        ":hash-library",
        ":json",
    ],
)

# Combined public headers tree with pcapplusplus/ prefix (matches upstream
# CMake install layout: /usr/include/pcapplusplus/*.h). dhcp4relay includes
# with <pcapplusplus/DhcpLayer.h> style paths.
cc_library(
    name = "common_prefix_hdrs",
    hdrs = glob(["Common++/header/*.h"]),
    include_prefix = "pcapplusplus",
    strip_include_prefix = "Common++/header",
)

cc_library(
    name = "packet_prefix_hdrs",
    hdrs = glob(["Packet++/header/*.h"]),
    include_prefix = "pcapplusplus",
    strip_include_prefix = "Packet++/header",
)

cc_library(
    name = "pcap_prefix_hdrs",
    hdrs = glob(["Pcap++/header/*.h"]),
    include_prefix = "pcapplusplus",
    strip_include_prefix = "Pcap++/header",
)

cc_library(
    name = "pcapplusplus_hdrs",
    deps = [
        ":common_prefix_hdrs",
        ":packet_prefix_hdrs",
        ":pcap_prefix_hdrs",
    ],
)

# Expose Pcap++ sources/headers so dhcprelay BUILD can compile them
# with libpcap headers from @dhcprelay_deps.
# Exclude conditional sources (XDP/DPDK/KNI/PF_RING/Windows-only) that
# require optional headers.
filegroup(
    name = "pcappp_srcs",
    srcs = glob(
        ["Pcap++/src/*.cpp"],
        exclude = [
            "Pcap++/src/XdpDevice.cpp",
            "Pcap++/src/DpdkDevice.cpp",
            "Pcap++/src/DpdkDeviceList.cpp",
            "Pcap++/src/KniDevice.cpp",
            "Pcap++/src/KniDeviceList.cpp",
            "Pcap++/src/MBufRawPacket.cpp",
            "Pcap++/src/PfRingDevice.cpp",
            "Pcap++/src/PfRingDeviceList.cpp",
            "Pcap++/src/PcapRemoteDevice.cpp",
            "Pcap++/src/PcapRemoteDeviceList.cpp",
            "Pcap++/src/WinPcapLiveDevice.cpp",
        ],
    ),
)

cc_library(
    name = "Pcap_headers",
    hdrs = glob(["Pcap++/header/*.h"]),
    includes = ["Pcap++/header"],
    deps = [
        ":Common++",
        ":EndianPortable",
        ":Packet++",
        ":light_pcapng",
    ],
)
