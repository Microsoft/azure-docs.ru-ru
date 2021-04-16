---
title: Руководство. Использование OpenSSL для создания тестовых сертификатов X.509 для Центра Интернета вещей Azure | Документация Майкрософт
description: Руководство. Использование OpenSSL для создания ЦС и сертификатов устройств для Центра Интернета вещей Azure
author: v-gpettibone
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 02/26/2021
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 0843e5d3a5e91cb4acdf18ad6bdf6f4f0c214f72
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378301"
---
# <a name="tutorial-using-openssl-to-create-test-certificates"></a>Руководство. Использование OpenSSL для создания тестовых сертификатов

Хотя вы всегда можете приобрести сертификаты X.509 у доверенного центра сертификации, создание собственной иерархии тестовых сертификатов или применение самозаверяющих сертификатов является альтернативой тестированию проверки подлинности устройств в центре Интернета вещей. В следующем примере с помощью [OpenSSL](https://www.openssl.org/) и [OpenSSL Cookbook](https://www.feistyduck.com/library/openssl-cookbook/online/ch-openssl.html) показано, как создать корневой центр сертификации (ЦС), подчиненный ЦС и сертификат устройства. Затем вы узнаете, как подписать подчиненный ЦС и сертификат устройства, создав иерархию сертификатов. Этот пример приводится только в образовательных целях.

## <a name="step-1---create-the-root-ca-directory-structure"></a>Шаг 1. Создание структуры каталогов для корневого ЦС

Создайте структуру каталогов для ЦС.

* Каталог **certs** содержит новые сертификаты.
* Каталог **db** содержит базу данных сертификатов.
* Каталог **private** содержит закрытый ключ ЦС.

```bash
  mkdir rootca
  cd rootca
  mkdir certs db private
  touch db/index
  openssl rand -hex 16 > db/serial
  echo 1001 > db/crlnumber
```

## <a name="step-2---create-a-root-ca-configuration-file"></a>Шаг 2. Создание файла конфигурации для корневого ЦС

Перед созданием ЦС нужно создать файл конфигурации и сохранить его с именем `rootca.conf` в каталоге rootca.

```xml
[default]
name                     = rootca
domain_suffix            = example.com
aia_url                  = http://$name.$domain_suffix/$name.crt
crl_url                  = http://$name.$domain_suffix/$name.crl
default_ca               = ca_default
name_opt                 = utf8,esc_ctrl,multiline,lname,align

[ca_dn]
commonName               = "Test Root CA"

[ca_default]
home                     = ../rootca 
database                 = $home/db/index
serial                   = $home/db/serial
crlnumber                = $home/db/crlnumber
certificate              = $home/$name.crt
private_key              = $home/private/$name.key
RANDFILE                 = $home/private/random
new_certs_dir            = $home/certs
unique_subject           = no
copy_extensions          = none
default_days             = 3650
default_crl_days         = 365
default_md               = sha256
policy                   = policy_c_o_match

[policy_c_o_match]
countryName              = optional
stateOrProvinceName      = optional
organizationName         = optional
organizationalUnitName   = optional
commonName               = supplied
emailAddress             = optional

[req]
default_bits             = 2048
encrypt_key              = yes
default_md               = sha256
utf8                     = yes
string_mask              = utf8only
prompt                   = no
distinguished_name       = ca_dn
req_extensions           = ca_ext

[ca_ext]
basicConstraints         = critical,CA:true
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[sub_ca_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:true,pathlen:0
extendedKeyUsage         = clientAuth,serverAuth
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[client_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
extendedKeyUsage         = clientAuth
keyUsage                 = critical,digitalSignature
subjectKeyIdentifier     = hash

```

## <a name="step-3---create-a-root-ca"></a>Шаг 3. Создание корневого ЦС

Для начала создайте ключ и запрос на подписывание сертификата в каталоге rootca.

```bash
  openssl req -new -config rootca.conf -out rootca.csr -keyout private/rootca.key
```

Затем создайте самозаверяющий сертификат ЦС. Самозаверяющий сертификат подходит для тестирования. Укажите расширения ca_ext для файлов конфигурации в командной строке. Это означает, что сертификат предназначен для корневого ЦС и может использоваться для подписывания сертификатов и списков отзыва сертификатов. Подпишите сертификат и зафиксируйте его в базе данных.

```bash
  openssl ca -selfsign -config rootca.conf -in rootca.csr -out rootca.crt -extensions ca_ext
```

## <a name="step-4---create-the-subordinate-ca-directory-structure"></a>Шаг 4. Создание структуры каталогов для подчиненного ЦС

Создайте структуру каталогов для подчиненного ЦС.

```bash
  mkdir subca
  cd subca
  mkdir certs db private
  touch db/index
  openssl rand -hex 16 > db/serial
  echo 1001 > db/crlnumber
```

## <a name="step-5---create-a-subordinate-ca-configuration-file"></a>Шаг 5. Создание файла конфигурации для подчиненного ЦС

Создайте файл конфигурации и сохраните его с именем subca.conf в каталоге `subca`.

```bash
[default]
name                     = subca
domain_suffix            = example.com
aia_url                  = http://$name.$domain_suffix/$name.crt
crl_url                  = http://$name.$domain_suffix/$name.crl
default_ca               = ca_default
name_opt                 = utf8,esc_ctrl,multiline,lname,align

[ca_dn]
commonName               = "Test Subordinate CA"

[ca_default]
home                     = . 
database                 = $home/db/index
serial                   = $home/db/serial
crlnumber                = $home/db/crlnumber
certificate              = $home/$name.crt
private_key              = $home/private/$name.key
RANDFILE                 = $home/private/random
new_certs_dir            = $home/certs
unique_subject           = no
copy_extensions          = copy 
default_days           
default_crl_days         = 90 
default_md               = sha256
policy                   = policy_c_o_match

[policy_c_o_match]
countryName              = optional
stateOrProvinceName      = optional
organizationName         = optional
organizationalUnitName   = optional
commonName               = supplied
emailAddress             = optional

[req]
default_bits             = 2048
encrypt_key              = yes
default_md               = sha256
utf8                     = yes
string_mask              = utf8only
prompt                   = no
distinguished_name       = ca_dn
req_extensions           = ca_ext

[ca_ext]
basicConstraints         = critical,CA:true
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[sub_ca_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:true,pathlen:0
extendedKeyUsage         = clientAuth,serverAuth
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[client_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
extendedKeyUsage         = clientAuth
keyUsage                 = critical,digitalSignature
subjectKeyIdentifier     = hash
```

## <a name="step-6---create-a-subordinate-ca"></a>Шаг 6. Создание подчиненного ЦС

Создайте новый серийный номер в файле `rootca/db/serial` для сертификата подчиненного ЦС.

```bash
  openssl rand -hex 16 > db/serial
```

>[!IMPORTANT]
>Вам нужно создавать новый серийный номер для каждого создаваемого сертификата подчиненного ЦС или сертификата устройства. Разные сертификаты не могут иметь один серийный номер.

В этом примере показано, как создать подчиненный ЦС или ЦС регистрации. Так как корневой ЦС может подписывать сертификаты, создание подчиненного ЦС не является обязательным. Но такая структура максимально близка к настоящей иерархии сертификатов в реальном мире, где корневой ЦС используется автономно, а клиентские сертификаты выдаются подчиненными ЦС.

Примените файл конфигурации для создания ключа и запроса на подписывание сертификата.

```bash
  openssl req -new -config subca.conf -out subca.csr -keyout private/subca.key
```

Передайте этот запрос на подписывание сертификата корневому ЦС и с его помощью выдайте и подпишите сертификат для подчиненного ЦС. Укажите аргумент расширений sub_ca_ext в командной строке. Эти расширения означают, что сертификат предназначен для ЦС, который может подписывать сертификаты и списки отзыва сертификатов. В ответ на запрос подпишите сертификат и зафиксируйте его в базе данных.

```bash
  openssl ca -config ../rootca/rootca.conf -in subca.csr -out subca.crt -extensions sub_ca_ext
```

## <a name="step-7---demonstrate-proof-of-possession"></a>Шаг 7. Подтверждение права владения

Теперь у вас есть сертификаты для корневого и подчиненного ЦС. Вы можете подписывать сертификаты устройств с помощью любого из них. Тот, который вы выберете, следует отправить в Центр Интернета вещей. В следующих шагах предполагается, что вы используете сертификат подчиненного ЦС. Чтобы отправить сертификат подчиненного ЦС в Центр Интернета вещей и зарегистрировать его:

1. На портале Azure перейдите к нужному Центру Интернета вещей и выберите **Параметры > Сертификаты**.

1. Щелкните **Добавить**, чтобы добавить новый сертификат подчиненного ЦС.

1. Введите отображаемое имя в поле **Имя сертификата** и выберите ранее созданный PEM-файл сертификата.

1. Щелкните **Сохранить**. Сертификат появится в списке сертификатов с состоянием **Не проверено**. Процесс подтверждения позволит подтвердить, что вы владеете сертификатом.

   
1. Выберите сертификат, чтобы открыть диалоговое окно **Сведения о сертификате**.

1. Щелкните **Создать код проверки**. Дополнительные сведения см. в руководстве [Подтверждение владения сертификатом центра сертификации](tutorial-x509-prove-possession.md).

1. Скопируйте код проверки в буфер обмена. Этот код проверки необходимо задать в качестве субъекта сертификата. Например, если код проверки имеет вид BB0C656E69AF75E3FB3C8D922C1760C58C1DA5B05AAA9D0A, добавьте его в качестве субъекта сертификата, как показано на шаге 9.

1. Создайте закрытый ключ.

  ```bash
    $ openssl genpkey -out pop.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
  ```

9. Создайте запрос на подписывание сертификата (CSR) на основе закрытого ключа. Добавьте код проверки в качестве субъекта сертификата.

  ```bash
  openssl req -new -key pop.key -out pop.csr
  
    -----
    Country Name (2 letter code) [XX]:.
    State or Province Name (full name) []:.
    Locality Name (eg, city) [Default City]:.
    Organization Name (eg, company) [Default Company Ltd]:.
    Organizational Unit Name (eg, section) []:.
    Common Name (eg, your name or your server hostname) []:BB0C656E69AF75E3FB3C8D922C1760C58C1DA5B05AAA9D0A
    Email Address []:

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:
 
  ```

10. Создайте сертификат, используя файл конфигурации корневого ЦС и CSR, чтобы подтвердить владение сертификатом.

  ```bash
    openssl ca -config rootca.conf -in pop.csr -out pop.crt -extensions client_ext

  ```

11. Выберите новый сертификат в представлении **Сведения о сертификате**. Чтобы найти файл PEM, перейдите в папку certs.

12. После отправки сертификата щелкните **Проверить**. Состояние сертификата ЦС должно измениться на **Проверено**.

## <a name="step-8---create-a-device-in-your-iot-hub"></a>Шаг 8. Создание устройства в Центре Интернета вещей

Перейдите к Центру Интернета вещей на портале Azure и создайте новое удостоверение устройства IoT со следующими значениями:

1. Укажите **идентификатор устройства**, соответствующий имени субъекта для сертификатов устройств.

1. Выберите тип проверки подлинности **Сертификат X.509, подписанный центром сертификации**.

1. Щелкните **Сохранить**.

## <a name="step-9---create-a-client-device-certificate"></a>Шаг 9. Создание сертификата для клиентского устройства

Чтобы создать сертификат клиента, необходимо сначала создать закрытый ключ. Следующая команда позволяет применить OpenSSL для создания закрытого ключа. Создайте ключ в каталоге subca.

```bash
openssl genpkey -out device.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
```

Создайте для этого ключа запрос на подписывание сертификата. Вам не нужно указывать пароль запроса или необязательное название компании. Но в поле Common name (Общее имя) необходимо ввести идентификатор устройства.

```bash
openssl req -new -key device.key -out device.csr

-----
Country Name (2 letter code) [XX]:.
State or Province Name (full name) []:.
Locality Name (eg, city) [Default City]:.
Organization Name (eg, company) [Default Company Ltd]:.
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server hostname) []:`<your device ID>`
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

```

Проверьте, что запрос на подписывание сертификата соответствует вашим требованиям.

```bash
openssl req -text -in device.csr -noout
```

Отправьте этот запрос подчиненному ЦС, чтобы подключиться к иерархии сертификатов. Укажите значение `client_ext` в параметре `-extensions`. Обратите внимание, что `Basic Constraints` в выданном сертификате означает, что он не предназначен для ЦС. Если вам нужно подписать несколько сертификатов, не забывайте обновлять серийный номер с помощью команды `rand -hex 16 > db/serial` OpenSSL перед созданием каждого из них.

```bash
openssl ca -config subca.conf -in device.csr -out device.crt -extensions client_ext
```

## <a name="next-steps"></a>Next Steps

Руководство [по тестированию проверки подлинности на основе сертификатов](tutorial-x509-test-certificate.md) поможет вам убедиться, что ваш сертификат позволяет устройству выполнять вход в Центр Интернета вещей.
