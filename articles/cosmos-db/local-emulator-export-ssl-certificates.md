---
title: Экспорт сертификатов эмулятора Azure Cosmos DB
description: Узнайте, как экспортировать сертификат эмулятора Azure Cosmos DB для использования с приложениями Java, Python и Node.js. Сертификаты должны быть экспортированы и использованы для языков и сред выполнения, которые не используют хранилище сертификатов Windows.
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/17/2020
author: deborahc
ms.author: dech
ms.custom: devx-track-python, devx-track-java, contperf-fy21q1
ms.openlocfilehash: 952be09662c2c74f883d63de72bba2b9cb58d0e0
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102554009"
---
# <a name="export-the-azure-cosmos-db-emulator-certificates-for-use-with-java-python-and-nodejs-apps"></a>Экспорт Azure Cosmos DB сертификатов эмулятора для использования с приложениями Java, Python и Node.js
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Эмулятор Azure Cosmos DB предоставляет локальную среду для разработки, которая эмулирует службу Azure Cosmos DB. Эмулятор Azure Cosmos DB поддерживает только безопасное подключение через TLS-подключения.

Сертификаты в локальном эмуляторе Azure Cosmos DB создаются при первом запуске эмулятора. Создается два сертификата: Один из них используется для подключения к локальному эмулятору, а другой используется для управления шифрованием данных эмулятора по умолчанию в эмуляторе. Вам нужно экспортировать сертификат подключения с именем DocumentDBEmulatorCertificate.

При использовании эмулятора для разработки приложений на разных языках, таких как Java, Python или Node.js, необходимо экспортировать сертификат эмулятора и импортировать его в необходимое хранилище сертификатов.

