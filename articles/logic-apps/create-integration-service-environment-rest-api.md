---
title: Создание сред службы интеграции (Исес) с помощью Logic Apps REST API
description: Создайте среду службы интеграции (ISE) с помощью Logic Apps REST API, чтобы получить доступ к виртуальным сетям Azure (виртуальных сетей) из Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: rarayudu, logicappspm
ms.topic: conceptual
ms.date: 02/03/2021
ms.openlocfilehash: d4500229800fa5d1743779b29927637777647e47
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99550663"
---
# <a name="create-an-integration-service-environment-ise-by-using-the-logic-apps-rest-api"></a>Создание среды службы интеграции (ISE) с помощью REST API Logic Apps 

В сценариях, где приложениям логики и учетным записям интеграции требуется доступ к [виртуальной сети Azure](../virtual-network/virtual-networks-overview.md), можно создать [ *среду службы интеграции* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) с помощью Logic Apps REST API. Дополнительные сведения о среде службы интеграции см. в статье [Доступ к ресурсам виртуальной сети Azure из Azure Logic Apps](connect-virtual-network-vnet-isolated-environment-overview.md).

В этой статье показано, как создать интегрированную среду сценариев с помощью Logic Apps REST API в целом. При необходимости можно также включить [назначенное системой или назначенное пользователем управляемое удостоверение](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) в интегрированной среде сценариев, но только с помощью REST API Logic Apps в данный момент. Это удостоверение позволяет интегрированной среде сценариев проверить подлинность доступа к защищенным ресурсам, таким как виртуальные машины и другие системы или службы, которые находятся в виртуальной сети Azure или подключены к ней. Таким образом, вам не нужно выполнять вход с использованием учетных данных.

Дополнительные сведения о других способах создания интегрированной среды сценариев см. в следующих статьях:

* [Создание интегрированной среды сценариев с помощью портал Azure](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)
* [Создание интегрированной среды сценариев с помощью образца шаблона быстрого запуска Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-integration-service-environment)
* [Создание интегрированной среды сценариев, поддерживающей использование управляемых клиентом ключей для шифрования неактивных данных](customer-managed-keys-integration-service-environment.md)

## <a name="prerequisites"></a>Предварительные условия

