# lab-09

Данная лабораторная работа посвещена изучению процесса создания артефактов на примере **Github Release**

# Task 1

Сначала скопируем репозиторий предыдущей ЛР (соответственно не будет заострять внимание на этом):
```
$ git clone https://github.com/alinoos/lab-08 lab-09
$ cd lab-09/
$ git remote remove origin
$ git remote add origin https://github.com/alinoos/lab-09
```

Создаем ключ шифрования:
```
$ gpg --list-secret-keys --keyid-format LONG
$ gpg --full-generate-key
$ gpg --list-secret-keys --keyid-format LONG #посмотреть секретные ключи с форматом идентификатора лонг (16 символов)
$ GPG_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep ssb | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}') #создаем переменную хранящую идентификатор открытого ключа
$ GPG_SEC_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}') #создаем переменную хранящую идентификатор секретного ключа
$ gpg --armor --export ${GPG_KEY_ID} #вывод ключа в текстовом формате (--armor=-a)
-----BEGIN PGP PUBLIC KEY BLOCK-----
*************** # - это ключ, который копируем в гитхаб настройки
-----END PGP PUBLIC KEY BLOCK-----

$ open https://github.com/settings/keys #открываем настройки гитхаба и копируем туда наш открытый gpg ключ
```

Генерируем `tgz` архив:
```
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
$ cmake --build _build --target package
```

Тегируем новый пуш, для дальнейшего релиза (аналогия с ЛР-6):
```
$ git tag -s v0.1.0.0 #-s make a GPG-signed tag, using the default e-mail address’s key (создает тэг использующий gpg-ключ)
$ git tag -v v0.1.0.0 #-v verify the GPG signature of the given tag names (выводит информацию о тэге и подписи)
$ git push origin master --tags
```

Создаем и загружаем ранее созданный релиз:
```
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

Проверяем что релиз загрузился и скачиваем архив:
```sh
$ github-release info -u ${GITHUB_USERNAME} -r lab09
$ wget https://github.com/${GITHUB_USERNAME}/lab09/releases/download/v0.1.0.0/${PACKAGE_FILENAME}
$ tar -ztf ${PACKAGE_FILENAME}
```
