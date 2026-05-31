# Laboratory work 6

### Создание CMakeLists.txt

Добавляем переменные версии в корневой `CMakeLists.txt`:

```
cmake_minimum_required(VERSION 3.11)
project(solver_pkg)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(formatter_lib)
add_subdirectory(formatter_ex_lib)
add_subdirectory(solver_lib)
add_subdirectory(solver_application)

set(SOLVER_VERSION "v1.0.0")

include(CPackConfig.cmake)
```

### Добавление install() в solver_application/CMakeLists.txt

```
project(solver_app)
add_executable(solver_app equation.cpp)
target_link_libraries(solver_app PRIVATE formatter_ex solver)
install(TARGETS solver_app DESTINATION bin)
```

### Создание файла CPackConfig.cmake

```
vim CPackConfig.cmake

set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_VERSION ${SOLVER_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "This is solver pack.")
set(CPACK_PACKAGE_VENDOR "dew-ls")
set(CPACK_PACKAGE_CONTACT "artemsuprankov6388@gmail.com")

include(CPack)
```

### Создание GitHub Actions workflow

Создаём файл `.github/workflows/release.yml`:

```
name: Release

on:
    push:
	tags:
        - 'v*'

jobs:

  build-deb:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure
        run: cmake -S . -B build -DCPACK_GENERATOR=DEB

      - name: Build
        run: cmake --build build

      - name: Pack
        run: cd build && cpack

      - uses: actions/upload-artifact@v4
        with:
          name: deb-pkg
          path: build/*.deb

  build-rpm:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install RPM dependencies
        run: sudo apt-get install -y rpm

      - name: Configure
        run: cmake -S . -B build -DCPACK_GENERATOR=RPM

      - name: Build
        run: cmake --build build

      - name: Pack
        run: cd build && cpack

      - uses: actions/upload-artifact@v4
        with:
          name: rpm-pkg
          path: build/*.rpm

  build-dmg:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure
        run: cmake -S . -B build -DCPACK_GENERATOR=DragNDrop

      - name: Build
        run: cmake --build build

      - name: Pack
        run: cd build && cpack

      - uses: actions/upload-artifact@v4
        with:
          name: dmg-pkg
          path: build/*.dmg

  build-msi:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure
        run: cmake -S . -B build -DCPACK_GENERATOR=WIX

      - name: Build
        run: cmake --build build --config Release

      - name: Pack
        run: cd build && cpack

      - uses: actions/upload-artifact@v4
        with:
          name: msi-pkg
          path: build/*.msi

  release:
    needs: [build-deb, build-rpm, build-dmg, build-msi]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/download-artifact@v4
        with:
          path: packages

      - uses: softprops/action-gh-release@v2
        with:
          files: packages/**/*
          generate_release_notes: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Публикация релиза

```
git add .
git commit -m "Packaging added."

[master 3f2a1c8] Packaging added.
 4 files changed, 89 insertions(+)
 create mode 100644 CPackConfig.cmake
 create mode 100644 .github/workflows/release.yml

git tag v1.0.0
git push origin master --tags

Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 8 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (10/10), 3.03 KiB | 1.51 MiB/s, done.
Total 8 (delta 2), reused 0 (delta 0), pack-reused (from 0)
To https://github.com/dew-ls/lab06.git
 * [new tag]         v1.0.0 -> v1.0.0
```
