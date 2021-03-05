---
title: Создание настраиваемой роли Azure Resource Manager и назначение ее субъекту-службе в Azure
description: В этой статье содержатся инструкции по созданию настраиваемой роли Azure Resource Manager и назначению субъекту-службе для службы Live Video Analytics на IoT Edge с помощью Azure CLI.
ms.topic: how-to
ms.date: 05/27/2020
ms.openlocfilehash: 80974c111dd451314635d06334766322bc68e437
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102210450"
---
# <a name="create-custom-azure-resource-manager-role-and-assign-to-service-principal"></a>Создание настраиваемой роли Azure Resource Manager и назначение ее субъекту-службе

Для правильной работы службы "Анализ видео в реальном времени" в экземпляре модуля IoT Edge требуется активная учетная запись служб мультимедиа Azure. Связь между проживая видео-аналитикой в модуле IoT Edge и учетной записью службы мультимедиа Azure устанавливается с помощью набора свойств модуля двойника. Одно из этих свойств двойника — это [субъект-служба](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) , который позволяет экземпляру модуля взаимодействовать с и активировать необходимые операции в учетной записи служб мультимедиа. Чтобы снизить вероятность неправильного использования и/или случайной раскрытия данных от пограничных устройств, этот субъект-служба должен иметь наименьший уровень привилегий.

В этой статье описаны действия по созданию пользовательской роли Azure Resource Manager с Azure Cloud Shell, которая затем используется для создания субъекта-службы.

## <a name="prerequisites"></a>Предварительные требования  

Ниже приведены необходимые условия для этой статьи.

* Подписка Azure с подпиской владельца.
* Azure Active Directory с правами на создание приложения и назначение роли субъекта-службы.

