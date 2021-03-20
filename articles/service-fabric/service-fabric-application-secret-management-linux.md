---
title: Настройка сертификата шифрования в кластерах Linux
description: Узнайте, как настроить сертификат шифрования и шифровать секреты в кластерах Linux.
author: shsha
ms.topic: conceptual
ms.date: 01/04/2019
ms.author: shsha
ms.openlocfilehash: b8e0a19e3f654fc561e7c7e26c6a2da463e24d5f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "78969029"
---
# <a name="set-up-an-encryption-certificate-and-encrypt-secrets-on-linux-clusters"></a>Настройка сертификата шифрования и шифрование секретов в кластерах Linux
В этой статье описывается, как настроить сертификат шифрования и использовать его для шифрования секретов в кластерах Linux. Инструкции для кластеров Windows приведены в разделе [Настройка сертификата шифрования и шифрование секретов в кластерах Windows][secret-management-windows-specific-link].

## <a name="obtain-a-data-encipherment-certificate"></a>Получение сертификата шифрования данных
Сертификат шифрования данных используются исключительно для шифрования и расшифровки [параметров][parameters-link] в файле Settings.xml службы и [переменных среды][environment-variables-link] в ее файле ServiceManifest.xml. Он не применяется для аутентификации или подписывания зашифрованного текста. Сертификат должен отвечать приведенным ниже требованиям.

* Сертификат должен содержать закрытый ключ.
* Использование ключа сертификата должно включать шифрование данных (10) и не должно включать аутентификацию сервера или клиента.

  Например, для создания требуемого сертификата с помощью OpenSSL можно использовать следующие команды.
  
  ```console
  user@linux:~$ openssl req -newkey rsa:2048 -nodes -keyout TestCert.prv -x509 -days 365 -out TestCert.pem
  user@linux:~$ cat TestCert.prv >> TestCert.pem
  ```

## <a name="install-the-certificate-in-your-cluster"></a>Установка сертификата в кластере
Этот сертификат должен быть установлен в папке `/var/lib/sfcerts` на каждом узле в кластере. У учетной записи пользователя, с помощью которой выполняется служба (по умолчанию — sfuser), должен быть **доступ на чтение** к установленному сертификату (то есть `/var/lib/sfcerts/TestCert.pem` в текущем примере).

## <a name="encrypt-secrets"></a>Шифрование секретов
Следующий фрагмент кода может использоваться для шифрования секрета. Он только шифрует значение, но **не** подписывает зашифрованный текст. Чтобы создать зашифрованные данные для значений секретов, **необходимо использовать** тот же сертификат шифрования, установленный в кластере.

```console
user@linux:$ echo "Hello World!" > plaintext.txt
user@linux:$ iconv -f ASCII -t UTF-16LE plaintext.txt | tr -d '\n' > plaintext_UTF-16.txt
user@linux:$ openssl smime -encrypt -in plaintext_UTF-16.txt -binary -outform der TestCert.pem | base64 > encrypted.txt
```
Результирующая строка в кодировке Base-64 в файле encrypted.txt содержит как зашифрованные данные секрета, так и сведения о сертификате, с помощью которого выполнено шифрование. Можно проверить его действительность, расшифровав его с помощью OpenSSL.
```console
user@linux:$ cat encrypted.txt | base64 -d | openssl smime -decrypt -inform der -inkey TestCert.prv
```

## <a name="next-steps"></a>Дальнейшие действия
Узнайте, как [указать зашифрованные секреты в приложении][secret-management-specify-encrypted-secrets-link].

<!-- Links -->
[parameters-link]:service-fabric-how-to-parameterize-configuration-files.md
[environment-variables-link]: service-fabric-how-to-specify-environment-variables.md
[secret-management-windows-specific-link]: service-fabric-application-secret-management-windows.md
[secret-management-specify-encrypted-secrets-link]: service-fabric-application-secret-management.md#specify-encrypted-secrets-in-an-application
