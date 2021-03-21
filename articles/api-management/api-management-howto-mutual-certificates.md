---
title: Серверная часть безопасного управления API с использованием проверки подлинности сертификата клиента
titleSuffix: Azure API Management
description: Узнайте, как управлять сертификатами клиентов и защищенными внутренними службами с помощью проверки подлинности на основе сертификата клиента в службе управления API Azure.
services: api-management
documentationcenter: ''
author: mikebudzynski
ms.service: api-management
ms.topic: article
ms.date: 01/26/2021
ms.author: apimpm
ms.openlocfilehash: 2e4a398ab71878134887fb8fba025cd8aa6122ad
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99492866"
---
# <a name="secure-backend-services-using-client-certificate-authentication-in-azure-api-management"></a>Защита серверных служб с помощью проверки подлинности сертификата клиента в службе управления API Azure

Управление API позволяет защитить доступ к серверной службе API с помощью сертификатов клиента. В этом руководство показано, как управлять сертификатами в экземпляре службы управления API Azure с помощью портал Azure. Здесь также объясняется, как настроить API для использования сертификата для доступа к серверной службе.

Вы также можете управлять сертификатами управления API с помощью [REST API управления API](/rest/api/apimanagement/2020-06-01-preview/certificate).

## <a name="certificate-options"></a>Параметры сертификата

Служба управления API предоставляет два варианта управления сертификатами, которые используются для защиты доступа к серверным службам.

* Ссылка на сертификат, управляемый в [Azure Key Vault](../key-vault/general/overview.md) 
* Добавление файла сертификата непосредственно в Управление API

Рекомендуется использовать сертификаты хранилища ключей, так как это помогает улучшить безопасность управления API:

* Сертификаты, хранящиеся в хранилищах ключей, можно повторно использовать в разных службах.
* Политики детализированного [доступа](../key-vault/general/secure-your-key-vault.md#data-plane-and-access-policies) можно применять к сертификатам, хранящимся в хранилищах ключей.
* Сертификаты, обновленные в хранилище ключей, автоматически поворачиваются в управлении API. После обновления в хранилище ключей сертификат в управлении API обновляется в течение 4 часов. Можно также вручную обновить сертификат с помощью портал Azure или с помощью REST API управления.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* Если экземпляр службы управления API еще не создан, см. раздел [Создание экземпляра службы управления API][Create an API Management service instance].
* Во внутренней службе должна быть настроена проверка подлинности с помощью сертификата клиента. Сведения о настройке проверки подлинности сертификата в службе приложений Azure см. в [этой статье][to configure certificate authentication in Azure WebSites refer to this article]. 
* Необходимо получить доступ к сертификату и паролю для управления в хранилище ключей Azure или отправить в службу управления API. Сертификат должен быть в формате **PFX** . Разрешено использовать самозаверяющие сертификаты.

### <a name="prerequisites-for-key-vault-integration"></a>Необходимые условия для интеграции хранилища ключей

1. Инструкции по созданию хранилища ключей см. в разделе [Краткое руководство. Создание хранилища ключей с помощью портал Azure](../key-vault/general/quick-create-portal.md).
1. Включите назначенное системой или назначенное пользователем [управляемое удостоверение](api-management-howto-use-managed-service-identity.md) в экземпляре управления API.
1. Назначьте управляемому удостоверению [политику доступа к хранилищу ключей](../key-vault/general/assign-access-policy-portal.md) с разрешениями на получение и вывод списка сертификатов из хранилища. Чтобы добавить политику, выполните следующие действия.
    1. На портале перейдите к своему хранилищу ключей.
    1. Выберите **параметры > политики доступа > + добавить политику доступа**.
    1. Выберите **разрешения сертификата**, а затем выберите **получить** и **вывести список**.
    1. В поле **Выбор субъекта** выберите имя ресурса управляемого удостоверения. Если вы используете назначенное системой удостоверение, участником является имя экземпляра управления API.
1. Создайте или импортируйте сертификат в хранилище ключей. См. раздел [Краткое руководство. Установка и извлечение сертификата из Azure Key Vault с помощью портал Azure](../key-vault/certificates/quick-create-portal.md).

[!INCLUDE [api-management-key-vault-network](../../includes/api-management-key-vault-network.md)]

## <a name="add-a-key-vault-certificate"></a>Добавление сертификата хранилища ключей

См. [Предварительные требования для интеграции хранилища ключей](#prerequisites-for-key-vault-integration).

> [!CAUTION]
> При использовании сертификата хранилища ключей в управлении API следует избегать удаления сертификата, хранилища ключей или управляемого удостоверения, используемого для доступа к хранилищу ключей.

Чтобы добавить сертификат хранилища ключей в Управление API, выполните следующие действия.

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com).
1. В разделе **Безопасность** выберите **Сертификаты**.
1. Выберите **Сертификаты**  >  **+ добавить**.
1. В окне **идентификатор** введите имя по своему усмотрению.
1. В разделе **сертификат** выберите **хранилище ключей**.
1. Введите идентификатор сертификата хранилища ключей или нажмите кнопку **выбрать** , чтобы выбрать сертификат из хранилища ключей.
    > [!IMPORTANT]
    > Если вы вводите идентификатор сертификата хранилища ключей самостоятельно, убедитесь, что у него нет сведений о версии. В противном случае сертификат не будет автоматически переключаться в службы управления API после обновления в хранилище ключей.
1. В списке **удостоверение клиента** выберите назначенное системой или существующее управляемое удостоверение, назначенное пользователем. Узнайте, как [добавлять или изменять управляемые удостоверения в службе управления API](api-management-howto-use-managed-service-identity.md).
    > [!NOTE]
    > Удостоверению требуются разрешения на получение и вывод списка сертификатов из хранилища ключей. Если вы еще не настроили доступ к хранилищу ключей, в службе управления API появится запрос, чтобы он мог автоматически настроить удостоверение с необходимыми разрешениями.
1. Выберите **Добавить**.

    :::image type="content" source="media/api-management-howto-mutual-certificates/apim-client-cert-kv.png" alt-text="Добавить сертификат хранилища ключей":::

## <a name="upload-a-certificate"></a>Загрузить сертификат

Чтобы передать сертификат клиента в Управление API, выполните следующие действия. 

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com).
1. В разделе **Безопасность** выберите **Сертификаты**.
1. Выберите **Сертификаты**  >  **+ добавить**.
1. В окне **идентификатор** введите имя по своему усмотрению.
1. В списке **сертификат** выберите **Пользовательский**.
1. Выберите PFX-файл сертификата и введите его пароль.
1. Выберите **Добавить**.

    :::image type="content" source="media/api-management-howto-mutual-certificates/apim-client-cert-add.png" alt-text="Отправить сертификат клиента":::

