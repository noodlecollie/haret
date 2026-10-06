## Relocations in `.text.preload` and `.text.irq` Sections

HaRET contains two sections of code that are not part of the program's executable context directly:

* `.text.preload` contains code that is copied to RAM specifically for the purpose of launching the Linux kernel.
* `.text.irq` contains code to intercept and log exceptions.

Since the code in these sections is designed to exist pretty much in isolation, there are two things it cannot do:

1. Call into other areas of HaRET code.
2. Embed absolute memory addresses to other parts of the same section. Since the code in these sections is copied
   around in RAM, addresses computed at link time will not be correct at runtime.

As a sanity check at compile time, the `tools/checkrelocs` script checks for any problematic "relocations" in the
`.text.preload` and `.text.irq` sections. A "relocation" is basically a code placeholder for an address that is not
known when an object file is generated, but that is filled in at link time instead. Relocations that point outside of
the section are prohibited, as are relocations which require an absolute memory address.

Presumably there were no relocation issues when this repo was originally created. However, I suspect that the more
recent `cegcc` toolchain is now able to do more sophisticated optimisations, and when I attempted to compile the code
in the repo using the existing `Makefile`, the script (mercifully!) flagged some relocations.

For context, relocations of the type `ARM_32` in `objdump` output are absolute 32-bit addresses. To see the disassembly
for a particular section, run the following command. Note that this requires `tmp_link.o` on disk, which is the
temporary object file that the `tools/checkrelocs` script uses to analyse relocations.

```bash
/opt/cegcc/bin/arm-mingw32ce-objdump -C -d -S -j <section_name> tmp_link.o > disassembly.txt
```

Searching for the offset of the relocation in the disassembly should show which function the relocation exists within.

### `.text.preload`

The relocations reported in `.text.preaload` were to do with "jump tables". In this context, the jump table was a list
of addresses embedded into the code, which are loaded directly into the program counter when eg. running a `switch`
statement. These addresses were absolute, and given that absolute addresses cannot be fully computed until link time,
they were flagged as relocations. The solution to this was to disable jump table generation for the specific functions
in `.text.preload` where this was an issue.

### `.text.irq`

The relocations reported in `.text.irq` were slightly different. These relocations were actually fine, because they were
generated from taking the addresses of callback functions that the `.text.irq` code did not actually use. These
addresses were simply stored in exception trace logs, so that the main HaRET code could receive these logs and call the
relevant functions. However, the relocations still showed up in `objdump` as they were technically references to code
outside of `.text.irq`.

The solution to this was to put the relevant callbacks into a specific section, `.text.trace_callbacks`. There is
nothing special about this section, it just has a unique name that can be used to reliably filter the relocations out of
the `objdump` output.