* Те же [условия](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#prerequisites) и [требования к доступу](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#enable-access) , что и при создании интегрированной среды сценариев в портал Azure

* Все дополнительные ресурсы, которые вы хотите использовать в интегрированной среде сценариев, чтобы можно было включать их сведения в определение ISE, например: 

  * Чтобы включить поддержку самозаверяющего сертификата, необходимо включить сведения об этом сертификате в определение ISE.

  * Чтобы включить управляемое пользователем удостоверение, необходимо создать это удостоверение заранее и включить `objectId` `principalId` свойства, и `clientId` их значения в определение ISE. Дополнительные сведения см. [в разделе Создание назначенного пользователем управляемого удостоверения в портал Azure](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

* Средство, которое можно использовать для создания интегрированной среды сценариев путем вызова REST API Logic Apps с запросом HTTPS-размещения. Например, можно использовать [POST](https://www.getpostman.com/downloads/)или создать приложение логики, которое будет выполнять эту задачу.

## <a name="create-the-ise"></a>Создание интегрированной среды сценариев

Чтобы создать интегрированную среду сценариев, вызвав REST API Logic Apps, сделайте запрос HTTPS to:

`PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01`

> [!IMPORTANT]
> Для Logic Apps REST API версии 2019-05-01 необходимо сделать собственный запрос HTTP-размещения для соединителей ISE.

Для завершения развертывания обычно требуется два часа. Изредка развертывание может занять до четырех часов. Чтобы проверить состояние развертывания, в [портал Azure](https://portal.azure.com)на панели инструментов Azure щелкните значок уведомления, чтобы открыть панель уведомлений.

> [!NOTE]
> Если развертывание завершится ошибкой или вы удалите среду службы интеграции, освобождение подсетей в Azure может занять до одного часа. Такая задержка означает, что вы не сможете сразу применить эти подсети для другой среды службы интеграции.
>
> При удалении виртуальной сети Azure, как правило, требуется до двух часов, прежде чем подсети будут свободны, но эта операция может занять и больше времени. 
> При удалении виртуальных сетей убедитесь, что в них нет подключенных ресурсов. 
> См. статью [Удаление виртуальной сети](../virtual-network/manage-virtual-network.md#delete-a-virtual-network).

## <a name="request-header"></a>Заголовок запроса

В заголовок запроса включите следующие свойства:

* `Content-type`: Задайте для этого свойства значение `application/json` .

* `Authorization`: Задайте для этого свойства токен носителя для клиента, который имеет доступ к подписке Azure или группе ресурсов, которые вы хотите использовать.

<a name="request-body"></a>

## <a name="request-body"></a>Текст запроса

В тексте запроса укажите определение ресурса для использования при создании интегрированной среды сценариев, включая сведения о дополнительных возможностях, которые необходимо включить в интегрированной среде сценариев, например:

* Чтобы создать интегрированную среду сценариев, позволяющую использовать самозаверяющий сертификат и сертификат, выданный центром сертификации предприятия, установленным в `TrustedRoot` расположении, включите `certificates` объект в раздел описания ISE `properties` , как описано в этой статье ниже.

* Чтобы создать интегрированную среду сценариев, использующую назначенное системой или назначенное пользователем управляемое удостоверение, включите `identity` объект с типом управляемого удостоверения и другими необходимыми сведениями в определение интегрированной среды, как описано в этой статье далее.

* Чтобы создать интегрированную среду сценариев, использующую управляемые клиентом ключи и Azure Key Vault для шифрования неактивных данных, включите [сведения, обеспечивающие поддержку ключа, управляемого клиентом](customer-managed-keys-integration-service-environment.md). Вы можете настроить ключи, управляемые клиентом, *только при их создании*, а не позже.

### <a name="request-body-syntax"></a>Синтаксис текста запроса

Ниже приведен синтаксис текста запроса, описывающий свойства, используемые при создании интегрированной среды сценариев.

```json
{
   "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Logic/integrationServiceEnvironments/{ISE-name}",
   "name": "{ISE-name}",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "{Azure-region}",
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   // Include the `identity` object to enable the system-assigned identity or user-assigned identity
   "identity": {
      "type": <"SystemAssigned" | "UserAssigned">,
      // When type is "UserAssigned", include the following "userAssignedIdentities" object:
      "userAssignedIdentities": {
         "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{user-assigned-managed-identity-object-ID}": {
            "principalId": "{principal-ID}",
            "clientId": "{client-ID}"
         }
      }
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-1}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-2}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-3}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-4}",
            }
         ]
      },
      // Include `certificates` object to enable self-signed certificate and the certificate issued by Enterprise Certificate Authority
      "certificates": {
         "testCertificate": {
            "publicCertificate": "{base64-encoded-certificate}",
            "kind": "TrustedRoot"
         }
      }
   }
}
```

### <a name="request-body-example"></a>Пример текста запроса

В этом примере текста запроса показаны примеры значений:

```json
{
   "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Logic/integrationServiceEnvironments/Fabrikam-ISE",
   "name": "Fabrikam-ISE",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "WestUS2",
   "identity": {
      "type": "UserAssigned",
      "userAssignedIdentities": {
         "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/*********************************": {
            "principalId": "*********************************",
            "clientId": "*********************************"
         }
      }
   },
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-1",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-2",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-3",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-4",
            }
         ]
      },
      "certificates": {
         "testCertificate": {
            "publicCertificate": "LS0tLS1CRUdJTiBDRV...",
            "kind": "TrustedRoot"
         }
      }
   }
}
```

## <a name="add-custom-root-certificates"></a>Добавление пользовательских корневых сертификатов

Интегрированная среда сценариев часто используется для подключения к пользовательским службам в виртуальной сети или локальной среде. Эти пользовательские службы часто защищаются сертификатом, выданным пользовательским корневым центром сертификации, например центром сертификации предприятия или самозаверяющим сертификатом. Дополнительные сведения об использовании самозаверяющих сертификатов см. [в разделе безопасный доступ и доступ к данным для исходящих вызовов других служб и систем](../logic-apps/logic-apps-securing-a-logic-app.md#secure-outbound-requests). Чтобы интегрированная среда сценариев успешно подключались к этим службам через протокол TLS, интегрированной среде сценариев требуется доступ к этим корневым сертификатам.

#### <a name="considerations-for-adding-custom-root-certificates"></a>Рекомендации по добавлению пользовательских корневых сертификатов

Перед обновлением интегрированной среды сценариев с пользовательским доверенным корневым сертификатом ознакомьтесь с приведенными ниже замечаниями.

* Обязательно отправьте корневой сертификат *и* все промежуточные сертификаты. Максимальное число сертификатов — 20.

* Отправка корневых сертификатов — это операция замены, при которой Последняя отправка перезаписывает предыдущие передачи. Например, если вы отправляете запрос, который отправляет один сертификат, а затем отправляете другой запрос на отправку другого сертификата, в интегрированной среде сценариев используется только второй сертификат. Если необходимо использовать оба сертификата, добавьте их вместе в один и тот же запрос.  

* Отправка корневых сертификатов — это асинхронная операция, которая может занять некоторое время. Чтобы проверить состояние или результат, можно отправить `GET` запрос, используя тот же универсальный код ресурса (URI). Ответное сообщение содержит `provisioningState` поле, которое возвращает `InProgress` значение, когда операция передачи все еще работает. Если `provisioningState` значение равно `Succeeded` , операция передачи завершена.

#### <a name="request-syntax"></a>Синтаксис запроса

Чтобы обновить интегрированную среду сценариев с помощью пользовательского доверенного корневого сертификата, отправьте следующий запрос исправления HTTPS в [URL-адрес Azure Resource Manager, который отличается в зависимости от среды Azure](../azure-resource-manager/management/control-plane-and-data-plane.md#control-plane), например:

| Среда | URL-адрес Azure Resource Manager |
|-------------|----------------------------|
| Глобальные Azure (несколько клиентов) | `PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01` |
| Azure для государственных организаций | `PATCH https://management.usgovcloudapi.net/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01` |
| Microsoft Azure China 21Vianet | `PATCH https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01` |
|||

#### <a name="request-body-syntax-for-adding-custom-root-certificates"></a>Синтаксис текста запроса для добавления пользовательских корневых сертификатов

Ниже приведен синтаксис текста запроса, описывающий свойства, используемые при добавлении корневых сертификатов.

```json
{
   "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Logic/integrationServiceEnvironments/{ISE-name}",
   "name": "{ISE-name}",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "{Azure-region}",
   "properties": {
      "certificates": {
         "testCertificate1": {
            "publicCertificate": "{base64-encoded-certificate}",
            "kind": "TrustedRoot"
         },
         "testCertificate2": {
            "publicCertificate": "{base64-encoded-certificate}",
            "kind": "TrustedRoot"
         }
      }
   }
}
```

## <a name="next-steps"></a>Дальнейшие действия

* [Добавление ресурсов в среды службы интеграции](../logic-apps/add-artifacts-integration-service-environment-ise.md)
* [Управление средами службы интеграции](../logic-apps/ise-manage-integration-service-environment.md#check-network-health)
