# CHERI ELF gABI Extensions

This document is based on the System V Application Binary Interface, otherwise
known as the ELF gABI specification.

## Table of Contents
1. [Note Section](#note-section)
2. [Program Header Table](#program-headers)
3. [Dynamic Table](#dynamic-table)

# <a name=note-section></a> Note Section

CHERI notes are located in an SHT_NOTE section with name `.note.cheri`. All
generic CHERI notes have a Name Size of 6 and a Name of `"CHERI\0"`. The
following notes are defined:

Type                    | Name
------------------------|----------------------------------------------
0x0                     | [NT_CHERI_GLOBALS_ABI](#nt-cheri-globals-abi)
0x1                     | [NT_CHERI_TLS_ABI](#nt-cheri-tls-abi)
0x80000000 - 0xffffffff | -- (Reserved for processor-specific use)

## <a name=nt-cheri-globals-abi></a> NT_CHERI_GLOBALS_ABI

This note describes the ABI variant in use for accessing globals. It has a Desc
Size of 4, and Desc should be interpreted as a 4-byte integer, with the
following variants defined:

Type                    | Name
------------------------|----------------------------------------------
0x0                     | CHERI_GLOBALS_ABI_PCREL
0x1                     | CHERI_GLOBALS_ABI_PLT_FPTR
0x2                     | CHERI_GLOBALS_ABI_FDESC
0x80000000 - 0xffffffff | -- (Reserved for processor-specific use)

* CHERI_GLOBALS_ABI_PCREL: Capabilities for globals are obtained by indexing a
  table relative to the program counter. Function pointers are sealed entry
  capabilities to code.

* CHERI_GLOBALS_ABI_PLT_FPTR: Capabilities for globals are obtained by indexing
  a table pointed to by a reserved register or equivalent that is defined on
  entry to functions. Function pointers are sealed entry capabilities to
  trampolines that install the correct value for the target. Whether the old
  value is saved by the caller, trampoline or callee, and how, is
  processor-specific, and more than one variant may exist differentiated in the
  ELF file in a processor-specific manner.

* CHERI_GLOBALS_ABI_FDESC: Capabilities for globals are obtained by indexing a
  table pointed to by a reserved register or equivalent that is defined on
  entry to functions. Function pointers are capabilities to function
  descriptors, the format of which is processor-specific.

The set of variants supported is processor-specific.

## <a name=nt-cheri-tls-abi></a> NT_CHERI_TLS_ABI

This note describes the ABI variant in use for accessing thread-locals. It has
a Desc Size of 4, and Desc should be interpreted as a 4-byte integer, with the
following variants defined:

Type                    | Name
------------------------|----------------------------------------------
0x0                     | CHERI_TLS_ABI_TRAD
0x1                     | CHERI_TLS_ABI_TGOT
0x80000000 - 0xffffffff | -- (Reserved for processor-specific use)

* CHERI_TLS_ABI_TRAD: Capabilities for thread-locals are obtained using
  traditional TLS Variant I or II with pointers implemented using capabilities.

* CHERI_TLS_ABI_TGOT: Capabilities for thread-locals are obtained using a
  Thread Global Offset Table ("TGOT") wih pointers implemented using
  capabilities.

The set of variants supported is processor-specific.

# <a name=program-headers></a> Program Header Table

CHERI adds the following segment types:

Name                      | Value      | Description
--------------------------|------------|-------------------------------------
PT_CHERI_TGOT             | 0x64348451 | TGOT template

# <a name=dynamic-table></a> Dynamic Table

CHERI adds the following dynamic table entries:

Enum       | ELF Dynamic Tab              | Description
-----------|------------------------------|-------------------------------------
0x64348450 | DT_CHERI_TGOTREL             | Start of the TGOT template relocations
0x64348451 | DT_CHERI_TGOTRELT            | Type of relocation used for the TGOT template
0x64348453 | DT_CHERI_TGOTRELSZ           | Total size in bytes of the TGOT template relocations
