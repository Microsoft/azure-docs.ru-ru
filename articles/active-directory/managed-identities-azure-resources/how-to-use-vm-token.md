---
title: Использование управляемых удостоверений на виртуальной машине для получения маркера доступа — Azure AD
description: Пошаговые инструкции и примеры использования управляемых удостоверений для ресурсов Azure на виртуальных машинах для получения маркера доступа OAuth.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/03/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 541f76ad825f492679530902c571096ca4b01902
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98726237"
---
# <a name="how-to-use-managed-identities-for-azure-resources-on-an-azure-vm-to-acquire-an-access-token"></a>Как использовать управляемые удостоверения для ресурсов Azure на виртуальной машине Azure для получения маркера доступа 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]  

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде. 

Эта статья содержит различные примеры кода и скриптов для получения маркера, а также инструкции по обработке ситуаций с просроченным маркером и устранению ошибок HTTP. 

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

Если вы планируете использовать примеры Azure PowerShell в этой статье, убедитесь, что установлена последняя версия [Azure PowerShell](/powershell/azure/install-az-ps).


> [!IMPORTANT]
> - Во всех примерах кода или сценариев в этой статье предполагается, что клиент выполняется на виртуальной машине, на которой включены управляемые удостоверения для ресурсов Azure. Используйте функцию подключения виртуальной машины на портале Azure для удаленного подключения к своей виртуальной машине. Дополнительные сведения о включении управляемых удостоверений для ресурсов Azure на виртуальной машине см. в статье [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью Azure CLI](qs-configure-portal-windows-vm.md) или в одном из вариантов статьи (с использованием PowerShell, интерфейса командной строки, шаблона или пакета SDK для Azure). 

> [!IMPORTANT]
> - Периметром безопасности управляемых удостоверений для ресурсов Azure является ресурс, к которому они применены. Все примеры кода и сценариев, выполняемые на виртуальной машине, могут запрашивать и получать маркеры для любого управляемого удостоверения, доступного на ней. 

## <a name="overview"></a>Обзор

