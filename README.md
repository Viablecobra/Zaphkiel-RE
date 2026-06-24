# Zaphkiel
[![License: Proprietary](https://img.shields.io/badge/License-Proprietary-red.svg)](LICENSE.txt)

A reverse engineering framework for Android via LeviLauncher. Injected as `libZaphkiel.so`, it performs static analysis on target binaries without requiring root.

## Features

- Static disassembly with inline string annotation and return-type guessing
- Auto signature generation (IDA / code / mask )
- VTable resolution and hooking via C++ RTTI
- Class hierarchy dumping
- String cross-reference with function decompilation
- Recursive caller/callee xref trees
- Struct offset scanning across the binary
- Full batch decompilation
- Header file parsing for field/method annotation
- Live memory string and symbol dumping

## Files

Place these in `/storage/emulated/0/Android/media/org.levimc.launcher/kurumi/`:

| File | Purpose |
|------|---------|
| `zaphkiel_config.txt` | Optional configuration |
| `zInput.txt` | Analysis target list (one per line) |
| `sig_targets.txt` | Signature generator targets |
| `headers/` | `.h` / `.hpp` files for annotation |

## Configuration

`zaphkiel_config.txt` (all keys optional — hardcoded defaults exist):

```
max_fn_insns = 120
max_callers = 6
max_callees = 6
callee_depth = 3
caller_depth = 2
annotate_inline_strings = 1
show_sub_call_addrs = 1
startup_delay_secs = 3
strings_path = /storage/emulated/0/Android/media/org.levimc.launcher/kurumi/zInput.txt
out_dir = /storage/emulated/0/Android/media/org.levimc.launcher/kurumi/
out_file = zaphkiel_out.txt
dump_rtti = true
dump_dynsym = true
dump_vtables = false
separate_dump_files = true
full_decompile = false
full_decompile_max_fns = 10000
full_decompile_max_insns = 60
get_strings = false
get_strings_min_len = 4
get_strings_demangle = true
get_strings_deobfuscate = true
headers_dir = /storage/emulated/0/Android/media/org.levimc.launcher/kurumi/headers/
```

## Target Syntax (`zInput.txt`)

```
fn<0x1234567>                    # Disassemble function at VA
add<0x1234567>                   # Resolve data address to containing function
scan<0x1234567>                  # Raw scan + caller analysis
range<0xSTART,0xEND>             # Disassemble address range
vft<19ClassName><0>              # Decompile vtable slot 0
vft<19ClassName><5><3>           # Slot 5, hint 3 params
Player::tick                     # String xref + decompile
strfn<"Player::tick">            # String xref + trace BL receiver
sig<AA ?? BB ?? CC>              # Byte pattern scan
struct<Level><0x180>             # Find all accesses to offset +0x180
struct<Level><0x180><1>          # Same, base register X1 only
xref_tree<0x1234567>             # Recursive caller/callee tree
fullf<0x1234567>                 # Decompile with no insn limit
fullvf<19ClassName>              # Decompile all vtable slots
fullvf<19ClassName><0><10>       # Decompile slots 0-9
hier<ClassName>                  # Dump C++ inheritance tree
h<ScreenView>                    # Parse header + dump struct layout
```

Class names accept raw RTTI (`19HudScreenController`) or unmangled (`HudScreenController`).

## Signature Generator

`sig_targets.txt` format:
```
fn<0x1234567>
add<0xabcdef0>
```

Output: `signatures.txt` (human-readable) and `signatures.json` (machine-parseable).

Signatures auto-wildcard PC-relative instructions (ADRP, ADR, BL, B, B.cond, CBZ/CBNZ, TBZ/TBNZ, LDR literal) for cross-build compatibility.

## Output Files

| File | Description |
|------|-------------|
| `zaphkiel_out.txt` | Main analysis output |
| `zaphkiel_log.txt` | Runtime debug log |
| `zaphkiel_fullout.txt` | Full decompile output |
| `str.txt` | Extracted strings |
| `rtti_dump.txt` | RTTI class names (currently broken) |
| `dynsym_dump.txt` | Dynamic symbols (currently broken) |
| `vtable_dump.txt` | Vtable pointers (currently broken) |
| `live_strings.txt` | Live memory strings |
| `signatures.txt` / `.json` | Generated signatures |
| `{Header}_parse.txt` | Parsed header layouts |

## Header Annotation

Place `.h` / `.hpp` files in `headers_dir/`. When analyzing `struct<Class><offset>` or `vft<Class><slot>`, matching field names and method signatures are displayed inline.

## Legal Notice

**Zaphkiel is an independent educational tool. It is not affiliated with, endorsed by, or sponsored by Mojang Studios, Microsoft Corporation, or any of their subsidiaries or affiliates.**

Zaphkiel is provided strictly for educational and research purposes. The authors and distributors of Zaphkiel assume no liability for any use or misuse of this software. By using Zaphkiel, you agree that:

- You are solely responsible for complying with all applicable laws, terms of service, and end-user license agreements in your jurisdiction.
- The authors bear no responsibility for any damages, bans, legal action, or other consequences resulting from the use of this software.
- All trademarks, game assets, and intellectual property referenced by user-generated targets (class names, strings, signatures) remain the property of their respective owners.
- This software is provided "as is" without warranty of any kind, express or implied.
