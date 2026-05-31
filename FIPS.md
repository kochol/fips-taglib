# fips-taglib

A [fips](https://github.com/floooh/fips)-ified fork of [TagLib](https://github.com/taglib/taglib)
(audio metadata library). Upstream sources are unchanged; only a fips build
wrapper is added so the library can be imported into a fips project as the
static lib target `tag`.

## What was added

- `fips.yml` — exports the `tag` module and the public header dirs.
- `CMakeLists.txt` — fips wrapper (replaces upstream's standalone build). It
  enables all format modules, runs `ConfigureChecks.cmake` to generate
  `config.h`, generates `taglib_config.h`, compiles every source under
  `taglib/` into the `tag` library, and puts all taglib sub-folders plus the
  bundled `utf8-cpp` on the public include path. Built statically
  (`TAGLIB_STATIC`). zlib is left off (optional, only used for compressed
  ID3v2 frames) and can be wired in later via `fips-zlib`.
- `fips`, `fips.cmd` — fips bootstrap wrappers.

The bundled `3rdparty/utfcpp` git submodule is required; clone with
`--recurse-submodules`.

## Use from a fips project

```yaml
# fips.yml
imports:
  fips-taglib:
    git: https://github.com/kochol/fips-taglib
```

```cmake
# CMakeLists.txt (no_auto_import projects)
fips_import_fips_taglib_tag()
...
fips_begin_app(myapp windowed)
    fips_deps(tag)
fips_end_app()
```

```cpp
#include <fileref.h>
#include <tag.h>

TagLib::FileRef f("song.mp3");
auto* t = f.tag();   // t->title(), t->artist(), ...
```

## Build standalone (smoke test)

```
./fips set config win64-vstudio-release
./fips gen
./fips build
```
