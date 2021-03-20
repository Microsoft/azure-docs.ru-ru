---
title: Поставщик кэша вывода ASP.NET для кэша Azure для Redis
description: Узнайте, как кэшировать выходные данные страницы ASP.NET с помощью кэша Azure для Redis. Поставщик кэша вывода Redis представляет собой механизм внепроцессного хранения для выходных данных кэширования.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.date: 04/22/2018
ms.openlocfilehash: 6d711b07a10e04dcdf31259f3e53c9687af28e28
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95993405"
---
# <a name="aspnet-output-cache-provider-for-azure-cache-for-redis"></a>Поставщик кэша вывода ASP.NET для кэша Azure для Redis

Поставщик кэша вывода Redis представляет собой механизм внепроцессного хранения для выходных данных кэширования. Эти данные предназначены специально для полных HTTP-ответов (кэширование вывода страниц). Поставщик подключается к новой точке расширения поставщика вывода кэша, которая появилась в ASP.NET 4. Для ASP.NET Core приложений [кэширование ответов считывается в ASP.NET Core](/aspnet/core/performance/caching/response). 

Чтобы использовать поставщик кэша вывода Redis, сначала настройте кэш, а затем приложение ASP.NET, используя пакет NuGet поставщика кэша вывода Redis. Этот раздел содержит информацию о том, как настроить приложение, чтобы оно использовало поставщик кэша вывода Redis. Дополнительные сведения о создании и настройке экземпляра кэша Azure для Redis см. в разделе [Создание кэша](cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).

## <a name="store-aspnet-page-output-in-the-cache"></a>Сохранение выходных данных страницы ASP.NET в кэше

Чтобы настроить клиентское приложение в Visual Studio, используя пакет NuGet состояний сеансов кэша Azure для Redis, в меню **Сервис** выберите **Диспетчер пакетов NuGet**, а затем **Консоль диспетчера пакетов**.

Выполните следующую команду в окне `Package Manager Console`:

```
Install-Package Microsoft.Web.RedisOutputCacheProvider
```

