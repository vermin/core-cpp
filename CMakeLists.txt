
cmake_minimum_required(VERSION 3.4.1)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED YES)

project(TEST)
enable_testing()

#Prep ourselves for compiling boost
find_package(Boost COMPONENTS
    unit_test_framework REQUIRED
    system REQUIRED
    thread REQUIRED
)

set(WAZN_SRC "contrib/wazniya-wazn")

include_directories(${Boost_INCLUDE_DIRS})
include_directories("src")
include_directories(${WAZN_SRC})
include_directories("${WAZN_SRC}/epee/include")
include_directories("${WAZN_SRC}/common")
include_directories("${WAZN_SRC}/vtlogger")
include_directories("${WAZN_SRC}/crypto")
include_directories("${WAZN_SRC}/cryptonote_basic")
include_directories("${WAZN_SRC}/multisig")
include_directories("${WAZN_SRC}/cryptonote_core")
include_directories("${WAZN_SRC}/cryptonote_protocol")
include_directories("${WAZN_SRC}/wallet")
include_directories("${WAZN_SRC}/rpc")
include_directories("${WAZN_SRC}/mnemonics")
include_directories("${WAZN_SRC}/contrib/libsodium/include") # support sodium/… paths
include_directories("${WAZN_SRC}/contrib/libsodium/include/sodium")
include_directories("test")

# keeping test files in a separate source directory
file(GLOB TEST_SRCS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} test/test_*.cpp)

set(
    SRC_FILES
    #
    src/wazn_address_utils.hpp
    src/wazn_address_utils.cpp
    src/wazn_paymentID_utils.hpp
    src/wazn_paymentID_utils.cpp
    src/wazn_key_image_utils.hpp
    src/wazn_key_image_utils.cpp
    src/wazn_fee_utils.hpp
    src/wazn_fee_utils.cpp
    src/wazn_transfer_utils.hpp
    src/wazn_transfer_utils.cpp
    src/wazn_send_routine.hpp
    src/wazn_send_routine.cpp
    src/wazn_fork_rules.hpp
    src/wazn_fork_rules.cpp
    src/wazn_wallet_utils.hpp
    src/wazn_wallet_utils.cpp
    src/serial_bridge_index.hpp
    src/serial_bridge_index.cpp
    src/serial_bridge_utils.hpp
    src/serial_bridge_utils.cpp
    src/tools__ret_vals.hpp
    src/tools__ret_vals.cpp
    #
    ${WAZN_SRC}/cryptonote_basic/cryptonote_basic_impl.cpp
    ${WAZN_SRC}/cryptonote_basic/account.cpp
    ${WAZN_SRC}/cryptonote_basic/cryptonote_format_utils.cpp
    ${WAZN_SRC}/crypto/crypto.cpp
    ${WAZN_SRC}/crypto/hash.c
    ${WAZN_SRC}/crypto/slow-hash-dummied.cpp
    ${WAZN_SRC}/crypto/oaes_lib.c
    ${WAZN_SRC}/crypto/crypto-ops.c
    ${WAZN_SRC}/crypto/crypto-ops-data.c
    ${WAZN_SRC}/crypto/keccak.c
    ${WAZN_SRC}/crypto/chacha.c
    ${WAZN_SRC}/crypto/random.c
    ${WAZN_SRC}/crypto/aesb.c
    ${WAZN_SRC}/crypto/tree-hash.c
    ${WAZN_SRC}/crypto/hash-extra-blake.c
    ${WAZN_SRC}/crypto/blake256.c
    ${WAZN_SRC}/crypto/hash-extra-groestl.c
    ${WAZN_SRC}/crypto/hash-extra-jh.c
    ${WAZN_SRC}/crypto/hash-extra-skein.c
    ${WAZN_SRC}/crypto/groestl.c
    ${WAZN_SRC}/crypto/jh.c
    ${WAZN_SRC}/crypto/skein.c
    ${WAZN_SRC}/cryptonote_core/cryptonote_tx_utils.cpp
    ${WAZN_SRC}/common/base58.cpp
    ${WAZN_SRC}/common/threadpool.cpp
    ${WAZN_SRC}/common/aligned.c
    ${WAZN_SRC}/common/util.cpp
    ${WAZN_SRC}/epee/src/hex.cpp
    ${WAZN_SRC}/epee/src/string_tools.cpp
    ${WAZN_SRC}/epee/src/memwipe.c
    ${WAZN_SRC}/epee/src/mlocker.cpp
    ${WAZN_SRC}/epee/src/wipeable_string.cpp
    ${WAZN_SRC}/device/device.cpp
    ${WAZN_SRC}/device/device_default.cpp
    ${WAZN_SRC}/ringct/rctOps.cpp
    ${WAZN_SRC}/ringct/rctTypes.cpp
    ${WAZN_SRC}/ringct/rctCryptoOps.c
    ${WAZN_SRC}/ringct/rctSigs.cpp
    ${WAZN_SRC}/ringct/bulletproofs.cc
    ${WAZN_SRC}/ringct/multiexp.cc
    ${WAZN_SRC}/mnemonics/electrum-words.cpp
    ${WAZN_SRC}/vtlogger/logger.cpp
    ${WAZN_SRC}/contrib/libsodium/src/crypto_verify/verify.c
)
# needed for slow-hash
add_compile_options(-maes)

#Run through each source
foreach(testSrc ${TEST_SRCS})
    # extract the filename without an extension (NAME_WE)
    get_filename_component(testName ${testSrc} NAME_WE)
    add_executable(
        ${testName}
        ${testSrc}
        ${SRC_FILES}
    )
    target_link_libraries(
        ${testName}
        #
        ${Boost_LIBRARIES}
    )
    set_target_properties(
        ${testName} PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY
        ${CMAKE_CURRENT_SOURCE_DIR}/build/products
    )
    add_test(
        NAME ${testName}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/build/products
        COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/build/products/${testName} --catch_system_error=yes
    )
endforeach(testSrc)
