---
title: Использование управляемого удостоверения в приложении
description: Использование управляемых удостоверений в коде приложения Azure Service Fabric для доступа к службам Azure.
ms.topic: article
ms.date: 10/09/2019
ms.openlocfilehash: e26a29020f26583f7e4aa16434c7e8647ba9a5a3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98871067"
---
# <a name="how-to-leverage-a-service-fabric-applications-managed-identity-to-access-azure-services"></a>Как использовать управляемое удостоверение приложения Service Fabric для доступа к службам Azure

Service Fabric приложения могут использовать управляемые удостоверения для доступа к другим ресурсам Azure, поддерживающим проверку подлинности на основе Azure Active Directory. Приложение может получить [маркер доступа](../active-directory/develop/developer-glossary.md#access-token) , представляющий удостоверение, которое может быть назначено системой или назначено пользователю, и использовать его в качестве токена носителя для проверки подлинности в другой службе, также называемой [защищенным сервером ресурсов](../active-directory/develop/developer-glossary.md#resource-server). Маркер представляет удостоверение, назначенное приложению Service Fabric, и будет выдаваться только ресурсам Azure (включая приложения SF), которые совместно используют это удостоверение. Подробное описание управляемых удостоверений, а также различие между назначенными системой и назначенными пользователем удостоверениями см. в документации по [управляемым удостоверениям](../active-directory/managed-identities-azure-resources/overview.md) . В рамках этой статьи мы будем называть приложение Service Fabric с поддержкой управляемых удостоверений в качестве [клиентского приложения](../active-directory/develop/developer-glossary.md#client-application) .

См. Сопутствующий пример приложения, демонстрирующий использование назначенных системой и назначенных пользователю [удостоверений Service Fabric приложений](https://github.com/Azure-Samples/service-fabric-managed-identity) с Reliable Services и контейнерами.

> [!IMPORTANT]
> Управляемое удостоверение представляет связь между ресурсом Azure и субъектом-службой в соответствующем клиенте Azure AD, связанном с подпиской, содержащей ресурс. Таким образом, в контексте Service Fabric управляемые удостоверения поддерживаются только для приложений, развернутых в качестве ресурсов Azure. 

> [!IMPORTANT]
> Прежде чем использовать управляемое удостоверение приложения Service Fabric, клиентскому приложению должен быть предоставлен доступ к защищенному ресурсу. Ознакомьтесь со списком [служб Azure, поддерживающих проверку подлинности Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-managed-identities-for-azure-resources) , чтобы проверить поддержку, а затем в соответствующую документацию службы, чтобы предоставить удостоверению доступ к интересующим ресурсам. 
 

## <a name="leverage-a-managed-identity-using-azureidentity"></a>Использование управляемого удостоверения с помощью Azure. Identity

Пакет SDK для удостоверений Azure теперь поддерживает Service Fabric. Использование Azure. Identity делает написание кода для использования Service Fabric управляемых приложением удостоверений проще, так как оно обрабатывает маркеры, маркеры кэширования и проверку подлинности сервера. При доступе к большинству ресурсов Azure концепция маркера скрыта.

Поддержка Service Fabric доступна в следующих версиях для следующих языков: 
- [C# в версии 1.3.0](https://www.nuget.org/packages/Azure.Identity). См. [пример на C#](https://github.com/Azure-Samples/service-fabric-managed-identity).
- [Python в версии 1.5.0](https://pypi.org/project/azure-identity/). См. [Пример Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/identity/azure-identity/tests/managed-identity-live/service-fabric/service_fabric.md).
- [Java в версии 1.2.0](/java/api/overview/azure/identity-readme).

Пример инициализации учетных данных на C# и использование учетных данных для выборки секрета из Azure Key Vault:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace MyMIService
{
    internal sealed class MyMIService : StatelessService
    {
        protected override async Task RunAsync(CancellationToken cancellationToken)
        {
            try
            {
                // Load the service fabric application managed identity assigned to the service
                ManagedIdentityCredential creds = new ManagedIdentityCredential();

                // Create a client to keyvault using that identity
                SecretClient client = new SecretClient(new Uri("https://mykv.vault.azure.net/"), creds);

                // Fetch a secret
                KeyVaultSecret secret = (await client.GetSecretAsync("mysecret", cancellationToken: cancellationToken)).Value;
            }
            catch (CredentialUnavailableException e)
            {
                // Handle errors with loading the Managed Identity
            }
            catch (RequestFailedException)
            {
                // Handle errors with fetching the secret
            }
            catch (Exception e)
            {
                // Handle generic errors
            }
        }
    }
}

```

## <a name="acquiring-an-access-token-using-rest-api"></a>Получение маркера доступа с помощью REST API
В кластерах, поддерживающих управляемое удостоверение, среда выполнения Service Fabric предоставляет конечную точку localhost, которую приложения могут использовать для получения маркеров доступа. Конечная точка доступна на каждом узле кластера и доступна для всех сущностей на этом узле. Полномочные вызывающие объекты могут получить маркеры доступа, вызвав эту конечную точку и выполнив код проверки подлинности. код создается средой выполнения Service Fabric для каждой отдельной активации пакета кода службы и привязана к времени существования процесса, в котором размещается пакет кода службы.

В частности, среда службы Service Fabric с поддержкой управляемых удостоверений будет заполнена следующими переменными:
- "IDENTITY_ENDPOINT": конечная точка localhost, соответствующая управляемому удостоверению службы
- "IDENTITY_HEADER": уникальный код проверки подлинности, представляющий службу на текущем узле
- "IDENTITY_SERVER_THUMBPRINT": отпечаток управляемого сервера удостоверений Service Fabric

> [!IMPORTANT]
> Код приложения должен рассматривать значение переменной среды "IDENTITY_HEADER" как конфиденциальные данные. она не должна регистрироваться или распространяться иным образом. Код проверки подлинности не имеет значения, находящегося за пределами локального узла, или после завершения процесса, в котором размещена служба, но он представляет удостоверение службы Service Fabric, поэтому следует рассматриваться с теми же мерами предосторожности, что и у маркера доступа.

Чтобы получить маркер, клиент выполняет следующие действия:
- формирует универсальный код ресурса (URI) путем сцепления управляемой конечной точки удостоверения (IDENTITY_ENDPOINT Value) с версией API и ресурсом (аудиторией), требуемым для токена.
- создает запрос GET HTTP (s) для указанного URI.
- Добавляет подходящую логику проверки сертификата сервера
- добавляет код проверки подлинности (IDENTITY_HEADER значение) в качестве заголовка в запрос
- отправляет запрос

Успешный ответ будет содержать полезные данные JSON, представляющие результирующий маркер доступа, а также метаданные, описывающие его. Неудачный ответ также будет включать объяснение сбоя. Дополнительные сведения об обработке ошибок см. ниже.

Маркеры доступа будут кэшироваться Service Fabric на различных уровнях (узел, кластер, служба поставщика ресурсов), поэтому успешный ответ не обязательно означает, что маркер был выдан непосредственно в ответ на запрос пользовательского приложения. Маркеры будут кэшироваться меньше времени существования, поэтому приложение гарантированно получит действительный токен. Рекомендуется, чтобы код приложения кэширует все маркеры доступа, которые он получает. ключ кэширования должен включать аудиторию (производную от). 

Пример запроса:
```http
GET 'https://localhost:2377/metadata/identity/oauth2/token?api-version=2019-07-01-preview&resource=https://vault.azure.net/' HTTP/1.1 Secret: 912e4af7-77ba-4fa5-a737-56c8e3ace132
```
где:

| Элемент | Описание |
| ------- | ----------- |
| `GET` | HTTP-команда, указывающая, что необходимо извлечь данные из конечной точки. В этом случае используется маркер доступа OAuth. | 
| `https://localhost:2377/metadata/identity/oauth2/token` | Управляемая конечная точка удостоверения для приложений Service Fabric, предоставляемых через переменную среды IDENTITY_ENDPOINT. |
| `api-version` | Параметр строки запроса, указывающий версию API службы токенов управляемого удостоверения; Сейчас единственным допустимым значением является `2019-07-01-preview` и может быть изменено. |
| `resource` | Параметр строки запроса, указывающий URI идентификатора приложения целевого ресурса. Это будет отражено в `aud` утверждении (аудитории) выданного маркера. В этом примере запрашивается маркер для доступа к Azure Key Vault, для которого URI идентификатора приложения — https: \/ /Vault.Azure.NET/. |
| `Secret` | Поле заголовка HTTP-запроса, необходимое для службы Service Fabric управляемого маркера идентификации для служб Service Fabric для проверки подлинности вызывающего объекта. Это значение предоставляется средой выполнения SF через переменную среды IDENTITY_HEADER. |


Пример ответа:
```json
HTTP/1.1 200 OK
Content-Type: application/json
{
    "token_type":  "Bearer",
    "access_token":  "eyJ0eXAiO...",
    "expires_on":  1565244611,
    "resource":  "https://vault.azure.net/"
}
```
где:

| Элемент | Описание |
| ------- | ----------- |
| `token_type` | Тип токена; в данном случае это маркер доступа носителя, который означает, что субъект (Bearer) этого маркера является предполагаемой темой маркера. |
| `access_token` | Запрашиваемый маркер доступа. При вызове защищенного REST API маркер внедряется в поле `Authorization` заголовка запроса в качестве маркера носителя, позволяя API выполнить проверку подлинности вызывающего объекта. | 
| `expires_on` | Метка времени истечения срока действия маркера доступа; представляется в виде числа секунд с "1970-01-01T0:0: 0Z UTC" и соответствует утверждению маркера `exp` . В этом случае срок действия маркера истекает в 2019-08-08T06:10:11 + 00:00 (в RFC 3339).|
| `resource` | Ресурс, для которого был выдан маркер доступа, указанный с помощью `resource` параметра строки запроса запроса; соответствует утверждению "AUD" маркера. |


## <a name="acquiring-an-access-token-using-c"></a>Получение маркера доступа с помощью языка C #
Приведенный выше объект в C# выглядит следующим образом:

```C#
namespace Azure.ServiceFabric.ManagedIdentity.Samples
{
    using System;
    using System.Net.Http;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web;
    using Newtonsoft.Json;

    /// <summary>
    /// Type representing the response of the SF Managed Identity endpoint for token acquisition requests.
    /// </summary>
    [JsonObject]
    public sealed class ManagedIdentityTokenResponse
    {
        [JsonProperty(Required = Required.Always, PropertyName = "token_type")]
        public string TokenType { get; set; }

        [JsonProperty(Required = Required.Always, PropertyName = "access_token")]
        public string AccessToken { get; set; }

        [JsonProperty(PropertyName = "expires_on")]
        public string ExpiresOn { get; set; }

        [JsonProperty(PropertyName = "resource")]
        public string Resource { get; set; }
    }

    /// <summary>
    /// Sample class demonstrating access token acquisition using Managed Identity.
    /// </summary>
    public sealed class AccessTokenAcquirer
    {
        /// <summary>
        /// Acquire an access token.
        /// </summary>
        /// <returns>Access token</returns>
        public static async Task<string> AcquireAccessTokenAsync()
        {
            var managedIdentityEndpoint = Environment.GetEnvironmentVariable("IDENTITY_ENDPOINT");
            var managedIdentityAuthenticationCode = Environment.GetEnvironmentVariable("IDENTITY_HEADER");
            var managedIdentityServerThumbprint = Environment.GetEnvironmentVariable("IDENTITY_SERVER_THUMBPRINT");
            // Latest api version, 2019-07-01-preview is still supported.
            var managedIdentityApiVersion = Environment.GetEnvironmentVariable("IDENTITY_API_VERSION");
            var managedIdentityAuthenticationHeader = "secret";
            var resource = "https://management.azure.com/";

            var requestUri = $"{managedIdentityEndpoint}?api-version={managedIdentityApiVersion}&resource={HttpUtility.UrlEncode(resource)}";

            var requestMessage = new HttpRequestMessage(HttpMethod.Get, requestUri);
            requestMessage.Headers.Add(managedIdentityAuthenticationHeader, managedIdentityAuthenticationCode);
            
            var handler = new HttpClientHandler();
            handler.ServerCertificateCustomValidationCallback = (httpRequestMessage, cert, certChain, policyErrors) =>
            {
                // Do any additional validation here
                if (policyErrors == SslPolicyErrors.None)
                {
                    return true;
                }
                return 0 == string.Compare(cert.GetCertHashString(), managedIdentityServerThumbprint, StringComparison.OrdinalIgnoreCase);
            };

            try
            {
                var response = await new HttpClient(handler).SendAsync(requestMessage)
                    .ConfigureAwait(false);

                response.EnsureSuccessStatusCode();

                var tokenResponseString = await response.Content.ReadAsStringAsync()
                    .ConfigureAwait(false);

                var tokenResponseObject = JsonConvert.DeserializeObject<ManagedIdentityTokenResponse>(tokenResponseString);

                return tokenResponseObject.AccessToken;
            }
            catch (Exception ex)
            {
                string errorText = String.Format("{0} \n\n{1}", ex.Message, ex.InnerException != null ? ex.InnerException.Message : "Acquire token failed");

                Console.WriteLine(errorText);
            }

            return String.Empty;
        }
    } // class AccessTokenAcquirer
} // namespace Azure.ServiceFabric.ManagedIdentity.Samples
```
## <a name="accessing-key-vault-from-a-service-fabric-application-using-managed-identity"></a>Доступ к Key Vault из Service Fabric приложения с помощью управляемого удостоверения
Этот пример строится на примере выше, чтобы продемонстрировать доступ к секрету, хранящемуся в Key Vault с помощью управляемого удостоверения.

```C#
        /// <summary>
        /// Probe the specified secret, displaying metadata on success.  
        /// </summary>
        /// <param name="vault">vault name</param>
        /// <param name="secret">secret name</param>
        /// <param name="version">secret version id</param>
        /// <returns></returns>
        public async Task<string> ProbeSecretAsync(string vault, string secret, string version)
        {
            // initialize a KeyVault client with a managed identity-based authentication callback
            var kvClient = new Microsoft.Azure.KeyVault.KeyVaultClient(new Microsoft.Azure.KeyVault.KeyVaultClient.AuthenticationCallback((a, r, s) => { return AuthenticationCallbackAsync(a, r, s); }));

            Log(LogLevel.Info, $"\nRunning with configuration: \n\tobserved vault: {config.VaultName}\n\tobserved secret: {config.SecretName}\n\tMI endpoint: {config.ManagedIdentityEndpoint}\n\tMI auth code: {config.ManagedIdentityAuthenticationCode}\n\tMI auth header: {config.ManagedIdentityAuthenticationHeader}");
            string response = String.Empty;

            Log(LogLevel.Info, "\n== {DateTime.UtcNow.ToString()}: Probing secret...");
            try
            {
                var secretResponse = await kvClient.GetSecretWithHttpMessagesAsync(vault, secret, version)
                    .ConfigureAwait(false);

                if (secretResponse.Response.IsSuccessStatusCode)
                {
                    // use the secret: secretValue.Body.Value;
                    response = String.Format($"Successfully probed secret '{secret}' in vault '{vault}': {PrintSecretBundleMetadata(secretResponse.Body)}");
                }
                else
                {
                    response = String.Format($"Non-critical error encountered retrieving secret '{secret}' in vault '{vault}': {secretResponse.Response.ReasonPhrase} ({secretResponse.Response.StatusCode})");
                }
            }
            catch (Microsoft.Rest.ValidationException ve)
            {
                response = String.Format($"encountered REST validation exception 0x{ve.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}' from {ve.Source}: {ve.Message}");
            }
            catch (KeyVaultErrorException kvee)
            {
                response = String.Format($"encountered KeyVault exception 0x{kvee.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}': {kvee.Response.ReasonPhrase} ({kvee.Response.StatusCode})");
            }
            catch (Exception ex)
            {
                // handle generic errors here
                response = String.Format($"encountered exception 0x{ex.HResult.ToString("X")} trying to access '{secret}' in vault '{vault}': {ex.Message}");
            }

            Log(LogLevel.Info, response);

            return response;
        }

        /// <summary>
        /// KV authentication callback, using the application's managed identity.
        /// </summary>
        /// <param name="authority">The expected issuer of the access token, from the KV authorization challenge.</param>
        /// <param name="resource">The expected audience of the access token, from the KV authorization challenge.</param>
        /// <param name="scope">The expected scope of the access token; not currently used.</param>
        /// <returns>Access token</returns>
        public async Task<string> AuthenticationCallbackAsync(string authority, string resource, string scope)
        {
            Log(LogLevel.Verbose, $"authentication callback invoked with: auth: {authority}, resource: {resource}, scope: {scope}");
            var encodedResource = HttpUtility.UrlEncode(resource);

            // This sample does not illustrate the caching of the access token, which the user application is expected to do.
            // For a given service, the caching key should be the (encoded) resource uri. The token should be cached for a period
            // of time at most equal to its remaining validity. The 'expires_on' field of the token response object represents
            // the number of seconds from Unix time when the token will expire. You may cache the token if it will be valid for at
            // least another short interval (1-10s). If its expiration will occur shortly, don't cache but still return it to the 
            // caller. The MI endpoint will not return an expired token.
            // Sample caching code:
            //
            // ManagedIdentityTokenResponse tokenResponse;
            // if (responseCache.TryGetCachedItem(encodedResource, out tokenResponse))
            // {
            //     Log(LogLevel.Verbose, $"cache hit for key '{encodedResource}'");
            //
            //     return tokenResponse.AccessToken;
            // }
            //
            // Log(LogLevel.Verbose, $"cache miss for key '{encodedResource}'");
            //
            // where the response cache is left as an exercise for the reader. MemoryCache is a good option, albeit not yet available on .net core.

            var requestUri = $"{config.ManagedIdentityEndpoint}?api-version={config.ManagedIdentityApiVersion}&resource={encodedResource}";
            Log(LogLevel.Verbose, $"request uri: {requestUri}");

            var requestMessage = new HttpRequestMessage(HttpMethod.Get, requestUri);
            requestMessage.Headers.Add(config.ManagedIdentityAuthenticationHeader, config.ManagedIdentityAuthenticationCode);
            Log(LogLevel.Verbose, $"added header '{config.ManagedIdentityAuthenticationHeader}': '{config.ManagedIdentityAuthenticationCode}'");

            var response = await httpClient.SendAsync(requestMessage)
                .ConfigureAwait(false);
            Log(LogLevel.Verbose, $"response status: success: {response.IsSuccessStatusCode}, status: {response.StatusCode}");

            response.EnsureSuccessStatusCode();

            var tokenResponseString = await response.Content.ReadAsStringAsync()
                .ConfigureAwait(false);

            var tokenResponse = JsonConvert.DeserializeObject<ManagedIdentityTokenResponse>(tokenResponseString);
            Log(LogLevel.Verbose, "deserialized token response; returning access code..");

            // Sample caching code (continuation):
            // var expiration = DateTimeOffset.FromUnixTimeSeconds(Int32.Parse(tokenResponse.ExpiresOn));
            // if (expiration > DateTimeOffset.UtcNow.AddSeconds(5.0))
            //    responseCache.AddOrUpdate(encodedResource, tokenResponse, expiration);

            return tokenResponse.AccessToken;
        }

        private string PrintSecretBundleMetadata(SecretBundle bundle)
        {
            StringBuilder strBuilder = new StringBuilder();

            strBuilder.AppendFormat($"\n\tid: {bundle.Id}\n");
            strBuilder.AppendFormat($"\tcontent type: {bundle.ContentType}\n");
            strBuilder.AppendFormat($"\tmanaged: {bundle.Managed}\n");
            strBuilder.AppendFormat($"\tattributes:\n");
            strBuilder.AppendFormat($"\t\tenabled: {bundle.Attributes.Enabled}\n");
            strBuilder.AppendFormat($"\t\tnbf: {bundle.Attributes.NotBefore}\n");
            strBuilder.AppendFormat($"\t\texp: {bundle.Attributes.Expires}\n");
            strBuilder.AppendFormat($"\t\tcreated: {bundle.Attributes.Created}\n");
            strBuilder.AppendFormat($"\t\tupdated: {bundle.Attributes.Updated}\n");
            strBuilder.AppendFormat($"\t\trecoveryLevel: {bundle.Attributes.RecoveryLevel}\n");

            return strBuilder.ToString();
        }

        private enum LogLevel
        {
            Info,
            Verbose
        };

        private void Log(LogLevel level, string message)
        {
            if (level != LogLevel.Verbose
                || config.DoVerboseLogging)
            {
                Console.WriteLine(message);
            }
        }

```

## <a name="error-handling"></a>Обработка ошибок
В поле "код состояния" заголовка HTTP-ответа указано состояние успешного выполнения запроса. состояние "200 ОК" указывает на успешное выполнение, и ответ будет включать маркер доступа, как описано выше. Ниже приведено краткое перечисление возможных ответов об ошибках.

| Код состояния | Причина ошибки | Способ устранения |
| ----------- | ------------ | ------------- |
| 404. Не найдено. | Неизвестный код проверки подлинности, или приложению не было назначено управляемое удостоверение. | Устраните настройку приложения или код получения маркера. |
| 429 — слишком много запросов |  Достигнут предел регулирования, накладываемый AAD или SF. | Повторите попытку с экспоненциальным увеличением задержки. См. инструкции ниже. |
| Ошибка 4xx в запросе. | Один или несколько параметров запроса неверные. | Не выполняйте повторную попытку.  Для получения дополнительной информации просмотрите сведения об ошибке.  Ошибки 4xx — это ошибки времени разработки.|
| Ошибка 5xx из службы. | Подсистема управляемого удостоверения или Azure Active Directory вернула временную ошибку. | Повторите попытку через некоторое время. При повторной попытке может попасть в условие регулирования (429).|

При возникновении ошибки соответствующий текст HTTP-ответа содержит объект JSON со сведениями об ошибке:

| Элемент | Описание |
| ------- | ----------- |
| code | Код ошибки. |
| correlationId | Идентификатор корреляции, который можно использовать для отладки. |
| message | Подробное описание ошибки. **Описания ошибок могут изменяться в любое время. Не зависят от самого сообщения об ошибке.**|

Пример ошибки:
```json
{"error":{"correlationId":"7f30f4d3-0f3a-41e0-a417-527f21b3848f","code":"SecretHeaderNotFound","message":"Secret is not found in the request headers."}}
```

Ниже приведен список типичных ошибок Service Fabric, характерных для управляемых удостоверений.

| Код | Сообщение | Описание | 
| ----------- | ----- | ----------------- |
| секресеадернотфаунд | Секрет не найден в заголовках запроса. | Код проверки подлинности не был предоставлен с запросом. | 
| манажедидентитинотфаунд | Управляемое удостоверение не найдено для указанного узла приложения. | Приложение не имеет удостоверения, или код проверки подлинности неизвестен. |
| аргументнуллоремпти | Параметр "Resource" не должен иметь значение null или быть пустой строкой. | Ресурс (аудитория) не указан в запросе. |
| InvalidApiVersion | Версия API "" не поддерживается. Поддерживаемая версия — "2019-07-01-Preview". | В URI запроса указана отсутствующая или неподдерживаемая версия API. |
| InternalServerError | Произошла ошибка. | В подсистеме управляемого удостоверения обнаружена ошибка, возможно, за пределами стека Service Fabric. Наиболее вероятной причиной является неверное значение, заданное для ресурса (проверьте наличие замыкающего символа "/"?) | 

## <a name="retry-guidance"></a>Рекомендации по использованию механизма повторов 

Как правило, единственным повторяемым кодом ошибки является 429 (слишком много запросов); внутренние ошибки сервера/5xx коды ошибок могут быть повторяемыми, хотя причина может быть постоянной. 

Ограничения регулирования применяются к числу вызовов управляемой подсистемы удостоверений — в частности, к "вышестоящим" зависимостям (управляемой службе удостоверений Azure или службе токенов безопасности). Service Fabric кэширует маркеры на различных уровнях конвейера, но учитывая распределенную природу вовлеченных компонентов, вызывающая сторона может столкнуться с непоследовательными ответами на регулирование (т. е. регулируется на одном узле или экземпляре приложения, но не на другом узле при запросе маркера для того же удостоверения). Если задано условие регулирования, последующие запросы из этого же приложения могут завершаться ошибкой с кодом состояния HTTP 429 (слишком много запросов) до тех пор, пока не будет сброшено условие.  

Не рекомендуется выполнять запросы, так как регулирование выполняется повторно с экспоненциальной задержкой, как показано ниже. 

| Индекс вызова | Действие при получении 429 | 
| --- | --- | 
| 1 | Подождите 1 секунду и повторите попытку |
| 2 | Подождите 2 секунды и повторите попытку |
| 3 | Подождите 4 секунды и повторите попытку |
| 4 | Подождите 8 секунд и повторите попытку |
| 4 | Подождите 8 секунд и повторите попытку |
| 5 | Подождите 16 секунд и повторите попытку |

## <a name="resource-ids-for-azure-services"></a>Идентификаторы ресурсов для служб Azure
Список ресурсов, поддерживающих Azure AD, и соответствующие идентификаторы ресурсов см. в статье [службы Azure, поддерживающие аутентификацию Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md) .

## <a name="next-steps"></a>Дальнейшие действия
* [Развертывание приложения Service Fabric Azure с управляемым удостоверением, назначенным системой](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
* [Развертывание приложения Service Fabric Azure с помощью управляемого удостоверения, назначенного пользователем](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)
* [Предоставление приложению Azure Service Fabric доступа к другим ресурсам Azure](./how-to-grant-access-other-resources.md)
* [Изучите пример приложения, используя управляемое удостоверение Service Fabric](https://github.com/Azure-Samples/service-fabric-managed-identity)