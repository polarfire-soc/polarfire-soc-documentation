# HSS and U-Boot Devicetree Interactions

## The HSS Options/Features

The HSS is capable of providing a devicetree to U-Boot (or any other subsequent stage) via
"ancillary-data" in a payload created by the payload generator. This devicetree will be
appended to U-Boot in memory and the address of this devicetree will be placed in the
`a1` register.

The HSS is also capable of providing a devicetree itself, when CONFIG_PROVIDE_DTB is enabled.
The HSS contains cut-down devicetrees for this purpose, but a user can provide their own dtb for
use. See CONFIG_DEFAULT_DEVICETREE for more information. Due to space constraints, it is advised
that any devicetree used with CONFIG_PROVIDE_DTB is reduced to the minimum possible nodes that
U-Boot would need to boot the OS.

A devicetree provided by ancillary-data in a payload will take priority over one provided by the
HSS using CONFIG_PROVIDE_DTB.

## The U-Boot Options/Features

U-Boot has three main options of interest: CONFIG_OF_BOARD, CONFIG_OF_SEPARATE and
CONFIG_MULTI_DTB_FIT.

CONFIG_OF_SEPARATE expects that the devicetree used by U-Boot will be located immediately after
U-Boot in memory. CONFIG_OF_BOARD allows for firmware to provide the address of the devicetree in
memory using the `a1` register. Both can be set at the same time. It is expected that
CONFIG_OF_SEPARATE will be set in all non-debug configurations of U-Boot as the alternative to
CONFIG_OF_SEPARATE, CONFIG_OF_EMBEDDED, is only intended for debug.

CONFIG_MULTI_DTB_FIT allows U-Boot to be built with multiple devicetree blobs built into it, and
the board file implements a function that selects which one U-Boot should use. In our case, the
function does this by matching the name of the configuration in the fit image (the filename
of the devicetree blob) with the compatible of the blob U-Boot was booted with.

CONFIG_MULTI_DTB_FIT will boot U-Boot with a complete devicetree rather than the stub that the HSS
carries with the bare minimum of nodes enabled that CONFIG_OF_BOARD will use.

Both CONFIG_OF_BOARD and CONFIG_MULTI_DTB_FIT can be used to build a U-Boot binary that can be
used on multiple different boards. CONFIG_OF_BOARD has an advantage in that it doesn't require
that U-Boot is built with devicetrees for the platforms it will be used on, as U-Boot will
run using the devicetree passed from the HSS and use that to determine which next stage payload
to boot. For example, we use the same U-Boot for Engineering Sample and Production Icicle kits,
and use the compatible from the HSS's minimal devicetree to determine which of the configurations
in the fit image containing the kernel to use. On the other hand, CONFIG_MULTI_DTB_FIT is more
suitable when U-Boot needs a more complete devicetree than the HSS can provide.

## Interactions

### CONFIG_OF_BOARD disabled & CONFIG_MULTI_DTB_FIT disabled

- If the HSS's payload contains u-boot.bin then the devicetree built with U-Boot will be used by
  U-boot, no matter whether or not CONFIG_PROVIDE_DTB is set in the HSS and ancillary data will be
  ignored.
- If the HSS's payload contains u-boot-nodtb.bin, which as the name suggests, contains no
  devicetree, a devicetree must be provided using ancillary-data in the payload. The HSS will load
  this devicetree immediately after U-Boot in memory, where U-Boot expects to find it.

- If the hss's payload contains u-boot-nodtb.bin, but there is no ancillary-data in the payload,
  U-Boot will hang on startup. Any devicetree provided using CONFIG_PROVIDE_DTB will be ignored by
  U-Boot as it will have been linked into the HSS and will not be placed after U-Boot in memory.

Using u-boot-nodtb.bin is not advised in any circumstance, the same ancillary-data behaviour can be
obtained with CONFIG_OF_BOARD enabled.

### CONFIG_OF_BOARD enabled & CONFIG_MULTI_DTB_FIT disabled

If the HSS's payload contains u-boot.bin then the prioritisation is:

1. the devicetree provided by ancillary-data
2. the devicetree provided by HSS option CONFIG_PROVIDE_DTB, if it is enabled
3. the devicetree built with U-Boot will be used

The HSS currently puts a dtb in its payload's ancillary-data immediately after U-Boot in memory,
setting CONFIG_OF_BOARD will produce identical behaviour in this case, but explicitly setting the
config option is more robust.

### CONFIG_MULTI_DTB_FIT enabled

CONFIG_MULTI_DTB_FIT allows U-Boot to choose between one of several devicetrees, specified by
CONFIG_OF_LIST, at runtime. On Icicle kits, this is done by looking at the devicetree provided
by the HSS and comparing the compatible with the devicetrees U-Boot has access to.

CONFIG_MULTI_DTB_FIT is essentially opting to use the minimal HSS provided devicetree is nothing
more than a tool for U-Boot to select from a list of complete devicetrees, so CONFIG_OF_BOARD is
ignored.

U-Boot will take the compatibles from HSS provided devicetree, strip the vendor prefix from them
and use the first configuration in the fit image where the configuration name matches one of the
compatibles. If none match, the first configuration in the fit image is used.

If no HSS provided devicetree exists, the first configuration in the fit image is used.

The U-Boot option DEFAULT_DEVICE_TREE is ignored when CONFIG_MULTI_DTB_FIT is enabled.

If the HSS provides a devicetree, one in ancillary-data is used before one from the HSS option
CONFIG_PROVIDE_DTB.
