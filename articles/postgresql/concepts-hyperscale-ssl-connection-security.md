---
title: Безопасность транспортного уровня (TLS) — масштабирование (Цитус) — база данных Azure для PostgreSQL
description: Инструкции и сведения о настройке базы данных Azure для PostgreSQL-Scale (Цитус) и связанных приложений для правильного использования TLS-подключений.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 07/16/2020
ms.openlocfilehash: 659f86a107e4b08db4ec5195635ea32d2260d677
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87071455"
---
# <a name="configure-tls-in-azure-database-for-postgresql---hyperscale-citus"></a>Настройка TLS в базе данных Azure для PostgreSQL — масштабирование (Цитус)
Для узла координатора Цитус требуется, чтобы клиентские приложения подключались с помощью протокола TLS. Применение протокола TLS между сервером базы данных и клиентскими приложениями помогает обеспечивать конфиденциальность данных при передаче. Дополнительные параметры проверки, описанные ниже, также защищают от атак "злоумышленник в середине".

## <a name="enforcing-tls-connections"></a>Принудительное подключение TLS
Приложения используют "строку подключения" для задания целевой базы данных и параметров соединения. Для разных клиентов требуются разные параметры. Чтобы просмотреть список строк подключения, используемых общими клиентами, просмотрите раздел **строки подключения** для группы серверов в портал Azure.

Параметры TLS `ssl` и `sslmode` меняются в зависимости от возможностей соединителя, например `ssl=true` или `sslmode=require` или `sslmode=required` .

## <a name="ensure-your-application-or-framework-supports-tls-connections"></a>Убедитесь, что приложение или платформа поддерживает подключения TLS.
Некоторые платформы приложений не включают TLS по умолчанию для подключений PostgreSQL. Однако без безопасного подключения приложение не может подключиться к узлу координатора Цитус. Чтобы узнать, как включить TLS, ознакомьтесь с документацией по приложению.

## <a name="applications-that-require-certificate-verification-for-tls-connectivity"></a>Приложения, для которых требуется проверка сертификата для подключения TLS
В некоторых случаях для безопасного подключения приложениям требуется локальный файл сертификата, созданный из файла сертификата (CER-файла) доверенного центра сертификации. Сертификат для подключения к базе данных Azure для PostgreSQL-Scale (Цитус) находится в папке https://cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem . Скачайте файл сертификата и сохраните его в предпочтительном расположении.

> [!NOTE]
>
> Чтобы проверить подлинность сертификата, можно проверить свой отпечаток SHA-256 с помощью программы командной строки OpenSSL:
>
> ```sh
> openssl x509 -in DigiCertGlobalRootCA.crt.pem -noout -sha256 -fingerprint
>
> # should output:
> # 43:48:A0:E9:44:4C:78:CB:26:5E:05:8D:5E:89:44:B4:D8:4F:96:62:BD:26:DB:25:7F:89:34:A4:43:C7:01:61
> ```

### <a name="connect-using-psql"></a>Подключение с помощью psql
В следующем примере показано, как подключиться к узлу координатора Цитус с помощью служебной программы командной строки psql. Используйте `sslmode=verify-full` параметр строки подключения для принудительной проверки сертификата TLS. Передайте путь к файлу локального сертификата `sslrootcert` параметру.

Ниже приведен пример строки подключения psql.
```
psql "sslmode=verify-full sslrootcert=DigiCertGlobalRootCA.crt.pem host=mydemoserver.postgres.database.azure.com dbname=citus user=citus password=your_pass"
```
> [!TIP]
> Убедитесь, что значение, передаваемое в, `sslrootcert` соответствует пути к файлу сохраненного сертификата.

## <a name="next-steps"></a>Дальнейшие действия
Повысьте безопасность с помощью [правил брандмауэра в базе данных Azure для PostgreSQL-Scale (Цитус)](concepts-hyperscale-firewall-rules.md).