После отправки сертификат отображается в окне **Сертификаты** . Если у вас много сертификатов, запишите отпечаток нужного сертификата, чтобы настроить API для использования сертификата клиента для [проверки подлинности шлюза](#configure-an-api-to-use-client-certificate-for-gateway-authentication).

> [!NOTE]
> Чтобы отключить проверку цепочки сертификатов при использовании, например, самозаверяющего сертификата, выполните действия, описанные в разделе « [самозаверяющие сертификаты](#self-signed-certificates)» далее в этой статье.

## <a name="configure-an-api-to-use-client-certificate-for-gateway-authentication"></a>Настройка API для использования сертификата клиента для проверки подлинности шлюза

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com).
1. В разделе **интерфейсы API** выберите **интерфейсы API**.
1. Выберите API из списка. 
2. На вкладке **конструктор** выберите значок Редактор в разделе **Серверная** часть.
3. В поле **учетные данные шлюза** выберите сертификат **клиента** и выберите его в раскрывающемся списке.
1. Щелкните **Сохранить**.

    :::image type="content" source="media/api-management-howto-mutual-certificates/apim-client-cert-enable-select.png" alt-text="Использовать сертификат клиента для проверки подлинности шлюза":::

> [!CAUTION]
> Это изменение вступает в силу немедленно, а вызовы к операциям этого API будут использовать сертификат для проверки подлинности на внутреннем сервере.

> [!TIP]
> Если сертификат указан для проверки подлинности шлюза для серверной службы API, он становится частью политики для этого API и может быть просмотрен в редакторе политик.

## <a name="self-signed-certificates"></a>Самозаверяющие сертификаты

Если используются самозаверяющие сертификаты, необходимо отключить проверку цепочки сертификатов для управления API для взаимодействия с серверной системой. В противном случае будет возвращена ошибка с кодом 500. Чтобы настроить это, можно использовать [`New-AzApiManagementBackend`](/powershell/module/az.apimanagement/new-azapimanagementbackend) командлеты PowerShell (для новой серверной части) или [`Set-AzApiManagementBackend`](/powershell/module/az.apimanagement/set-azapimanagementbackend) (для существующих серверных компонентов) и задать `-SkipCertificateChainValidation` для параметра значение `True` .

```powershell
$context = New-AzApiManagementContext -resourcegroup 'ContosoResourceGroup' -servicename 'ContosoAPIMService'
New-AzApiManagementBackend -Context  $context -Url 'https://contoso.com/myapi' -Protocol http -SkipCertificateChainValidation $true
```

## <a name="delete-a-client-certificate"></a>Удаление сертификата клиента

Чтобы удалить сертификат, выберите его и в контекстном меню (**...**) выберите **Удалить** .

:::image type="content" source="media/api-management-howto-mutual-certificates/apim-client-cert-delete-new.png" alt-text="Удаление сертификата":::

> [!IMPORTANT]
> Если на сертификат ссылаются какие-либо политики, отображается экран предупреждения. Чтобы удалить сертификат, необходимо сначала удалить сертификат из всех политик, настроенных для его использования.

## <a name="next-steps"></a>Дальнейшие действия

* [Защита API-интерфейсов с помощью аутентификации на основе сертификата клиента в службе управления API Azure](api-management-howto-mutual-certificates-for-clients.md)
* Сведения о [политиках в управлении API](api-management-howto-policies.md)


[How to add operations to an API]: ./mock-api-responses.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: ../api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-policies.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[Azure API Management REST API Certificate entity]: ./api-management-caching-policies.md
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet
[to configure certificate authentication in Azure WebSites refer to this article]: ../app-service/app-service-web-configure-tls-mutual-auth.md

