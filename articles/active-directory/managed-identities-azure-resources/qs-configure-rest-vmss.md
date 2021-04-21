---
title: Настройка управляемых удостоверений в масштабируемом наборе виртуальных машин Azure с помощью REST — Azure AD
description: Пошаговые инструкции по настройке управляемых удостоверений, назначаемых системой и назначаемых пользователем, в масштабируемом наборе виртуальных машин Azure с использованием CURL для выполнения вызовов REST API.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/29/2021
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 585e31ab566cc990af2819fcf9cdde0506560208
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780179"
---
# <a name="configure-managed-identities-for-azure-resources-on-a-virtual-machine-scale-set-using-rest-api-calls"></a>Настройка управляемых удостоверений для ресурсов Azure в масштабируемом наборе виртуальных машин с помощью вызовов REST API

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые системные удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде. 

В этой статье вы узнаете, как выполнять приведенные ниже операции с управляемыми удостоверениями для ресурсов Azure, используя CURL для выполнения вызовов к конечной точке REST Azure Resource Manager.

- Включение и отключение управляемого удостоверения, назначаемого системой, в масштабируемом наборе виртуальных машин Azure
- Добавление и удаление управляемого удостоверения, назначаемого пользователем, в масштабируемом наборе виртуальных машин Azure

Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжать.

## <a name="prerequisites"></a>Обязательные условия

