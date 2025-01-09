Based on https://jornvernee.github.io/hsdis/2022/04/30/hsdis.html by Jorn Vernee
Adapted to build on AArch64 with the latest on 1/9/2025 versions of the stuff
by Pavel Samolysov

to build, please run CMake from the `build` directory next to the file
with the following parameters:
$ JAVA_HOME=~/dev/openjdk/aarch64/jdk-25 cmake -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=install -DCMAKE_TOOLCHAIN_FILE=cross-clang-aarch64.cmake -GNinja ..
$ cmake --build . --target install

to copy the library into a pre-built JDK, do the following:
$ cp ~/dev/jdk-dev/hsdis/build/install/lib/hsdis-aarch64.so ~/dev/openjdk/aarch64/jdk-25/lib/server/
where "~/dev/openjdk/aarch64/jdk-25" is JAVA_HOME what was used to build the library.

