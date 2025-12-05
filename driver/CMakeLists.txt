cmake_minimum_required(VERSION 3.15)
project(smi_modules VERSION 0.1.0 LANGUAGES C)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Module info
add_definitions(-D__KERNEL__ -DMODULE)

# Find the kernel release
execute_process(
        COMMAND uname -r
        OUTPUT_VARIABLE KERNEL_RELEASE
        OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Find the headers
find_path(
        KERNELHEADERS_DIR
        NAMES
            include/linux/user.h
            include/generated/autoconf.h
        PATHS /usr/src/linux-headers-${KERNEL_RELEASE}
)

message(STATUS "Kernel release: ${KERNEL_RELEASE}")
message(STATUS "Kernel headers: ${KERNELHEADERS_DIR}")

# Check kernel version for class_create API change (changed in 6.4)
string(REGEX MATCH "^([0-9]+)\\.([0-9]+)" KERNEL_VERSION_MATCH ${KERNEL_RELEASE})
set(KERNEL_MAJOR ${CMAKE_MATCH_1})
set(KERNEL_MINOR ${CMAKE_MATCH_2})

# Kernel 6.4+ uses class_create without owner parameter
if((KERNEL_MAJOR GREATER 6) OR ((KERNEL_MAJOR EQUAL 6) AND (KERNEL_MINOR GREATER_EQUAL 4)))
    set(HAVE_CLASS_WITHOUT_OWNER 1)
else()
    set(HAVE_CLASS_WITHOUT_OWNER 0)
endif()

message(STATUS "HAVE_CLASS_WITHOUT_OWNER: ${HAVE_CLASS_WITHOUT_OWNER}")

function(compile_module obj)
    set(TARGET_NAME ${obj})
    add_custom_target(${TARGET_NAME} ALL cp -f ${CMAKE_CURRENT_SOURCE_DIR}/*.c ${CMAKE_CURRENT_SOURCE_DIR}/*.h ${CMAKE_CURRENT_BINARY_DIR}/
        COMMAND echo "compiling module ${obj}.ko...")
    list(LENGTH ARGN argn_len)
    set(i 0)
    set(depend_objlist "")
    while(i LESS ${argn_len})  
        list(GET ARGN ${i} argn_value)  
        set(depend_objlist "${depend_objlist} ${argn_value}.o") 
        math(EXPR i "${i} + 1")  
    endwhile() 

    add_custom_command(TARGET ${TARGET_NAME}
        POST_BUILD
        COMMAND cp Makefile Makefile.bak
        COMMAND echo "obj-m += ${obj}.o" > ${CMAKE_CURRENT_BINARY_DIR}/Makefile
        #COMMAND echo "MY_CFLAGS += -g -DDEBUG" >> ${CMAKE_CURRENT_BINARY_DIR}/Makefile
        #COMMAND echo "ccflags-y += -g -DDEBUG" >> ${CMAKE_CURRENT_BINARY_DIR}/Makefile
        #COMMAND echo "CC += -g -DDEBUG" >> ${CMAKE_CURRENT_BINARY_DIR}/Makefile
        COMMAND echo "ccflags-y += -O2 -DMODULE -D__KERNEL__ -DHAVE_CLASS_WITHOUT_OWNER=${HAVE_CLASS_WITHOUT_OWNER}" >> ${CMAKE_CURRENT_BINARY_DIR}/Makefile
        COMMAND echo "${obj}-objs:=${depend_objlist}" >> ${CMAKE_CURRENT_BINARY_DIR}/Makefile
        #COMMAND make -C ${KERNELHEADERS_DIR} M=${CMAKE_CURRENT_BINARY_DIR} modules EXTRA_CFLAGS="-g"
        COMMAND make -C ${KERNELHEADERS_DIR} M=${CMAKE_CURRENT_BINARY_DIR} modules
        COMMAND cp Makefile Makefile.op
        COMMAND cp Makefile.bak Makefile
        )
endfunction()

#compile_module(bcm2835_smi)
compile_module(smi_stream_dev)
#compile_module(bcm2835_smi_dev)
