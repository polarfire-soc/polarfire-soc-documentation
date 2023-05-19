# Hart Software Services Function Profiling

- [Function Profiling](#function-profiling)
- [Viewing Counters from TinyCLI](#viewing-counters-from-tinycli)
- [Enabling Profiling](#enabling-profiling)
- [Excluding Functions from Profiling](#excluding-functions-from-profiling)

This document describes:

- What Function Profiling in the HSS is all about.
- How to view function profiling counters, and how to convert those counters back to meaningful information with function names.
- Enabling profiling through Kconfig, and excluding certain functions from the profiling capture.

<a name="#function-profiling"></a>

## Function Profiling

Function profiling in the HSS can be enabled using the `CONFIG_DEBUG_PROFILING_SUPPORT` Kconfig option.

Function profiling allows capturing of the time spent in each C function (through the use of `__cyg_profile_func_enter` and `__cyg_profile_func_exit`. This information can be logged to the serial console through calling the `HSS_Profile_DumpAll()` function at an appropriate time, depending on what is being debugged.

<a name="#viewing-counters-from-tinycli"></a>

## Viewing Counters from TinyCLI

If the HSS's CLI is enabled, then the command `debug profile` will dump the captured profiling information to the HSS console.  This information contains function addresses, and a tick count. The function addresses can be parsed back to function symbol names by copying and pasting the information into a CSV file, and processing it with `tools/profiling/gen-prof-report.py`.  For example, consider the following:

```csv
    # log.csv
    a00c408, 792590076
    a006fd8, 222793198
    a006f8c, 66192
    a0205fa, 5211
    a02058e, 5241
    a00c31a, 5183557
    a00a24e, 252666457
    a002a8a, 8941
    a00a12e, 30944151
    a0061b6, 5452
    a00638e, 72715
    a00620e, 2770
    a00630c, 8906
    a00623a, 5676
    a006150, 55560
    a006124, 11442
```

Processing this as follows:

```shell-session
    $ tools/profiling/gen-prof-report.py Default/hss-l2scratch.elf log.csv

```

would generate...

```csv
    uart_getchar, 792590076
    HSS_MMCInit, 252666457
    MSS_UART_get_rx, 222793198
    mmc_init_sdcard, 30944151
    HSS_BoardLateInit, 5183557
    switch_mssio_config, 72715
    MSS_UART_get_rx_status, 66192
    switch_demux_using_fabric_ip, 55560
    fabric_sd_emmc_demux_present, 11442
    mmc_reset_block, 8941
    io_mux_and_bank_config, 8906
    set_bank2_and_bank4_volts, 5676
    mss_does_xml_ver_support_switch, 5452
    HSS_DDRHi_GetStart, 5241
    HSS_DDRHi_GetSize, 5211
    mss_is_alternate_io_setting_sd, 2770
```

**NOTE**: `gen-prof-report.py` requires the pyelftools Python library. `pyelftools` is a pure-Python library for parsing and analyzing ELF files and DWARF debugging information. It can typically be installed using `pip install pyelftools`.

<a name="#enabling-profiling"></a>

## Enabling Profiling

To enable profiling, ensure that the following options are set in your `.config`:

```Kconfig
    CONFIG_DEBUG_PROFILING_SUPPORT=y
    CONFIG_DEBUG_PROFILING_MAX_NUM_FUNCTIONS=128
```

Profiling is captured into an array, on a first-come-first-served basis. The array size is configured with `CONFIG_DEBUG_PROFILING_MAX_NUM_FUNCTIONS`. Depending on how many files are compiled for profiling, this array can fill very quickly. When full, new entries are ignored silently.

By default, this array (`modules/debug/profiling.c::profileStats`) is placed in L2 Scratchpad.  This could be placed into an arbitrary section using the section attribute, and this section placed appropriately in larger memory.

```C
    /* modules/debug/profiling.c */
    struct ProfileNode profileStats[CONFIG_DEBUG_PROFILING_MAX_NUM_FUNCTIONS] = { 0 } __attribute__((section(".profilingStats");
```

<a name="#excluding-functions-from-profiling"></a>

## Excluding Functions from Profiling

Function profiling is implemented using GCC functionality that allows you to specify which functions should be instrumented for profiling and which ones should be excluded.

GCC provides the `-fprofile-arcs` and `-ftest-coverage` options to enable function profiling and code coverage analysis. To exclude specific functions from profiling, you can use the `-fno-profile-arcs` or `-fno-test-coverage` options in combination with function attributes.

When `CONFIG_DEBUG_PROFILING_SUPPORT` is enabled, almost all functions are eligible for profiling, unless explicitly excluded.

Functions can be excluded from profiling by being marked with ``__attribute__((no_instrument_function))``. They can also be added to the `-finstrument-functions-exclude-function-list` in `modules/debug/Makefile`. Finally, as a more coarse-grained alternative, a list of files to exclude can be added to the `-finstrument-functions-exclude-file-list` in `modules/debug/Makefile`.
