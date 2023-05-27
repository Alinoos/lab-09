## Laboratory work IX

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

## Report

Подготовка к работе
```sh
$ git clone https://github.com/IvanGalk1n/lab08 lab09 #клонируем репозиторий
$ cd lab09/
$ git remote remove origin
$ git remote add origin https://github.com/IvanGalk1n/lab09 #настраиваем удаленный репозиторий
```

Создаем ключ шифрования
```sh
$ gpg --list-secret-keys --keyid-format LONG  #посмотреть секретные ключи с форматом идентификатора лонг (16 символов)
$ gpg --full-generate-key #сгенерировать полноценную пару ключей
$ gpg --list-secret-keys --keyid-format LONG #посмотреть секретные ключи с форматом идентификатора лонг (16 символов)
$ GPG_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep ssb | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}') #создаем переменную хранящую идентификатор открытого ключа
$ GPG_SEC_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}') #создаем переменную хранящую идентификатор секретного ключа
$ gpg --armor --export ${GPG_KEY_ID} #вывод ключа в текстовом формате (--armor=-a)
-----BEGIN PGP PUBLIC KEY BLOCK-----
*************** # - это ключ, который копируем в гитхаб настройки
-----END PGP PUBLIC KEY BLOCK-----

$ open https://github.com/settings/keys #открываем настройки гитхаба и копируем туда наш открытый gpg ключ
```

Генерируем tgz архив
```sh
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
$ cmake --build _build --target package
```

Создаем тэг и пушим его
```sh
$ git tag -s v0.1.0.0 #-s make a GPG-signed tag, using the default e-mail address’s key (создает тэг использующий gpg-ключ)
$ git tag -v v0.1.0.0 #-v verify the GPG signature of the given tag names (выводит информацию о тэге и подписи)
$ git push origin master --tags
```

Создание и аплоад релиза из тэга
```sh
$ github-release --version
$ github-release info -u ${GITHUB_USERNAME} -r lab09
$ github-release release \
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "logger" \
    --description "my first release"
    
$ export PACKAGE_OS=`uname -s` PACKAGE_ARCH=`uname -m` 
$ export PACKAGE_FILENAME=print-${PACKAGE_OS}-${PACKAGE_ARCH}.tar.gz
$ github-release upload \
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "${PACKAGE_FILENAME}" \
    --file _build/*.tar.gz
```

Проверяем что релиз загрузился и скачиваем архив
```sh
$ github-release info -u ${GITHUB_USERNAME} -r lab09
$ wget https://github.com/${GITHUB_USERNAME}/lab09/releases/download/v0.1.0.0/${PACKAGE_FILENAME}
$ tar -ztf ${PACKAGE_FILENAME}
```

## Links

- [Create Release](https://help.github.com/articles/creating-releases/)
- [Get GitHub Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
- [Signing Commits](https://help.github.com/articles/signing-commits-with-gpg/)
- [Go Setup](http://www.golangbootcamp.com/book/get_setup)
- [github-release](https://github.com/aktau/github-release)

```
Copyright (c) 2015-2021 The ISC Authors
```
