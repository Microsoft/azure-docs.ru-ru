---
title: Настройка OpenSSL для Linux
titleSuffix: Azure Cognitive Services
description: Узнайте, как настроить OpenSSL для Linux.
services: cognitive-services
author: jhakulin
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/16/2020
ms.author: jhakulin
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: a6225fec30a87ca0bbe57e414733bc21489f87ad
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104577450"
---
# <a name="configure-openssl-for-linux"></a>Настройка OpenSSL для Linux

При использовании любой версии пакета SDK для распознавания речи перед 1.9.0, [OpenSSL](https://www.openssl.org) динамически настраивается в качестве версии системы узла. В более поздних версиях пакета SDK OpenSSL (версия [1.1.1 b](https://mta.openssl.org/pipermail/openssl-announce/2019-February/000147.html)) статически связывается с основной библиотекой пакета SDK для распознавания речи.

Чтобы обеспечить подключение, убедитесь, что сертификаты OpenSSL установлены в системе. Выполните команду:
```bash
openssl version -d
```

Выходные данные в системах на базе Ubuntu/Debian должны быть:
```
OPENSSLDIR: "/usr/lib/ssl"
```

Проверьте наличие `certs` подкаталога в разделе опенсслдир. В приведенном выше примере это будет `/usr/lib/ssl/certs` .

* Если есть `/usr/lib/ssl/certs` и он содержит много отдельных файлов сертификатов (с `.crt` или `.pem` расширением), дальнейшие действия не требуются.

* Если ОПЕНССЛДИР — это что-то еще `/usr/lib/ssl` и/или имеется отдельный файл пакета сертификатов, а не несколько отдельных файлов, необходимо задать соответствующую переменную среды SSL, чтобы указать, где можно найти сертификаты.

## <a name="examples"></a>Примеры

- ОПЕНССЛДИР — `/opt/ssl` . Существует `certs` подкаталог с множеством `.crt` `.pem` файлов или.
Задайте переменную среды `SSL_CERT_DIR` для указания `/opt/ssl/certs` перед запуском программы, использующей РЕЧЕВОЙ пакет SDK. Пример:
```bash
export SSL_CERT_DIR=/opt/ssl/certs
```

- ОПЕНССЛДИР `/etc/pki/tls` (как и в системах на базе RHEL/CentOS). `certs`Например, существует подкаталог с файлом пакета сертификатов `ca-bundle.crt` .
Установите переменную среды, `SSL_CERT_FILE` чтобы она указывала на этот файл, перед запуском программы, использующей речевой пакет SDK. Пример:
```bash
export SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt
```

## <a name="certificate-revocation-checks"></a>Проверки отзыва сертификатов
При подключении к службе речи речевой пакет SDK проверит, не отозван ли сертификат TLS, используемый службой речи. Для выполнения этой проверки речевому пакету SDK потребуется доступ к точкам распространения списков отзыва сертификатов для центров сертификации, используемых в Azure. Список возможных расположений загрузки списков отзыва сертификатов можно найти в [этом документе](https://docs.microsoft.com/azure/security/fundamentals/tls-certificate-changes). Если сертификат был отозван или не удается скачать его, пакет SDK для распознавания речи прерывает подключение и вызывает отмененное событие.

В случае, если сеть, в которой используется речевой пакет SDK, настроена таким образом, что не позволяет получить доступ к расположениям скачивания списка отзыва сертификатов, проверка списка отзыва сертификатов может быть отключена или настроена на неудачу, если не удается извлечь список отзыва сертификатов. Эта конфигурация выполняется через объект конфигурации, используемый для создания объекта распознавателя.

Чтобы продолжить подключение, когда не удается получить список отзыва сертификатов, установите свойство OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE.

::: zone pivot="programming-language-csharp"

```csharp
config.SetProperty("OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE", "true");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
config->SetProperty("OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE", "true");
```

::: zone-end

::: zone pivot="programming-language-java"

```java
config.setProperty("OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE", "true");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_config.set_property_by_name("OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE", "true")?
```

::: zone-end

::: zone pivot="programming-language-more"

```ObjectiveC
[config setPropertyTo:@"true" byName:"OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE"];
```

::: zone-end
Если задано значение "true", будет предпринята попытка получить список отзыва сертификатов и, если извлечение будет успешным, сертификат будет проверен на отзыв, если при извлечении произойдет сбой, подключение будет разрешено.

Чтобы полностью отключить проверку отзыва сертификатов, присвойте свойству OPENSSL_DISABLE_CRL_CHECK значение true.
::: zone pivot="programming-language-csharp"

```csharp
config.SetProperty("OPENSSL_DISABLE_CRL_CHECK", "true");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
config->SetProperty("OPENSSL_DISABLE_CRL_CHECK", "true");
```

::: zone-end

::: zone pivot="programming-language-java"

```java
config.setProperty("OPENSSL_DISABLE_CRL_CHECK", "true");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_config.set_property_by_name("OPENSSL_DISABLE_CRL_CHECK", "true")?
```

::: zone-end

::: zone pivot="programming-language-more"

```ObjectiveC
[config setPropertyTo:@"true" byName:"OPENSSL_DISABLE_CRL_CHECK"];
```

::: zone-end


> [!NOTE]
> Также следует отметить, что в некоторых дистрибутивах Linux не определена переменная среды TMP или TMPDIR. Это приведет к тому, что речевой пакет SDK скачивает список отзыва сертификатов (CRL) каждый раз, а не кэширует его на диск для повторного использования до истечения срока действия. Чтобы улучшить начальную производительность подключения, можно [создать переменную среды с именем TMPDIR и задать путь к выбранному временному каталогу.](https://help.ubuntu.com/community/EnvironmentVariables)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о пакете SDK службы "Речь"](speech-sdk.md)