- Если вы не работали с управляемыми удостоверениями для ресурсов Azure, см. статью [Что такое управляемые удостоверения для ресурсов Azure](overview.md). Сведения об удостоверениях, назначаемых системой и назначаемых пользователем, см. в разделе [Типы управляемых удостоверений](overview.md#managed-identity-types).

- Для выполнения операций управления, описанных в этой статье, учетной записи нужно назначить следующие роли Azure:

  - [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor): для создания масштабируемого набора виртуальных машин, а также для включения и удаления управляемого удостоверения, назначаемого системой, и (или) управляемого удостоверения, назначаемого пользователем, из масштабируемого набора виртуальных машин.

  - Роль [Участник управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-contributor): для создания управляемого удостоверения, назначаемого пользователем.

  - Роль [Оператор управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator): для назначения и удаления удостоверения, назначаемого пользователем, в масштабируемом наборе виртуальных машин.

  > [!NOTE]
  > Дополнительные назначения ролей в каталоге Azure AD не требуются.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="system-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое системой

В этом разделе описывается, как включить и отключить управляемое удостоверение, назначаемое системой, в масштабируемом наборе виртуальных машин, используя CURL для выполнения вызовов к конечной точке REST Azure Resource Manager.

### <a name="enable-system-assigned-managed-identity-during-creation-of-a-virtual-machine-scale-set"></a>Включение управляемого удостоверения, назначаемого системой, во время создания масштабируемого набора виртуальных машин

Для создания масштабируемого набора виртуальных машин с включенным управляемым удостоверением, назначаемым системой, создайте масштабируемый набор виртуальных машин и получите маркер доступа, чтобы использовать CURL для вызова конечной точки Resource Manager со значением типа управляемого удостоверения, назначаемого системой.

1. Чтобы сохранить и развернуть масштабируемый набор виртуальных машин и связанные с ним ресурсы, создайте [группу ресурсов](../../azure-resource-manager/management/overview.md#terminology) с помощью команды [az group create](/cli/azure/group/#az_group_create). Если вы уже создали группу ресурсов, которую можно использовать, этот шаг можно пропустить:

   ```azurecli-interactive 
   az group create --name myResourceGroup --location westus
   ```

2. Создайте [сетевой интерфейс](/cli/azure/network/nic#az_network_nic_create) для масштабируемого набора виртуальных машин:

   ```azurecli-interactive
    az network nic create -g myResourceGroup --vnet-name myVnet --subnet mySubnet -n myNic
   ```

3. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ``` 

4. С помощью Azure Cloud Shell создайте масштабируемый набор виртуальных машин, используя CURL для вызова конечной точки REST Azure Resource Manager. Приведенный ниже пример создает в группе ресурсов *myResourceGroup* масштабируемый набор виртуальных машин *myVMSS* с управляемым удостоверением, назначаемым системой, в соответствии со значением `"identity":{"type":"SystemAssigned"}` в тексте запроса. Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PUT -d '{"sku":{"tier":"Standard","capacity":3,"name":"Standard_D1_v2"},"location":"eastus","identity":{"type":"SystemAssigned"},"properties":{"overprovision":true,"virtualMachineProfile":{"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"createOption":"FromImage"}},"osProfile":{"computerNamePrefix":"myVMSS","adminUsername":"azureuser","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaceConfigurations":[{"name":"myVMSS","properties":{"primary":true,"enableIPForwarding":true,"ipConfigurations":[{"name":"myVMSS","properties":{"subnet":{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"}}}]}}]}},"upgradePolicy":{"mode":"Manual"}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "sku":{
          "tier":"Standard",
          "capacity":3,
          "name":"Standard_D1_v2"
       },
       "location":"eastus",
       "identity":{
          "type":"SystemAssigned"
       },
       "properties":{
          "overprovision":true,
          "virtualMachineProfile":{
             "storageProfile":{
                "imageReference":{
                   "sku":"2016-Datacenter",
                   "publisher":"MicrosoftWindowsServer",
                   "version":"latest",
                   "offer":"WindowsServer"
                },
                "osDisk":{
                   "caching":"ReadWrite",
                   "managedDisk":{
                      "storageAccountType":"Standard_LRS"
                   },
                   "createOption":"FromImage"
                }
             },
             "osProfile":{
                "computerNamePrefix":"myVMSS",
                "adminUsername":"azureuser",
                "adminPassword":"myPassword12"
             },
             "networkProfile":{
                "networkInterfaceConfigurations":[
                   {
                      "name":"myVMSS",
                      "properties":{
                         "primary":true,
                         "enableIPForwarding":true,
                         "ipConfigurations":[
                            {
                               "name":"myVMSS",
                               "properties":{
                                  "subnet":{
                                     "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"
                                  }
                               }
                            }
                         ]
                      }
                   }
                ]
             }
          },
          "upgradePolicy":{
             "mode":"Manual"
          }
       }
    }  
   ```  

### <a name="enable-system-assigned-managed-identity-on-an-existing-virtual-machine-scale-set"></a>Включение управляемого удостоверения, назначаемого системой, в существующем масштабируемом наборе виртуальных машин

Чтобы включить управляемое удостоверение, назначаемое системой, в существующем масштабируемом наборе виртуальных машин, нужно получить маркер доступа, а затем воспользоваться CURL для вызова конечной точки REST Resource Manager для изменения типа удостоверения.

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Используйте приведенную ниже команду CURL для вызова конечной точки REST Azure Resource Manager, чтобы включить управляемое удостоверение, назначаемое системой, в масштабируемом наборе виртуальных машин в соответствии со значением `{"identity":{"type":"SystemAssigned"}` в тексте запроса для масштабируемого набора виртуальных машин *myVMSS*.  Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.
   
   > [!IMPORTANT]
   > Чтобы предотвратить удаление существующих управляемых удостоверений, назначаемых пользователем, для масштабируемого набора виртуальных машин, нужно перечислить эти удостоверения с помощью следующей команды CURL: `curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachineScaleSets/<VMSS NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"`. Если для масштабируемого набора виртуальных машин заданы управляемые удостоверения, назначаемые пользователем, как указано в значении `identity` в ответе, перейдите к шагу 3, где описано, как сохранить управляемые удостоверения, назначаемые пользователем, при включении управляемого удостоверения, назначаемого системой, в масштабируемом наборе виртуальных машин.

   ```bash
    curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned"}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned"
       }
    }
   ```

3. Чтобы включить управляемое удостоверение, назначаемое системой, в масштабируемом наборе виртуальных машин с существующими управляемыми удостоверениями, назначаемыми пользователем, нужно добавить `SystemAssigned` в значение `type`.  
   
   Например, если для масштабируемого набора виртуальных машин заданы управляемые удостоверения, назначаемые системой, `ID1` и `ID2`, и вы хотите добавить для него управляемое удостоверение, назначаемое системой, используйте приведенный ниже вызов CURL. Замените `<ACCESS TOKEN>` и `<SUBSCRIPTION ID>` значениями, уместными для вашей среды.

   API версии `2018-06-01` хранит управляемые удостоверения, назначаемые пользователем, в значении `userAssignedIdentities` в формате словаря в отличие от значения `identityIds` в формате массива, используемого в API версии `2017-12-01`.
   
   **API версии 2018-06-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned,UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{},"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{}}}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. |
 
   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned,UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
             },
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{
    
             }
          }
       }
    }
   ```
   
   **API версии 2017-12-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned,UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1","/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"]}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned,UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1",
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"
          ]
       }
    }
   ```

