metapost.js
===========

Metapost running directly in a browser, thanks to Emscripten and asm.js

How to compile mpost with Emscripten
------------------------------------

This is the raw shell notes from @Troush; some of these instructions may not be accurate, so beware - and send pull requests with fixes :)

```sh
# MANUAL: Follow the Emscripten installation guide on a Linux machine:
# https://github.com/kripken/emscripten/wiki/Getting-Started-on-Ubuntu-12.10

# MANUAL: Patch emcc in the Emscripted folder root:
#
# -        os.path.join('libc', 'stdlib', 'getopt_long.c'),
# +#        os.path.join('libc', 'stdlib', 'getopt_long.c'),
#
# See https://github.com/manuels/texlive.js/issues/14

# MANUAL: Delete ~/.emscripten_cache

# AUTO: Create a workspace folder.
mkdir metapost-workspace
cd metapost-workspace

# AUTO: Download Metapost source code.
wget "https://foundry.supelec.fr/frs/download.php/file/15758/metapost-1.890.tar.bz2"

# AUTO: Create separate branches for the native and the JS build.
tar -xf metapost-1.890.tar.bz2
cp -r metapost-1.890 native
cp -r metapost-1.890 js

# AUTO: Run the full native build.
cd native
./build.sh
cd ..

# AUTO: Now start working on the JS build:
cd js

# MANUAL: Perform some patch on the JS branch
#
# source/libs/libpng/configure:
# -ac_fn_c_check_func "$LINENO" "zlibVersion" "ac_cv_func_zlibVersion"
# -if test "x$ac_cv_func_zlibVersion" = xyes; then :
# -
# -else
# -  as_fn_error $? "zlib not found" "$LINENO" 5
# -fi
# +#ac_fn_c_check_func "$LINENO" "zlibVersion" "ac_cv_func_zlibVersion"
# +#if test "x$ac_cv_func_zlibVersion" = xyes; then :
# +#
# +#else
# +#  as_fn_error $? "zlib not found" "$LINENO" 5
# +#fi
#
# source/libs/cairo/configure:
# -ac_fn_c_check_func "$LINENO" "pixman_version_string"
"ac_cv_func_pixman_version_string"
# -if test "x$ac_cv_func_pixman_version_string" = xyes; then :
# -
# -else
# -  as_fn_error $? "pixman not found" "$LINENO" 5
# -fi
# +#ac_fn_c_check_func "$LINENO" "pixman_version_string"
"ac_cv_func_pixman_version_string"
# +#if test "x$ac_cv_func_pixman_version_string" = xyes; then :
# +#
# +#else
# +#  as_fn_error $? "pixman not found" "$LINENO" 5
# +#fi
#
# source/libs/cairo/cairo-1.12.8/configure:
# -#define HAVE___UINT128_T 1

# AUTO: Run the full JS build, while copying the native intermediate binaries.
mkdir out
cd out
emconfigure ../source/configure \
    --disable-all-pkgs \
    --disable-shared    \
    --disable-largefile \
    --disable-ptex \
    --enable-mp  \
    --enable-compiler-warnings=max \
    --without-system-harfbuzz \
    --without-system-cairo \
    --without-system-libpng \
    --without-ptexenc \
    --without-system-ptexenc \
    --without-system-kpathsea \
    --without-system-xpdf \
    --without-system-freetype \
    --without-system-freetype2 \
    --without-system-gd \
    --without-system-teckit \
    --without-system-t1lib \
    --without-system-icu \
    --without-system-graphite \
    --without-system-zziplib \
    --without-mf-x-toolkit --without-x
emmake make

cd libs/zlib
emmake make
cd ../..

cd libs/libpng
emmake make
cd ../..

cd texk/web2c
emmake make mpost  # This fails, as expected.

cp ../../../../native/build/texk/web2c/ctangleboot .
chmod +x ctangleboot
emmake make mpost  # This gets further but still fails, as expected.

cp ../../../../native/build/texk/web2c/ctangle .
chmod +x ctangle
emmake make mpost  # This finally succeeds

# AUTO: Compile the final output to JS.
cp mpost mpost.bc
emcc mpost.bc -o mpost.js

# TODO: Setup the file system. See:
# https://github.com/kripken/emscripten/wiki/Filesystem-Guide
# https://github.com/manuels/texlive.js/blob/master/pre.js

# AUTO: Try out mpost.
node mpost.js
```
