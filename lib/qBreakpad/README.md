# qBreakpad

[![Build status](https://travis-ci.org/buzzySmile/qBreakpad.svg?branch=master)](https://travis-ci.org/buzzySmile/qBreakpad)

qBreakpad is Qt library to use google-breakpad crash reporting facilities (and using it conviniently).
Supports
* Windows (but crash dump decoding will not work with MinGW compiler)
* Linux
* MacOS X

## How to run demo in Qt Creator

* Clone repository recursively

```bash
cd ~/work/my-project/src
git submodule add git@github.com:jiakuan/qBreakpad.git
git submodule update --init --recursive
```

* Build qBreakpad static library (qBreakpad/handler/)

```bash
#!/usr/bin/env bash

#-------------------------------------------------------------------------------
# Variables that can be changed according to the dev environment
PROJECT_ROOT=$HOME/work/my-app
BP_PROJECT_DIR=${PROJECT_ROOT}/src/qBreakpad/
BP_PROJECT_FILE=${BP_PROJECT_DIR}/qBreakpad.pro
BP_BUILD_DIR=${PROJECT_ROOT}/build/qBreakpad

QT_BIN_DIR=$HOME/Qt/5.8/clang_64/bin
#-------------------------------------------------------------------------------

SCRIPTPATH="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
echo "Script path: $SCRIPTPATH"

rm -rf ${BP_BUILD_DIR} && mkdir -p ${BP_BUILD_DIR}
rc=$?; if [[ $rc != 0 ]]; then exit $rc; fi

cd ${BP_BUILD_DIR}
${QT_BIN_DIR}/qmake ${BP_PROJECT_FILE} -r -spec macx-clang CONFIG+=x86_64
/usr/bin/make
rc=$?; if [[ $rc != 0 ]]; then exit $rc; fi

if [[ $rc == 0 ]]; then echo -e "\033[0;32mBreakpad: BUILD SUCCESSFUL\033[0m\n"; fi
```

To run the demo project, open qBreakpad.pro in Qt Creator as a separate project, and then configure build directory of all build configurations to:

```
On Mac:

$HOME/work/my-app/src/qBreakpad/build

On Windows:

C:\work\my-app\src\qBreakpad\build
```

After that, you can build and run demo apps in Qt Creator.


## How to use in your project

* Include "qBreakpad/qBreakpad.pri" to your target Qt project

```c++
include($$PWD/../qBreakpad/qBreakpad.pri)
```

* Use ```QBreakpadHandler``` singleton class to enable automatic crash dumps generation on any failure; example:

```c++
#include <QBreakpadHandler.h>

int main(int argc, char* argv[])
{
    ...
    QBreakpadInstance.setDumpPath(QLatin1String("crashes"));
    ...
}
```

* Read Google Breakpad documentation to know further workflow

## Generate symbol file 

Firstly, we need to build the dump_syms executable.

```
PROJECT_ROOT=$HOME/work/my-app
DUMP_SYMS_SOURCE_DIR=${PROJECT_ROOT}/src/qBreakpad/third_party/breakpad/src/tools/mac/dump_syms

cd ${DUMP_SYMS_SOURCE_DIR}
xcodebuild -project dump_syms.xcodeproj -configuration Release

DUMP_SYMS_BUILD_DIR=${DUMP_SYMS_SOURCE_DIR}/build/Release

cd ${PROJECT_ROOT}/build
for macho in `find . |grep "\.o$"`; do
    ${DUMP_SYMS_BUILD_DIR}/dump_syms $macho >> ${PROJECT_ROOT}/build/my-app.breadpad
done

```

## Build Breadpad processors

Follow the steps mentioned on this page: https://github.com/google/breakpad, and then we will get:

```
src/processor/minidump_stackwalk
src/processor/minidump_dump
```

## Getting started with Google Breakpad

https://chromium.googlesource.com/breakpad/breakpad/+/master/docs/getting_started_with_breakpad.md

Detail description about integration `qBreakpad` into your system and platform you could find in **[Wiki](https://github.com/buzzySmile/qBreakpad/wiki)**.