Клиентское приложение может запрашивать [маркер доступа к приложению](../develop/developer-glossary.md#access-token) управляемых удостоверений для ресурсов Azure, чтобы получить доступ к заданному ресурсу. Этот маркер [создан с помощью субъекта-службы управляемых удостоверений для ресурсов Azure](overview.md#managed-identity-types). Таким образом клиенту не нужно регистрироваться для получения маркера доступа для собственного субъекта-службы. Маркер подходит для использования в качестве маркера носителя при [выполнении вызовов между службами, требующих учетных данных клиента](../develop/v2-oauth2-client-creds-grant-flow.md).

| Ссылка | Описание |
| -------------- | -------------------- |
| [Получение маркера с использованием HTTP](#get-a-token-using-http) | Сведения о протоколе для конечной точки маркера управляемых удостоверений для ресурсов Azure |
| [Получение токена с помощью библиотеки Microsoft.Azure.Services.AppAuthentication для .NET](#get-a-token-using-the-microsoftazureservicesappauthentication-library-for-net) | Пример использования библиотеки Microsoft.Azure.Services.AppAuthentication из клиента .NET
| [Получение маркера с использованием C#](#get-a-token-using-c) | Пример использования конечной точки REST управляемых удостоверений для ресурсов Azure из клиента C# |
| [Получение маркера с использованием Java](#get-a-token-using-java) | Пример использования конечной точки REST управляемых удостоверений для ресурсов Azure из клиента Java. |
| [Получение маркера с использованием Go](#get-a-token-using-go) | Пример использования конечной точки REST управляемых удостоверений для ресурсов Azure из клиента Go |
| [Получение маркера с использованием Azure PowerShell](#get-a-token-using-azure-powershell) | Пример использования конечной точки REST управляемых удостоверений для ресурсов Azure из клиента PowerShell |
| [Получение маркера с использованием CURL](#get-a-token-using-curl) | Пример использования конечной точки REST управляемых удостоверений для ресурсов Azure из клиента Bash или CURL |
| Обработка кэширования маркеров | Рекомендации по обработке срока действия маркера доступа |
| [Обработка ошибок](#error-handling) | Рекомендации по обработке ошибок HTTP, возвращаемых из конечной точки маркера управляемых удостоверений для ресурсов Azure |
| [Идентификаторы ресурсов для служб Azure](#resource-ids-for-azure-services) | Сведения о том, где можно получить идентификаторы ресурсов для поддерживаемых служб Azure |

## <a name="get-a-token-using-http"></a>Получение маркера с использованием HTTP 

Основной интерфейс для получения маркера доступа создан на основе REST, что делает его доступным для любого клиентского приложения, выполняющегося на виртуальной машине,которая может выполнять вызовы HTTP REST. Это похоже на модель программирования Azure AD, за исключением того, что клиент использует конечную точку на виртуальной машине (а не конечную точку Azure AD).

Пример запроса с использованием конечной точки службы метаданных экземпляров (IMDS) Azure *(рекомендуется)*.

```
GET 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/' HTTP/1.1 Metadata: true
```

| Элемент | Описание |
| ------- | ----------- |
| `GET` | HTTP-команда, указывающая, что необходимо извлечь данные из конечной точки. В этом случае используется маркер доступа OAuth. | 
| `http://169.254.169.254/metadata/identity/oauth2/token` | Конечная точка управляемых удостоверений для ресурсов Azure для Службы метаданных экземпляров. |
| `api-version`  | Параметр строки запроса, указывающий версию API для конечной точки IMDS. Используйте версию API `2018-02-01` или выше. |
| `resource` | Параметр строки запроса, указывающий URI идентификатора приложения целевого ресурса. Он также отображается в утверждении (аудитории) `aud` выданного маркера. В этом примере запрашивается маркер для доступа к Azure Resource Manager, который имеет универсальный код ресурса (URI) идентификатора приложения `https://management.azure.com/`. |
| `Metadata` | Поле заголовка HTTP-запроса, необходимое управляемым удостоверениям для ресурсов Azure для устранения рисков от атак с подделкой запроса со стороны сервера (SSRF). Должно быть присвоено значение true (все в нижнем регистре). |
| `object_id` | (Необязательно.) Параметр строки запроса, указывающий object_id управляемого удостоверения, для которого требуется токен. Требуется, если виртуальная машина имеет несколько управляемых удостоверений, назначаемых пользователем.|
| `client_id` | (Необязательно.) Параметр строки запроса, указывающий client_id управляемого удостоверения, для которого требуется токен. Требуется, если виртуальная машина имеет несколько управляемых удостоверений, назначаемых пользователем.|
| `mi_res_id` | Используемых Параметр строки запроса, указывающий mi_res_id (идентификатор ресурса Azure) управляемого удостоверения, для которого вы хотите получить маркер. Требуется, если виртуальная машина имеет несколько управляемых удостоверений, назначаемых пользователем. |

Пример запроса с использованием управляемых удостоверений для ресурсов Azure к конечной точке расширения виртуальной машины *(в январе 2019 года эта служба будет объявлена устаревшей)*:

```http
GET http://localhost:50342/oauth2/token?resource=https%3A%2F%2Fmanagement.azure.com%2F HTTP/1.1
Metadata: true
```

| Элемент | Описание |
| ------- | ----------- |
| `GET` | HTTP-команда, указывающая, что необходимо извлечь данные из конечной точки. В этом случае используется маркер доступа OAuth. | 
| `http://localhost:50342/oauth2/token` | Конечная точка управляемых удостоверений для ресурсов Azure, где 50342 — порт по умолчанию, который можно настроить. |
| `resource` | Параметр строки запроса, указывающий URI идентификатора приложения целевого ресурса. Он также отображается в утверждении (аудитории) `aud` выданного маркера. В этом примере запрашивается маркер для доступа к Azure Resource Manager, который имеет универсальный код ресурса (URI) идентификатора приложения `https://management.azure.com/`. |
| `Metadata` | Поле заголовка HTTP-запроса, необходимое управляемым удостоверениям для ресурсов Azure для устранения рисков от атак с подделкой запроса со стороны сервера (SSRF). Должно быть присвоено значение true (все в нижнем регистре).|
| `object_id` | (Необязательно.) Параметр строки запроса, указывающий object_id управляемого удостоверения, для которого требуется токен. Требуется, если виртуальная машина имеет несколько управляемых удостоверений, назначаемых пользователем.|
| `client_id` | (Необязательно.) Параметр строки запроса, указывающий client_id управляемого удостоверения, для которого требуется токен. Требуется, если виртуальная машина имеет несколько управляемых удостоверений, назначаемых пользователем.|

Пример ответа:

```json
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

| Элемент | Описание |
| ------- | ----------- |
| `access_token` | Запрашиваемый маркер доступа. При вызове защищенного REST API маркер внедряется в поле `Authorization` заголовка запроса в качестве маркера носителя, позволяя API выполнить проверку подлинности вызывающего объекта. | 
| `refresh_token` | Не используется управляемыми удостоверениями для ресурсов Azure. |
| `expires_in` | Количество секунд, в течение которых маркер доступа является действительным, прежде чем его срок действия истечет (с момента выдачи). Время выдачи можно найти в утверждении `iat` маркера. |
| `expires_on` | Период времени после истечения срока действия маркера доступа. Дата представляется как количество секунд с 1970-01-01T0:0:0Z в формате UTC (соответствует утверждению `exp` маркера). |
| `not_before` | Период времени, когда маркер доступа вступает в силу и может быть принят. Дата представляется как количество секунд с 1970-01-01T0:0:0Z в формате UTC (соответствует утверждению `nbf` маркера). |
| `resource` | Ресурс, для которого был запрошен маркер доступа, соответствует параметру строки запроса `resource`. |
| `token_type` | Тип маркера, который является маркером доступа носителя, что значит, что ресурс может предоставлять доступ носителю этого маркера. |

## <a name="get-a-token-using-the-microsoftazureservicesappauthentication-library-for-net"></a>Получение токена с помощью библиотеки Microsoft.Azure.Services.AppAuthentication для .NET

Самый простой способ для приложений и функций .NET работать с управляемыми удостоверениями для ресурсов Azure заключается в использовании пакета Microsoft.Azure.Services.AppAuthentication. Эта библиотека также позволяет локально тестировать код на компьютере разработки с использованием учетной записи пользователя из Visual Studio, [Azure CLI](/cli/azure) или встроенной проверки подлинности Active Directory. Дополнительные сведения о параметрах локальной разработки с помощью этой библиотеки см. в [Справочник Microsoft.Azure.Services.AppAuthentication](/dotnet/api/overview/azure/service-to-service-authentication). В этом разделе показано, как начать работу с библиотекой в коде.

1. Добавьте ссылки на пакеты NuGet [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) и [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault) в приложение.

2.  Добавьте в приложение следующий код:

    ```csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;
    // ...
    var azureServiceTokenProvider = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://management.azure.com/");
    // OR
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));
    ```
    
Дополнительные сведения о пакете Microsoft.Azure.Services.AppAuthentication и операциях, которые он предоставляет, приведены в [справочнике по Microsoft.Azure.Services.AppAuthentication ](/dotnet/api/overview/azure/service-to-service-authentication) и [примере .NET, использующем службу приложений и хранилище ключей с управляемыми удостоверениями для ресурсов Azure](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet).

## <a name="get-a-token-using-c"></a>Получение маркера с использованием C#

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Web.Script.Serialization; 

// Build request to acquire managed identities for Azure resources token
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/");
request.Headers["Metadata"] = "true";
request.Method = "GET";

try
{
    // Call /token endpoint
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();

    // Pipe response Stream to a StreamReader, and extract access token
    StreamReader streamResponse = new StreamReader(response.GetResponseStream()); 
    string stringResponse = streamResponse.ReadToEnd();
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    string accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}

```

## <a name="get-a-token-using-java"></a>Получение маркера с использованием Java

Используйте эту [библиотеку JSON](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core/2.9.4) для получения маркера с помощью Java.

```Java
import java.io.*;
import java.net.*;
import com.fasterxml.jackson.core.*;
 
class GetMSIToken {
    public static void main(String[] args) throws Exception {
 
        URL msiEndpoint = new URL("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/");
        HttpURLConnection con = (HttpURLConnection) msiEndpoint.openConnection();
        con.setRequestMethod("GET");
        con.setRequestProperty("Metadata", "true");
 
        if (con.getResponseCode()!=200) {
            throw new Exception("Error calling managed identity token endpoint.");
        }
 
        InputStream responseStream = con.getInputStream();
 
        JsonFactory factory = new JsonFactory();
        JsonParser parser = factory.createParser(responseStream);
 
        while(!parser.isClosed()){
            JsonToken jsonToken = parser.nextToken();
 
            if(JsonToken.FIELD_NAME.equals(jsonToken)){
                String fieldName = parser.getCurrentName();
                jsonToken = parser.nextToken();
 
                if("access_token".equals(fieldName)){
                    String accesstoken = parser.getValueAsString();
                    System.out.println("Access Token: " + accesstoken.substring(0,5)+ "..." + accesstoken.substring(accesstoken.length()-5));
                    return;
                }
            }
        }
    }
}
```

## <a name="get-a-token-using-go"></a>Получение маркера с использованием Go

```
package main

import (
  "fmt"
  "io/ioutil"
  "net/http"
  "net/url"
  "encoding/json"
)

type responseJson struct {
  AccessToken string `json:"access_token"`
  RefreshToken string `json:"refresh_token"`
  ExpiresIn string `json:"expires_in"`
  ExpiresOn string `json:"expires_on"`
  NotBefore string `json:"not_before"`
  Resource string `json:"resource"`
  TokenType string `json:"token_type"`
}

func main() {
    
    // Create HTTP request for a managed services for Azure resources token to access Azure Resource Manager
    var msi_endpoint *url.URL
    msi_endpoint, err := url.Parse("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01")
    if err != nil {
      fmt.Println("Error creating URL: ", err)
      return 
    }
    msi_parameters := url.Values{}
    msi_parameters.Add("resource", "https://management.azure.com/")
    msi_endpoint.RawQuery = msi_parameters.Encode()
    req, err := http.NewRequest("GET", msi_endpoint.String(), nil)
    if err != nil {
      fmt.Println("Error creating HTTP request: ", err)
      return 
    }
    req.Header.Add("Metadata", "true")

    // Call managed services for Azure resources token endpoint
    client := &http.Client{}
    resp, err := client.Do(req) 
    if err != nil{
      fmt.Println("Error calling token endpoint: ", err)
      return
    }

    // Pull out response body
    responseBytes,err := ioutil.ReadAll(resp.Body)
    defer resp.Body.Close()
    if err != nil {
      fmt.Println("Error reading response body : ", err)
      return
    }

    // Unmarshall response body into struct
    var r responseJson
    err = json.Unmarshal(responseBytes, &r)
    if err != nil {
      fmt.Println("Error unmarshalling the response:", err)
      return
    }

    // Print HTTP response and marshalled response body elements to console
    fmt.Println("Response status:", resp.Status)
    fmt.Println("access_token: ", r.AccessToken)
    fmt.Println("refresh_token: ", r.RefreshToken)
    fmt.Println("expires_in: ", r.ExpiresIn)
    fmt.Println("expires_on: ", r.ExpiresOn)
    fmt.Println("not_before: ", r.NotBefore)
    fmt.Println("resource: ", r.Resource)
    fmt.Println("token_type: ", r.TokenType)
}
```

## <a name="get-a-token-using-azure-powershell"></a>Получение маркера с использованием Azure PowerShell

В следующем примере демонстрируется использование конечной точки REST управляемых удостоверений для ресурсов Azure из клиента PowerShell для выполнения следующих задач.

1. Получение маркера доступа.
2. Использование маркера доступа для вызова REST API Azure Resource Manager и получения информации о виртуальной машине. Не забудьте указать свой идентификатор подписки, имя группы ресурсов и имя виртуальной машины для `<SUBSCRIPTION-ID>`, `<RESOURCE-GROUP>` и `<VM-NAME>` соответственно.

```azurepowershell
Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Headers @{Metadata="true"}
```

Пример анализа маркера доступа, полученного из ответа:
```azurepowershell
# Get an access token for managed identities for Azure resources
$response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' `
                              -Headers @{Metadata="true"}
$content =$response.Content | ConvertFrom-Json
$access_token = $content.access_token
echo "The managed identities for Azure resources access token is $access_token"

# Use the access token to get resource information for the VM
$vmInfoRest = (Invoke-WebRequest -Uri 'https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.Compute/virtualMachines/<VM-NAME>?api-version=2017-12-01' -Method GET -ContentType "application/json" -Headers @{ Authorization ="Bearer $access_token"}).content
echo "JSON returned from call to get VM info:"
echo $vmInfoRest

```

## <a name="get-a-token-using-curl"></a>Получение маркера с использованием CURL

```bash
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true -s
```


Пример анализа маркера доступа, полученного из ответа:

```bash
response=$(curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true -s)
access_token=$(echo $response | python -c 'import sys, json; print (json.load(sys.stdin)["access_token"])')
echo The managed identities for Azure resources access token is $access_token
```

## <a name="token-caching"></a>Кэширование маркеров

Хотя используемая подсистема управляемых удостоверений для ресурсов Azure (IMDS или расширение виртуальной машины для управляемых удостоверений для ресурсов Azure) кэширует маркеры, рекомендуется также реализовать кэширование маркеров в своем коде. Поэтому следует подготовиться к сценариям, когда ресурс указывает, что срок действия маркера истек. 

Сетевые вызовы к Azure AD приносят результат, только когда:
- произошел промах кэша из-за отсутствия маркера в кэше подсистемы управляемых удостоверений для ресурсов Azure;
- истек срок действия маркера в кэше.

## <a name="error-handling"></a>Обработка ошибок

Конечная точка управляемых удостоверений для Azure сообщает об ошибках в поле кода состояния заголовка ответного сообщения HTTP в виде ошибок 4xx или 5xx.

| Код состояния | Причина ошибки | Способ устранения |
| ----------- | ------------ | ------------- |
| 404. Не найдено. | Конечная точка IMDS обновляется. | Повторите попытку с экспоненциальным увеличением задержки. См. инструкции ниже. |
| 429 — слишком много запросов |  Достигнут предел регулирования IMDS. | Повторите попытку с экспоненциальным увеличением задержки. См. инструкции ниже. |
| Ошибка 4xx в запросе. | Один или несколько параметров запроса неверные. | Не выполняйте повторную попытку.  Для получения дополнительной информации просмотрите сведения об ошибке.  Ошибки 4xx — это ошибки времени разработки.|
| Временная ошибка 5xx службы. | Подсистема управляемых удостоверений для ресурсов Azure или служба Azure Active Directory вернула временную ошибку. | По прошествии 1 секунды можно безопасно повторить попытку.  Если повторные попытки будут выполняться слишком быстро или слишком часто, IMDS и (или) Azure AD могут вернуть ошибку ограничения частоты (429).|
| timeout | Конечная точка IMDS обновляется. | Повторите попытку с экспоненциальным увеличением задержки. См. инструкции ниже. |

Если возникает ошибка, в соответствующем тексте ответа HTTP содержится код JSON с подробностями об ошибке:

| Элемент | Описание |
| ------- | ----------- |
| Ошибка   | Идентификатор ошибки. |
| error_description | Подробное описание ошибки. **Описания ошибок могут изменяться в любое время. Не создавайте код, который выполняет ветвление на основе значений в описании ошибки.**|

### <a name="http-response-reference"></a>Ссылка ответа HTTP

В этом разделе описываются возможные сообщения об ошибках. Состояние "200 ОК" — это успешный ответ, а маркер доступа содержится в коде JSON текста ответа в элементе access_token.

| Код состояния | Ошибка | Описание ошибки | Решение |
| ----------- | ----- | ----------------- | -------- |
| 400 — недопустимый запрос | invalid_resource | AADSTS50001: приложение с именем *\<URI\>* не найдено в клиенте с именем *\<TENANT-ID\>* . Это может произойти, если приложение установил не администратор клиента или если пользователь в клиенте не предоставил к нему разрешение. Возможно, вы отправили запрос на проверку подлинности не тому клиенту. | (только для Linux) |
| 400 — недопустимый запрос | bad_request_102 | Необходимый заголовок метаданных не указан | Поле заголовка запроса `Metadata` отсутствует или имеет неправильный формат. Должно быть указано значение `true` в нижнем регистре. В качестве примера см. "Пример запроса" в предыдущем разделе для REST.|
| 401 — недостаточно прав | unknown_source | Неизвестный источник *\<URI\>* | Убедитесь, что универсальный код ресурса (URI) запроса HTTP GET имеет правильный формат. Часть `scheme:host/resource-path` должна быть указана как `http://localhost:50342/oauth2/token`. В качестве примера см. "Пример запроса" в предыдущем разделе для REST.|
|           | invalid_request | В запросе отсутствует требуемый параметр, он включает недопустимое значение параметра, параметр указан несколько раз или как-то искажен. |  |
|           | unauthorized_client | Клиент не авторизован для запроса маркера доступа с помощью этого метода. | Вызвано запросом, который не использовал локальное замыкание на себя для вызова расширения или на виртуальной машине без управляемых удостоверений для ресурсов Azure, настроенных правильно. Если вам нужна помощь с конфигурацией виртуальной машины, ознакомьтесь с разделом [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине с помощью портала Azure](qs-configure-portal-windows-vm.md). |
|           | access_denied | Владелец ресурса или сервер авторизации отклонил запрос. |  |
|           | unsupported_response_type | Сервер авторизации не поддерживает получение маркера доступа с помощью этого метода. |  |
|           | invalid_scope | Запрошенная область недопустима, неизвестна или неверна. |  |
| 500 — внутренняя ошибка сервера | неизвестно | Не удалось извлечь маркер из Active Directory. Дополнительные сведения см. в статье вход в систему. *\<file path\>* | Убедитесь, что на виртуальной машине включены управляемые удостоверения для ресурсов Azure. Если вам нужна помощь с конфигурацией виртуальной машины, ознакомьтесь с разделом [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине с помощью портала Azure](qs-configure-portal-windows-vm.md).<br><br>Кроме того, убедитесь, что универсальный код ресурса (URI) запроса HTTP GET отформатирован правильно, в частности универсальный код ресурса (URI), указанный в строке запроса. В качестве примера см. "Пример запроса" в предыдущем разделе для REST или раздел [Службы Azure, поддерживающие аутентификацию Azure AD](./services-support-managed-identities.md) для получения списка служб и соответствующих идентификаторов ресурсов.

## <a name="retry-guidance"></a>Рекомендации по использованию механизма повторов 

Если вы получите код с ошибкой 404, 429 или 5xx, советуем повторить попытку (сведения см. в разделе [Обработка ошибок](#error-handling) выше).

Ограничения регулирования применяются к числу вызовов к конечной точке IMDS. Если превышен порог регулирования, конечная точка IMDS ограничивает каждый последующий запрос, пока действует регулирование. В течение этого периода конечная точка IMDS возвращает код состояния HTTP 429 ("Too many requests" (Слишком много запросов)), и запросы не выполняются. 

Для повторных попыток рекомендуем следующую стратегию: 

| **Стратегия повторов** | **Параметры** | **Значения** | **Принцип работы** |
| --- | --- | --- | --- |
|ExponentialBackoff |Число повторных попыток<br />Минимальная задержка<br />Максимальная задержка<br />Разностная задержка<br />Первый быстрый повтор |5<br />0 с<br />60 с<br />2 с<br />false |Попытка 1 — задержка 0 с<br />Попытка 2 — задержка 2 с<br />Попытка 3 — задержка 6 с<br />Попытка 4 — задержка 14 с<br />Попытка 5 — задержка 30 с |

## <a name="resource-ids-for-azure-services"></a>Идентификаторы ресурсов для служб Azure

Список ресурсов, которые поддерживают Azure AD и были протестированы с управляемыми удостоверениями для ресурсов Azure и соответствующими идентификаторами ресурсов, приведен в разделе [Службы Azure, поддерживающие аутентификацию Azure AD](./services-support-managed-identities.md).


## <a name="next-steps"></a>Дальнейшие действия

- Чтобы включить управляемые удостоверения для ресурсов Azure на виртуальной машине Azure, ознакомьтесь с разделом [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине с помощью портала Azure](qs-configure-portal-windows-vm.md).