Проверить, есть ли у вас соответствующие разрешения, проще всего на портале. Ознакомьтесь с [проверкой наличия необходимых разрешений](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

## <a name="overview"></a>Обзор  

Мы рассмотрим шаги по созданию настраиваемой роли и ее связыванию с субъектом-службой в следующем порядке:

1. Создайте учетную запись службы мультимедиа, если она еще не создана.
1. Создание субъекта-службы.
1. Создайте настраиваемую роль Azure Resource Manager с ограниченными привилегиями.
1. "Ограничьте" привилегии субъекта-службы с помощью созданной настраиваемой роли.
1. Выполните простой тест, чтобы узнать, удалось ли успешно ограничить субъект-службу.
1. Запишите параметры, которые будут использоваться в манифестах развертывания IoT Edge.

### <a name="create-a-media-services-account"></a>Создание учетной записи служб мультимедиа  

Если у вас нет учетной записи службы мультимедиа, выполните следующие действия, чтобы создать ее.

1. Перейдите к [Cloud Shell](https://shell.azure.com/).
1. Выберите "bash" в качестве среды в раскрывающемся списке в левой части окна оболочки.

    ![На снимках экрана показана bash, выбранная из окна оболочки.](./media/create-custom-azure-resource-manager-role-how-to/bash.png)
1. Задайте подписку Azure в качестве учетной записи по умолчанию, используя следующий шаблон команды:
    
    ```
    az account set --subscription " <yourSubscriptionName or yourSubscriptionId>"
    ```
1. Создайте [группу ресурсов](/cli/azure/group#az-group-create) и [учетную запись хранения](/cli/azure/storage/account#az-storage-account-create).
1. Теперь создайте учетную запись службы мультимедиа Azure с помощью следующего шаблона команды в Cloud Shell:

    ```
    az ams account create --name <yourAMSAccountName>  --resource-group <yourResouceGroup>  --storage-account <yourStorageAccountName>
    ```

### <a name="create-service-principal"></a>Создание субъекта-службы  

Теперь мы создадим новый субъект-службу и связываем его с учетной записью службы мультимедиа.

Без параметров проверки подлинности проверка подлинности на основе пароля используется со случайным паролем для субъекта-службы. В Cloud Shell используйте следующий шаблон команды:

```
az ams account sp create --account-name < yourAMSAccountName > --resource-group < yourResouceGroup >
```

Эта команда создает ответ следующим образом:

```
{
  "AadClientId": "00000000-0000-0000-0000-000000000000",
  "AadEndpoint": "https://login.microsoftonline.com",
  "AadSecret": "<yourServicePrincipalPassword>",
  "AadTenantId": "00000000-0000-0000-0000-000000000000",
  "AccountName": " <yourAMSAccountName >",
  "ArmAadAudience": "https://management.core.windows.net/",
  "ArmEndpoint": "https://management.azure.com/",
  "Region": "Central US",
  "ResourceGroup": " <yourResouceGroup >",
  "SubscriptionId": "<yourSubscriptionId>"
}

```
1. Выходные данные для субъекта-службы с проверкой пароля включают в себя ключ пароля, который в данном случае является параметром "Аадсекрет". 

    Обязательно скопируйте это значение, так как его нельзя получить повторно. Если вы забыли пароль, [сбросьте учетные данные для субъекта-службы](/cli/azure/create-an-azure-service-principal-azure-cli#reset-credentials).
1. Идентификатор appId и ключ клиента отображаются в выходных данных как "Аадклиентид" и "Aadtenantid и" соответственно. Они используются в проверке подлинности субъекта-службы. Запишите их значения, но при необходимости вы можете получить их в любой момент с помощью команды [az ad sp list](/cli/azure/ad/sp#az-ad-sp-list).

### <a name="create-a-custom-role-definition"></a>Создание пользовательского определения роли  

Чтобы создать настраиваемую роль, выполните следующие действия.

1. Создайте JSON-файл определения роли в локальной системе и сохраните следующий текст в файле. 
    1. Замените < Йоурсубскриптионид> ИДЕНТИФИКАТОРом подписки Azure.
    1. Для этой роли разрешены только следующие действия:
        * Листконтаинерсас — помогает использовать в списке модулей URL-адреса контейнеров для хранения и загрузки содержимого ресурсов.
        * Запись ресурсов — помогает модулю создать или обновить любой ресурс.
        * ЛистеджеполиЦиес — список политик, применяемых к пограничному устройству.  
        
        ```
        {
          "Name": "LVAEdge User",
          "IsCustom": true,
          "Description": "Can create assets, view list of containers and view Edge policies",
          "Actions": [
            "Microsoft.Media/mediaservices/assets/listContainerSas/action",
            "Microsoft.Media/mediaservices/assets/write",
            "Microsoft.Media/mediaservices/listEdgePolicies/action"
          ],
          "NotActions": [],
          "DataActions": [],
          "NotDataActions": [],
          "AssignableScopes": [
            "/subscriptions/<yourSubscriptionId>"
          ]
        }
        ```  
          
1. После создания запустите следующий шаблон команды, чтобы создать новое определение роли в подписке.
    
    ```
    az role definition create --role-definition "<location of the Role Definition JSON file >"
    ```

    После успешного выполнения команды вы увидите следующие выходные данные:
    
    ```
    {
      "assignableScopes": [
      "/subscriptions/<yourSubscriptionId>"
      ],
      "description": "Can create assets, view list of containers and view Edge policies",
      "id": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<unique name>",
      "name": "<unique name>",
      "permissions": [
        {
          "actions": [
            "Microsoft.Media/mediaservices/assets/listContainerSas/action",
            "Microsoft.Media/mediaservices/assets/write",
            "Microsoft.Media/mediaservices/listEdgePolicies/action",
          ],
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": " LVAEdge User ",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```

### <a name="create-role-assignment"></a>Создание назначения роли  

Чтобы добавить назначение ролей, вам потребуется идентификатор объекта субъекта-службы, которому вы хотите назначить только что созданную пользовательскую роль.

Чтобы получить objectId, используйте следующую команду в Cloud Shell:

```
az ad sp show --id "<appId>" | Select-String "objectId"
```

> [!NOTE]
> `<appId>` можно получить из выходных данных шага [создания субъекта-службы](#create-service-principal) .

Приведенная выше команда выведет на печать идентификатор objectId субъекта-службы. 

```
“objectId” : “<yourObjectId>”,
```

Используйте [команду AZ Role назначение Create](/cli/azure/role/assignment#az-role-assignment-create) , чтобы связать настраиваемую роль с субъектом-службой.

```
az role assignment create --role “LVAEdge User” --assignee-object-id < objectId>    
```

Параметры

|Параметры|Описание| 
|---|---|
|--Role |Имя пользовательской роли или идентификатор. В нашем случае: "пользователь Лваедже".|
|--Присваиваемый объект-Object-ID|Идентификатор объекта субъекта-службы, который будет использоваться.|

Результат будет выглядеть следующим образом:

```
{
  "canDelegate": null,
  "id": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleAssignments/<yourCustomRoleId>",
  "name": "<yourCustomRoleId>",
  "principalId": "<yourServicePrincipalId>",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<yourRoleDefinitionId>",
  "scope": "/subscriptions/<yourSubscriptionId>",
  "type": "Microsoft.Authorization/roleAssignments"
} 
```

### <a name="confirm-that-role-assignment-happened"></a>Подтверждение назначения роли

Чтобы убедиться, что субъект-служба теперь связан с только что созданной пользовательской ролью, выполните следующую команду:

```
az role assignment list  --assignee < objectId>
```

Результат должен выглядеть следующим образом:

```
[
  {
    "canDelegate": null,
    "id": "/subscriptions/xxx/providers/Microsoft.Authorization/roleAssignments/00000000-0000-0000-0000-000000000000",
    "name": "00000000-0000-0000-0000-000000000000",
    "principalId": "<yourServicePrincipalID>",
    "principalName": "<yourServicePrincipalName>",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/xxx/providers/Microsoft.Authorization/roleDefinitions/zzz",
    "roleDefinitionName": "LVAEdge User",
    "scope": "/subscriptions/<yourSubscription ID>",
    "type": "Microsoft.Authorization/roleAssignments
  }
]  
```
 
Найдите "Роледефинитионнаме" и убедитесь, что для него задано значение "Лваедже User". 

Это подтверждает, что мы связали пользовательскую роль пользователя с субъектом-службой, который используется для нашего приложения.

### <a name="test-the-service-principal-access-control"></a>Тестирование контроля доступа субъекта-службы

1. Вход с использованием субъекта-службы. Для этого нам потребуется 3 части информации для Azure Active Directory предоставить нам соответствующий маркер доступа, который можно получить из выходных данных шага [создания субъекта-службы](#create-service-principal) :
    1. аадклиентид 
    1. AadSecret
    1. AADTenantId
1. Теперь попробуйте выполнить вход с помощью шаблона команды ниже:
    
    ```
    az login --service-principal --username "< AadClientID>" --password " <AadSecret>" --tenant "<AadTenantId>"
    ```
3.  Теперь давайте посмотрим, ограничено ли имя входа субъекту-службе с ролью "пользователь Лваедже", пытаясь создать группу ресурсов, чтобы убедиться в ее сбое. Выполните следующую команду в Cloud Shell:

    ```
    az group create --location "central us" --name "testresourcegroup"
    ```

    Эта команда должна завершиться ошибкой и будет выглядеть следующим образом:
    
    ```
    The client '<AadClientId>' with object id '<AadClientId>' does not have authorization to perform action 'Microsoft.Resources/subscriptions/resourcegroups/write' over scope '/subscriptions/<yourSubscriptionId>/resourcegroups/testresourcegroup' or the scope is invalid. If access was recently granted, please refresh your credentials.
    ```

## <a name="next-steps"></a>Дальнейшие действия  

Обратите внимание на следующие значения из этой статьи. Эти значения будут необходимы для настройки свойств двойника в модуле Live Video Analytics для IoT Edge модуля. см. раздел [ДВОЙНИКА JSON Schema](module-twin-configuration-schema.md).

| Переменная из этой статьи|Имя свойства двойника для функции Live Video Analytics на IoT Edge|
|---|---|
|AadSecret |    аадсервицепринЦипалпассворд|
|AADTenantId |  aadTenantId|
|AadClientId    |aadServicePrincipalAppId|
