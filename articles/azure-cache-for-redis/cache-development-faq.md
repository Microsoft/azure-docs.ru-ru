---
title: Часто задаваемые вопросы по разработке кэша Azure для Redis
description: Ознакомьтесь с ответами на часто задаваемые вопросы, которые помогут вам в разработке кэша Azure для Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 08/06/2020
ms.openlocfilehash: bafd8a9752d2587ec52fe586e442e3bfc86d7537
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97585774"
---
# <a name="azure-cache-for-redis-development-faqs"></a>Часто задаваемые вопросы по разработке кэша Azure для Redis

В этой статье содержатся ответы на часто задаваемые вопросы о разработке для кэша Azure для Redis.

## <a name="common-questions-and-answers"></a>Частые вопросы и ответы
В этом разделе рассматриваются следующие вопросы и ответы:

* [Как мне начать работу с кэшем Azure для Redis?](#how-can-i-get-started-with-azure-cache-for-redis)
* [Что делают параметры конфигурации StackExchange.Redis?](#what-do-the-stackexchangeredis-configuration-options-do)
* [Каких клиентов кэша Azure для Redis я могу использовать?](#what-azure-cache-for-redis-clients-can-i-use)
* [Существует ли локальный эмулятор кэша Azure для Redis?](#is-there-a-local-emulator-for-azure-cache-for-redis)
* [Как выполнять команды Redis?](#how-can-i-run-redis-commands)
* [Почему в кэше Azure для Redis отсутствует ссылка на библиотеку классов MSDN?](#why-doesnt-azure-cache-for-redis-have-an-msdn-class-library-reference)
* [Можно ли использовать кэш Azure для Redis как кэш сеанса PHP?](#can-i-use-azure-cache-for-redis-as-a-php-session-cache)
* [Что такое базы данных Redis?](#what-are-redis-databases)

### <a name="how-can-i-get-started-with-azure-cache-for-redis"></a>Как мне начать работу с кэшем Azure для Redis?
Существует несколько способов приступить к работе с кэшем Azure для Redis.

* Вы можете ознакомиться с одним из наших руководств, доступных для [.NET](cache-dotnet-how-to-use-azure-redis-cache.md), [ASP.NET](cache-web-app-howto.md), [Java](cache-java-get-started.md), [Node.js](cache-nodejs-get-started.md) и [Python](cache-python-get-started.md).
* Вы можете посмотреть видео [Как создавать высокопроизводительные приложения с помощью кэша Microsoft Azure для Redis](https://azure.microsoft.com/documentation/videos/how-to-build-high-performance-apps-using-microsoft-azure-cache/).
* Можно ознакомиться с документацией по клиентам, которые поддерживают язык, используемый при разработке проекта, чтобы узнать, как использовать Redis. Существует множество клиентов Redis, которые можно использовать с кэшем Azure для Redis. Список клиентов Redis доступен на странице [https://redis.io/clients](https://redis.io/clients).

Если у вас нет учетной записи Azure, то вы можете сделать следующее.

* [Открыть бесплатную учетную запись Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=redis_cache_hero). Вы получаете кредиты, которые можно использовать, чтобы попробовать платные службы Azure. После израсходования кредитов ваша учетная запись не исчезнет. Вы сможете использовать ее для работы с бесплатными службами и функциями Azure.
* [Активировать преимущества подписчика Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=redis_cache_hero). Ваша подписка MSDN каждый месяц приносит вам кредиты, которые можно использовать для оплаты использования служб Azure.

### <a name="what-do-the-stackexchangeredis-configuration-options-do"></a>Что делают параметры конфигурации StackExchange.Redis?
StackExchange.Redis имеет много параметров. В этом разделе рассказывается о некоторых распространенных параметрах. Более подробные сведения о параметрах StackExchange.Redis см. в статье [StackExchange.Redis configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) (Конфигурация StackExchange.Redis).

| Параметры конфигурации | Описание | Рекомендация |
| --- | --- | --- |
| AbortOnConnectFail |Если задано значение true, соединение не будет восстанавливаться после сбоя сети. |Установите значение false и позвольте StackExchange.Redis автоматически восстанавливать соединение. |
| ConnectRetry |Количество повторных попыток подключения во время первоначального подключения. |Руководствуйтесь следующими примечаниями. |
| ConnectTimeout |Время ожидания в миллисекундах для операций подключения. |Руководствуйтесь следующими примечаниями. |

Обычно подходят значения по умолчанию для клиента. Вы можете точнее настроить параметры в зависимости от своей рабочей нагрузки.

* **Повторы**
  * Общая рекомендация для ConnectRetry и ConnectTimeout — завершение работы при первой ошибке и повторение попытки. Эта рекомендация основана на вашей рабочей нагрузке и на том, сколько времени в среднем занимает в вашем клиенте выдача команды Redis и получение ответа.
  * Вам не нужно проверять состояние и подключаться самостоятельно — StackExchange.Redis переподключается автоматически. **Избегайте использования свойства ConnectionMultiplexer.IsConnected**.
  * Снежный ком — иногда возникает проблема, когда вы повторяете снова и снова, это нарастает и никогда не исправляется. При возникновении снежного кома следует использовать алгоритм экспоненциальной отсрочки повтора, как описано в [общих рекомендациях по повторным попыткам](/azure/architecture/best-practices/transient-faults), опубликованных группой Microsoft Patterns & Practices.
  
* **Значения времени ожидания**
  * Исследуйте вашу рабочую нагрузку и установите соответствующие значения. Если вы храните большие значения, установите более высокое значение времени ожидания.
  * Установите для `AbortOnConnectFail` значение false и позвольте StackExchange.Redis восстанавливать подключение.
  * Используйте один экземпляр ConnectionMultiplexer для приложения. Вы можете применять LazyConnection, чтобы создать один экземпляр, который возвращается свойством Connection, как показано в разделе [Подключение к кэшу с использованием класса ConnectionMultiplexer](cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-the-cache).
  * Задайте в свойстве `ConnectionMultiplexer.ClientName` уникальное имя экземпляра приложения в целях диагностики.
  * Используйте несколько экземпляров `ConnectionMultiplexer` для пользовательских рабочих нагрузок.
    * Вы можете следовать этой модели, если в приложении имеется меняющаяся нагрузка. Пример:
    * можно иметь один мультиплексор для работы с большими ключами;
    * можно иметь один мультиплексор для работы с небольшими ключами;
    * можно задать разные значения времени ожидания подключения и логики повторов для каждого используемого ConnectionMultiplexer.
    * Установите свойство `ClientName` в каждом мультиплексоре для облегчения диагностики.
    * Эта рекомендация может обеспечить более оптимизированную задержку на каждый класс `ConnectionMultiplexer`.

### <a name="what-azure-cache-for-redis-clients-can-i-use"></a>Каких клиентов кэша Azure для Redis я могу использовать?
Одним из основных преимуществ Redis является то, что существует множество клиентов, поддерживающих множество языков разработки. Текущий список клиентов Redis см. [здесь](https://redis.io/clients). Дополнительные сведения см. в статье [Краткое руководство. Использование кэша Azure для Redis с приложениями .NET](cache-dotnet-how-to-use-azure-redis-cache.md). Похожие статьи вы найдете в таблице содержимого.

[!INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

### <a name="is-there-a-local-emulator-for-azure-cache-for-redis"></a>Существует ли локальный эмулятор кэша Azure для Redis?
Локального эмулятора кэша Azure для Redis нет, но можно запустить версию MSOpenTech redis-server.exe из [программ командной строки Redis](https://github.com/MSOpenTech/redis/releases/) на локальном компьютере и подключиться к нему, чтобы получить возможности, аналогичные локальному эмулятору кэша, как показано в следующем примере:

```csharp
private static Lazy<ConnectionMultiplexer>
    lazyConnection = new Lazy<ConnectionMultiplexer> (() =>
    {
        // Connect to a locally running instance of Redis to simulate
        // a local cache emulator experience.
        return ConnectionMultiplexer.Connect("127.0.0.1:6379");
    });

public static ConnectionMultiplexer Connection
{
    get
    {
        return lazyConnection.Value;
    }
}
```

При необходимости можно настроить файл [redis.conf](https://redis.io/topics/config) для более точного соответствия [параметрам кэша по умолчанию](cache-configure.md#default-redis-server-configuration) для подключенного к сети кэша Azure для Redis.

### <a name="how-can-i-run-redis-commands"></a>Как выполнять команды Redis?
Вы можете использовать любую из команд, перечисленных в [командах Redis](https://redis.io/commands#), за исключением команд, указанных в перечне [команд, не поддерживаемых в кэше Azure для Redis](cache-configure.md#redis-commands-not-supported-in-azure-cache-for-redis). Выполнять команды Redis можно несколькими способами.

* При наличии кэша уровня "Стандартный" или "Премиум" вы можете запускать команды Redis с помощью [консоли Redis](cache-configure.md#redis-console). Консоль Redis — это защищенный способ выполнения команд Redis на портале Azure.
* Также можно использовать программы командной строки Redis. Чтобы воспользоваться ими, выполните следующие действия:
* Скачайте [программы командной строки Redis](https://github.com/MSOpenTech/redis/releases/).
* Подключитесь к кэшу с помощью `redis-cli.exe`. Передайте конечную точку кэша, используя параметр -h и ключ с параметром -a, как показано в следующем примере:
* `redis-cli -h <Azure Cache for Redis name>.redis.cache.windows.net -a <key>`

> [!NOTE]
> Программы командной строки Redis не работают с портом TLS, но можно использовать служебную программу, такую как `stunnel`, для безопасного подключения этих программ к порту TLS в соответствии с указаниями в записи блога [Использование инструмента командной строки Redis с Кэшем Azure для Redis](./cache-how-to-redis-cli-tool.md).
>
>

### <a name="why-doesnt-azure-cache-for-redis-have-an-msdn-class-library-reference"></a>Почему в кэше Azure для Redis отсутствует ссылка на библиотеку классов MSDN?
Microsoft Azure кэш для Redis основан на популярном хранилище данных в памяти с открытым исходным кодом, Redis. Доступ к нему можно получить с помощью широкого спектра [клиентов Redis](https://redis.io/clients) для многих языков программирования. Каждый клиент имеет собственный API, который вызывает экземпляр кэша Azure для Redis с помощью [команд Redis](https://redis.io/commands).

Поскольку все клиенты разные, отсутствует одна централизованная ссылка на классы в MSDN, а каждый клиент поддерживает свою собственную справочную документацию. Помимо справочной документации имеется несколько учебников, показывающих, как приступить к работе с кэшем Azure для Redis, используя разные языки и клиенты кэша. Чтобы получить доступ к этим руководствам, см. в статью [Краткое руководство. Использование кэша Azure для Redis с приложениями .NET](cache-dotnet-how-to-use-azure-redis-cache.md). Похожие статьи вы найдете в таблице содержимого.

### <a name="can-i-use-azure-cache-for-redis-as-a-php-session-cache"></a>Можно ли использовать кэш Azure для Redis как кэш сеанса PHP?
Да. Для использования кэша Azure для Redis в качестве кэша сеанса PHP укажите строку подключения к экземпляру кэша Azure для Redis в пути `session.save_path`.

> [!IMPORTANT]
> При использовании кэша Azure для Redis в качестве кэша сеанса PHP необходимо кодировать в URL-адресе ключ безопасности, используемый для подключения к кэшу, как показано в следующем примере:
>
> `session.save_path = "tcp://mycache.redis.cache.windows.net:6379?auth=<url encoded primary or secondary key here>";`
>
> Если ключ не кодирован в URL-адресе, то может отобразиться сообщение об исключении, подобное этому: `Failed to parse session.save_path`
>

Дополнительные сведения об использовании кэша Azure для Redis в качестве кэша сеанса PHP с клиентом PhpRedis см. в разделе [Обработчик сеанса PHP](https://github.com/phpredis/phpredis#php-session-handler).

### <a name="what-are-redis-databases"></a>Что такое базы данных Redis?

Базы данных Redis — это просто логическое разделение данных внутри одного экземпляра Redis. Кэш-память совместно используется всеми базами данных, и фактический объем памяти, используемый определенной базой данных, зависит от пар "ключ-значение", хранящихся в ней. Например, объем кэша C6 составляет 53 ГБ, а P5 — 120 ГБ. Вы можете предоставить все 53 или 120 ГБ одной базе данных или разделить их между несколькими базами данных. 

> [!NOTE]
> При использовании кэша Azure для Redis уровня "Премиум" с включенной кластеризацией доступна только база данных 0. Это внутреннее ограничение Redis, и оно не относится к кэшу Azure для Redis. Дополнительные сведения см. в разделе [необходимость внесения изменений в клиентское приложение для использования кластеризации](cache-how-to-premium-clustering.md#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering).
> 
> 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о другом [кэше Azure для Redis часто задаваемых вопросов](cache-faq.md).