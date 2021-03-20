---
title: Использование частных конечных точек с голосовыми службами
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать службы речи с частными конечными точками, предоставляемыми частной ссылкой Azure
services: cognitive-services
author: alexeyo26
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/04/2021
ms.author: alexeyo
ms.openlocfilehash: c9af0cda14261e8eab7f1ecc05c50a289d7ddfdb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99559662"
---
# <a name="use-speech-services-through-a-private-endpoint"></a>Использование речевых служб через закрытую конечную точку

[Частная ссылка Azure](../../private-link/private-link-overview.md) позволяет подключаться к службам в Azure с помощью [частной конечной точки](../../private-link/private-endpoint-overview.md). Частная конечная точка — это частный IP-адрес, доступный только в пределах определенной [виртуальной сети](../../virtual-network/virtual-networks-overview.md) и подсети.

В этой статье объясняется, как настроить и использовать закрытые ссылки и частные конечные точки с голосовыми службами в Azure Cognitive Services.

> [!NOTE]
> Прежде чем продолжить, изучите, [как использовать виртуальные сети с Cognitive Services](../cognitive-services-virtual-networks.md).

В этой статье также описано [, как удалить частные конечные точки позже, но по-прежнему использовать речевой ресурс](#use-a-speech-resource-with-a-custom-domain-name-and-without-private-endpoints).

## <a name="create-a-custom-domain-name"></a>Создание пользовательского доменного имени

Частным конечным точкам требуется [имя пользовательского поддомена для Cognitive Services](../cognitive-services-custom-subdomains.md). Используйте следующие инструкции, чтобы создать его для своего речевого ресурса.

> [!WARNING]
> В речевом ресурсе с включенным именем пользовательского домена используется другой способ взаимодействия со службами речи. Может потребоваться настроить код приложения для обоих сценариев: [Частная конечная точка](#use-a-speech-resource-with-a-custom-domain-name-and-a-private-endpoint-enabled) включена, а [ *не* частная конечная точка](#use-a-speech-resource-with-a-custom-domain-name-and-without-private-endpoints).
>
> При включении имени личного домена операция [необратима](../cognitive-services-custom-subdomains.md#can-i-change-a-custom-domain-name). Единственный способ вернуться к [региональному имени](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) — создать новый речевой ресурс.
>
> Если у вашего речевого ресурса много связанных пользовательских моделей и проектов, созданных с помощью [Speech Studio](https://speech.microsoft.com/), мы настоятельно рекомендуем выполнить настройку с помощью тестового ресурса, прежде чем изменять ресурс, используемый в рабочей среде.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы создать пользовательское доменное имя с помощью портал Azure, выполните следующие действия.

1. Откройте [портал Azure](https://portal.azure.com/) и войдите в свою учетную запись Azure.
1. Выберите необходимый речевой ресурс.
1. В группе **Управление ресурсами** на левой панели выберите **сеть**.
1. На вкладке **брандмауэры и виртуальные сети** выберите **создать пользовательское доменное имя**. Появится новая правая панель с инструкциями по созданию уникального пользовательского поддомена для ресурса.
1. На панели **создать имя пользовательского домена** введите имя личного домена. Ваш полный личный домен будет выглядеть следующим образом: `https://{your custom name}.cognitiveservices.azure.com` . 
    
    Помните, что после создания имени пользовательского домена его _нельзя_ изменить.
    
    Введя имя личного домена, нажмите кнопку **сохранить**.
1. После завершения операции в группе **Управление ресурсами** выберите **ключи и конечная точка**. Убедитесь, что новое имя конечной точки ресурса запускается таким образом: `https://{your custom name}.cognitiveservices.azure.com` .

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы создать пользовательское доменное имя с помощью PowerShell, убедитесь, что на компьютере установлен PowerShell версии 7. x или более поздней версии с Azure PowerShellным модулем 5.1.0 или более поздней версией. Чтобы просмотреть версии этих средств, выполните следующие действия.

1. В окне PowerShell введите:

    `$PSVersionTable`

    Убедитесь, что `PSVersion` значение равно 7. x или более поздней версии. Чтобы обновить PowerShell, следуйте инструкциям по [установке различных версий PowerShell](/powershell/scripting/install/installing-powershell).

1. В окне PowerShell введите:

    `Get-Module -ListAvailable Az`

    Если ничего не отображается или если эта версия модуля Azure PowerShell более ранняя, чем 5.1.0, следуйте инструкциям по [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps) для обновления.

Прежде чем продолжить, выполните команду, `Connect-AzAccount` чтобы создать подключение к Azure.

## <a name="verify-that-a-custom-domain-name-is-available"></a>Убедитесь, что пользовательское доменное имя доступно.

Проверьте, доступен ли личный домен, который вы хотите использовать. Следующий код подтверждает, что домен доступен с помощью операции [проверки доступности домена](/rest/api/cognitiveservices/accountmanagement/checkdomainavailability/checkdomainavailability) в Cognitive Services REST API.

> [!TIP]
> Следующий код *не* будет работать в Azure Cloud Shell.

```azurepowershell
$subId = "Your Azure subscription Id"
$subdomainName = "custom domain name"

# Select the Azure subscription that contains the Speech resource.
# You can skip this step if your Azure account has only one active subscription.
Set-AzContext -SubscriptionId $subId

# Prepare the OAuth token to use in the request to the Cognitive Services REST API.
$Context = Get-AzContext
$AccessToken = (Get-AzAccessToken -TenantId $Context.Tenant.Id).Token
$token = ConvertTo-SecureString -String $AccessToken -AsPlainText -Force

# Prepare and send the request to the Cognitive Services REST API.
$uri = "https://management.azure.com/subscriptions/" + $subId + `
    "/providers/Microsoft.CognitiveServices/checkDomainAvailability?api-version=2017-04-18"
$body = @{
subdomainName = $subdomainName
type = "Microsoft.CognitiveServices/accounts"
}
$jsonBody = $body | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri $uri -ContentType "application/json" -Authentication Bearer `
    -Token $token -Body $jsonBody | Format-List
```
Если требуемое имя доступно, вы увидите следующий ответ:
```azurepowershell
isSubdomainAvailable : True
reason               :
type                 :
subdomainName        : my-custom-name
```
Если имя уже используется, вы увидите следующий ответ:
```azurepowershell
isSubdomainAvailable : False
reason               : Sub domain name 'my-custom-name' is already used. Please pick a different name.
type                 :
subdomainName        : my-custom-name
```
## <a name="create-your-custom-domain-name"></a>Создание пользовательского доменного имени

Чтобы включить пользовательское доменное имя для выбранного ресурса речи, используйте командлет [Set-азкогнитивесервицесаккаунт](/powershell/module/az.cognitiveservices/set-azcognitiveservicesaccount) .

> [!WARNING]
> После успешного выполнения следующего кода вы создадите пользовательское доменное имя для своего ресурса речи. Помните, что это имя *нельзя* изменить.

```azurepowershell
$resourceGroup = "Resource group name where Speech resource is located"
$speechResourceName = "Your Speech resource name"
$subdomainName = "custom domain name"

# Select the Azure subscription that contains the Speech resource.
# You can skip this step if your Azure account has only one active subscription.
$subId = "Your Azure subscription Id"
Set-AzContext -SubscriptionId $subId

# Set the custom domain name to the selected resource.
# WARNING: THIS CANNOT BE CHANGED OR UNDONE!
Set-AzCognitiveServicesAccount -ResourceGroupName $resourceGroup `
    -Name $speechResourceName -CustomSubdomainName $subdomainName
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

Для работы с этим разделом требуется последняя версия Azure CLI. Если вы используете Azure Cloud Shell, то последняя версия уже установлена.

## <a name="verify-that-the-custom-domain-name-is-available"></a>Убедитесь, что пользовательское доменное имя доступно.

Проверьте, не является ли личный домен, который вы хотите использовать, бесплатным. Используйте метод [проверки доступности домена](/rest/api/cognitiveservices/accountmanagement/checkdomainavailability/checkdomainavailability) из REST API Cognitive Services.

Скопируйте следующий блок кода, вставьте предпочитаемое имя личного домена и сохраните его в файл `subdomain.json` .

```json
{
    "subdomainName": "custom domain name",
    "type": "Microsoft.CognitiveServices/accounts"
}
```

Скопируйте файл в текущую папку или отправьте его в Azure Cloud Shell и выполните следующую команду. Замените `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` идентификатором своей подписки Azure.

```azurecli-interactive
az rest --method post --url "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.CognitiveServices/checkDomainAvailability?api-version=2017-04-18" --body @subdomain.json
```
Если требуемое имя доступно, вы увидите следующий ответ:
```azurecli
{
  "isSubdomainAvailable": true,
  "reason": null,
  "subdomainName": "my-custom-name",
  "type": null
}
```

Если имя уже используется, вы увидите следующий ответ:
```azurecli
{
  "isSubdomainAvailable": false,
  "reason": "Sub domain name 'my-custom-name' is already used. Please pick a different name.",
  "subdomainName": "my-custom-name",
  "type": null
}
```
## <a name="enable-a-custom-domain-name"></a>Включение пользовательского доменного имени

Чтобы включить пользовательское доменное имя для выбранного ресурса речи, используйте команду [AZ cognitiveservices Account Update](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_update) .

Выберите подписку Azure, которая содержит ресурс речи. Если ваша учетная запись Azure имеет только одну активную подписку, этот шаг можно пропустить. Замените `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` идентификатором своей подписки Azure.
```azurecli-interactive
az account set --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```
Задайте имя пользовательского домена для выбранного ресурса. Замените примеры значений параметров на фактические и выполните следующую команду.

> [!WARNING]
> После успешного выполнения следующей команды вы создадите пользовательское доменное имя для своего ресурса речи. Помните, что это имя *нельзя* изменить.

```azurecli
az cognitiveservices account update --name my-speech-resource-name --resource-group my-resource-group-name --custom-domain my-custom-name
```

***

## <a name="enable-private-endpoints"></a>Включить частные конечные точки

Мы рекомендуем использовать [закрытую зону DNS](../../dns/private-dns-overview.md) , подключенную к виртуальной сети, с необходимыми обновлениями для частных конечных точек. В процессе подготовки вы по умолчанию создаете частную зону DNS. Если вы используете собственный DNS-сервер, вам также может потребоваться изменить конфигурацию DNS. 

*Прежде чем* подготавливать частные конечные точки для рабочего речевого ресурса, примите решение о стратегии DNS. И протестируйте изменения DNS, особенно если вы используете собственный DNS-сервер.

Для создания частных конечных точек используйте одну из следующих статей. В этих статьях для включения в частные конечные точки используется веб-приложение в качестве примера ресурса.

- [Создание частной конечной точки с помощью портала Azure](../../private-link/create-private-endpoint-portal.md)
- [Создание частной конечной точки с помощью Azure PowerShell](../../private-link/create-private-endpoint-powershell.md)
- [Создание частной конечной точки с помощью Azure CLI](../../private-link/create-private-endpoint-cli.md)

Используйте эти параметры вместо параметров в выбранной статье:

| Параметр             | Значение                                    |
|---------------------|------------------------------------------|
| Тип ресурса       | **Microsoft. CognitiveServices/учетные записи** |
| Ресурс            | **\<your-speech-resource-name>**         |
| Целевой подресурс | **учетная запись**                              |

**DNS для частных конечных точек:** Ознакомьтесь с общими принципами [DNS для частных конечных точек в Cognitive Servicesных ресурсах](../cognitive-services-virtual-networks.md#dns-changes-for-private-endpoints). Затем убедитесь, что конфигурация DNS работает правильно, выполнив проверки, описанные в следующих разделах.

### <a name="resolve-dns-from-the-virtual-network"></a>Разрешение DNS из виртуальной сети

Эта проверка *обязательна*.

Выполните следующие действия, чтобы проверить пользовательскую запись DNS из виртуальной сети.

1. Войдите в виртуальную машину, расположенную в виртуальной сети, к которой присоединена частная конечная точка. 
1. Откройте командную строку Windows или оболочку Bash, запустите `nslookup` и убедитесь, что она успешно разрешает имя личного домена ресурса.

   ```dos
   C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
   Server:  UnKnown
   Address:  168.63.129.16

   Non-authoritative answer:
   Name:    my-private-link-speech.privatelink.cognitiveservices.azure.com
   Address:  172.28.0.10
   Aliases:  my-private-link-speech.cognitiveservices.azure.com
   ```

1. Убедитесь, что IP-адрес соответствует IP-адресу частной конечной точки.

### <a name="resolve-dns-from-other-networks"></a>Разрешение DNS-имен из других сетей

Выполните эту проверку, только если вы включили параметр " **все сети** " или " **Выбранные сети и доступ к частным конечным точкам** " в разделе " **Сетевые подключения** " ресурса. 

Если вы планируете доступ к ресурсу только с помощью частной конечной точки, этот раздел можно пропустить.

1. Войдите на компьютер, подключенный к сети, которой разрешен доступ к ресурсу.
2. Откройте командную строку Windows или оболочку Bash, запустите `nslookup` и убедитесь, что она успешно разрешает имя личного домена ресурса.

   ```dos
   C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
   Server:  UnKnown
   Address:  fe80::1

   Non-authoritative answer:
   Name:    vnetproxyv1-weu-prod.westeurope.cloudapp.azure.com
   Address:  13.69.67.71
   Aliases:  my-private-link-speech.cognitiveservices.azure.com
             my-private-link-speech.privatelink.cognitiveservices.azure.com
             westeurope.prod.vnet.cog.trafficmanager.net
   ```

> [!NOTE]
> Разрешенный IP-адрес указывает на конечную точку прокси виртуальной сети, которая передает сетевой трафик в частную конечную точку для ресурса Cognitive Services. Поведение будет отличаться для ресурса с именем личного домена, но *без* закрытых конечных точек. Дополнительные сведения см. в [этом разделе](#dns-configuration) .

## <a name="adjust-existing-applications-and-solutions"></a>Настройка существующих приложений и решений

Речевой ресурс с включенным пользовательским доменом использует другой способ взаимодействия со службами речи. Это справедливо для ресурса речи с поддержкой пользовательского домена как с закрытыми конечными точками, так и без них. Сведения в этом разделе относятся к обоим сценариям.

### <a name="use-a-speech-resource-with-a-custom-domain-name-and-a-private-endpoint-enabled"></a>Использование речевого ресурса с пользовательским именем домена и частной конечной точкой

В речевом ресурсе с пользовательским именем домена и частной конечной точкой используется другой способ взаимодействия со службами речи. В этом разделе объясняется, как использовать такой ресурс с интерфейсами API и [пакетом SDK](speech-sdk.md)для речевых служб.

> [!NOTE]
> Речевой ресурс без закрытых конечных точек, но с включенным пользовательским именем домена также имеет особый способ взаимодействия со службами речи. Этот способ отличается от сценария для речевого ресурса с поддержкой закрытой конечной точки. Если у вас есть такой ресурс (например, ресурс с частными конечными точками, но затем решил удалить их), см. раздел [использование речевого ресурса с пользовательским именем домена и без закрытых конечных точек](#use-a-speech-resource-with-a-custom-domain-name-and-without-private-endpoints).

#### <a name="speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-rest-apis"></a>Речевой ресурс с пользовательским именем домена и частной конечной точкой: использование с API-интерфейсами RESTFUL

`my-private-link-speech.cognitiveservices.azure.com`Для этого раздела мы будем использовать в качестве примера DNS-имя ресурса речи (личный домен).

В речевых службах есть интерфейсы API для преобразования [речи в текст](rest-speech-to-text.md) и преобразования [текста в речь](rest-text-to-speech.md). Примите во внимание следующие сведения о сценарии с поддержкой частных конечных точек.

Речь-to-Text имеет два интерфейса API для других интерфейсов. Каждый API-интерфейс служит другой целью, использует разные конечные точки и требует другого подхода при использовании в сценарии с поддержкой закрытой конечной точки.

API-интерфейсы RESTFUL для преобразования речи в текст:
- Преобразование [речи в текст REST API v 3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30), используемое для записи [пакетов](batch-transcription.md) и [пользовательское распознавание речи](custom-speech-overview.md). Версия 3.0 является [последователем версии 2.0](./migrate-v2-to-v3.md)
- [REST API преобразования речи в текст для коротких аудиофайлов](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio), используемых для записи в Интернете 

Использование REST API преобразования речи в текст для коротких аудио и преобразования текста в REST API речь в сценарии частной конечной точки аналогично. Это эквивалентно [случаю речевого пакета SDK](#speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-speech-sdk) , описанного далее в этой статье. 

Преобразование речи в текст REST API v 3.0 использует другой набор конечных точек, поэтому для сценария с поддержкой закрытой конечной точки требуется другой подход.

В следующих подразделах описываются оба варианта.

##### <a name="speech-to-text-rest-api-v30"></a>Преобразование речи в текст REST API v 3.0

Обычно речевые ресурсы используют [Cognitive Services региональных конечных точек](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) для взаимодействия с [речью в тексте REST API v 3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30). Эти ресурсы имеют следующий формат именования: <p/>`{region}.api.cognitive.microsoft.com`.

Вот пример URL-адреса запроса:

```http
https://westeurope.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions
```

> [!NOTE]
> Ознакомьтесь с [этой статьей](sovereign-clouds.md) для конечных точек Azure для государственных организаций и Azure для Китая.

После включения пользовательского домена для речевого ресурса (который необходим для частных конечных точек) этот ресурс будет использовать следующий шаблон DNS-имени для базовой конечной точки REST API: <p/>`{your custom name}.cognitiveservices.azure.com`.

Это означает, что в нашем примере имя конечной точки REST API будет следующим: <p/>`my-private-link-speech.cognitiveservices.azure.com`.

И URL-адрес образца запроса необходимо преобразовать в:
```http
https://my-private-link-speech.cognitiveservices.azure.com/speechtotext/v3.0/transcriptions
```
Этот URL-адрес должен быть доступен из виртуальной сети с подключенной частной конечной точкой (при условии, что [разрешение DNS указано правильно](#resolve-dns-from-the-virtual-network)).

После включения пользовательского доменного имени для речевого ресурса обычно имя узла заменяется во всех URL-адресах запросов на новое имя узла пользовательского домена. Все остальные части запроса (например, путь `/speechtotext/v3.0/transcriptions` в предыдущем примере) остаются неизменными.

> [!TIP]
> Некоторые клиенты разрабатывают приложения, использующие регион в DNS-имени региональной конечной точки (например, для отправки запроса в ресурс речи, развернутый в определенном регионе Azure).
>
> Личный домен для речевого ресурса *не* содержит сведений о регионе, в котором развернут ресурс. Поэтому логика приложения, описанная выше, *не* будет работать и должна быть изменена.

##### <a name="speech-to-text-rest-api-for-short-audio-and-text-to-speech-rest-api"></a>REST API преобразования речи в текст для коротких REST API аудио и текста в речь

[REST API речи в текст для коротких аудио](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) и [текста в речь REST API](rest-text-to-speech.md) использовать два типа конечных точек:
- [Cognitive Services региональные конечные точки](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) для взаимодействия с Cognitive Services REST API для получения маркера авторизации
- Специальные конечные точки для всех других операций

> [!NOTE]
> Ознакомьтесь с [этой статьей](sovereign-clouds.md) для конечных точек Azure для государственных организаций и Azure для Китая.

Подробное описание специальных конечных точек и способ преобразования их URL-адресов в закрытые конечные точки приведены в [этом подразделе](#construct-endpoint-url) об использовании с пакетом SDK для распознавания речи. Тот же принцип, который описан для пакета SDK, относится к REST APIм преобразования речи в текст для коротких аудио и REST API текста в речь.

Ознакомьтесь с материалами в подразделе, упомянутом в предыдущем абзаце, и см. Следующий пример. В примере описывается REST API преобразования текста в речь. Использование REST API преобразования речи в текст для коротких аудиозаписей полностью эквивалентно.

> [!NOTE]
> При использовании REST API преобразования текста в речь для коротких аудио-и текстовых REST API в сценариях частных конечных точек используйте ключ подписки, передаваемый через `Ocp-Apim-Subscription-Key` заголовок. (См. Дополнительные сведения о [REST APIе преобразования речи в текст для коротких аудио](rest-speech-to-text.md#request-headers) и [текста в речь REST API](rest-text-to-speech.md#request-headers))
>
> Использование маркера авторизации и его передача в специальную конечную точку через `Authorization` заголовок будет работать *только* в том случае, если включен параметр " **все сети** " в разделе " **Сетевые подключения** " вашего речевого ресурса. В других случаях при `Forbidden` `BadRequest` попытке получить маркер авторизации возникнет ошибка или.

**Пример использования преобразования текста в речь REST API**

В качестве примера региона Azure мы будем использовать Западная Европа, а в `my-private-link-speech.cognitiveservices.azure.com` качестве примера имя DNS-имени ресурса речи (личный домен). Имя личного домена `my-private-link-speech.cognitiveservices.azure.com` в нашем примере принадлежит к речевому ресурсу, созданному в регионе Западной Европы.

Чтобы получить список голосов, поддерживаемых в регионе, выполните следующий запрос:

```http
https://westeurope.tts.speech.microsoft.com/cognitiveservices/voices/list
```
Дополнительные сведения см. в [документации REST API текста в речь](rest-text-to-speech.md).

Для ресурса речи с поддержкой закрытой конечной точки необходимо изменить URL-адрес конечной точки для той же операции. Тот же запрос будет выглядеть следующим образом:

```http
https://my-private-link-speech.cognitiveservices.azure.com/tts/cognitiveservices/voices/list
```
См. подробное описание в подразделе [Создание URL-адреса конечной точки](#construct-endpoint-url) для пакета SDK для распознавания речи.

#### <a name="speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-speech-sdk"></a>Речевой ресурс с пользовательским именем домена и частной конечной точкой: использование с пакетом SDK для распознавания речи

Для использования речевого пакета SDK с пользовательским именем домена и речевыми ресурсами, поддерживающими закрытые конечные точки, необходимо проверить и, скорее всего, изменить код приложения.

`my-private-link-speech.cognitiveservices.azure.com`Для этого раздела мы будем использовать в качестве примера DNS-имя ресурса речи (личный домен).

##### <a name="construct-endpoint-url"></a>Создание URL-адреса конечной точки

Обычно в сценариях пакета SDK (а также в REST APIе преобразования текста в речь для коротких речевых и текстовых REST API сценариев) в речевых ресурсах используются выделенные региональные конечные точки для различных предложений услуг. Для этих конечных точек используется формат DNS-имен:

`{region}.{speech service offering}.speech.microsoft.com`

Пример DNS-имени:

`westeurope.stt.speech.microsoft.com`

Все возможные значения для региона (первый элемент DNS-имени) перечислены в [области Поддерживаемые службы речи](regions.md). (Дополнительные сведения об Azure для государственных организаций и Azure для Китая см. в [этой статье](sovereign-clouds.md) .) В следующей таблице представлены возможные значения для предложения служб речи (второй элемент DNS-имени):

| Значение DNS-имени | Предложение службы речи                                    |
|----------------|-------------------------------------------------------------|
| `commands`     | [Пользовательские команды](custom-commands.md)                       |
| `convai`       | [Транскрибирование бесед](conversation-transcription.md) |
| `s2s`          | [Перевод речи](speech-translation.md)                 |
| `stt`          | [Преобразование речи в текст](speech-to-text.md)                         |
| `tts`          | [Преобразование текста в речь](text-to-speech.md)                         |
| `voice`        | [Custom Voice](how-to-custom-voice.md) (Пользовательские голосовые модели)                      |

Поэтому предыдущий пример ( `westeurope.stt.speech.microsoft.com` ) означает конечную точку преобразования речи в текст в Западной Европе.

Конечные точки с поддержкой частных конечных точек взаимодействуют с голосовыми службами через специальный прокси-сервер. Поэтому *необходимо изменить URL-адреса подключения конечной точки*. 

URL-адрес конечной точки "Стандартный" выглядит следующим образом: <p/>`{region}.{speech service offering}.speech.microsoft.com/{URL path}`

URL-адрес частной конечной точки выглядит следующим образом: <p/>`{your custom name}.cognitiveservices.azure.com/{speech service offering}/{URL path}`

**Пример 1.** Приложение взаимодействует с использованием следующего URL-адреса (распознавание речи с использованием базовой модели для английского языка США в Западной Европе):

```
wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```

Чтобы использовать его в сценарии с поддержкой частных конечных точек, когда имя пользовательского домена для ресурса речи — `my-private-link-speech.cognitiveservices.azure.com` , необходимо изменить URL-адрес следующим образом:

```
wss://my-private-link-speech.cognitiveservices.azure.com/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```

Обратите внимание на сведения:

- Имя узла `westeurope.stt.speech.microsoft.com` заменяется именем узла пользовательского домена `my-private-link-speech.cognitiveservices.azure.com` .
- Второй элемент исходного DNS-имени ( `stt` ) преобразуется в первый элемент URL-пути и предшествует исходному пути. Поэтому исходный URL-адрес `/speech/recognition/conversation/cognitiveservices/v1?language=en-US` станет `/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US` .

**Пример 2.** Приложение использует следующий URL-адрес для синтезирования речи в Западной Европе с помощью настраиваемой голосовой модели:
```http
https://westeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId=974481cc-b769-4b29-af70-2fb557b897c4
```

Следующий эквивалентный URL-адрес использует закрытую конечную точку, где пользовательское доменное имя ресурса речи `my-private-link-speech.cognitiveservices.azure.com` :

```http
https://my-private-link-speech.cognitiveservices.azure.com/voice/cognitiveservices/v1?deploymentId=974481cc-b769-4b29-af70-2fb557b897c4
```

Применяется тот же принцип, что и в примере 1, но элемент key на данный момент равен `voice` .

##### <a name="modifying-applications"></a>Изменение приложений

Чтобы изменить код, выполните следующие действия.

1. Определите URL-адрес конечной точки приложения:

   - [Включите ведение журнала для приложения](how-to-use-logging.md) и запустите его для ведения журнала действий.
   - В файле журнала выполните поиск по запросу `SPEECH-ConnectionUrl` . В соответствующих строках `value` параметр содержит полный URL-адрес, который приложение использует для доступа к речевым службам.

   Пример

   ```
   (114917): 41ms SPX_DBG_TRACE_VERBOSE:  property_bag_impl.cpp:138 ISpxPropertyBagImpl::LogPropertyAndValue: this=0x0000028FE4809D78; name='SPEECH-ConnectionUrl'; value='wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?traffictype=spx&language=en-US'
   ```

   Итак, URL-адрес, используемый приложением в этом примере:

   ```
   wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US
   ```

2. Создайте `SpeechConfig` экземпляр с помощью полного URL-адреса конечной точки:

   1. Измените только что определенную конечную точку, как описано в разделе [URL-адрес конечной точки конструкции](#construct-endpoint-url) выше.

   1. Измените метод создания экземпляра `SpeechConfig` . Скорее всего, приложение использует нечто вроде:
      ```csharp
      var config = SpeechConfig.FromSubscription(subscriptionKey, azureRegion);
      ```
      Это не сработает для ресурса речи с закрытой конечной точкой из-за изменений имени узла и URL-адреса, описанных в предыдущих разделах. Если вы попытаетесь запустить существующее приложение без каких-либо изменений с помощью ключа ресурса с закрытой конечной точкой, вы получите ошибку проверки подлинности (401).

      Чтобы сделать его работоспособным, измените способ создания экземпляра `SpeechConfig` класса и используйте инициализацию "из конечной точки"/"с конечной точкой". Предположим, что определены следующие две переменные:
      - `subscriptionKey` содержит ключ ресурса речи с закрытой конечной точкой.
      - `endPoint` содержит полный *измененный* URL-адрес конечной точки (с использованием типа, необходимого для соответствующего языка программирования). В нашем примере эта переменная должна содержать:
        ```
        wss://my-private-link-speech.cognitiveservices.azure.com/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US
        ```

      Создайте экземпляр `SpeechConfig`:
      ```csharp
      var config = SpeechConfig.FromEndpoint(endPoint, subscriptionKey);
      ```
      ```cpp
      auto config = SpeechConfig::FromEndpoint(endPoint, subscriptionKey);
      ```
      ```java
      SpeechConfig config = SpeechConfig.fromEndpoint(endPoint, subscriptionKey);
      ```
      ```python
      import azure.cognitiveservices.speech as speechsdk
      speech_config = speechsdk.SpeechConfig(endpoint=endPoint, subscription=subscriptionKey)
      ```
      ```objectivec
      SPXSpeechConfiguration *speechConfig = [[SPXSpeechConfiguration alloc] initWithEndpoint:endPoint subscription:subscriptionKey];
      ```

> [!TIP]
> Параметры запроса, указанные в URI конечной точки, не изменяются, даже если они задаются другими API. Например, если язык распознавания определен в URI как параметр запроса `language=en-US` и для него также задано значение `ru-RU` через соответствующее свойство, используется языковой стандарт URI. Затем действующий язык `en-US` .
>
> Параметры, заданные в URI конечной точки, всегда имеют приоритет. Другие API-интерфейсы могут переопределять только те параметры, которые не указаны в универсальном коде ресурса (URI) конечной точки.

После этого приложение должно работать с речевыми ресурсами с поддержкой частных конечных точек. Мы работаем над более простой поддержкой сценариев частных конечных точек.

### <a name="use-a-speech-resource-with-a-custom-domain-name-and-without-private-endpoints"></a>Использование речевого ресурса с пользовательским именем домена и без закрытых конечных точек

В этой статье мы указали несколько раз, что включение пользовательского домена для речевого ресурса *необратимо*. Такой ресурс будет использовать другой способ связи со службами речи, по сравнению с теми, которые используют [имена региональных конечных точек](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints).

В этом разделе объясняется, как использовать речевой ресурс с включенным пользовательским именем домена, но *без* частных конечных точек с интерфейсами API-интерфейса и [речевым пакетом SDK](speech-sdk.md)для речевых служб. Это может быть ресурс, который был бы использован в сценарии частной конечной точки, но в нем были удалены частные конечные точки.

#### <a name="dns-configuration"></a>Настройка DNS

Помните, как DNS-имя личного домена с поддержкой закрытых конечных точек [разрешается из общедоступных сетей](#resolve-dns-from-other-networks). В этом случае IP-адрес, который разрешен, указывает на конечную точку прокси-сервера для виртуальной сети. Эта конечная точка используется для диспетчеризации сетевого трафика в ресурс Cognitive Services с поддержкой частных конечных точек.

Однако при удалении *всех* частных конечных точек ресурса (или непосредственно после включения имени личного домена) запись CNAME для речевого ресурса повторно подготавливается. Теперь он указывает на IP-адрес соответствующей [Cognitive Services региональной конечной точки](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints).

Поэтому выходные данные `nslookup` команды будут выглядеть следующим образом:
```dos
C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
Server:  UnKnown
Address:  fe80::1

Non-authoritative answer:
Name:    apimgmthskquihpkz6d90kmhvnabrx3ms3pdubscpdfk1tsx3a.cloudapp.net
Address:  13.93.122.1
Aliases:  my-private-link-speech.cognitiveservices.azure.com
          westeurope.api.cognitive.microsoft.com
          cognitiveweprod.trafficmanager.net
          cognitiveweprod.azure-api.net
          apimgmttmdjylckcx6clmh2isu2wr38uqzm63s8n4ub2y3e6xs.trafficmanager.net
          cognitiveweprod-westeurope-01.regional.azure-api.net
```
Сравните его с выходными данными [этого раздела](#resolve-dns-from-other-networks).

#### <a name="speech-resource-with-a-custom-domain-name-and-without-private-endpoints-usage-with-the-rest-apis"></a>Речевой ресурс с пользовательским именем домена и без закрытых конечных точек: использование с API-интерфейсами RESTFUL

##### <a name="speech-to-text-rest-api-v30"></a>Преобразование речи в текст REST API v 3.0

Использование преобразования речи в текст REST API v 3.0 полностью эквивалентно случаю [речевых ресурсов с поддержкой частных конечных точек](#speech-to-text-rest-api-v30).

##### <a name="speech-to-text-rest-api-for-short-audio-and-text-to-speech-rest-api"></a>REST API преобразования речи в текст для коротких REST API аудио и текста в речь

В этом случае использование преобразования речи в текст REST API для короткого аудио и использования REST API не имеет отличий от общего варианта, за одним исключением. (См. следующее примечание.) Следует использовать оба интерфейса API, как описано в REST API преобразования текста в речь [для коротких аудио](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) -и [речевых REST API](rest-text-to-speech.md) документации.

> [!NOTE]
> При использовании REST API речи в текст для коротких аудио и текста в речь REST API в сценариях с пользовательскими доменами используйте ключ подписки, передаваемый через `Ocp-Apim-Subscription-Key` заголовок. (См. Дополнительные сведения о [REST APIе преобразования речи в текст для коротких аудио](rest-speech-to-text.md#request-headers) и [текста в речь REST API](rest-text-to-speech.md#request-headers))
>
> Использование маркера авторизации и его передача в специальную конечную точку через `Authorization` заголовок будет работать *только* в том случае, если включен параметр " **все сети** " в разделе " **Сетевые подключения** " вашего речевого ресурса. В других случаях при `Forbidden` `BadRequest` попытке получить маркер авторизации возникнет ошибка или.

#### <a name="speech-resource-with-a-custom-domain-name-and-without-private-endpoints-usage-with-the-speech-sdk"></a>Речевой ресурс с пользовательским именем домена и без закрытых конечных точек: использование с пакетом SDK для распознавания речи

Использование речевого пакета SDK с пользовательскими доменными активами *без* закрытых конечных точек эквивалентно общему регистру, как описано в [документации по пакету SDK для распознавания речи](speech-sdk.md).

Если вы изменили код для использования с [голосовым ресурсом с закрытой конечной точкой](#speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-speech-sdk), учитывайте следующее.

В разделе о [речевых ресурсах с поддержкой частных конечных точек](#speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-speech-sdk)мы объяснили, как определить URL-адрес конечной точки, изменить его и заставить его работать с помощью инициализации экземпляра класса из конечной точки или с конечной точкой `SpeechConfig` .

Однако при попытке запустить то же приложение после удаления всех частных конечных точек (что позволяет выполнить некоторое время для соответствующей повторной инициализации записи DNS) возникает внутренняя ошибка службы (404). Причина заключается в том, что [запись DNS](#dns-configuration) теперь указывает на региональную конечную Cognitive Services, а не на прокси-сервер виртуальной сети, и URL-пути, такие как, `/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US` не будут найдены.

Необходимо выполнить откат приложения к стандартному созданию экземпляра `SpeechConfig` в стиле следующего кода:

```csharp
var config = SpeechConfig.FromSubscription(subscriptionKey, azureRegion);
```

## <a name="pricing"></a>Цены

Дополнительные сведения о ценах см. на [странице цен на службу "Приватный канал" Azure](https://azure.microsoft.com/pricing/details/private-link).

## <a name="learn-more"></a>Дополнительные сведения

* [Приватный канал Azure](../../private-link/private-link-overview.md)
* [пакет SDK для службы "Речь"](speech-sdk.md);
* [REST API преобразования речи в текст;](rest-speech-to-text.md)
* [REST API преобразования текста в речь.](rest-text-to-speech.md)