### <a name="disable-system-assigned-managed-identity-from-a-virtual-machine-scale-set"></a>Отключение управляемого удостоверения, назначаемого системой, в масштабируемом наборе виртуальных машин

Чтобы отключить удостоверение, назначаемое системой, в существующем масштабируемом наборе виртуальных машин, нужно получить маркер доступа, а затем воспользоваться CURL для вызова конечной точки REST Resource Manager для изменения типа удостоверения на `None`.

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Измените масштабируемый набор виртуальных машин, используя CURL для вызова конечной точки REST Azure Resource Manager, чтобы отключить управляемое удостоверение, назначаемое системой.  Приведенный ниже пример отключает управляемое удостоверение, назначаемое системой, в масштабируемом наборе виртуальных машин *myVMSS* в соответствии со значением `{"identity":{"type":"None"}}` в тексте запроса.  Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.

   > [!IMPORTANT]
   > Чтобы предотвратить удаление существующих управляемых удостоверений, назначаемых пользователем, для масштабируемого набора виртуальных машин, нужно перечислить эти удостоверения с помощью следующей команды CURL: `curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachineScaleSets/<VMSS NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"`. Если для масштабируемого набора виртуальных машин задано управляемое удостоверение, назначаемое пользователем, перейдите к шагу 3, где описано, как сохранить управляемые удостоверения, назначаемые пользователем, при удалении управляемого удостоверения, назначаемого системой, из масштабируемого набора виртуальных машин.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"None"}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"None"
       }
    }
   ```

   Чтобы удалить управляемое удостоверение, назначаемое системой, из масштабируемого набора виртуальных машин, в котором есть управляемые удостоверения, назначаемые пользователем, удалите `SystemAssigned` из значения `{"identity":{"type:" "}}`, сохранив значение `UserAssigned` и значения словаря `userAssignedIdentities`, если используется **API версии 2018-06-01**. При использовании **API версии 2017-12-01** или более ранних версий сохраните массив `identityIds`.

## <a name="user-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое пользователем

В этом разделе описывается, как добавить и удалить управляемое удостоверение, назначаемое пользователем, в масштабируемом наборе виртуальных машин, используя CURL для выполнения вызовов к конечной точке REST Azure Resource Manager.

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-a-virtual-machine-scale-set"></a>Задание управляемого удостоверения, назначаемого пользователем, при создании масштабируемого набора виртуальных машин

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Создайте [сетевой интерфейс](/cli/azure/network/nic#az_network_nic_create) для масштабируемого набора виртуальных машин:

   ```azurecli-interactive
    az network nic create -g myResourceGroup --vnet-name myVnet --subnet mySubnet -n myNic
   ```

3. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ``` 

4. Создайте пользовательское удостоверение, как описано в разделе [Создание управляемого удостоверения, назначаемого пользователем](how-to-manage-ua-identity-rest.md#create-a-user-assigned-managed-identity).

5. Создайте масштабируемый набор виртуальных машин, используя CURL для вызова конечной точки REST Azure Resource Manager. Приведенный ниже пример создает в группе ресурсов *myResourceGroup* масштабируемый набор виртуальных машин *myVMSS* с управляемым удостоверением, назначаемым пользователем, `ID1` в соответствии со значением `"identity":{"type":"UserAssigned"}` в тексте запроса. Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.
 
   **API версии 2018-06-01**

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PUT -d '{"sku":{"tier":"Standard","capacity":3,"name":"Standard_D1_v2"},"location":"eastus","identity":{"type":"UserAssigned","userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{}}},"properties":{"overprovision":true,"virtualMachineProfile":{"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"createOption":"FromImage"}},"osProfile":{"computerNamePrefix":"myVMSS","adminUsername":"azureuser","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaceConfigurations":[{"name":"myVMSS","properties":{"primary":true,"enableIPForwarding":true,"ipConfigurations":[{"name":"myVMSS","properties":{"subnet":{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"}}}]}}]}},"upgradePolicy":{"mode":"Manual"}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "sku":{
          "tier":"Standard",
          "capacity":3,
          "name":"Standard_D1_v2"
       },
       "location":"eastus",
       "identity":{
          "type":"UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
    
             }
          }
       },
       "properties":{
          "overprovision":true,
          "virtualMachineProfile":{
             "storageProfile":{
                "imageReference":{
                   "sku":"2016-Datacenter",
                   "publisher":"MicrosoftWindowsServer",
                   "version":"latest",
                   "offer":"WindowsServer"
                },
                "osDisk":{
                   "caching":"ReadWrite",
                   "managedDisk":{
                      "storageAccountType":"Standard_LRS"
                   },
                   "createOption":"FromImage"
                }
             },
             "osProfile":{
                "computerNamePrefix":"myVMSS",
                "adminUsername":"azureuser",
                "adminPassword":"myPassword12"
             },
             "networkProfile":{
                "networkInterfaceConfigurations":[
                   {
                      "name":"myVMSS",
                      "properties":{
                         "primary":true,
                         "enableIPForwarding":true,
                         "ipConfigurations":[
                            {
                               "name":"myVMSS",
                               "properties":{
                                  "subnet":{
                                     "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"
                                  }
                               }
                            }
                         ]
                      }
                   }
                ]
             }
          },
          "upgradePolicy":{
             "mode":"Manual"
          }
       }
    }
   ```   

   **API версии 2017-12-01**

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01' -X PUT -d '{"sku":{"tier":"Standard","capacity":3,"name":"Standard_D1_v2"},"location":"eastus","identity":{"type":"UserAssigned","identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]},"properties":{"overprovision":true,"virtualMachineProfile":{"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"createOption":"FromImage"}},"osProfile":{"computerNamePrefix":"myVMSS","adminUsername":"azureuser","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaceConfigurations":[{"name":"myVMSS","properties":{"primary":true,"enableIPForwarding":true,"ipConfigurations":[{"name":"myVMSS","properties":{"subnet":{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"}}}]}}]}},"upgradePolicy":{"mode":"Manual"}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. |
 
   **Текст запроса**

   ```JSON
    {
       "sku":{
          "tier":"Standard",
          "capacity":3,
          "name":"Standard_D1_v2"
       },
       "location":"eastus",
       "identity":{
          "type":"UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       },
       "properties":{
          "overprovision":true,
          "virtualMachineProfile":{
             "storageProfile":{
                "imageReference":{
                   "sku":"2016-Datacenter",
                   "publisher":"MicrosoftWindowsServer",
                   "version":"latest",
                   "offer":"WindowsServer"
                },
                "osDisk":{
                   "caching":"ReadWrite",
                   "managedDisk":{
                      "storageAccountType":"Standard_LRS"
                   },
                   "createOption":"FromImage"
                }
             },
             "osProfile":{
                "computerNamePrefix":"myVMSS",
                "adminUsername":"azureuser",
                "adminPassword":"myPassword12"
             },
             "networkProfile":{
                "networkInterfaceConfigurations":[
                   {
                      "name":"myVMSS",
                      "properties":{
                         "primary":true,
                         "enableIPForwarding":true,
                         "ipConfigurations":[
                            {
                               "name":"myVMSS",
                               "properties":{
                                  "subnet":{
                                     "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"
                                  }
                               }
                            }
                         ]
                      }
                   }
                ]
             }
          },
          "upgradePolicy":{
             "mode":"Manual"
          }
       }
    }
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-virtual-machine-scale-set"></a>Задание управляемого удостоверения, назначаемого пользователем, для существующего масштабируемого набора виртуальных машин Azure

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2.  Создайте управляемое удостоверение, назначаемое пользователем, как описано в разделе [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью вызовов REST API](how-to-manage-ua-identity-rest.md#create-a-user-assigned-managed-identity).

3. Чтобы предотвратить удаление существующих управляемых удостоверений, назначаемых пользователем или назначаемых системой, для масштабируемого набора виртуальных машин, нужно получить список назначенных ему типов удостоверений с помощью приведенной ниже команды CURL. Если у вас есть управляемые удостоверения, назначенные масштабируемому набору виртуальных машин, они перечислены в значении `identity`.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachineScaleSets/<VMSS NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachineScaleSets/<VMSS NAME>?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. |   
 

4. Если для вашего масштабируемого набора виртуальных машин не заданы управляемые удостоверения, назначаемые пользователем или назначаемые системой, используйте приведенную ниже команду CURL для вызова конечной точки REST Azure Resource Manager, чтобы задать первое управляемое удостоверение, назначаемое пользователем, для масштабируемого набора виртуальных машин.  Если для масштабируемого набора виртуальных машин заданы управляемые удостоверения, назначаемые пользователем или назначаемые системой, перейдите к шагу 5, где описано, как добавить несколько управляемых удостоверений, назначаемых пользователем, в масштабируемый набор виртуальных машин, сохранив при этом управляемое удостоверение, назначаемое системой.

   Следующий пример задает управляемое удостоверение, назначаемое пользователем, `ID1` для масштабируемого набора виртуальных машин *myVMSS* в группе ресурсов *myResourceGroup*.  Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.

   **API версии 2018-06-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-12-01' -X PATCH -d '{"identity":{"type":"userAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{}}}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-12-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"userAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
    
             }
          }
       }
    }
   ``` 
    
   **API версии 2017-12-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"userAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"userAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       }
    }
   ```  

5. Если для масштабируемого набора виртуальных машин задано управляемое удостоверение, назначаемое пользователем или назначаемого системой, сделайте следующее.
   
   **API версии 2018-06-01**

   Добавьте управляемое удостоверение, назначаемое пользователем, в значение словаря `userAssignedIdentities`.

   Например, если для масштабируемого набора виртуальных машин заданы управляемое удостоверение, назначаемое системой, и управляемое удостоверение, назначаемое пользователем, `ID1` и вы хотите добавить для него управляемое удостоверение, назначаемое пользователем, `ID2`, сделайте следующее.

   ```bash
   curl  'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{},"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{}}}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
    
             },
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{
    
             }
          }
       }
    }
   ```

   **API версии 2017-12-01**

   При добавлении нового управляемого удостоверения, назначаемого пользователем, сохраните управляемые удостоверения, назначаемые пользователем, которые вы хотите оставить в значении массива `identityIds`.

   Например, если для масштабируемого набора виртуальных машин заданы удостоверение, назначаемое системой, и управляемое удостоверение, назначаемое пользователем, `ID1` и вы хотите добавить для него управляемое удостоверение, назначаемое пользователем, `ID2`, сделайте следующее.

    ```bash
   curl  'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1","/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"]}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01 HTTP/1.1
   ```

    **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1",
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"
          ]
       }
    }
   ```

### <a name="remove-a-user-assigned-managed-identity-from-a-virtual-machine-scale-set"></a>Удаление управляемого удостоверения, назначаемого пользователем, из масштабируемого набора виртуальных машин

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Чтобы предотвратить удаление существующих управляемых удостоверений, назначаемых пользователем, которые нужно сохранить для масштабируемого набора виртуальных машин, или удалить управляемое удостоверение, назначаемое системой, нужно перечислить управляемые удостоверения с помощью следующей команды CURL.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachineScaleSets/<VMSS NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>" 
   ```

   ```HTTP
   GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachineScaleSets/<VMSS NAME>?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. |
   
   Если у вас есть управляемые удостоверения, назначенные виртуальной машине, они перечислены в значении `identity` в ответе. 
    
   Например, если для масштабируемого набора виртуальных машин заданы управляемые удостоверения, назначаемые пользователем, `ID1` и `ID2`, и требуется оставить только `ID1`, сохранив при этом управляемое удостоверение, назначаемое системой.

   **API версии 2018-06-01**

   К управляемому удостоверению, назначаемому пользователем, которое требуется удалить, необходимо добавить `null`.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":null}}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":null
          }
       }
    }
   ```

   **API версии 2017-12-01**

   В массиве `identityIds` оставьте только те управляемые удостоверения, назначаемые пользователем, которые вы хотите сохранить.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned,UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
   ```   

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2017-12-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned,UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       }
    }
   ```

Если у масштабируемого набора виртуальных машин есть управляемые удостоверения, назначаемые системой и назначаемые пользователем, вы можете удалить все управляемые удостоверения, назначаемые пользователем, переключившись на использование только управляемого удостоверения, назначаемого системой, с помощью следующей команды.

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned"}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
```

```HTTP
PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
```

**Заголовки запроса**

|Заголовок запроса  |Описание  |
|---------|---------|
|*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
|*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

**Текст запроса**

```JSON
{
   "identity":{
      "type":"SystemAssigned"
   }
}
```
    
Если ваш масштабируемый набор виртуальных машин имеет только управляемые удостоверения, назначаемые пользователем, и вы хотите удалить их все, используйте следующую команду.

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"None"}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
```

```HTTP
PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myVMSS?api-version=2018-06-01 HTTP/1.1
```

**Заголовки запроса**

|Заголовок запроса  |Описание  |
|---------|---------|
|*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
|*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`. | 

**Текст запроса**

```JSON
{
   "identity":{
      "type":"None"
   }
}
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о создании и удалении управляемых удостоверений, назначаемых пользователем, а также получении их списка с помощью REST:

- [Создание и удаление управляемых удостоверений, назначаемых пользователем, а также получение их списка с помощью вызовов REST API](how-to-manage-ua-identity-rest.md)
