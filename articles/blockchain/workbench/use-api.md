---
title: Использование интерфейсов API службы Azure Блокчейн Workbench
description: Сценарии использования предварительной версии Azure Блокчейн Workbench REST API
ms.date: 03/05/2020
ms.topic: how-to
ms.reviewer: brendal
ms.openlocfilehash: 696f1f2f96034f7a044f6a39182774c02804518f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96004846"
---
# <a name="using-the-azure-blockchain-workbench-preview-rest-api"></a>Использование предварительной версии Azure Блокчейн Workbench REST API

Azure Блокчейн Workbench Preview REST API предоставляет разработчикам и информационным работникам способ создания богатой интеграции с приложениями блокчейн. В этой статье описываются несколько сценариев использования REST API Workbench. Например, предположим, что вы хотите создать настраиваемый клиент блокчейн, позволяющий войти в пользователей для просмотра назначенных им приложений блокчейн и взаимодействия с ними. Клиент может использовать API Блокчейн Workbench для просмотра экземпляров контракта и выполнения действий с интеллектуальными контрактами.

## <a name="blockchain-workbench-api-endpoint"></a>Конечная точка API блокчейн Workbench

Доступ к API-интерфейсам блокчейн Workbench осуществляется через конечную точку развертывания. Чтобы получить URL-адрес конечной точки API для развертывания, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).
1. В области навигации слева выберите **группы ресурсов**.
1. Выберите имя группы ресурсов, в которой развернута Блокчейн Workbench.
1. Щелкните заголовок столбца **Тип**, чтобы отсортировать список в алфавитном порядке.
1. Существует два ресурса с типом **Служба приложений**. Выберите ресурс типа **служба приложений** *с* суффиксом-API.
1. В разделе **Обзор** службы приложений скопируйте значение **URL-адреса** , которое представляет URL-адрес конечной точки API для развернутого блокчейн Workbench.

    ![URL-адрес конечной точки API службы приложений](media/use-api/app-service-api.png)

## <a name="authentication"></a>Аутентификация

Запросы к Блокчейн Workbench REST API защищены с помощью Azure Active Directory (Azure AD).

