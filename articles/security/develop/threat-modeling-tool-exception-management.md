---
title: Управление исключениями. Средство моделирования угроз Microsoft Azure | Документация Майкрософт
description: Сведения об управлении исключениями в Threat Modeling Tool. См. сведения об устранении рисков и просмотр примеров кода.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: devx-track-csharp
ms.openlocfilehash: 41532e554623c47e9728c6ccab92d99500e42021
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94517433"
---
# <a name="security-frame-exception-management--mitigations"></a>Механизм безопасности. Управление исключениями | Устранение рисков 
| Продукт или служба | Статья |
| --------------- | ------- |
| **WCF** | <ul><li>[WCF — не добавляйте узел serviceDebug в файл конфигурации](#servicedebug)</li><li>[WCF — не добавляйте узел serviceMetadata в файл конфигурации](#servicemetadata)</li></ul> |
| **Веб-интерфейс API** | <ul><li>[Обеспечьте правильную обработку исключений в веб-API ASP.NET](#exception)</li></ul> |
| **Веб-приложение** | <ul><li>[Не раскрывайте сведения о безопасности в сообщениях об ошибках](#messages)</li><li>[Реализуйте страницу обработки ошибок по умолчанию](#default)</li><li>[Настройте метод развертывания retail в IIS](#deployment)</li><li>[Обеспечьте безопасную обработку исключений](#fail)</li></ul> |

## <a name="wcf--do-not-include-servicedebug-node-in-configuration-file"></a><a id="servicedebug"></a>WCF — не добавляйте узел serviceDebug в файл конфигурации

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | WCF | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальные, .NET Framework 3 |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_debug_information) |
| **Шаги** | Службы Windows Communication Framework (WCF) можно настроить для предоставления отладочной информации. Отладочную информацию не следует использовать в рабочих средах. Тег `<serviceDebug>` определяет, включена ли функция отладочной информации для службы WCF. Если атрибуту includeExceptionDetailInFaults присвоено значение true, клиентам будут возвращены сведения об исключении из приложения. Злоумышленники могут использовать дополнительные сведения, полученные из результатов отладки, для совершения целенаправленных атак на платформу, базу данных или другие ресурсы, используемые приложением. |

### <a name="example"></a>Пример
Следующий файл конфигурации содержит тег `<serviceDebug>`: 
```
<configuration> 
<system.serviceModel> 
<behaviors> 
<serviceBehaviors> 
<behavior name=""MyServiceBehavior""> 
<serviceDebug includeExceptionDetailInFaults=""True"" httpHelpPageEnabled=""True""/> 
... 
```
Отключите отладочную информацию в службе. Это можно сделать, удалив тег `<serviceDebug>` в файле конфигурации приложения. 

## <a name="wcf--do-not-include-servicemetadata-node-in-configuration-file"></a><a id="servicemetadata"></a>WCF — не добавляйте узел serviceMetadata в файл конфигурации

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | WCF | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Универсальные, .NET Framework 3 |
| **Справочные материалы**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_service_enumeration) |
| **Шаги** | В случае публичного раскрытия информации о службе злоумышленники могут получить ценные сведения о том, как ее можно использовать в своих целях. Тег `<serviceMetadata>` включает функцию публикации метаданных. Метаданные службы могут содержать конфиденциальные сведения, к которым нельзя предоставлять общий доступ. Как минимум доступ к метаданным следует разрешать только доверенным пользователям, а также необходимо гарантировать, что ненужная информация не предоставляется. А еще лучше полностью отключите возможность публикации метаданных. Безопасная конфигурация WCF не будет содержать тег `<serviceMetadata>`. |

## <a name="ensure-that-proper-exception-handling-is-done-in-aspnet-web-api"></a><a id="exception"></a>Обеспечьте правильную обработку исключений в веб-API ASP.NET

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-API | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | MVC 5, MVC 6 |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Обработка исключений в веб-API ASP.NET](https://www.asp.net/web-api/overview/error-handling/exception-handling), [проверка модели в веб-API ASP.NET](https://www.asp.net/web-api/overview/formats-and-model-binding/model-validation-in-aspnet-web-api) |
| **Шаги** | По умолчанию большинство неперехваченных исключений в веб-API ASP.NET преобразовываются в HTTP-ответ с кодом состояния `500, Internal Server Error`.|

### <a name="example"></a>Пример
Для управления кодом состояния, возвращенным API, можно использовать `HttpResponseException`, как показано ниже. 
```csharp
public Product GetProduct(int id)
{
    Product item = repository.Get(id);
    if (item == null)
    {
        throw new HttpResponseException(HttpStatusCode.NotFound);
    }
    return item;
}
```

### <a name="example"></a>Пример
Для дальнейшего управления ответом исключения можно использовать класс `HttpResponseMessage`, как показано ниже. 
```csharp
public Product GetProduct(int id)
{
    Product item = repository.Get(id);
    if (item == null)
    {
        var resp = new HttpResponseMessage(HttpStatusCode.NotFound)
        {
            Content = new StringContent(string.Format("No product with ID = {0}", id)),
            ReasonPhrase = "Product ID Not Found"
        }
        throw new HttpResponseException(resp);
    }
    return item;
}
```
Чтобы перехватить необработанные исключения, которые не относятся к типу `HttpResponseException`, можно использовать фильтры исключений. Они реализовывают интерфейс `System.Web.Http.Filters.IExceptionFilter`. Самый простой способ создать фильтр исключений — использовать класс `System.Web.Http.Filters.ExceptionFilterAttribute` и переопределить метод OnException. 

### <a name="example"></a>Пример
Ниже приведен фильтр, который преобразует исключения `NotImplementedException` в код состояния HTTP `501, Not Implemented`. 
```csharp
namespace ProductStore.Filters
{
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Web.Http.Filters;

    public class NotImplExceptionFilterAttribute : ExceptionFilterAttribute 
    {
        public override void OnException(HttpActionExecutedContext context)
        {
            if (context.Exception is NotImplementedException)
            {
                context.Response = new HttpResponseMessage(HttpStatusCode.NotImplemented);
            }
        }
    }
}
```

Существует несколько способов регистрации фильтра исключений веб-API:
- Для действия
- Для контроллера
- Глобально

### <a name="example"></a>Пример
Чтобы применить фильтр к определенному действию, добавьте его в качестве атрибута в действие: 
```csharp
public class ProductsController : ApiController
{
    [NotImplExceptionFilter]
    public Contact GetContact(int id)
    {
        throw new NotImplementedException("This method is not implemented");
    }
}
```
### <a name="example"></a>Пример
Чтобы применить фильтр ко всем действиям в `controller`, добавьте его в качестве атрибута в класс `controller`: 

```csharp
[NotImplExceptionFilter]
public class ProductsController : ApiController
{
    // ...
}
```

### <a name="example"></a>Пример
Чтобы применить фильтр ко всем контроллерам веб-API глобально, добавьте экземпляр фильтра в коллекцию `GlobalConfiguration.Configuration.Filters`. В этой коллекции фильтры исключений применяются к любому действию контроллера веб-API. 
```csharp
GlobalConfiguration.Configuration.Filters.Add(
    new ProductStore.NotImplExceptionFilterAttribute());
```

### <a name="example"></a>Пример
Для проверки модели можно настроить передачу состояния модели методу CreateErrorResponse, как показано ниже. 
```csharp
public HttpResponseMessage PostProduct(Product item)
{
    if (!ModelState.IsValid)
    {
        return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
    }
    // Implementation not shown...
}
```

Просмотрите ссылки в разделе справочных сведений, чтобы получить дополнительные сведения об обработке исключений и проверке модели в веб-API ASP.NET. 

## <a name="do-not-expose-security-details-in-error-messages"></a><a id="messages"></a>Не раскрывайте сведения о безопасности в сообщениях об ошибках

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | <p>Общие сообщения об ошибках предоставляются непосредственно пользователю и не содержат конфиденциальные данные приложения. К конфиденциальным данным относятся:</p><ul><li>имена серверов;</li><li>Строки подключения</li><li>имена пользователей;</li><li>паролей</li><li>процедуры SQL;</li><li>подробные сведения о динамических ошибках SQL;</li><li>трассировка стека и строки кода;</li><li>переменные, хранимые в памяти;</li><li>расположения дисков и папок;</li><li>точки установки приложения;</li><li>параметры конфигурации узла;</li><li>другие внутренние сведения о приложении.</li></ul><p>Перехват всех ошибок в приложении и предоставление общих сообщений об ошибках, а также включение пользовательских ошибок в службах IIS позволяют предотвратить раскрытие информации. Для пользователя-злоумышленника обработка исключений .NET и базы данных SQL Server, помимо других архитектур обработки ошибок, является чрезвычайно подробной и полезной при профилировании приложения. Не отображайте напрямую содержимое класса, производного от класса исключений .NET, и настройте правильную обработку исключений, чтобы непредвиденное исключение случайно не было отправлено непосредственно пользователю.</p><ul><li>Предоставляйте общие сообщения об ошибках напрямую пользователю, который абстрагирует подробные сведения, находящиеся непосредственно в сообщении об ошибке или исключении.</li><li>Не отображайте содержимое класса исключений .NET непосредственно пользователю.</li><li>Перехватывайте все сообщения об ошибках и при необходимости оповещайте пользователя с помощью общего сообщения об ошибке, отправленного клиенту приложения.</li><li>Не предоставляйте содержимое класса исключений непосредственно пользователю, особенно значение, возвращаемое `.ToString()`, а также значения свойств сообщения или трассировки стека. Безопасно записывайте эти сведения в журнал и отображайте сообщение без конфиденциальных сведений для пользователя.</li></ul>|

## <a name="implement-default-error-handling-page"></a><a id="default"></a>Реализуйте страницу обработки ошибок по умолчанию

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Сведения о диалоговом окне изменения параметров страниц ошибок ASP.NET](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd569096(v=ws.10)) |
| **Шаги** | <p>Если приложение ASP.NET перестает работать и приводит к возникновению ошибки "HTTP/1.x 500 — внутренняя ошибка сервера" или конфигурация компонентов (например, фильтрация запросов) предотвращает отображение страницы, создается сообщение об ошибке. Администраторы могут выбрать, следует ли приложению отображать понятное сообщение клиенту, подробное сообщение об ошибке клиенту или подробное сообщение только на локальном узле. Тег `<customErrors>` в файле web.config имеет три режима:</p><ul><li>**On.** Указывает, что пользовательские ошибки включены. Если атрибут defaultRedirect не указан, для пользователей отобразится общая ошибка. Пользовательские ошибки отображаются для удаленных клиентов и на локальном узле.</li><li>**Off.** Указывает, что пользовательские ошибки выключены. Подробные ошибки ASP.NET отображаются для удаленных клиентов и на локальном узле.</li><li>**RemoteOnly.** Указывает, что пользовательские ошибки отображаются только для удаленных клиентов, а также что ошибки ASP.NET отображаются на локальном узле. Это значение по умолчанию.</li></ul><p>Откройте файл `web.config` для приложения или сайта и убедитесь, что для тега определен либо режим `<customErrors mode="RemoteOnly" />`, либо `<customErrors mode="On" />`.</p>|

