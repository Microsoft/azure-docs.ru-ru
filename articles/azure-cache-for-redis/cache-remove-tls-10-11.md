---
title: Удаление поддержки TLS 1.0 и 1.1 при работе с Кэшем Azure для Redis
description: Узнайте, как удалить TLS 1.0 и 1.1 из своего приложения при взаимодействии с Кэшем Azure для Redis
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 10/22/2019
ms.author: yegu
ms.openlocfilehash: fd0e6f893d152259c46ff06e9ec20af54395c5e6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95994389"
---
# <a name="remove-tls-10-and-11-from-use-with-azure-cache-for-redis"></a>Удаление поддержки TLS 1.0 и 1.1 при работе с Кэшем Azure для Redis

В рамках всей отрасли существует стойкая тенденция к использованию исключительно протокола TLS 1.2 и более поздних версий. Известно, что протоколы TLS версий 1.0 и 1.1 подвержены атакам, таким как BEAST и POODLE, и имеют другие общеизвестные уязвимости (CVE). Кроме того, они не поддерживают современные методы шифрования и наборы шифров, рекомендуемые стандартами платежных карт (PCI). В [блоге по безопасности TLS](https://www.acunetix.com/blog/articles/tls-vulnerabilities-attacks-final-part/) подробно рассматриваются некоторые из этих уязвимостей.

В рамках этих усилий мы планируем внести следующие изменения в Кэш Azure для Redis.

* **Этап 1.** Мы настроим в качестве минимальной версии по умолчанию для протокола TLS значение 1,2 для вновь создаваемых экземпляров кэша (ранее это был TLS 1,0). На этом этапе существующие экземпляры кэша затронуты не будут. Вы по-прежнему можете использовать портал Azure или другие API управления, чтобы [изменить минимальную версию TLS](cache-configure.md#access-ports) на 1,0 или 1,1 для обеспечения обратной совместимости, если это необходимо.
* **Этап 2.** Мы будем прекращать поддержку TLS 1,1 и TLS 1,0. После этого изменения приложение должно использовать TLS 1,2 или более поздней версии для обмена данными с кэшем. Ожидается, что кэш Azure для службы Redis будет доступен, пока мы выполним перенос для поддержки только TLS 1,2 или более поздней версии.

  > [!NOTE]
  > Этап 2 предварительно запланирован на начало не раньше 31 декабря 2020. Однако настоятельно рекомендуется начать планирование этого изменения сейчас и заблаговременно обновлять клиенты для поддержки TLS 1,2 или более поздней версии. 
  >

В рамках этого изменения мы также удалим поддержку старых наборов шифр, которые не являются безопасными. Наши Поддерживаемые наборы шифр будут ограничены следующими наборами, если для кэша настроено минимальное количество TLS 1,2:

* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384_P384
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256_P256

В этой статье содержатся общие рекомендации по обнаружению зависимостей от предыдущих версий TLS и их удаление из приложения.

Даты вступления этих изменений в силу.

| Cloud                | Начало этапа 1 | Начало этапа 2         |
|----------------------|--------------------|----------------------------|
| Azure (глобальная)       |  13 января 2020 г.  | Отложено из-за КОВИД-19  |
| Azure для государственных организаций     |  13 марта 2020 г.    | Отложено из-за КОВИД-19  |
| Azure для Германии        |  13 марта 2020 г.    | Отложено из-за КОВИД-19  |
| Azure China 21Vianet |  13 марта 2020 г.    | Отложено из-за КОВИД-19  |

> [!NOTE]
> Этап 2 предварительно запланирован на начало не раньше 31 декабря 2020. Эта статья будет обновлена, если заданы определенные даты.
>

## <a name="check-whether-your-application-is-already-compliant"></a>Проверка приложений на соответствие требованиям

Самый простой способ узнать, будет ли приложение работать с TLS 1,2, — установить значение **минимальной версии TLS** для TLS 1,2 в тестовом или промежуточном кэше, а затем выполнить тесты. Параметр **Минимальная версия TLS** находится в [дополнительных параметрах](cache-configure.md#advanced-settings) экземпляра кэша на портале Azure.  Если после этого изменения приложение продолжит работать должным образом, оно, вероятно, соответствует требованиям. Возможно, потребуется настроить клиентскую библиотеку Redis, используемую приложением, чтобы включить TLS 1,2 для подключения к кэшу Azure для Redis.

## <a name="configure-your-application-to-use-tls-12"></a>Настройка приложения для использования TLS 1.2

Большинство приложений используют клиентские библиотеки Redis для управления связью со своим кэшем. Ниже приведены инструкции по настройке некоторых популярных клиентских библиотек, в различных языках программирования и платформах, для использования TLS 1.2.

### <a name="net-framework"></a>.NET Framework

Клиенты Redis .NET в .NET Framework 4.5.2 и более ранних версиях по умолчанию используют самую раннюю версию TLS, а в .NET Framework 4.6 и более поздних версиях используют последнюю версию TLS. Если вы используете старую версию .NET Framework, то TLS 1.2 можно включить вручную.

* **StackExchange.Redis:** задайте `ssl=true` и `sslprotocols=tls12` в строке подключения.
* **ServiceStack.Redis:** следуйте инструкциям в [ServiceStack. Redis](https://github.com/ServiceStack/ServiceStack.Redis#servicestackredis-ssl-support), требуется ServiceStack.Redis версии не ниже 5.6.

### <a name="net-core"></a>.NET Core

Клиенты Redis .NET Core по умолчанию имеют версию протокола TLS, установленную по умолчанию в ОС, которая, очевидно, зависит от самой ОС. 

В зависимости от версии ОС и всех примененных исправлений, действующая версия TLS по умолчанию может отличаться. Хотя существует один источник сведений об этом, [ниже](/dotnet/framework/network-programming/tls#support-for-tls-12) приведена статья для Windows. 

Однако если вы используете старую ОС или просто хотите убедиться, рекомендуется вручную настроить предпочтительную версию TLS с помощью клиента.


### <a name="java"></a>Java

Клиенты Redis Java используют протокол TLS 1.0 в Java 6 и более ранних версиях. Jedis, Lettuce и Redisson не смогут подключиться к кэшу Azure для Redis, если в нем отключен TLS 1.0. Обновите платформу Java, чтобы использовать новые версии TLS.

Для Java 7 клиенты Redis не используют TLS 1.2 по умолчанию, но могут быть настроены для этого. Jedis позволяет указать базовые параметры TLS с помощью следующего фрагмента кода.

``` Java
SSLSocketFactory sslSocketFactory = (SSLSocketFactory) SSLSocketFactory.getDefault();
SSLParameters sslParameters = new SSLParameters();
sslParameters.setEndpointIdentificationAlgorithm("HTTPS");
sslParameters.setProtocols(new String[]{"TLSv1.2"});
 
URI uri = URI.create("rediss://host:port");
JedisShardInfo shardInfo = new JedisShardInfo(uri, sslSocketFactory, sslParameters, null);
 
shardInfo.setPassword("cachePassword");
 
Jedis jedis = new Jedis(shardInfo);
```

Клиенты Lettuce и Redisson пока не поддерживают указание версии TLS, поэтому они будут прекращать работу, если кэш принимает только подключения TLS 1.2. Исправления для этих клиентов рассматриваются, поэтому проверяйте обновления версий этих пакетов на наличие такой поддержки.

В Java 8 протокол TLS 1.2 используется по умолчанию, и в большинстве случаев обновления конфигурации клиента не требуется. На всякий случай протестируйте свое приложение.

### <a name="nodejs"></a>Node.js

Node Redis и IORedis по умолчанию используют TLS 1.2.

### <a name="php"></a>PHP

#### <a name="predis"></a>Predis
 
* Версии PHP ниже 7. Predis поддерживает только TLS 1.0. Эти версии не работают с TLS 1.2, для использования TLS 1.2 необходимо выполнить обновление.
 
* PHP версий от 7.0 до 7.2.1. По умолчанию Predis использует только TLS 1.0 или 1.1. Для использования TLS 1.2 можно использовать следующий обходной путь. При создании экземпляра клиента укажите TLS 1.2.

  ``` PHP
  $redis=newPredis\Client([
      'scheme'=>'tls',
      'host'=>'host',
      'port'=>6380,
      'password'=>'password',
      'ssl'=>[
          'crypto_type'=>STREAM_CRYPTO_METHOD_TLSv1_2_CLIENT,
      ],
  ]);
  ```

* PHP 7.3 и более поздние версии. Predis использует последнюю версию TLS.

#### <a name="phpredis"></a>PhpRedis

PhpRedis не поддерживает TLS ни в какой версии PHP.

### <a name="python"></a>Python

Redis-py по умолчанию использует TLS 1.2.

### <a name="go"></a>GO

Redigo по умолчанию использует TLS 1.2.

## <a name="additional-information"></a>Дополнительные сведения

- [Настройка кэша Azure для Redis](cache-configure.md)