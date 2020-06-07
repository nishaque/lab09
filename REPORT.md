## Laboratory work IX

<a href="https://yandex.ru/efir/?stream_id=vYrKRcFKi46o"><img src="https://raw.githubusercontent.com/tp-labs/lab09/master/preview.png" width="640"/></a>

Данная лабораторная работа посвещена изучению процесса создания артефактов на примере **Github Release**

```sh
$ open https://help.github.com/articles/creating-releases/
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab09** на сервисе **GitHub**
- [x] 2. Ознакомиться со ссылками учебного материала
- [x] 3. Получить токен для доступа к репозиториям сервиса **GitHub**
- [x] 4. Выполнить инструкцию учебного материала
- [x] 5. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial
Создание переменных среды и установка их значений.
```sh
$ export GITHUB_TOKEN=fcaad8fb6d613b6859110ff8a3ae6553279e4aa9
$ export GITHUB_USERNAME=nishaque
$ export PACKAGE_MANAGER=brew
$ export GPG_PACKAGE_NAME=gpg
```
Установка утилиты **xclip**, которая представляет доступ к буферу обмена X из командной строки.
```sh
# for *-nix system
$ $PACKAGE_MANAGER install xclip
$ alias gsed=sed
# Связывание команды pbcopy с записью данных в секцию clipboard
$ alias pbcopy='xclip -selection clipboard'
# Связывание команды pbpaste с чтением данных из секции clipboard
$ alias pbpaste='xclip -selection clipboard -o'
```
Установка приложения командной строки **github-release**, для работы с релизами на **Github**.
```sh
# Переход в  рабочую директорию
$ cd ${GITHUB_USERNAME}/workspace
$ pushd . # Сохранение текущего каталога в стек
# Активация Node.js и rvm
$ source scripts/activate
# Скачивание и установка пакета, написанного на языке программирования Go
$ go get github.com/aktau/github-release
```
Настройка git-репозитория **lab09** для работы.
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab08 projects/lab09
$ cd projects/lab09
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab09
```
Замена всех упоминаний `lab08` на `lab09`.
```sh
$ gsed -i "" 's/lab08/lab09/g' README.md
```
Установка и работа с программой **GPG** для шифрования информации и создания электронных цифровых подписей.
```sh
# Установка программы
$ $PACKAGE_MANAGER install ${GPG_PACKAGE_NAME}
$ gpg --list-secret-keys --keyid-format LONG
$ gpg --full-generate-key
$ gpg --list-secret-keys --keyid-format LONG
$ gpg -K ${GITHUB_USERNAME}
public and secret key created and signed.

pub   rsa2048 2020-06-07 [SC]
      F595D023301AC6AC9B9801BE74277C96ED394E3C
uid                      Yulia (//) <timoshenkojulie01@gmail.com>
sub   rsa2048 2020-06-07 [E]

# Создание переменной окружения с сохраненным в ней публичным ключом
$ GPG_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep ssb | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
# Создание переменной окружения с сохраненным в ней секретным ключом
$ GPG_SEC_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
# Вывести ключ в ASCII и копировать его в буфер
$ gpg --armor --export ${GPG_KEY_ID} | pbcopy
# Вставить из буфера
$ pbpaste
-----BEGIN PGP PUBLIC KEY BLOCK-----
...
-----END PGP PUBLIC KEY BLOCK-----
$ open https://github.com/settings/keys
# Добавление секретного ключа к настройкам Git'a
$ git config user.signingkey ${GPG_SEC_KEY_ID}
$ git config gpg.program gpg
```
Создание скрипта дя добавления сообщения к тегу
```sh
$ test -r ~/.zsh_profile && echo 'export GPG_TTY=$(tty)' >> ~/.zsh_profile
$ echo 'export GPG_TTY=$(tty)' >> ~/.profile
```
Генерация пакета с расширением `.tar.gz`
```sh
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"

-- Build files have been written to: /Users/nishaque/nishaque/workspace/projects/lab09/_build

$ cmake --build _build --target package
Scanning dependencies of target print
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print
Run CPack packaging tool...
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /Users/nishaque/nishaque/workspace/projects/lab09/_build/print-0.1.0.0-Darwin.tar.gz generated.
```
Инициализация проекта на **Travis CI**
```sh
$ travis login --auto --pro
Successfully logged in as nishaque!
$ travis enable --pro
nishaque/lab09: enabled :)
```
Создание тега для коммита
```sh
# Создание тега с сообщением с информацией
$ git tag -s v0.12.0.0
$ git tag -v v0.1.0.0
$ git show v0.1.0.0
$ git push origin master --tags
Enumerating objects: 101, done.
Counting objects: 100% (101/101), done.
Delta compression using up to 4 threads
Compressing objects: 100% (67/67), done.
Writing objects: 100% (101/101), 52.76 KiB | 13.19 MiB/s, done.
Total 101 (delta 19), reused 101 (delta 19)
remote: Resolving deltas: 100% (19/19), done.
To https://github.com/nishaque/lab09
 * [new branch]      master -> master

```
Создание релиза
```sh
# Проверка версии приложения
$ github-release --version
github-release v0.8.1
$ github-release info -u ${GITHUB_USERNAME} -r lab09
tags:
- v0.1.0.0 (commit: https://api.github.com/repos/nishaque/lab09/commits/b77e72be8000ff846e42e4ccc4d878ebf02b7187)
releases:
# Создание релиза
$ github-release release \
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "libprint" \
    --description "my first release"
```
Добавление артефакта с указанием ОС и архитектуры, на которых происходила компиляция библиотеки
```sh
$ export PACKAGE_OS=`uname -s` PACKAGE_ARCH=`uname -m`
$ export PACKAGE_FILENAME=print-${PACKAGE_OS}-${PACKAGE_ARCH}.tar.gz
$ github-release upload \
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "${PACKAGE_FILENAME}" \
    --file _build/*.tar.gz
```

```sh
$ github-release info -u ${GITHUB_USERNAME} -r lab09tags:
# Скачивание артефакта из раздела релизов для проверки
$ wget https://github.com/${GITHUB_USERNAME}/lab09/releases/download/v0.1.0.0/${PACKAGE_FILENAME}
# Просмотр содержимого архива
$ tar -ztf ${PACKAGE_FILENAME}
print-0.1.0.0-Darwin/cmake/
print-0.1.0.0-Darwin/cmake/print-config-noconfig.cmake
print-0.1.0.0-Darwin/cmake/print-config.cmake
print-0.1.0.0-Darwin/bin/
print-0.1.0.0-Darwin/bin/demo
print-0.1.0.0-Darwin/include/
print-0.1.0.0-Darwin/include/print.hpp
print-0.1.0.0-Darwin/lib/
print-0.1.0.0-Darwin/lib/libprint.a
```

## Report
Создание отчета по ЛР № 9
```sh
$ popd
$ export LAB_NUMBER=09
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ atom REPORT.md
$ gistup -m "lab${LAB_NUMBER}"
```

## Links

- [Create Release](https://help.github.com/articles/creating-releases/)
- [Get GitHub Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
- [Signing Commits](https://help.github.com/articles/signing-commits-with-gpg/)
- [Go Setup](http://www.golangbootcamp.com/book/get_setup)
- [github-release](https://github.com/aktau/github-release)

```
Copyright (c) 2015-2020 The ISC Authors
```
