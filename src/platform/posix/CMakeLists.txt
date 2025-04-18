#
# Copyright 2023 Staysail Systems, Inc. <info@staystail.tech>
#
# This software is supplied under the terms of the MIT License, a
# copy of which should be located in the distribution where this
# file was obtained (LICENSE.txt).  A copy of the license may also be
# found online at https://opensource.org/licenses/MIT.
#

#  POSIX.

# We cannot use nng_sources_if because these tests don't go into
# the static library unless they also go into the dynamic.
if (NNG_PLATFORM_POSIX)

    nng_find_package(Threads)
    nng_link_libraries(Threads::Threads)

    # Unconditionally declare the following feature test macros.  These are
    # needed for some platforms (glibc and SunOS/illumos) and are harmless
    # on the others.
    nng_defines(_GNU_SOURCE)
    nng_defines(_REENTRANT)
    nng_defines(_THREAD_SAFE)
    nng_defines(_POSIX_PTHREAD_SEMANTICS)

    nng_check_func(lockf NNG_HAVE_LOCKF)
    nng_check_func(flock NNG_HAVE_FLOCK)
    nng_check_func(getentropy NNG_HAVE_GETENTROPY)
    nng_check_func(getrandom NNG_HAVE_GETRANDOM)
    nng_check_func(arc4random_buf NNG_HAVE_ARC4RANDOM)
    nng_check_func(recvmsg NNG_HAVE_RECVMSG)
    nng_check_func(sendmsg NNG_HAVE_SENDMSG)

    nng_check_func(clock_gettime NNG_HAVE_CLOCK_GETTIME_LIBC)
    if (NNG_HAVE_CLOCK_GETTIME_LIBC)
        nng_defines_if(NNG_HAVE_CLOCK_GETTIME_LIBC NNG_HAVE_CLOCK_GETTIME)
    else()
        nng_check_lib(rt clock_gettime NNG_HAVE_CLOCK_GETTIME)
    endif()
    nng_check_lib(pthread sem_wait NNG_HAVE_SEMAPHORE_PTHREAD)
    nng_check_lib(pthread pthread_atfork NNG_HAVE_PTHREAD_ATFORK_PTHREAD)
    nng_check_lib(pthread pthread_set_name_np NNG_HAVE_PTHREAD_SET_NAME_NP)
    nng_check_lib(pthread pthread_setname_np NNG_HAVE_PTHREAD_SETNAME_NP)
    nng_check_lib(nsl gethostbyname NNG_HAVE_LIBNSL)
    nng_check_lib(socket socket NNG_HAVE_LIBSOCKET)

    # GCC needs libatomic on some architectures (e.g. ARM) because the
    # underlying architecture may lack the necessary atomic primitives.
    # One hopes that the libatomic implementation is superior to just using
    # a pthread mutex.  The symbol chosen here was identified from GCC's
    # libatomic map file.
    #
    # Arguably when using clang, compiler-rt might be better.
    nng_check_lib(atomic __atomic_load_1 NNG_HAVE_LIBATOMIC)

    nng_check_sym(AF_UNIX sys/socket.h NNG_HAVE_UNIX_SOCKETS)
    nng_check_sym(backtrace_symbols_fd execinfo.h NNG_HAVE_BACKTRACE)
    nng_check_struct_member(msghdr msg_control sys/socket.h NNG_HAVE_MSG_CONTROL)
    nng_check_sym(eventfd sys/eventfd.h NNG_HAVE_EVENTFD)
    nng_check_sym(kqueue sys/event.h NNG_HAVE_KQUEUE)
    nng_check_sym(port_create port.h NNG_HAVE_PORT_CREATE)
    nng_check_sym(epoll_create sys/epoll.h NNG_HAVE_EPOLL)
    nng_check_sym(epoll_create1 sys/epoll.h NNG_HAVE_EPOLL_CREATE1)
    nng_check_sym(poll poll.h NNG_HAVE_POLL)
    nng_check_sym(select sys/select.h NNG_HAVE_SELECT)
    nng_check_sym(getpeereid unistd.h NNG_HAVE_GETPEEREID)
    nng_check_sym(SO_PEERCRED sys/socket.h NNG_HAVE_SOPEERCRED)
    nng_check_struct_member(sockpeercred uid sys/socket.h NNG_HAVE_SOCKPEERCRED)
    nng_check_sym(LOCAL_PEERCRED sys/un.h NNG_HAVE_LOCALPEERCRED)
    nng_check_sym(LOCAL_PEERPID sys/un.h NNG_HAVE_LOCALPEERPID)
    nng_check_sym(getpeerucred ucred.h NNG_HAVE_GETPEERUCRED)
    nng_check_sym(atomic_flag_test_and_set stdatomic.h NNG_HAVE_STDATOMIC)
    nng_check_sym(socketpair sys/socket.h NNG_HAVE_SOCKETPAIR)
    nng_check_sym(AF_INET6 netinet/in.h NNG_HAVE_INET6)
    nng_check_sym(AF_INET6 netinet6/in6.h NNG_HAVE_INET6_BSD)
    nng_check_sym(timespec_get time.h NNG_HAVE_TIMESPEC_GET)
    nng_check_sym(getentropy sys/random.h NNG_HAVE_SYS_RANDOM)

    nng_sources(
            posix_impl.h
            posix_aio.h
            posix_ipc.h
            posix_config.h
            posix_pollq.h
            posix_tcp.h

            posix_alloc.c
            posix_atomic.c
            posix_clock.c
            posix_debug.c
            posix_file.c
            posix_ipcconn.c
            posix_ipcdial.c
            posix_ipclisten.c
            posix_peerid.c
            posix_pipe.c
            posix_resolv_gai.c
            posix_sockaddr.c
            posix_socketpair.c
            posix_sockfd.c
            posix_tcpconn.c
            posix_tcpdial.c
            posix_tcplisten.c
            posix_thread.c
            posix_udp.c
    )

    set(NNG_POLLQ_POLLER "auto" CACHE STRING "Poller used for multiplexing I/O")
    set_property(CACHE NNG_POLLQ_POLLER PROPERTY STRINGS auto ports kqueue epoll poll select)
    mark_as_advanced(NNG_POLLQ_POLLER)
    if (NNG_POLLQ_POLLER STREQUAL "ports")
        set(NNG_POLLQ_PORTS ON)
    elseif (NNG_POLLQ_POLLER STREQUAL "kqueue")
        set(NNG_POLLQ_KQUEUE ON)
    elseif (NNG_POLLQ_POLLER STREQUAL "epoll")
        set(NNG_POLLQ_EPOLL ON)
    elseif (NNG_POLLQ_POLLER STREQUAL "poll")
        set(NNG_POLLQ_POLL ON)
    elseif (NNG_POLLQ_POLLER STREQUAL "select")
        set(NNG_POLLQ_SELECT ON)
    elseif (NNG_HAVE_PORT_CREATE)
        set(NNG_POLLQ_PORTS ON)
    elseif (NNG_HAVE_KQUEUE)
        set(NNG_POLLQ_KQUEUE ON)
    elseif (NNG_HAVE_EPOLL AND NNG_HAVE_EVENTFD)
        set(NNG_POLLQ_EPOLL ON)
    elseif (NNG_HAVE_POLL)
        set(NNG_POLLQ_POLL ON)
    elseif (NNG_HAVE_SELECT)
        set(NNG_POLLQ_SELECT ON)
    endif()

    if (NNG_POLLQ_PORTS)
        message(STATUS "Using port events for multiplexing I/O.")
        nng_defines(NNG_POLLQ_PORTS)
        nng_sources(posix_pollq_port.c)
    elseif (NNG_POLLQ_KQUEUE)
        message(STATUS "Using kqueue for multiplexing I/O.")
        nng_defines(NNG_POLLQ_KQUEUE)
        nng_sources(posix_pollq_kqueue.c)
    elseif (NNG_POLLQ_EPOLL)
        message(DEBUG "Using epoll for multiplexing I/O.")
        nng_defines(NNG_POLLQ_EPOLL)
        nng_sources(posix_pollq_epoll.c)
    elseif (NNG_POLLQ_POLL)
        message(STATUS "Using poll for multiplexing I/O.")
        nng_defines(NNG_POLLQ_POLL)
        nng_sources(posix_pollq_poll.c)
    elseif (NNG_POLLQ_SELECT)
        message(STATUS "Using select for multiplexing I/O.")
        nng_defines(NNG_POLLQ_SELECT)
        nng_sources(posix_pollq_select.c)
    else()
        message(FATAL_ERROR "No suitable poller found for multiplexing I/O.")
    endif ()

    if (NNG_HAVE_ARC4RANDOM)
        nng_sources(posix_rand_arc4random.c)
    elseif (NNG_HAVE_GETENTROPY)
        nng_sources(posix_rand_getentropy.c)
    elseif (NNG_HAVE_GETRANDOM)
        nng_sources(posix_rand_getrandom.c)
    else ()
        nng_sources(posix_rand_urandom.c)
    endif ()

    nng_test(posix_ipcwinsec_test)

endif ()