Язык и среда выполнения .NET используют хранилище сертификатов Windows для безопасного подключения к Azure Cosmos DB локальному эмулятору при запуске приложения на узле ОС Windows. Другие языки используют собственные методы для управления сертификатами и использования сертификатов. Например, Java использует собственное [хранилище сертификатов](https://docs.oracle.com/cd/E19830-01/819-4712/ablqw/index.html), Python использует программы- [оболочки сокетов](https://docs.python.org/2/library/ssl.html), а Node.js использует [тлссоккет](https://nodejs.org/api/tls.html#tls_tls_connect_options_callback).

В этой статье показано, как экспортировать сертификаты TLS/SSL для использования на разных языках и средах выполнения, которые не интегрируются с хранилищем сертификатов Windows. Дополнительные сведения об эмуляторе см. в статье об [использовании эмулятора Azure Cosmos DB для разработки и тестирования](./local-emulator.md).

## <a name="export-the-azure-cosmos-db-tlsssl-certificate"></a><a id="export-emulator-certificate"></a>Экспорт сертификата Azure Cosmos DB TLS/SSL

Чтобы успешно использовать конечную точку эмулятора из языков и сред выполнения, которые не интегрируются с хранилищем сертификатов Windows, необходимо экспортировать сертификат эмулятора. Сертификат можно экспортировать с помощью диспетчера сертификатов Windows. Используйте следующие пошаговые инструкции для экспорта сертификата "DocumentDBEmulatorCertificate" в виде файла X. 509 (. cer) в кодировке BASE-64.

1. Запустите диспетчер сертификатов Windows, запустив certlm. msc и перейдя в папку Личные->Certificates, а затем откройте сертификат с понятным именем **DocumentDbEmulatorCertificate**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-1.png" alt-text="Экспорт данных в локальном эмуляторе Azure Cosmos DB (шаг 1)":::

1. Щелкните **Сведения**, а затем нажмите кнопку **ОК**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-2.png" alt-text="Экспорт данных в локальном эмуляторе Azure Cosmos DB (шаг 2)":::

1. Щелкните **Копировать в файл...**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-3.png" alt-text="Экспорт данных в локальном эмуляторе Azure Cosmos DB (шаг 3)":::

1. Щелкните **Далее**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-4.png" alt-text="Экспорт данных в локальном эмуляторе Azure Cosmos DB (шаг 4)":::

1. Щелкните **No, do not export private key** (Не экспортировать закрытый ключ), а затем — **Далее**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-5.png" alt-text="Экспорт данных в локальном эмуляторе Azure Cosmos DB (шаг 5)":::

1. Щелкните **Файлы X.509 (.CER) в кодировке Base-64**, а затем — **Далее**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-6.png" alt-text="Экспорт данных в локальном эмуляторе Azure Cosmos DB (шаг 6)":::

1. Укажите имя сертификата. В этом случае — **documentdbemulatorcert**. Щелкните **Далее**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-7.png" alt-text="Экспорт данных в локальном эмуляторе Azure Cosmos DB (шаг 7)":::

1. Нажмите кнопку **Готово**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-8.png" alt-text="Экспорт данных в локальном эмуляторе Azure Cosmos DB (шаг 8)":::

## <a name="use-the-certificate-with-java-apps"></a>Использование сертификата с приложениями Java

При запуске приложений Java или приложений MongoDB, использующих клиент на основе Java, проще установить сертификат в хранилище сертификатов Java по умолчанию, чем при передаче `-Djavax.net.ssl.trustStore=<keystore> -Djavax.net.ssl.trustStorePassword="<password>"` флагов. Например, включенное демонстрационное приложение Java ( `https://localhost:8081/_explorer/index.html` ) зависит от хранилища сертификатов по умолчанию.

Следуйте инструкциям в разделе [Добавление сертификата в хранилище сертификатов Java](https://docs.oracle.com/cd/E54932_01/doc.705/e54936/cssg_create_ssl_cert.htm) , чтобы импортировать сертификат X. 509 в хранилище сертификатов Java по умолчанию. Помните, что при запуске keytool вы будете работать в каталоге *% JAVA_HOME%* . После импорта сертификата в хранилище сертификатов клиенты SQL и API Azure Cosmos DB для MongoDB смогут подключаться к эмулятору Azure Cosmos DB.

Кроме того, для импорта сертификата можно выполнить следующий скрипт Bash:

```bash
#!/bin/bash

# If emulator was started with /AllowNetworkAccess, replace the below with the actual IP address of it:
EMULATOR_HOST=localhost
EMULATOR_PORT=8081
EMULATOR_CERT_PATH=/tmp/cosmos_emulator.cert
openssl s_client -connect ${EMULATOR_HOST}:${EMULATOR_PORT} </dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > $EMULATOR_CERT_PATH
# delete the cert if already exists
sudo $JAVA_HOME/bin/keytool -cacerts -delete -alias cosmos_emulator
# import the cert
sudo $JAVA_HOME/bin/keytool -cacerts -importcert -alias cosmos_emulator -file $EMULATOR_CERT_PATH
```

После установки сертификата TLS/SSL "CosmosDBEmulatorCertificate" приложение должно иметь возможность подключения и использования локального эмулятора Azure Cosmos DB. Если у вас возникнут проблемы, см. статью [Отладка соединений SSL/TLS](https://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/ReadDebug.html) . В большинстве случаев сертификат не может быть установлен в хранилище *% JAVA_HOME%/жре/либ/секурити/кацертс%* . Например, при наличии нескольких установленных версий Java приложение может использовать другое хранилище cacerts, отличное от того, которое вы обновили.

## <a name="use-the-certificate-with-python-apps"></a>Использование сертификата с приложениями Python

При подключении к эмулятору из приложений Python проверка TLS отключена. По умолчанию [пакет SDK для Python (версия 2.0.0 или более поздней версии)](sql-api-sdk-python.md) для API SQL не будет пытаться использовать сертификат TLS/SSL при подключении к локальному эмулятору. Если вы хотите использовать проверку TLS, можно воспользоваться примерами в документации по [оболочкам сокетов Python](https://docs.python.org/2/library/ssl.html) .

## <a name="how-to-use-the-certificate-in-nodejs"></a>Как использовать сертификат в Node.js

При подключении к эмулятору из Node.js SDK проверка TLS отключена. По умолчанию [Node.js SDK (версия 1.10.1 или более поздней версии)](sql-api-sdk-node.md) для API SQL не будет пытаться использовать сертификат TLS/SSL при подключении к локальному эмулятору. Если вы хотите использовать проверку TLS, можно воспользоваться примерами в [ документации поNode.js](https://nodejs.org/api/tls.html#tls_tls_connect_options_callback).

## <a name="rotate-emulator-certificates"></a>Повернуть сертификаты эмулятора

Вы можете принудительно повторно создать сертификаты эмулятора, выбрав **Сброс данных** из эмулятора Azure Cosmos DB, работающего в панели задач Windows. Обратите внимание, что это действие также приведет к очистке всех данных, хранящихся локально в эмуляторе.

:::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-reset-data.png" alt-text="Сброс данных в локальном эмуляторе Azure Cosmos DB":::

Если вы установили сертификат в хранилище сертификатов Java или использовали его в других местах, необходимо повторно импортировать их с использованием текущих сертификатов. Приложение не может подключиться к локальному эмулятору, пока не будут обновлены сертификаты.

## <a name="next-steps"></a>Дальнейшие действия

* [Использование параметров командной строки и команд PowerShell для управления эмулятором](emulator-command-line-parameters.md)
* [Отладка проблем с эмулятором](troubleshoot-local-emulator.md)