## <a name="set-deployment-method-to-retail-in-iis"></a><a id="deployment"></a>Настройте метод развертывания retail в IIS

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Развертывание |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Элемент deployment (схема параметров ASP.NET)](/previous-versions/dotnet/netframework-2.0/ms228298(v=vs.80)) |
| **Шаги** | <p>Параметр `<deployment retail>` предназначен для использования рабочими серверами IIS. Он используется для выполнения приложений с максимальной производительностью и минимальной утечкой сведений о безопасности. Это возможно благодаря исключению возможностей приложения создавать выходные данные трассировки на странице, отображать подробные сообщения об ошибках для конечных пользователей и отключать параметр отладки.</p><p>Зачастую параметры, предназначенные для разработчиков, например трассировка и отладка неудачно завершенных запросов, включаются во время активной разработки. Мы советуем настроить метод развертывания retail для любого рабочего сервера. Откройте файл machine.config и убедитесь, что параметру `<deployment retail="true" />` присвоено значение true.</p>|

## <a name="exceptions-should-fail-safely"></a><a id="fail"></a>Обеспечьте безопасную обработку исключений

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Безопасная обработка ошибок](https://owasp.org/www-community/Fail_securely) |
| **Шаги** | Приложение должно безопасно обрабатывать ошибки. Любой метод, возвращающий логическое значение, в зависимости от принятого решения, должен содержать тщательно созданный блок исключений. Существует множество логических ошибок, из-за которых возникают проблемы с безопасностью, например при наличии небрежно созданного блока исключений.|

### <a name="example"></a>Пример
```csharp
        public static bool ValidateDomain(string pathToValidate, Uri currentUrl)
        {
            try
            {
                if (!string.IsNullOrWhiteSpace(pathToValidate))
                {
                    var domain = RetrieveDomain(currentUrl);
                    var replyPath = new Uri(pathToValidate);
                    var replyDomain = RetrieveDomain(replyPath);

                    if (string.Compare(domain, replyDomain, StringComparison.OrdinalIgnoreCase) != 0)
                    {
                        //// Adding additional check to enable CMS urls if they are not hosted on same domain.
                        if (!string.IsNullOrWhiteSpace(Utilities.CmsBase))
                        {
                            var cmsDomain = RetrieveDomain(new Uri(Utilities.Base.Trim()));
                            if (string.Compare(cmDomain, replyDomain, StringComparison.OrdinalIgnoreCase) != 0)
                            {
                                return false;
                            }
                            else
                            {
                                return true;
                            }
                        }

                        return false;
                    }
                }

                return true;
            }
            catch (UriFormatException ex)
            {
                LogHelper.LogException("Utilities:ValidateDomain", ex);
                return true;
            }
        }
```
Если исключение произошло, приведенный выше метод всегда возвращает значение true. Если конечный пользователь предоставляет URL-адрес неправильного формата, который поддерживает браузер, но отклоняет конструктор `Uri()`, выдается исключение и пользователь перенаправляется на допустимый URL-адрес неправильного формата.