Пакет NuGet поставщика кэша вывода Redis имеет зависимость от пакета StackExchange.Redis.StrongName. Если в проекте отсутствует пакет StackExchange.Redis.StrongName, то он будет установлен. Дополнительные сведения о пакете NuGet RedisOutputCacheProvider см. на странице NuGet [RedisOutputCacheProvider](https://www.nuget.org/packages/Microsoft.Web.RedisOutputCacheProvider/).

>[!NOTE]
>Помимо пакета со строгим именем StackExchange.Redis.StrongName существует также версия с нестрогим именем StackExchange.Redis. Если проект использует версию StackExchange. Redis без строгого имени, ее необходимо удалить. в противном случае в проекте будут возникать конфликты имен. Дополнительные сведения об этих пакетах см. в разделе [Настройка клиентов кэша .NET](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients).

Пакет NuGet скачивает и добавляет требуемые ссылки на сборку и добавляет следующий раздел в файл web.config. Этот раздел содержит необходимые настройки для приложения ASP.NET, позволяющие использовать поставщик кэша вывода Redis.

```xml
<caching>
  <outputCache defaultProvider="MyRedisOutputCache">
    <providers>
      <add name="MyRedisOutputCache" type="Microsoft.Web.Redis.RedisOutputCacheProvider"
           host=""
           accessKey=""
           ssl="true" />
    </providers>
  </outputCache>
</caching>
```

Задайте для атрибутов значения из колонки своего кэша на портале Microsoft Azure и настройте другие параметры по своему усмотрению. Инструкции по доступу к свойствам кэша см. в разделе [Настройка параметров кэша Azure для Redis](cache-configure.md#configure-azure-cache-for-redis-settings).

| attribute | Тип | По умолчанию | Описание |
| --------- | ---- | ------- | ----------- |
| *хост* | строка | localhost | IP-адрес или имя узла сервера Redis |
| *port* | положительное целое число | 6379 (без TLS/SSL)<br/>6380 (TLS/SSL) | Порт сервера Redis |
| *accessKey* | строка | "" | Пароль сервера Redis, если включена авторизация Redis. Значение по умолчанию — пустая строка, что означает, что поставщик состояний сеанса не будет использовать пароль при подключении к серверу Redis. **Если сервер Redis находится в общедоступной сети, такой как кэш Redis для Azure, обязательно включите авторизацию Redis для повышения безопасности и обеспечьте безопасный пароль.** |
| *технологию* | Логическое | **false** | Следует ли подключаться к серверу Redis по протоколу TLS. По умолчанию это значение равно **false** , так как Redis не поддерживает TLS из поля. **Если вы используете кэш Redis для Azure, который поддерживает протокол SSL, установите значение true для повышения безопасности.**<br/><br/>Порт, отличный от TLS, по умолчанию отключен для новых кэшей. Укажите **значение true** для этого параметра, чтобы использовать порт, отличный от TLS. Дополнительные сведения о включении порта без TLS см. в разделе [порты доступа](cache-configure.md#access-ports) статьи [Настройка кэша](cache-configure.md) . |
| *датабасеиднумбер* | положительное целое число | 0 | *Этот атрибут можно указать только с помощью web.config или AppSettings.*<br/><br/>Укажите, какую базу данных Redis использовать. |
| *коннектионтимеаутинмиллисекондс* | положительное целое число | Предоставлено StackExchange. Redis | Используется для задания *ConnectTimeout* при создании StackExchange. Redis. ConnectionMultiplexer. |
| *оператионтимеаутинмиллисекондс* | положительное целое число | Предоставлено StackExchange. Redis | Используется для задания *SyncTimeout* при создании StackExchange. Redis. ConnectionMultiplexer. |
| *ConnectionString* (допустимая строка подключения StackExchange. Redis) | строка | *н/д* | Либо ссылка на параметр AppSettings, либо web.config, либо, в противном случае, допустимая строка подключения StackExchange. Redis. Этот атрибут может предоставлять значения для *узлов*, *портов*, *AccessKey*, *SSL* и других атрибутов StackExchange. Redis. Более подробные сведения о *ConnectionString* см. в разделе [Настройка ConnectionString](#setting-connectionstring) в [заметках атрибута](#attribute-notes) . |
| *сеттингскласснаме*<br/>*сеттингсмесоднаме* | строка<br/>строка | *н/д* | *Эти атрибуты можно указать только с помощью web.config или AppSettings.*<br/><br/>Используйте эти атрибуты для предоставления строки подключения. *сеттингскласснаме* должно быть полным именем класса сборки, содержащим метод, заданный параметром *сеттингсмесоднаме*.<br/><br/>Метод, заданный параметром *сеттингсмесоднаме* , должен быть открытым, статическим и void (не принимать параметры) с типом возвращаемого значения **String**. Этот метод возвращает фактическую строку подключения. |
| *логгингкласснаме*<br/>*логгингмесоднаме* | строка<br/>строка | *н/д* | *Эти атрибуты можно указать только с помощью web.config или AppSettings.*<br/><br/>Используйте эти атрибуты для отладки приложения, предоставив журналы из состояния сеанса/кэша вывода вместе с журналами из StackExchange. Redis. *логгингкласснаме* должно быть полным именем класса сборки, содержащим метод, заданный параметром *логгингмесоднаме*.<br/><br/>Метод, заданный параметром *логгингмесоднаме* , должен быть открытым, статическим и void (не принимать параметры) с типом возвращаемого значения **System. IO. TextWriter**. |
| *applicationName* | строка | Имя модуля текущего процесса или "/" | *Только Сессионстатепровидер*<br/>*Этот атрибут можно указать только с помощью web.config или AppSettings.*<br/><br/>Префикс имени приложения для использования в кэше Redis. Клиент может использовать один кэш Redis для разных целей. Чтобы убедиться, что ключи сеанса не противоречат друг другу, можно указать имя приложения с префиксом. |
| *throwOnError* | Логическое | Да | *Только Сессионстатепровидер*<br/>*Этот атрибут можно указать только с помощью web.config или AppSettings.*<br/><br/>Следует ли создавать исключение при возникновении ошибки.<br/><br/>Дополнительные сведения о *throwOnError* см. в разделе [Примечания к *throwOnError*](#notes-on-throwonerror) в статье [Примечания к атрибутам](#attribute-notes) . |>*Microsoft. Web. Redis. RedisSessionStateProvider. ластексцептион*. |
| *ретритимеаутинмиллисекондс* | положительное целое число | 5000 | *Только Сессионстатепровидер*<br/>*Этот атрибут можно указать только с помощью web.config или AppSettings.*<br/><br/>Время повтора при сбое операции. Если это значение меньше *оператионтимеаутинмиллисекондс*, поставщик не будет повторять попытки.<br/><br/>Дополнительные сведения о *ретритимеаутинмиллисекондс* см. в разделе [Примечания к *ретритимеаутинмиллисекондс*](#notes-on-retrytimeoutinmilliseconds) в статье [Примечания к атрибутам](#attribute-notes) . |
| *редиссериализертипе* | строка | *н/д* | Указывает полное имя типа сборки класса, реализующего Microsoft. Web. Redis. Исериализер и, содержащий пользовательскую логику для сериализации и десериализации значений. Дополнительные сведения см. в разделе [About *Редиссериализертипе*](#about-redisserializertype) статьи [Attribute Notes](#attribute-notes) . |

## <a name="attribute-notes"></a>Примечания к атрибутам

### <a name="setting-connectionstring"></a>Настройка *ConnectionString*

Значение *ConnectionString* используется в качестве ключа для выборки фактической строки подключения из appSettings, если такая строка существует в appSettings. Если в AppSettings не найдено, значение *ConnectionString* будет использоваться в качестве ключа для выборки действительной строки подключения из раздела web.config **ConnectionString** , если этот раздел существует. Если строка подключения не существует в параметре AppSettings или в разделе web.config **ConnectionString** , литеральное значение *ConnectionString* будет использоваться в качестве строки подключения при создании StackExchange. Redis. ConnectionMultiplexer.

В следующих примерах показано, как используется *ConnectionString* .

#### <a name="example-1"></a>Пример 1

```xml
<connectionStrings>
    <add name="MyRedisConnectionString" connectionString="mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False" />
</connectionStrings>
```

В `web.config` используйте ключ выше в качестве значения параметра вместо фактического значения.

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "MyRedisConnectionString"/>
    </providers>
</sessionState>
```

#### <a name="example-2"></a>Пример 2

```xml
<appSettings>
    <add key="MyRedisConnectionString" value="mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False" />
</appSettings>
```

В `web.config` используйте ключ выше в качестве значения параметра вместо фактического значения.

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "MyRedisConnectionString"/>
    </providers>
</sessionState>
```

#### <a name="example-3"></a>Пример 3

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False"/>
    </providers>
</sessionState>
```

### <a name="notes-on-throwonerror"></a>Примечания о *throwOnError*

В настоящее время при возникновении ошибки во время операции сеанса поставщик состояния сеанса вызывает исключение. Это завершает работу приложения.

Это поведение было изменено способом, который поддерживает ожидания существующих пользователей поставщика состояний сеансов ASP.NET, а также предоставляет возможность действовать с исключениями при необходимости. Поведение по умолчанию по-прежнему вызывает исключение при возникновении ошибки, которое согласуется с другими поставщиками состояний сеансов ASP.NET; существующий код должен работать так же, как и раньше.

Если задать для *throwOnError* **значение false**, то вместо создания исключения при возникновении ошибки произойдет сбой. Чтобы проверить, произошла ли ошибка, и, если да, выяснить, что это за исключением, проверьте статическое свойство *Microsoft. Web. Redis. RedisSessionStateProvider. ластексцептион*.

### <a name="notes-on-retrytimeoutinmilliseconds"></a>Примечания о *ретритимеаутинмиллисекондс*

Это предоставляет некоторую логику повторных попыток, чтобы упростить выполнение некоторой операции сеанса в случае сбоя из-за неисправности в сети, а также позволяет управлять временем ожидания повторных попыток или отказаться от повторных попыток.

Если задать для *ретритимеаутинмиллисекондс* число, например 2000, то при сбое операции сеанса она повторит попытку в течение 2000 миллисекунд, прежде чем рассматривать ее как ошибку. Чтобы поставщик состояний сеанса применял эту логику повторных попыток, просто настройте время ожидания. Первая повторная попытка произойдет через 20 миллисекунд, что вполне достаточно в большинстве случаев при возникновении сбоя сети. После этого он будет повторять каждую секунду, пока не истечет время ожидания. Сразу после истечения времени ожидания будет повторяться еще одно время, чтобы убедиться в том, что время ожидания не обрезается (не более одного секунды).

Если вы не считаете, что вам требуется повторная попытка (например, при запуске сервера Redis на том же компьютере, что и приложение), или если вы хотите самостоятельно реализовать логику повторных попыток, присвойте параметру *ретритимеаутинмиллисекондс* значение 0.

### <a name="about-redisserializertype"></a>О *редиссериализертипе*

По умолчанию сериализация для хранения значений в Redis выполняется в двоичном формате, предоставляемом классом **BinaryFormatter** . Используйте *редиссериализертипе* , чтобы указать полное имя типа сборки класса, реализующего **Microsoft. Web. Redis. исериализер** и имеющего пользовательскую логику для сериализации и десериализации значений. Например, ниже приведен класс сериализатора JSON с помощью JSON.NET:

```cs
namespace MyCompany.Redis
{
    public class JsonSerializer : ISerializer
    {
        private static JsonSerializerSettings _settings = new JsonSerializerSettings() { TypeNameHandling = TypeNameHandling.All };

        public byte[] Serialize(object data)
        {
            return Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(data, _settings));
        }

        public object Deserialize(byte[] data)
        {
            if (data == null)
            {
                return null;
            }
            return JsonConvert.DeserializeObject(Encoding.UTF8.GetString(data), _settings);
        }
    }
}
```

Если этот класс определен в сборке с именем **микомпанидлл**, можно задать параметр *редиссериализертипе* для его использования:

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvider"
             name = "MySessionStateStore"
             redisSerializerType = "MyCompany.Redis.JsonSerializer,MyCompanyDll"
             ... />
    </providers>
</sessionState>
```

## <a name="output-cache-directive"></a>Директива выходного кэша

Добавьте директиву OutputCache для каждой страницы, для которой требуется кэшировать выходные данные.

```xml
<%@ OutputCache Duration="60" VaryByParam="*" %>
```

В предыдущем примере кэшированные данные страницы остаются в кэше в течение 60 секунд, а для каждой комбинации параметров кэшируется другая версия страницы. Дополнительные сведения об директиве OutputCache см. в разделе [@OutputCache](/previous-versions/dotnet/netframework-4.0/hdxfb6cy(v=vs.100)) .

Когда эти действия будут выполнены, приложение будет соответствующим образом настроено и сможет использовать поставщик кэша вывода Redis.

## <a name="third-party-output-cache-providers"></a>Сторонние поставщики кэша вывода

* [NCache](https://www.alachisoft.com/blogs/how-to-use-a-distributed-cache-for-asp-net-output-cache/)
* [Apache Ignite](https://apacheignite-net.readme.io/docs/aspnet-output-caching)


## <a name="next-steps"></a>Дальнейшие действия

Посетите страницу [ASP.NET Session State Provider for Azure Cache for Redis](cache-aspnet-session-state-provider.md) (Поставщик состояний сеансов ASP.NET для кэша Azure для Redis).