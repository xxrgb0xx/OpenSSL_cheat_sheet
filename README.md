# OpenSSL_cheat_sheet | Шпаргалка OpenSSL
## Просмотр данных сертификата (формат PEM)
```bash
openssl x509 -text -noout -in ИМЯ_СЕРТИФИКАТА
```
*alias certinfo='openssl x509 -text -noout -in'*
## Скачать сертификат с сервера
```bash
openssl s_client -connect АДРЕС_СЕРВЕРА:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM > ИМЯ_ФАЙЛА
```
*Принятно сохранять с именем .pem / .cer / .crt*
## Отладка соединения SSL/TLS (инфо о сертфикате, алгоритмах шифрования и т.д.)
```bash
openssl s_client -connect ИМЯ_СЕРВЕРА:443
```
## Проверка соотвествия сертификата и приватного ключа
Сертификат:
```bash
openssl x509 -noout -modulus -in $ИМЯ_СЕРТИФИКАТА | openssl md5 
```
Ключ:
```bash
openssl rsa -noout -modulus -in $ИМЯ_ПРИВАТНОГО_КЛЮЧА | openssl md5 
```
Запрос CSR:
```bash
openssl req -noout -modulus -in $ИМЯ_ЗАПРОСА_CSR | openssl md5
```
*Хэши модулей должны совпадать.*

## Самозаверенный сертификат одной командой
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
## Пакет ca-certificates
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
## Работа с ГОСТ сертификатами
### Выпуск сертфиката сторонним УЦ
1.  Генерируем закрытый ключ:
```bash
openssl genpkey -algorithm gost2012_256 -pkeyopt paramset:A -out gost.test.org.key
```
2. Создаем запрос в УЦ (CSR).
```bash
openssl req -new  -md_gost12_256 -key gost.test.org.key -out gost.test.org.csr \
-subj "/C=RU/L=SPb/O=Test_org/CN=gost.test.org" \
-addext "subjectAltName=DNS:gost.test.org,DNS:www.gost.test.org"
```
3. Выводим содержимое сформированного файла-запроса:
```bash
openssl req -text -noout -verify -in gost.test.org.csr
```
### Выпуск собственным УЦ
1. Генерируем закрытый ключ и запрос на выпуск сертификата (см. раздел "Выпуск сертфиката сторонним УЦ", п. 1-2 ).
2. Генерируем закрытый ключ УЦ:
```bash
openssl genpkey -algorithm gost2012_256 -pkeyopt paramset:A -out ca.key
```
3. Генерируем сертификат УЦ:
```bash
openssl req -new -x509 -md_gost12_256 -days 365 -key ca.key -out ca.cer \
-subj "/C=RU/ST=Russia/L=SPb/O=Test_org/OU=Test_OU/CN=Test_CA"
```
4. Создаем и подписываем запрашиваемый сертификат:
```bash
openssl x509 -req \
-extfile <(printf "subjectAltName=DNS:gost.test.org,DNS:www.gost.test.org \n crlDistributionPoints=URI:http://gost.test.org/ca.crl") \
-days 365 -in gost.test.org.csr -CA ca.cer -CAkey ca.key -CAcreateserial -out gost.test.org.cer
```
## Конвертация PKCS#12 / PFX (сертификат + приватный ключ) в PEM
Приватный ключ:
```bash
openssl pkcs12 -in ИМЯ_СЕРТИФИКАТА.pfx -out key.pem -nodes
```
Сертификат:
```bash
openssl pkcs12 -in ИМЯ_СЕРТИФИКАТА.pfx -clcerts -nokeys -out cert.pem
```

## Конвертация PEM в PKCS#12 / PFX (сертификат + приватный ключ) 
```bash
openssl pkcs12 -inkey ПРИВАТНЫЙ_КЛЮЧ.pem -in СЕРТИФИКАТ.pem -export -out СЕРТИФИКАТ_С_ПРИВАТНЫМ_КЛЮЧОМ.pfx
```

## Зашифровать / расшифровать приватный ключ
Зашифровать:
```bash
openssl rsa -aes256 -in ПРИВАТНЫЙ_КЛЮЧ.pem -out ЗАШИФРОВАННЫЙ_ПРИВАТНЫЙ_КЛЮЧ.pem
```
Расшифровать:
```bash
openssl rsa -in ЗАШИФРОВАННЫЙ_ПРИВАТНЫЙ_КЛЮЧ.pem -out ПРИВАТНЫЙ_КЛЮЧ.pem
```
