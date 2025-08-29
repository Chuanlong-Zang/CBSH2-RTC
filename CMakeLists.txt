cmake_minimum_required(VERSION 3.16)
project(cbsh2rtc LANGUAGES CXX)

# Build the CLI driver too?
option(CBSH2RTC_BUILD_DRIVER "Build the cbs command-line driver" ON)

# --- Sources ------------------------------------------------------------------
file(GLOB CBSH2RTC_ALL_SOURCES CONFIGURE_DEPENDS src/*.cpp)
set(CBSH2RTC_DRIVER_SRC src/driver.cpp)
list(REMOVE_ITEM CBSH2RTC_ALL_SOURCES ${CBSH2RTC_DRIVER_SRC})

# --- Core library -------------------------------------------------------------
add_library(cbsh2rtc_core STATIC ${CBSH2RTC_ALL_SOURCES})
add_library(cbsh2rtc::core ALIAS cbsh2rtc_core)

target_include_directories(cbsh2rtc_core
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc>
        $<INSTALL_INTERFACE:include>
)

# Prefer features instead of forcing global flags
target_compile_features(cbsh2rtc_core PUBLIC cxx_std_17)
set_target_properties(cbsh2rtc_core PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Most core code only needs header-only Boost (tokenizer, etc.).
find_package(Boost REQUIRED)  # header-only OK
target_link_libraries(cbsh2rtc_core PUBLIC Boost::boost)

# If your core sources use Boost.Filesystem (rare), uncomment:
# find_package(Boost REQUIRED COMPONENTS filesystem)
# target_link_libraries(cbsh2rt_core PUBLIC Boost::filesystem)

# --- Optional CLI driver ------------------------------------------------------
if(CBSH2RTC_BUILD_DRIVER)
    find_package(Boost REQUIRED COMPONENTS program_options filesystem)
    add_executable(cbs ${CBSH2RTC_DRIVER_SRC})
    target_link_libraries(cbs PRIVATE cbsh2rtc_core Boost::program_options Boost::filesystem)
    target_compile_features(cbs PRIVATE cxx_std_17)
endif()