# lab06
# lab06
## Laboratory work VI

Данная лабораторная работа посвещена изучению средств пакетирования на примере **CPack**

```sh
$ open https://cmake.org/Wiki/CMake:CPackPackageGenerators
```

## Tasks

- [ ] 1. Создать публичный репозиторий с названием **lab06** на сервисе **GitHub**
- [ ] 2. Выполнить инструкцию учебного материала
- [ ] 3. Ознакомиться со ссылками учебного материала
- [ ] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_USERNAME=<имя_пользователя>
$ export GITHUB_EMAIL=<адрес_почтового_ящика>
$ alias edit=<nano|vi|vim|subl>
$ alias gsed=sed # for *-nix system
```

```sh
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab05 projects/lab06
$ cd projects/lab06
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab06
```

```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION\
  \${PRINT_VERSION_MAJOR}.\${PRINT_VERSION_MINOR}.\${PRINT_VERSION_PATCH}.\${PRINT_VERSION_TWEAK})
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_TWEAK 0)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_PATCH 0)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MINOR 1)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MAJOR 0)
' CMakeLists.txt
$ git diff
```

```sh
$ touch DESCRIPTION && edit DESCRIPTION
$ touch ChangeLog.md
$ export DATE="`LANG=en_US date +'%a %b %d %Y'`"
$ cat > ChangeLog.md <<EOF
* ${DATE} ${GITHUB_USERNAME} <${GITHUB_EMAIL}> 0.1.0.0
- Initial RPM release
EOF
```

```sh
$ cat > CPackConfig.cmake <<EOF
include(InstallRequiredSystemLibraries)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF
set(CPACK_PACKAGE_CONTACT ${GITHUB_EMAIL})
set(CPACK_PACKAGE_VERSION_MAJOR \${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR \${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH \${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK \${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION \${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE \${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RESOURCE_FILE_LICENSE \${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RPM_PACKAGE_NAME "print-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print")
set(CPACK_RPM_CHANGELOG_FILE \${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_DEBIAN_PACKAGE_NAME "libprint-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

include(CPack)
EOF
```

```sh
$ cat >> CMakeLists.txt <<EOF

include(CPackConfig.cmake)
EOF
```

```sh
$ gsed -i 's/lab05/lab06/g' README.md
```

```sh
$ git add .
$ git commit -m"added cpack config"
$ git tag v0.1.0.0
$ git push origin master --tags
```

```sh
$ travis login --auto
$ travis enable
```

```sh
$ cmake -H. -B_build
$ cmake --build _build
$ cd _build
$ cpack -G "TGZ"
$ cd ..
```

```sh
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
$ cmake --build _build --target package
```

```sh
$ mkdir artifacts
$ mv _build/*.tar.gz artifacts
$ tree artifacts
```

## Report

```sh
$ popd
$ export LAB_NUMBER=06
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Homework

После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку [releases](https://github.com/tp-labs/lab06/releases)).</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1)
Таким образом, каждый новый релиз будет состоять из следующих компонентов:
- архивы с файлами исходного кода (`.tar.gz`, `.zip`)
- пакеты с бинарным файлом _solver_ (`.deb`, `.rpm`, `.msi`, `.dmg`)

В качестве подсказки:
```sh
$ cat .travis.yml
os: osx
script:
...
- cpack -G DragNDrop # dmg

$ cat .travis.yml
os: linux
script:
...
- cpack -G DEB # deb

$ cat .travis.yml
os: linux
addons:
  apt:
    packages:
    - rpm
script:
...
- cpack -G RPM # rpm

$ cat appveyor.yml
platform:
- x86
- x64
build_script:
...
- cpack -G WIX # msi
```

Для этого нужно добавить ветвление в конфигурационные файлы для **CI** со следующей логикой:</br>
если **commit** помечен тэгом, то необходимо собрать пакеты (`DEB, RPM, WIX, DragNDrop, ...`) </br>
и разместить их на сервисе **GitHub**. (см. пример для [Travi CI](https://docs.travis-ci.com/user/deployment/releases))</br>

```sh
$ git clone https://github.com/Alinoos/lab04 lab06
Клонирование в «lab06»...
remote: Enumerating objects: 130, done.
remote: Counting objects: 100% (130/130), done.
remote: Compressing objects: 100% (76/76), done.
remote: Total 130 (delta 41), reused 119 (delta 38), pack-reused 0
Получение объектов: 100% (130/130), 51.19 КиБ | 1.02 МиБ/с, готово.
Определение изменений: 100% (41/41), готово.
$ git remote add origin https://github.com/Alinoos/lab06.git
$ git push origin master
Username for 'https://github.com': Alinoos
Password for 'https://Alinoos@github.com': 
Перечисление объектов: 115, готово.
Подсчет объектов: 100% (115/115), готово.
При сжатии изменений используется до 8 потоков
Сжатие объектов: 100% (105/105), готово.
Запись объектов: 100% (115/115), 45.64 КиБ | 6.52 МиБ/с, готово.
Всего 115 (изменений 37), повторно использовано 0 (изменений 0), повторно использовано пакетов 0
remote: Resolving deltas: 100% (37/37), done.
remote: 
remote: Create a pull request for 'master' on GitHub by visiting:
remote:      https://github.com/Alinoos/lab06/pull/new/master
remote: 
To https://github.com/Alinoos/lab06.git
 * [new branch]      master -> master
$ touch CPackConfig.cmake
$ nano CPackConfig.cmake
$ rm -rf hello_world_application/
$ git add .
$ git commit -m "added CPackConfig.cmake"
[master 2af3946] added CPackConfig.cmake
 17 files changed, 1508 deletions(-)
 create mode 100644 CPackConfig.cmake
 delete mode 100644 hello_world_application/CMakeLists.txt
 delete mode 100644 hello_world_application/_build/CMakeCache.txt
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/CMakeDirectoryInformation.cmake
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/DependInfo.cmake
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/build.make
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/cmake_clean.cmake
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/compiler_depend.internal
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/compiler_depend.make
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/depend.make
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/flags.make
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o.d
 delete mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/progress.marks
 delete mode 100644 hello_world_application/_build/formatter_ex/cmake_install.cmake
 delete mode 100644 hello_world_application/_build/formatter_ex/libformatter_ex.a
 delete mode 100644 hello_world_application/hello_world.cpp

```
## Links

- [DMG](https://cmake.org/cmake/help/latest/module/CPackDMG.html)
- [DEB](https://cmake.org/cmake/help/latest/module/CPackDeb.html)
- [RPM](https://cmake.org/cmake/help/latest/module/CPackRPM.html)
- [NSIS](https://cmake.org/cmake/help/latest/module/CPackNSIS.html)

```
Copyright (c) 2015-2021 The ISC Authors
```
