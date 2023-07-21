# OpenSSL_cheat_sheet | Шпаргалка OpenSSL
## Просмотр данных сертификата (формат PEM):
```bash
openssl x509 -text -noout -in ИМЯ_СЕРТИФИКАТА
```
*alias certinfo='openssl x509 -text -noout -in'*
## Скачать сертификат с сервера:
```bash
openssl s_client -connect АДРЕС_СЕРВЕРА:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM > ИМЯ_ФАЙЛА
```
*Принятно сохранять с именем .pem / .cer / .crt*
## Отладка соединения SSL/TLS (инфо о сертфикате, алгоритмах шифрования и т.д.)
```bash
openssl s_client -connect ИМЯ_СЕРВЕРА:443
```
## Самозаверенный сертификат одной командой:
```bash
openssl req -nodes -x509 -sha256 -newkey rsa:4096 \
  -keyout ИМЯ_ЗАКРЫТОГО_КЛЮЧА.key \
  -out ИМЯ_СЕРТИФИКАТА.crt \
  -days 356 \
  -subj "/C=RU/ST=SPb/L=SPb/O=CORP_NAME/OU=CORP_UNIT/CN=example.localnet.ru" \
  -extensions san \
  -config <( \
  echo '[req]'; \
  echo 'distinguished_name=req'; \
  echo '[san]'; \
  echo 'subjectAltName=DNS:example.localnet.ru')
```
*Можно сгенерировать wildcard сертификат указав в CommonName и subjectAltName адрес вида **\*.domain.org**.*
## Пакет ca-certificates:
Добавить сертификат в доверенные ЦС:
1. Файл сертификата обязательно должен заканчиваться на .cer.
2. Копируем файл в "/usr/local/share/ca-certificates/".
3. Обновляем список сертификатов:
```bash
sudo update-ca-certificates     
```
Пример успешного вывода:
```
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```
## Работа с ГОСТ сертификатами:
### Выпуск сертфиката сторонним УЦ:
1.  Генерируем закрытый ключ:
```bash
openssl genpkey -algorithm gost2012_256 -pkeyopt paramset:A -out gost.test.org.key
```
2. Создаем запрос в УЦ (CSR).
```bash
openssl req -new  -md_gost12_256 -key gost.test.org.key -out gost.test.org.csr -subj "/C=RU/L=SPb/O=Test_org/CN=gost.test.org" -addext "subjectAltName=DNS:gost.test.org,DNS:www.gost.test.org"
```
3. Выводим содержимое сформированного файла-запроса:
```bash
openssl req -text -noout -verify -in gost.test.org.csr
```
