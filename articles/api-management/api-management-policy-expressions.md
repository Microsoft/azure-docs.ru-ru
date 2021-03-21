---
title: Выражения политики в службе управления API Azure | Документация Майкрософт
description: Сведения о выражениях политики в службе управления API Azure. См. примеры и Просмотр дополнительных доступных ресурсов.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: ea160028-fc04-4782-aa26-4b8329df3448
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/22/2019
ms.author: apimpm
ms.openlocfilehash: aec1967f0652e18c4a24ca258c14a103355b22af
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99219321"
---
# <a name="api-management-policy-expressions"></a>Выражения политики в службе управления API
В этой статье рассматривается синтаксис выражений политики в C# 7. Каждому выражению предоставлен доступ к неявно заданной переменной [context](api-management-policy-expressions.md#ContextVariables) и допустимому [подмножеству](api-management-policy-expressions.md#CLRTypes) типов платформы .NET Framework.

Дополнительные сведения

- Способы передачи сведений о контексте во внутреннюю службу. Используйте политики [настройки параметра строки запроса](api-management-transformation-policies.md#SetQueryStringParameter) и [настройки HTTP-заголовка](api-management-transformation-policies.md#SetHTTPheader), чтобы передать эти сведения.
- Использование политики [проверки JWT](api-management-access-restriction-policies.md#ValidateJWT) для предварительной авторизации доступа к операциям на основе утверждений маркеров.
- Сведения об использовании трассировки с помощью [инспектора API](./api-management-howto-api-inspector.md) для оценки политик, а также результаты этой оценки.
- Использование выражений с политиками [получения из кэша](api-management-caching-policies.md#GetFromCache) и [хранения в кэше](api-management-caching-policies.md#StoreToCache) для настройки кэширования ответа службы управления API. Установите такую же длительность, как и для кэширования ответа во внутренней службе, которая задается директивой `Cache-Control` на сервере.
- Сведения о том, как отфильтровать содержимое. Удалите элементы данных из ответа, полученного из внутренней службы с помощью политик [потока управления](api-management-advanced-policies.md#choose) и [задания текста](api-management-transformation-policies.md#SetBody).
- Сведения о загрузке инструкций политики см. в репозитории GitHub [API-Management-Samples/Policies](https://github.com/Azure/api-management-samples/tree/master/policies) .


## <a name="syntax"></a><a name="Syntax"></a> Синтаксис
Выражения с одним оператором заключаются в элементы `@(expression)`, где `expression` — оператор выражения C# правильного формата.

Выражения с несколькими операторами заключаются в элементы `@{expression}`. Все ветви кода в выражениях с несколькими операторами должны заканчиваться оператором `return`.

## <a name="examples"></a><a name="PolicyExpressionsExamples"></a> Примеры

```
@(true)

@((1+1).ToString())

@("Hi There".Length)

@(Regex.Match(context.Response.Headers.GetValueOrDefault("Cache-Control",""), @"max-age=(?<maxAge>\d+)").Groups["maxAge"]?.Value)

@(context.Variables.ContainsKey("maxAge") ? int.Parse((string)context.Variables["maxAge"]) : 3600)

@{
  string[] value;
  if (context.Request.Headers.TryGetValue("Authorization", out value))
  {
      if(value != null && value.Length > 0)
      {
          return Encoding.UTF8.GetString(Convert.FromBase64String(value[0]));
      }
  }
  return null;

}
```

## <a name="usage"></a><a name="PolicyExpressionsUsage"></a>Загрузка
Выражения можно использовать в качестве значений атрибутов или текстовых значений в любой [политике](api-management-policies.md) службы управления API, если в справочнике по политике не указано иное.

> [!IMPORTANT]
> При использовании выражений политики выполняется только ограниченная проверка выражений политики во время ее определения. Выражения выполняет шлюз во время выполнения. Все исключения, создаваемые выражениями политики, приводят к ошибке среды выполнения.

## <a name="net-framework-types-allowed-in-policy-expressions"></a><a name="CLRTypes"></a> Типы .NET Framework, допустимые в выражениях политики
В следующей таблице перечислены типы .NET Framework и их члены, допустимые в выражениях политики.

|Type|Поддерживаемые члены|
|--------------|-----------------------|
|Newtonsoft.Js. Форматирован|Все|
|Newtonsoft.Json.Jsное преобразование|Для SerializeObject, методов DeserializeObject|
|Newtonsoft.Json.Linq.Extensions|Все|
|Newtonsoft.Json.Linq.JArray|Все|
|Newtonsoft.Json.Linq.JConstructor|Все|
|Newtonsoft.Json.Linq.JContainer|Все|
|Newtonsoft.Json.Linq.JObject|Все|
|Newtonsoft.Json.Linq.JProperty|Все|
|Newtonsoft.Json.Linq.JRaw|Все|
|Newtonsoft.Json.Linq.JToken|Все|
|Newtonsoft.Json.Linq.JTokenType|Все|
|Newtonsoft.Json.Linq.JValue|Все|
|System.Array|Все|
|System.BitConverter|Все|
|System.Boolean|Все|
|System.Byte|Все|
|System.Char|Все|
|System.Collections.Generic.Dictionary<TKey, TValue>|Все|
|System. Collections. Generic. hash\<T>|Все|
|System. Collections. Generic. ICollection\<T>|Все|
|System. Collections. Generic. IDictionary<TKey, TValue>|Все|
|System.Collections.Generic.IEnumerable\<T>|Все|
|System. Collections. Generic. IEnumerator\<T>|Все|
|System.Collections.Generic.IList\<T>|Все|
|System. Collections. Generic. IReadOnlyCollection\<T>|Все|
|System. Collections. Generic. IReadOnlyDictionary<TKey, TValue>|Все|
|System. Collections. Generic. ISet\<T>|Все|
|System. Collections. Generic. KeyValuePair<TKey, TValue>|Все|
|System. Collections. Generic. List\<T>|Все|
|System. Collections. Generic. Queue\<T>|Все|
|System. Collections. Generic. Stack\<T>|Все|
|System.Convert|Все|
|System.DateTime|(Конструктор), Add, AddDays, AddHours, Аддмиллисекондс, Аддминутес, Аддмонсс, Аддсекондс, Аддтиккс, Аддеарс, Дата, день, DayOfWeek, DayOfYear, Дайсинмонс, Hour, Исдайлигхтсавингтиме, IsLeapYear, MaxValue, миллисекунда, минута, MinValue, month, Now, анализ, секунда, вычитание, такты, TimeOfDay, сегодня, ToString, UtcNow, Year|
|System.DateTimeKind|Время в формате UTC|
|System.DateTimeOffset|Все|
|System.Decimal|Все|
|System.Double|Все|
|System.Exception|Все|
|System.Guid|Все|
|System.Int16|Все|
|System.Int32|Все|
|System.Int64|Все|
|System. IO. StringReader|Все|
|System. IO. Стрингвритер|Все|
|System. LINQ. Enumerable|Все|
|System.Math|Все|
|System.MidpointRounding|Все|
|System.Net.WebUtility|Все|
|System.Nullable|Все|
|System.Random|Все|
|System.SByte|Все|
|System.Security.Cryptography.AsymmetricAlgorithm|Все|
|System. Security. Cryptography. CipherMode|Все|
|System.Security.Cryptography.HashAlgorithm|Все|
|System. Security. Cryptography. Хашалгорисмнаме|Все|
|System.Security.Cryptography.HMAC|Все|
|System.Security.Cryptography.HMACMD5|Все|
|System.Security.Cryptography.HMACSHA1|Все|
|System.Security.Cryptography.HMACSHA256|Все|
|System.Security.Cryptography.HMACSHA384|Все|
|System.Security.Cryptography.HMACSHA512|Все|
|System.Security.Cryptography.KeyedHashAlgorithm|Все|
|System.Security.Cryptography.MD5|Все|
|System. Security. Cryptography. OID|Все|
|System. Security. Cryptography. PaddingMode|Все|
|System.Security.Cryptography.RNGCryptoServiceProvider|Все|
|System.Security.Cryptography.RSA|Все|
|System. Security. Cryptography. Рсаенкриптионпаддинг|Все|
|System. Security. Cryptography. Рсасигнатурепаддинг|Все|
|System.Security.Cryptography.SHA1|Все|
|System.Security.Cryptography.SHA1Managed|Все|
|System.Security.Cryptography.SHA256|Все|
|System.Security.Cryptography.SHA256Managed|Все|
|System.Security.Cryptography.SHA384|Все|
|System.Security.Cryptography.SHA384Managed|Все|
|System.Security.Cryptography.SHA512|Все|
|System.Security.Cryptography.SHA512Managed|Все|
|System. Security. Cryptography. SymmetricAlgorithm|Все|
|System. Security. Cryptography. X509Certificates. PublicKey|Все|
|System. Security. Cryptography. X509Certificates. Рсацертификатикстенсионс|Все|
|System. Security. Cryptography. X509Certificates. X500DistinguishedName|Имя|
|System. Security. Cryptography. X509Certificates. X509Certificate|Все|
|System.Security.Cryptography.X509Certificates.X509Certificate2|Все|
|System. Security. Cryptography. X509Certificates. X509ContentType|Все|
|System. Security. Cryptography. X509Certificates. X509NameType|Все|
|System.Single|Все|
|System.String|Все|
|System.StringComparer|Все|
|System.StringComparison|Все|
|System.StringSplitOptions|Все|
|System.Text.Encoding|Все|
|System.Text.RegularExpressions.Capture|Index, Length, Value|
|System.Text.RegularExpressions.CaptureCollection|Count, Item|
|System.Text.RegularExpressions.Group|Captures, Success|
|System.Text.RegularExpressions.GroupCollection|Count, Item|
|System.Text.RegularExpressions.Match|Empty, Groups, Result|
|System.Text.RegularExpressions.Regex|(Конструктор), Match, Match, Matches, Replace, Unescape, разбиение|
|System.Text.RegularExpressions.RegexOptions|Все|
|System.Text.StringBuilder|Все|
|System.TimeSpan|Все|
|System.TimeZone|Все|
|System. TimeZoneInfo. массива AdjustmentRule|Все|
|System. TimeZoneInfo. Транситионтиме|Все|
|System.TimeZoneInfo|Все|
|System.Tuple|Все|
|System.UInt16|Все|
|System.UInt32|Все|
|System.UInt64|Все|
|System.Uri|Все|
|System.UriPartial|Все|
|System.Xml.Linq.Extensions|Все|
|System.Xml.Linq.XAttribute|Все|
|System.Xml.Linq.XCData|Все|
|System.Xml.Linq.XComment|Все|
|System.Xml.Linq.XContainer|Все|
|System.Xml.Linq.XDeclaration|Все|
|System.Xml.Linq.XDocument|Все, за исключением: Load|
|System.Xml.Linq.XDocumentType|Все|
|System.Xml.Linq.XElement|Все|
|System.Xml.Linq.XName|Все|
|System.Xml.Linq.XNamespace|Все|
|System.Xml.Linq.XNode|Все|
|System.Xml.Linq.XNodeDocumentOrderComparer|Все|
|System.Xml.Linq.XNodeEqualityComparer|Все|
|System.Xml.Linq.XObject|Все|
|System.Xml.Linq.XProcessingInstruction|Все|
|System.Xml.Linq.XText|Все|
|System.Xml.XmlNodeType|Все|

## <a name="context-variable"></a><a name="ContextVariables"></a> Переменная контекста
Переменная с именем `context` неявно доступна в каждом [выражении](api-management-policy-expressions.md#Syntax) политики. Ее члены предоставляют сведения, относящиеся к переменной `\request`. Все члены `context` доступны только для чтения.

|Переменная контекста|Допустимые методы, свойства и значения параметров|
|----------------------|-------------------------------------------------------|
|контекст|[API](#ref-context-api): [иапи](#ref-iapi)<br /><br /> [Развертывание](#ref-context-deployment)<br /><br /> Elapsed: TimeSpan — интервал времени между значением Timestamp и текущим временем<br /><br /> [LastError](#ref-context-lasterror)<br /><br /> [Операция](#ref-context-operation)<br /><br /> [Продукт](#ref-context-product)<br /><br /> [Запрос](#ref-context-request)<br /><br /> RequestId: Guid — уникальный идентификатор запроса<br /><br /> [Ответ](#ref-context-response)<br /><br /> [Подписка](#ref-context-subscription)<br /><br /> Timestamp: DateTime — время получения запроса<br /><br /> Tracing: логическое значение — указывает, включена ли трассировка <br /><br /> [Пользователь](#ref-context-user)<br /><br /> [Переменные](#ref-context-variables): строка<IReadOnlyDictionary, объект><br /><br /> void Trace(message: строка)|
|<a id="ref-context-api"></a>context.Api|Id: строка<br /><br /> IsCurrentRevision: bool<br /><br />  Name: строка<br /><br /> Path: строка<br /><br /> Revision: строка<br /><br /> ServiceUrl: [иурл](#ref-iurl)<br /><br /> Version: строка |
|<a id="ref-context-deployment"></a>context.Deployment|Region: строка<br /><br /> ServiceName: строка<br /><br /> Certificates: IReadOnlyDictionary<строка, X509Certificate2>|
|<a id="ref-context-lasterror"></a>context.LastError|Source: строка<br /><br /> Reason: строка<br /><br /> Message: строка<br /><br /> Scope: строка<br /><br /> Section: строка<br /><br /> Path: строка<br /><br /> PolicyId: строка<br /><br /> Дополнительные сведения о переменной context.LastError см. в разделе [Error handling](api-management-error-handling-policies.md) (Обработка ошибок).|
|<a id="ref-context-operation"></a>context.Operation|Id: строка<br /><br /> Method: строка<br /><br /> Name: строка<br /><br /> UrlTemplate: строка|
|<a id="ref-context-product"></a>context.Product|API-интерфейсы: IEnumerable<[иапи](#ref-iapi)\><br /><br /> ApprovalRequired: логическое значение<br /><br /> Группы: IEnumerable<[играуп](#ref-igroup)\><br /><br /> Id: строка<br /><br /> Name: строка<br /><br /> State: enum ProductState {NotPublished, Published}<br /><br /> SubscriptionLimit: целое число?<br /><br /> SubscriptionRequired: логическое значение|
|<a id="ref-context-request"></a>context.Request|Body: [имессажебоди](#ref-imessagebody) или, `null` Если запрос не имеет текста.<br /><br /> Certificate: System.Security.Cryptography.X509Certificates.X509Certificate2<br /><br /> [Заголовки](#ref-context-request-headers): IReadOnlyDictionary<строка, String [] ><br /><br /> IpAddress: строка<br /><br /> MatchedParameters: IReadOnlyDictionary<string, string><br /><br /> Method: строка<br /><br /> Оригиналурл: [иурл](#ref-iurl)<br /><br /> URL-адрес: [иурл](#ref-iurl)|
|<a id="ref-context-request-headers"></a>string context.Request.Headers.GetValueOrDefault(headerName: строка, defaultValue: строка)|headerName: строка<br /><br /> defaultValue: строка<br /><br /> Возвращает значения заголовков запросов, разделенные запятыми, или значение `defaultValue`, если заголовок не найден.|
|<a id="ref-context-response"></a>context.Response|Текст: [имессажебоди](#ref-imessagebody)<br /><br /> [Заголовки](#ref-context-response-headers): IReadOnlyDictionary<строка, String [] ><br /><br /> StatusCode: целое число<br /><br /> StatusReason: строка|
|<a id="ref-context-response-headers"></a>string context.Response.Headers.GetValueOrDefault(headerName: строка, defaultValue: строка)|headerName: строка<br /><br /> defaultValue: строка<br /><br /> Возвращает значения заголовков ответов, разделенные запятыми, или значение `defaultValue`, если заголовок не найден.|
|<a id="ref-context-subscription"></a>context.Subscription|CreatedDate: Дата и время<br /><br /> EndDate: дата и время?<br /><br /> Id: строка<br /><br /> Key: строка<br /><br /> Name: строка<br /><br /> PrimaryKey: строка<br /><br /> SecondaryKey: строка<br /><br /> StartDate: дата и время?|
|<a id="ref-context-user"></a>context.User|Email: строка<br /><br /> FirstName: строка<br /><br /> Группы: IEnumerable<[играуп](#ref-igroup)\><br /><br /> Id: строка<br /><br /> Удостоверения: IEnumerable<[иусеридентити](#ref-iuseridentity)\><br /><br /> LastName: строка<br /><br /> Note: строка<br /><br /> RegistrationDate: дата и время|
|<a id="ref-iapi"></a>IApi|Id: строка<br /><br /> Name: строка<br /><br /> Path: строка<br /><br /> Protocols: IEnumerable<string\><br /><br /> ServiceUrl: [иурл](#ref-iurl)<br /><br /> Субскриптионкэйпараметернамес: [исубскриптионкэйпараметернамес](#ref-isubscriptionkeyparameternames)|
|<a id="ref-igroup"></a>IGroup|Id: строка<br /><br /> Name: строка|
|<a id="ref-imessagebody"></a>IMessageBody|Как<T \> (пресервеконтент: bool = false): где T: строка, Byte [], JObject, JToken, жаррай, XNode, XElement, XDocument<br /><br /> Методы `context.Request.Body.As<T>` и `context.Response.Body.As<T>` используются для чтения текста сообщения запроса или ответа в одном из заданных форматов `T`. По умолчанию метод использует исходный поток текста сообщения и делает его недоступным после возвращения. Чтобы избежать этого, задайте для параметра `preserveContent` значение `true`, чтобы метод работал с копией потока текста. Перейдите по [этой](api-management-transformation-policies.md#SetBody) ссылке, чтобы увидеть пример.|
|<a id="ref-iurl"></a>IUrl|Host: строка<br /><br /> Path: строка<br /><br /> Port: целое число<br /><br /> [Запрос](#ref-iurl-query): IReadOnlyDictionary<строка, String [] ><br /><br /> QueryString: строка<br /><br /> Scheme: строка|
|<a id="ref-iuseridentity"></a>IUserIdentity|Id: строка<br /><br /> Provider: строка|
|<a id="ref-isubscriptionkeyparameternames"></a>ISubscriptionKeyParameterNames|Header: строка<br /><br /> Query: строка|
|<a id="ref-iurl-query"></a>string IUrl.Query.GetValueOrDefault(queryParameterName: строка, defaultValue: строка)|queryParameterName: строка<br /><br /> defaultValue: строка<br /><br /> Возвращает разделенные запятыми значения параметров запроса или значение `defaultValue`, если параметр не найден.|
|<a id="ref-context-variables"></a>T контекст. Variables. GetValueOrDefault<T \> (variablename: String, DefaultValue: T)|variableName: строка<br /><br /> defaultValue: T<br /><br /> Возвращает значение переменной, приведенное к типу `T` или `defaultValue`, если переменная не найдена.<br /><br /> Этот метод выдает исключение, если указанный тип не соответствует фактическому типу возвращаемой переменной.|
|BasicAuthCredentials AsBasic(input: this string)|input: строка<br /><br /> Если входной параметр содержит допустимое значение заголовка запроса авторизации для обычной проверки подлинности HTTP, метод возвращает объект типа `BasicAuthCredentials`; в противном случае метод возвращает значение NULL.|
|bool TryParseBasic(input: this string, result: out BasicAuthCredentials)|input: строка<br /><br /> result: выходное значение BasicAuthCredentials<br /><br /> Если входной параметр содержит допустимое значение заголовка запроса авторизации для обычной проверки подлинности и HTTP, метод возвращает значение `true`, а параметр результата содержит значение типа `BasicAuthCredentials`. В противном случае метод возвращает значение `false`.|
|BasicAuthCredentials|Password: строка<br /><br /> UserId: строка|
|Jwt AsJwt(input: this string)|input: строка<br /><br /> Если входной параметр содержит допустимое значение маркера JWT, метод возвращает объект типа `Jwt`; в противном случае метод возвращает значение `null`.|
|bool TryParseJwt(input: this string, result: out Jwt)|input: строка<br /><br /> result: выходное значение типа Jwt<br /><br /> Если входной параметр содержит допустимое значение маркера JWT, метод возвращает значение `true`, а параметр результата содержит значение типа `Jwt`; в противном случае метод возвращает значение `false`.|
|Jwt|Algorithm: строка<br /><br /> Аудитории: строка<IEnumerable\><br /><br /> Claims: IReadOnlyDictionary<string, string[]><br /><br /> ExpirationTime: дата и время?<br /><br /> Id: строка<br /><br /> Issuer: строка<br /><br /> Иссуедат: Дата и время?<br /><br /> NotBefore: дата и время?<br /><br /> Subject: строка<br /><br /> Type: строка|
|string Jwt.Claims.GetValueOrDefault(claimName: string, defaultValue: string)|claimName: строка<br /><br /> defaultValue: строка<br /><br /> Возвращает значения утверждений, разделенные запятыми, или значение `defaultValue`, если заголовок не найден.|
|byte[] Encrypt(input: this byte[], alg: строка, key:byte[], iv:byte[])|input — открытый текст, который нужно зашифровать<br /><br />alg — имя алгоритма симметричного шифрования<br /><br />key — ключ шифрования<br /><br />iv — вектор инициализации<br /><br />Возвращает открытый текст в зашифрованном виде.|
|byte[] Encrypt(input: this byte[], alg: System.Security.Cryptography.SymmetricAlgorithm)|input — открытый текст, который нужно зашифровать<br /><br />alg — алгоритм шифрования<br /><br />Возвращает открытый текст в зашифрованном виде.|
|byte[] Encrypt(input: this byte[], alg: System.Security.Cryptography.SymmetricAlgorithm, key:byte[], iv:byte[])|input — открытый текст, который нужно зашифровать<br /><br />alg — алгоритм шифрования<br /><br />key — ключ шифрования<br /><br />iv — вектор инициализации<br /><br />Возвращает открытый текст в зашифрованном виде.|
|byte[] Decrypt(input: this byte[], alg: строка, key:byte[], iv:byte[])|input — зашифрованный текст, который нужно расшифровать<br /><br />alg — имя алгоритма симметричного шифрования<br /><br />key — ключ шифрования<br /><br />iv — вектор инициализации<br /><br />Возвращает открытый текст.|
|byte[] Decrypt(input: this byte[], alg: System.Security.Cryptography.SymmetricAlgorithm)|input — зашифрованный текст, который нужно расшифровать<br /><br />alg — алгоритм шифрования<br /><br />Возвращает открытый текст.|
|byte[] Decrypt(input: this byte[], alg: System.Security.Cryptography.SymmetricAlgorithm, key:byte[], iv:byte[])|input — зашифрованный текст, который нужно расшифровать<br /><br />alg — алгоритм шифрования<br /><br />key — ключ шифрования<br /><br />iv — вектор инициализации<br /><br />Возвращает открытый текст.|
|bool Верифиноревокатион (входные данные: Этот системный. Security. Cryptography. X509Certificates. X509Certificate2)|Выполняет проверку цепочки X. 509 без проверки состояния отзыва сертификата.<br /><br />input-Certificate, объект<br /><br />Возвращает значение `true` , если проверка прошла успешно; `false` значение, если проверка завершается неудачно.|


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о работе с политиками см. в следующих статьях:

+ [Политики в управлении API](api-management-howto-policies.md)
+ [Преобразование API-интерфейсов](transform-api.md).
+ Полный перечень операторов политик и их параметров см. в [справочнике по политикам](./api-management-policies.md).
+ [Примеры политик](./policy-reference.md).
