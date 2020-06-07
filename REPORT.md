## Laboratory work VII


<a href="https://yandex.ru/efir/?stream_id=vDHLoKtKoa3o"><img src="https://raw.githubusercontent.com/tp-labs/lab07/master/preview.png" width="640"/></a>

Данная лабораторная работа посвещена изучению систем управления пакетами на примере **Hunter**

```sh
$ open https://github.com/ruslo/hunter
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab07** на сервисе **GitHub**
- [x] 2. Выполнить инструкцию учебного материала
- [x] 3. Ознакомиться со ссылками учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial
Создание переменных среды и установка их значений, а также связывание команд с их "новыми" названиями.
```sh
$ export GITHUB_USERNAME=nishaque
$ alias gsed=sed
```
Начало работы в каталоге workspace
```sh
# Переход в  рабочую директорию
$ cd ${GITHUB_USERNAME}/workspace
$ pushd . # Сохранение текущего каталога в стек
# Активация Node.js и rvm
$ source scripts/activate
```
Настройка git-репозитория **lab07** для работы
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab06 projects/lab07
$ cd projects/lab07
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab07
```
Скачивание и подключение модуля `HunterGate`
```sh
$ mkdir -p cmake # Создание директории где будут храниться файлы Hunter
# Скачивание данных из файла в удаленном репозитории и их запись в файл HunterGate.cmake
$ wget https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake -O cmake/HunterGate.cmake
--2020-06-06 20:11:07--  https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.84.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.84.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 17070 (17K) [text/plain]
Saving to: ‘cmake/HunterGate.cmake’

cmake/HunterGate.cm 100%[===================>]  16.67K  --.-KB/s    in 0.03s   

2020-06-06 20:11:07 (498 KB/s) - ‘cmake/HunterGate.cmake’ saved [17070/17070]
# Добавление HunterGate к CMake
$ gsed -i "" '/cmake_minimum_required(VERSION 3.4)/a\

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https:\//github.com/cpp-pm/hunter/archive/v0.23.251.tar.gz"
    SHA1 "5659b15dc0884d4b03dbd95710e6a1fa0fc3258d"
)
' CMakeLists.txt
```
Теперь не нужно скачивать **GTest** самостоятельно. **Hunter** сам подтянет добавленные с помощью функции `hunter_add_package`.
```sh
# Удаление подмодуля с GTest
$ git rm -rf third-party/gtest
rm 'third-party/gtest'
# Добавение через hunter пакета gtest и его поиск
$ gsed -i "" '/set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")/a\

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)
' CMakeLists.txt
# Удаление строки с добавлением поддиректории gtest
$ gsed -i "" 's/gtest_main/GTest::gtest_main/' CMakeLists.txt
# Замена обращение к gtest gtest_main на GTest::gtest_main
$ gsed -i "" 's/gtest_main/GTest::gtest_main/' CMakeLists.txt
```
Сборка прокта при помощи **Hunter**.
```sh
# Видим как полключаются пакеты при помощи Hanter'a
$ cmake -H. -B_builds -DBUILD_TESTS=ON
...
 -- Build files have been written to: /Users/nishaque/nishaque/workspace/projects/lab07/_builds

$ cmake --build _builds
Scanning dependencies of target print
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print

$ cmake --build _builds --target test

Running tests...
Test project  /Users/nishaque/nishaque/workspace/projects/lab07/_builds
    Start 1: check
1/1 Test #1: check ............................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.00 sec

# Вывод файлов из директории .hunter
$ ls -la $HOME/.hunter
total 0
drwxr-xr-x   3 nishaque  staff    96 Mar  4 18:02 .
drwxr-xr-x+ 42 nishaque  staff  1344 Jun  5 22:38 ..
drwxr-xr-x   6 nishaque  staff   192 Mar  4 19:37 _Base

```
Установка **Hunter** в систему
```sh
$ git clone https://github.com/cpp-pm/hunter $HOME/nishaque/workspace/projects/hunter
$ export HUNTER_ROOT=/Users/nishaque/nishaque/workspace/projects/hunter
$ rm -rf _builds
$ cmake -H. -B_builds -DBUILD_TESTS=ON

-- Build files have been written to: /Users/nishaque/nishaque/workspace/projects/lab07/_builds

$ cmake --build _builds
$ cmake --build _builds --target test
```
Добавление конфигурационного файла в проект, который будет содержать необходимую версию GTest.
```sh
# Просмотр дефолтной версии GTest
$ cat $HUNTER_ROOT/cmake/configs/default.cmake | grep GTest
  hunter_default_version(GTest VERSION 1.7.0-hunter-6)
  hunter_default_version(GTest VERSION 1.10.0)
# Просмотр всех версий GTest, поддерживаемых Hunter'ом
$ cat $HUNTER_ROOT/cmake/projects/GTest/hunter.cmake
$ mkdir cmake/Hunter
# Установка нужной версии GTest
$ cat > cmake/Hunter/config.cmake <<EOF
hunter_config(GTest VERSION 1.7.0-hunter-9)
EOF
# add LOCAL in HunterGate function
```
Добавление файла, использующего библиотеку `print`
```sh
$ mkdir demo
#
$ cat > demo/main.cpp <<EOF
#include <print.hpp>

#include <cstdlib>

int main(int argc, char* argv[])
{
  const char* log_path = std::getenv("LOG_PATH");
  if (log_path == nullptr)
  {
    std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
    return 1;
  }
  std::string text;
  while (std::cin >> text)
  {
    std::ofstream out{log_path, std::ios_base::app};
    print(text, out);
    out << std::endl;
  }
}
EOF

$ gsed -i  '/endif()/a\

add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print)
install(TARGETS demo RUNTIME DESTINATION bin)
' CMakeLists.txt
```
Добавление подмодуля **polly**, который содержит инструкции для сборки проектов с установленным **Hunter**.
```sh
$ mkdir tools
$ git submodule add https://github.com/ruslo/polly tools/polly
$ tools/polly/bin/polly.py --test

...
Log saved: /Users/nishaque/nishaque/workspace/projects/lab07/_logs/polly/default/log.txt
-
Generate: 0:00:04.861035s
Build: 0:00:01.853576s
Test: 0:00:00.040277s
-
Total: 0:00:06.755267s
-
SUCCESS
```
Добавим изменения на удаленный репозиторий
```sh
$ git add .
$ git commit -m "Add lab"
$ git push origin master
```

## Report

```sh
$ popd
$ export LAB_NUMBER=07
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Links

- [Create Hunter package](https://docs.hunter.sh/en/latest/creating-new/create.html)
- [Custom Hunter config](https://github.com/ruslo/hunter/wiki/example.custom.config.id)
- [Polly](https://github.com/ruslo/polly)

```
Copyright (c) 2015-2020 The ISC Authors
```
