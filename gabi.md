# CHERI ELF gABI Extensions

This document is based on the System V Application Binary Interface, otherwise
known as the ELF gABI specification.

## Table of Contents
1. [Note Section](#note-section)
2. [C18n String Table](#c18n-strtab)
3. [C18n Ranges Table](#c18n-rangetab)
4. [Section Header Table](#section-headers)
5. [Program Header Table](#program-headers)
6. [Dynamic Table](#dynamic-table)

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

# <a name=c18n-strtab></a> C18n String Table

CHERI compartmentalization adds a new memory-resident string table in
a section named `.c18nstrtab` containing strings such as sub-object
compartment names.

# <a name=c18n-rangetab></a> C18n Ranges Table

CHERI compartmentalization adds a new table containing an array of
structures describing virtal address ranges associated with sub-object
compartments.  A single compartment may be described by multiple
address ranges.  This table is stored in a new section named
`.c18nrangetab`.  Each address range is described by the following
structure:

```
typedef struct {
	Elf_Addr	r_addr;
	Elf_Size	r_size;
	Elf_Word	r_name;
	Elf_Word	r_reserved;
} Elf_Range;
```

The `r_addr` and `r_size` members describe a range of virtual
addresses similar to the `p_vaddr` and `p_memsz` fields of a program
header.  The `r_name` member holds the offset of a compartment name
from the c18n string table similar to the `st_name` field of symbol
table entries.  The `r_reserved` field should be cleared to zero.

Note that this section is only specified for ELF files containing
program headers, and each address range should be a subset of the
addresses described by `PT_LOAD` segments.

# <a name=section-headers></a> Section Header Table

CHERI adds the following section types:

Name                      | Value      | Description
--------------------------|------------|-------------------------------------
SHT_C18N_RANGE            | 0x64331380 | c18n address ranges table

* SHT_C18N_RANGE should be marked as SHF_ALLOC and its `sh_link` field
  should reference the corresponding c18n string table section.

# <a name=program-headers></a> Program Header Table

CHERI adds the following segment types:

Name                      | Value      | Description
--------------------------|------------|-------------------------------------
PT_C18N_NAME              | 0x64331380 | Sub-object compartment
PT_CHERI_PCC              | 0x64348450 | PCC bounds
PT_CHERI_TGOT             | 0x64348451 | TGOT template

* PT_C18N_NAME describes a range of virtual addresses belonging to a
  named compartment.  Most fields are similar to PT_GNU_RELRO except
  for `p_paddr` which holds the offset of the nul-terminated ASCII
  compartment name in the c18n string table.

  **Note:** This segment is deprecated and will be replaced by
  DT_C18N_RANGETAB.

* PT_CHERI_PCC is similar to PT_GNU_RELRO in that it describes a property
  of an existing virtual address range.  Specifically, capabilities for
  code pointers should use the bounds of the PT_CHERI_PCC segment which
  contains the pointer's target address.  The static linker must ensure
  that the bounds are precise.  If necessary, the static linker must
  insert suitable padding.

# <a name=dynamic-table></a> Dynamic Table

CHERI adds the following dynamic table entries:

Enum       | ELF Dynamic Tab              | Description
-----------|------------------------------|-------------------------------------
0x64331380 | DT_C18N_STRTAB               | Start of the c18n string table
0x64331381 | DT_C18N_STRTABSZ             | Total size in bytes of the c18n string table
0x64331382 | DT_C18N_RANGETAB             | Start of the c18n address ranges table
0x64331383 | DT_C18N_RANGETABSZ           | Total size in bytes of the c18n address ranges table
0x64348450 | DT_CHERI_TGOTREL             | Start of the TGOT template relocations
0x64348451 | DT_CHERI_TGOTRELT            | Type of relocation used for the TGOT template
0x64348453 | DT_CHERI_TGOTRELSZ           | Total size in bytes of the TGOT template relocations