Чтобы выполнить запрос, прошедший проверку подлинности в API-интерфейсах, клиентский код требует проверки подлинности с действительными учетными данными перед вызовом API. Проверка подлинности согласовывается между различными субъектами Azure AD и предоставляет клиенту [маркер доступа](../../active-directory/develop/developer-glossary.md#access-token) в качестве подтверждения проверки подлинности. Затем маркер отправляется в заголовке авторизации HTTP запросов REST API. Дополнительные сведения о проверке подлинности Azure AD см. в статье [Azure Active Directory для разработчиков](../../active-directory/develop/index.yml).

Примеры проверки подлинности см. в разделе [REST API Samples](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/rest-api-samples) .

## <a name="using-postman"></a>Использование Postman

Если вы хотите протестировать или поэкспериментировать с API-интерфейсами Workbench, вы можете использовать [POST](https://www.postman.com) для выполнения вызовов API к развертыванию. [Скачайте пример коллекции запросов API Workbench](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/rest-api-samples/postman) из GitHub. Дополнительные сведения о проверке подлинности и использовании примеров запросов API см. в файле сведений.

## <a name="create-an-application"></a>Создание приложения

Для создания приложения Блокчейн Workbench используются два вызова API. Этот метод может выполняться только пользователями, которые являются администраторами Workbench.

Используйте [API-интерфейс передачи приложений](/rest/api/azure-blockchain-workbench/applications/applicationspost) , чтобы передать JSON-файл приложения и получить идентификатор приложения.

### <a name="applications-post-request"></a>Запросы приложений POST

Используйте параметр **аппфиле** , чтобы отправить файл конфигурации как часть текста запроса.

``` http
POST /api/v1/applications
Content-Type: multipart/form-data;
Authorization : Bearer {access token}
Content-Disposition: form-data; name="appFile"; filename="/C:/smart-contract-samples/HelloWorld.json"
Content-Type: application/json
```

### <a name="applications-post-response"></a>Ответ после ответа приложений

Созданный идентификатор приложения возвращается в ответе. Идентификатор приложения необходим для связывания файла конфигурации с файлом кода при вызове следующего API.

``` http
HTTP/1.1 200 OK
Content-Type: "application/json"
1
```

### <a name="contract-code-post-request"></a>Запрос POST кода контракта

Используйте [API-интерфейс POST кода контракта приложений](/rest/api/azure-blockchain-workbench/applications/contractcodepost) , передав идентификатор приложения, чтобы отправить файл кода для надежной работы приложения. Полезной нагрузкой может быть файл с одним расширением или сжатый ZIP-файл, содержащий файлы со сплошной плотностью.

Измените следующие значения:

| Параметр | Значение |
|-----------|-------|
| Кодов | Возвращаемое значение из API-интерфейса POST приложений. |
| {Леджерид} | Индекс книги учета. Значение обычно равно 1. Кроме того, можно проверить [таблицу книги](data-sql-management-studio.md) на наличие значения. |

``` http
POST /api/v1/applications/{applicationId}/contractCode?ledgerId={ledgerId}
Content-Type: multipart/form-data;
Authorization : Bearer {access token}
Content-Disposition: form-data; name="contractFile"; filename="/C:/smart-contract-samples/HelloWorld.sol"
```

### <a name="contract-code-post-response"></a>Ответ на POST код контракта

В случае успеха ответ включает созданный идентификатор кода контракта из [таблицы контракткоде](data-sql-management-studio.md).

``` http
HTTP/1.1 200 OK
Content-Type: "application/json"
2
```

## <a name="assign-roles-to-users"></a>Назначение ролей пользователям

Используйте [API назначения ролей приложений](/rest/api/azure-blockchain-workbench/applications/roleassignmentspost) , передав идентификатор приложения, идентификатор пользователя и идентификатор роли приложения, чтобы создать сопоставление пользователя с ролью в указанном приложении блокчейн. Этот метод может выполняться только пользователями, которые являются администраторами Workbench.

### <a name="role-assignments-post-request"></a>Запрос POST назначений ролей

Измените следующие значения:

| Параметр | Значение |
|-----------|-------|
| Кодов | Возвращаемое значение из API-интерфейса POST приложений. |
| {userId} | Значение идентификатора пользователя из [пользовательской таблицы](data-sql-management-studio.md). |
| {Аппликатионролеид} | Значение идентификатора роли приложения, связанное с ИДЕНТИФИКАТОРом приложения из [таблицы ApplicationRole](data-sql-management-studio.md). |

``` http
POST /api/v1/applications/{applicationId}/roleAssignments
Content-Type: application/json;
Authorization : Bearer {access token}

{
  "userId": {userId},
  "applicationRoleId": {applicationRoleId}
}
```

### <a name="role-assignments-post-response"></a>Назначение ролей после ответа

В случае успеха ответ включает созданный идентификатор назначения роли из [таблицы RoleAssignment](data-sql-management-studio.md).

``` http
HTTP/1.1 200
1
```

## <a name="list-applications"></a>Список приложений

Используйте [API Get Applications](/rest/api/azure-blockchain-workbench/applications/applicationsget) для получения всех приложений блокчейн Workbench для пользователя. В этом примере у пользователя, выполнившего вход, есть доступ к двум приложениям:

- [Передача активов](https://github.com/Azure-Samples/blockchain/blob/master/blockchain-workbench/application-and-smart-contract-samples/asset-transfer/readme.md)
- [Транспортировка с охлаждением](https://github.com/Azure-Samples/blockchain/blob/master/blockchain-workbench/application-and-smart-contract-samples/refrigerated-transportation/readme.md)

### <a name="applications-get-request"></a>Запрос на получение приложений

``` http
GET /api/v1/applications
Authorization : Bearer {access token}
```

### <a name="applications-get-response"></a>ПОЛУЧЕНИЕ ответа приложениями

Ответ содержит список всех приложений на базе блокчейна, к которым пользователь имеет доступ в Blockchain Workbench. Администраторы Blockchain Workbench получают все блокчейн-приложения. Администраторы, не являющиеся администраторами Workbench, получают все блокчейн приложения, для которых у них есть по крайней мере одна связанная роль приложения или связанная роль экземпляра смарт-контракта.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/applications?skip=2",
    "applications": [
        {
            "id": 1,
            "name": "AssetTransfer",
            "description": "Allows transfer of assets between a buyer and a seller, with appraisal/inspection functionality",
            "displayName": "Asset Transfer",
            "createdByUserId": 1,
            "createdDtTm": "2018-04-28T05:59:14.4733333",
            "enabled": true,
            "applicationRoles": null
        },
        {
            "id": 2,
            "name": "RefrigeratedTransportation",
            "description": "Application to track end-to-end transportation of perishable goods.",
            "displayName": "Refrigerated Transportation",
            "createdByUserId": 7,
            "createdDtTm": "2018-04-28T18:25:38.71",
            "enabled": true,
            "applicationRoles": null
        }
    ]
}
```

## <a name="list-workflows-for-an-application"></a>Получение списка рабочих процессов для приложения

Использование [рабочих процессов приложения получение API](/rest/api/azure-blockchain-workbench/applications/workflowsget) для перечисления всех рабочих процессов указанного приложения блокчейн, к которым пользователь имеет доступ в блокчейн Workbench. Для каждого приложения на базе блокчейна доступен один или несколько рабочих процессов, и для каждого рабочего процесса могут быть доступны экземпляры смарт-контракта. Для клиентского приложения на базе блокчейна, у которого есть только один рабочий процесс, мы рекомендуем пропустить поток взаимодействия с пользователем, позволяющий пользователям выбрать соответствующий рабочий процесс.

### <a name="application-workflows-request"></a>Запрос рабочих процессов приложения

``` http
GET /api/v1/applications/{applicationId}/workflows
Authorization: Bearer {access token}
```

### <a name="application-workflows-response"></a>Ответ рабочих процессов приложения

Администраторы Blockchain Workbench получают все рабочие процессы блокчейна. Остальные администраторы (не Workbench) получают все рабочие процессы, для которых у них есть по крайней мере одна роль связанного приложения или экземпляра смарт-контракта.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/applications/1/workflows?skip=1",
    "workflows": [
        {
            "id": 1,
            "name": "AssetTransfer",
            "description": "Handles the business logic for the asset transfer scenario",
            "displayName": "Asset Transfer",
            "applicationId": 1,
            "constructorId": 1,
            "startStateId": 1
        }
    ]
}
```

## <a name="create-a-contract-instance"></a>Создание экземпляра контракта

Используйте [API контрактов версии 2](/rest/api/azure-blockchain-workbench/contractsv2/contractpost) , чтобы создать новый экземпляр смарт-контракта для рабочего процесса. Пользователи могут создать новый экземпляр смарт-контракта только в том случае, если пользователь связан с ролью приложения, который может инициировать для рабочего процесса экземпляр интеллектуального контракта.

> [!NOTE]
> В этом примере используется версия 2 API. API-интерфейсы контракта версии 2 обеспечивают более подробную детализацию для связанных полей Провисионингстатус.

### <a name="contracts-post-request"></a>Запрос POST контрактов

Измените следующие значения:

| Параметр | Значение |
|-----------|-------|
| WorkflowId | Значение идентификатора рабочего процесса — это Конструкторид контракта из [таблицы рабочего процесса](data-sql-management-studio.md). |
| {Контракткодеид} | Значение идентификатора кода контракта из [таблицы контракткоде](data-sql-management-studio.md). Сопоставьте идентификатор приложения и идентификатор книги учета для создаваемого экземпляра контракта. |
| ConnectionId | Значение идентификатора соединения из [таблицы подключения](data-sql-management-studio.md). |

Для текста запроса задайте значения, используя следующие сведения:

| Параметр | Значение |
|-----------|-------|
| воркфловфунктионид | ИДЕНТИФИКАТОР из [таблицы воркфловфунктион](data-sql-management-studio.md). |
| воркфловактионпараметерс | Пары "имя значение" параметров, переданных конструктору. Для каждого параметра используйте значение Воркфловфунктионпараметерид из таблицы [воркфловфунктионпараметер](data-sql-management-studio.md) . |

``` http
POST /api/v2/contracts?workflowId={workflowId}&contractCodeId={contractCodeId}&connectionId={connectionId}
Content-Type: application/json;
Authorization : Bearer {access token}

{
  "workflowFunctionID": 2,
  "workflowActionParameters": [
    {
      "name": "message",
      "value": "Hello, world!",
      "workflowFunctionParameterId": 3
    }
  ]
}
```

### <a name="contracts-post-response"></a>Ответ POST контрактов

В случае успеха API назначения ролей возвращает Контрактактионид из [таблицы контрактактионпараметер](data-sql-management-studio.md).

``` http
HTTP/1.1 200 OK
4
```

## <a name="list-smart-contract-instances-for-a-workflow"></a>Вывести список экземпляров смарт-контракта для рабочего процесса.

Используйте [контракты Get API](/rest/api/azure-blockchain-workbench/contractsv2/contractsget) , чтобы отобразить все экземпляры смарт-контракта для рабочего процесса. Кроме того, можно разрешить пользователям изучить любой из отображаемых экземпляров смарт-контрактов.

### <a name="contracts-request"></a>Запрос контрактов

В этом примере пользователь применит один из смарт-контрактов для выполнения действий.

``` http
GET api/v1/contracts?workflowId={workflowId}
Authorization: Bearer {access token}
```

### <a name="contracts-response"></a>Ответ контрактов

В ответе перечисляются все экземпляры смарт-контракта указанного рабочего процесса. Администраторы Workbench получают все экземпляры смарт-контрактов. Остальные администраторы (не Workbench) получают все экземпляры смарт-контрактов, для которых у них есть по крайней мере одна роль связанного приложения или экземпляра смарт-контракта.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/contracts?skip=3&workflowId=1",
    "contracts": [
        {
            "id": 1,
            "provisioningStatus": 2,
            "connectionID": 1,
            "ledgerIdentifier": "0xbcb6127be062acd37818af290c0e43479a153a1c",
            "deployedByUserId": 1,
            "workflowId": 1,
            "contractCodeId": 1,
            "contractProperties": [
                {
                    "workflowPropertyId": 1,
                    "value": "0"
                },
                {
                    "workflowPropertyId": 2,
                    "value": "My first car"
                },
                {
                    "workflowPropertyId": 3,
                    "value": "54321"
                },
                {
                    "workflowPropertyId": 4,
                    "value": "0"
                },
                {
                    "workflowPropertyId": 5,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 6,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 7,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 8,
                    "value": "0xd882530eb3d6395e697508287900c7679dbe02d7"
                }
            ],
            "transactions": [
                {
                    "id": 1,
                    "connectionId": 1,
                    "transactionHash": "0xf3abb829884dc396e03ae9e115a770b230fcf41bb03d39457201449e077080f4",
                    "blockID": 241,
                    "from": "0xd882530eb3d6395e697508287900c7679dbe02d7",
                    "to": null,
                    "value": 0,
                    "isAppBuilderTx": true
                }
            ],
            "contractActions": [
                {
                    "id": 1,
                    "userId": 1,
                    "provisioningStatus": 2,
                    "timestamp": "2018-04-29T23:41:14.9333333",
                    "parameters": [
                        {
                            "name": "Description",
                            "value": "My first car"
                        },
                        {
                            "name": "Price",
                            "value": "54321"
                        }
                    ],
                    "workflowFunctionId": 1,
                    "transactionId": 1,
                    "workflowStateId": 1
                }
            ]
        }
    ]
}
```

## <a name="list-available-actions-for-a-contract"></a>Получение списка доступных действий для контракта

Использование [действия контракта Get API](/rest/api/azure-blockchain-workbench/contractsv2/contractactionget) для отображения доступных действий пользователя по состоянию контракта. 

### <a name="contract-action-request"></a>Запрос действия контракта

В этом примере пользователь просматривает все доступные действия для нового смарт-контракта, который они создали.

``` http
GET /api/v1/contracts/{contractId}/actions
Authorization: Bearer {access token}
```

### <a name="contract-action-response"></a>Ответ действия контракта

Ответ содержит список всех действий, к которым пользователь может применить текущее состояние указанного экземпляра смарт-контракта.

* Изменить: это действие позволяет пользователю изменить описание и цену ресурса.
* Завершить: это действие дает пользователю возможность завершить контракт ресурса.

Пользователи получают все необходимые действия, если они имеют роль связанного приложения или экземпляра смарт-контракта для текущего состояния указанного экземпляра смарт-контракта.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/contracts/1/actions?skip=2",
    "workflowFunctions": [
        {
            "id": 2,
            "name": "Modify",
            "description": "Modify the description/price attributes of this asset transfer instance",
            "displayName": "Modify",
            "parameters": [
                {
                    "id": 1,
                    "name": "description",
                    "description": "The new description of the asset",
                    "displayName": "Description",
                    "type": {
                        "id": 2,
                        "name": "string",
                        "elementType": null,
                        "elementTypeId": 0
                    }
                },
                {
                    "id": 2,
                    "name": "price",
                    "description": "The new price of the asset",
                    "displayName": "Price",
                    "type": {
                        "id": 3,
                        "name": "money",
                        "elementType": null,
                        "elementTypeId": 0
                    }
                }
            ],
            "workflowId": 1
        },
        {
            "id": 3,
            "name": "Terminate",
            "description": "Used to cancel this particular instance of asset transfer",
            "displayName": "Terminate",
            "parameters": [],
            "workflowId": 1
        }
    ]
}
```

## <a name="execute-an-action-for-a-contract"></a>Выполнение действия для контракта

Используйте [API действия контракта](/rest/api/azure-blockchain-workbench/contractsv2/contractactionpost) , чтобы выполнить действие для указанного экземпляра смарт-контракта.

### <a name="contract-action-post-request"></a>Запрос POST действия контракта

В этом случае рассмотрим сценарий, в котором пользователь хочет изменить описание и цену ресурса.

``` http
POST /api/v1/contracts/{contractId}/actions
Authorization: Bearer {access token}
actionInformation: {
    "workflowFunctionId": 2,
    "workflowActionParameters": [
        {
            "name": "description",
            "value": "My updated car"
        },
        {
            "name": "price",
            "value": "54321"
        }
    ]
}
```

Пользователи могут выполнить действие только с учетом текущего состояния указанного экземпляра смарт-контракта и роли связанного приложения или экземпляра смарт-контракта пользователя.

### <a name="contract-action-post-response"></a>Ответ на запрос POST действия контракта

При успешном выполнении вызова POST возвращается ответ HTTP 200 OK без текста ответа.

``` http
HTTP/1.1 200 OK
Content-type: application/json
```

## <a name="next-steps"></a>Дальнейшие действия

Справочные сведения об API-интерфейсах Блокчейн Workbench см. в [справочнике по Azure Блокчейн workbench REST API](/rest/api/azure-blockchain-workbench).
