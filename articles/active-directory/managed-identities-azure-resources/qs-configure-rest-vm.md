---
title: Настройка управляемых удостоверений на виртуальной машине Azure с помощью REST — Azure AD
description: Пошаговые инструкции по настройке управляемых удостоверений, назначаемых системой и назначаемых пользователем, на виртуальной машине Azure с использованием CURL для выполнения вызовов REST API.
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
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9b74e7d312133c24daad448e029a3c3d4cbdce79
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773087"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-rest-api-calls"></a>Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью вызовов REST API

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые системные удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде. 

В этой статье вы узнаете, как выполнять приведенные ниже операции с управляемыми удостоверениями для ресурсов Azure на виртуальной машине Azure, используя CURL для выполнения вызовов к конечной точке REST Azure Resource Manager.

- Включение и отключение управляемого удостоверения, назначаемого системой, на виртуальной машине Azure
- Добавление и удаление управляемого удостоверения, назначаемого пользователем, в виртуальной машине Azure

Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжать.

## <a name="prerequisites"></a>Обязательные условия

- Если вы не работали с управляемыми удостоверениями для ресурсов Azure, см. статью [Что такое управляемые удостоверения для ресурсов Azure](overview.md). Сведения об удостоверениях, назначаемых системой и назначаемых пользователем, см. в разделе [Типы управляемых удостоверений](overview.md#managed-identity-types).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="system-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое системой

Этот раздел описывает, как включить и отключить управляемое удостоверение, назначаемое системой, на виртуальной машине Azure, используя CURL для выполнения вызовов к конечной точке REST Azure Resource Manager.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>Включение управляемого удостоверения, назначаемого системой, во время создания виртуальной машины Azure

Чтобы создать виртуальную машину Azure с включенным управляемым удостоверением, назначаемым системой, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. Создайте [группу ресурсов](../../azure-resource-manager/management/overview.md#terminology) с помощью параметра [az group create](/cli/azure/group/#az_group_create), чтобы сохранить и развернуть виртуальную машину и связанные с ней ресурсы. Если вы уже создали группу ресурсов, которую можно использовать, этот шаг можно пропустить:

   ```azurecli-interactive 
   az group create --name myResourceGroup --location westus
   ```

2. Создайте [сетевой интерфейс](/cli/azure/network/nic#az_network_nic_create) для виртуальной машины:

   ```azurecli-interactive
    az network nic create -g myResourceGroup --vnet-name myVnet --subnet mySubnet -n myNic
   ```

3. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания виртуальной машины с удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ``` 

4. С помощью Azure Cloud Shell создайте виртуальную машину, используя CURL для вызова конечной точки REST Azure Resource Manager. Приведенный ниже пример создает виртуальную машину *myVM* с управляемым удостоверением, назначаемым системой, в соответствии со значением `"identity":{"type":"SystemAssigned"}` в тексте запроса. Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"SystemAssigned"},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"<SECURE PASSWORD STRING>"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   
   **Заголовки запроса**
   
   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 
   
   **Текст запроса**

   ```JSON
     {
       "location":"westus",
       "name":"myVM",
       "identity":{
          "type":"SystemAssigned"
       },
       "properties":{
          "hardwareProfile":{
             "vmSize":"Standard_D2_v2"
          },
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
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{
             "networkInterfaces":[
                {
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{
                      "primary":true
                   }
                }
             ]
          }
       }
    }  
   ```

### <a name="enable-system-assigned-identity-on-an-existing-azure-vm"></a>Включение удостоверения, назначаемого системой, на имеющейся виртуальной машине Azure

Чтобы включить назначаемое системой управляемое удостоверение на виртуальной машине, которая была подготовлена без него, вашей учетной записи должна быть назначена роль [Участника виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания виртуальной машины с удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Используйте приведенную ниже команду CURL для вызова конечной точки REST Azure Resource Manager, чтобы включить управляемое удостоверение, назначаемое системой, на виртуальной машине в соответствии со значением `{"identity":{"type":"SystemAssigned"}` в тексте запроса для виртуальной машины *myVM*.  Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.
   
   > [!IMPORTANT]
   > Чтобы предотвратить удаление существующих управляемых удостоверений, назначаемых пользователем, для виртуальной машины, нужно получить список этих удостоверений с помощью следующей команды CURL: `curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"`. Если для виртуальной машины заданы управляемые удостоверения, назначаемые пользователем, как указано в значении `identity` в ответе, перейдите к шагу 3, где описано, как сохранить управляемые удостоверения, назначаемые пользователем, при включении на виртуальной машине управляемого удостоверения, назначаемого системой.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 
   
   **Текст запроса**
    
   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned"
       }
    }
   ```

3. Чтобы включить управляемое удостоверение, назначаемое системой, на виртуальной машине с существующими управляемыми удостоверениями, назначаемыми пользователем, нужно добавить `SystemAssigned` в значение `type`.  
   
   Например, если для виртуальной машины назначены управляемые удостоверения, назначаемые пользователем, `ID1` и `ID2`, и вы хотите добавить для нее управляемое удостоверение, назначаемое системой, используйте приведенный ниже вызов CURL. Замените `<ACCESS TOKEN>` и `<SUBSCRIPTION ID>` значениями, уместными для вашей среды.

   API версии `2018-06-01` хранит управляемые удостоверения, назначаемые пользователем, в значении `userAssignedIdentities` в формате словаря в отличие от значения `identityIds` в формате массива, используемого в API версии `2017-12-01`.
   
   **API версии 2018-06-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{},"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

   **Текст запроса**

   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{  
             "/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{  
    
             },
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{  
    
             }
          }
       }
    }
   ```

   **API версии 2017-12-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "identityIds":["/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1","/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```
    
   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

   **Текст запроса**

   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned, UserAssigned",
          "identityIds":[  
             "/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1",
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"
          ]
       }
    }
   ```   

### <a name="disable-system-assigned-managed-identity-from-an-azure-vm"></a>Отключение управляемого удостоверения, назначаемого системой, на виртуальной машине Azure

Чтобы отключить назначаемое системой управляемое удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания виртуальной машины с удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Измените виртуальную машину, используя CURL для вызова конечной точки REST Azure Resource Manager, чтобы отключить управляемое удостоверение, назначаемое системой.  Приведенный ниже пример отключает управляемое удостоверение, назначаемое системой, на виртуальной машине *myVM* в соответствии со значением `{"identity":{"type":"None"}}` в тексте запроса.  Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.

   > [!IMPORTANT]
   > Чтобы предотвратить удаление существующих управляемых удостоверений, назначаемых пользователем, для виртуальной машины, нужно получить список этих удостоверений с помощью следующей команды CURL: `curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"`. Если для виртуальной машины заданы управляемые удостоверения, назначаемые пользователем, как указано в значении `identity` в ответе, перейдите к шагу 3, где описано, как сохранить управляемые удостоверения, назначаемые пользователем, при отключении на виртуальной машине управляемого удостоверения, назначаемого системой.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"None"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

   **Текст запроса**

   ```JSON
    {  
       "identity":{  
          "type":"None"
       }
    }
   ```

   Чтобы удалить управляемое удостоверение, назначаемое системой, из виртуальной машины, в которой есть управляемые удостоверения, назначаемые пользователем, удалите `SystemAssigned` из значения `{"identity":{"type:" "}}`, сохранив значение `UserAssigned` и значения словаря `userAssignedIdentities`, если используется **API версии 2018-06-01**. При использовании **API версии 2017-12-01** или более ранних версий сохраните массив `identityIds`.

## <a name="user-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое пользователем

В этом разделе описывается, как добавить и удалить управляемое удостоверение, назначаемое пользователем, в виртуальной машине Azure, используя CURL для выполнения вызовов к конечной точке REST Azure Resource Manager.

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-an-azure-vm"></a>Задание управляемого удостоверения, назначаемого пользователем, во время создания виртуальной машины Azure

Чтобы присвоить назначаемое пользователем удостоверение виртуальной машине, вашей учетной записи должны быть назначены роли [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) и [Оператор управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Назначать другие роли в каталоге Azure AD не требуется.

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания виртуальной машины с удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Создайте [сетевой интерфейс](/cli/azure/network/nic#az_network_nic_create) для виртуальной машины:

   ```azurecli-interactive
    az network nic create -g myResourceGroup --vnet-name myVnet --subnet mySubnet -n myNic
   ```

3. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания виртуальной машины с удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ``` 

4. Создайте пользовательское удостоверение, как описано в разделе [Создание управляемого удостоверения, назначаемого пользователем](how-to-manage-ua-identity-rest.md#create-a-user-assigned-managed-identity).

5. Создайте виртуальную машину, используя CURL для вызова конечной точки REST Azure Resource Manager. Приведенный ниже пример создает в группе ресурсов *myResourceGroup* виртуальную машину *myVM* с управляемым удостоверением, назначаемым пользователем, `ID1` в соответствии со значением `"identity":{"type":"UserAssigned"}` в тексте запроса. Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.
 
   **API версии 2018-06-01**

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"UserAssigned","identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```
    
   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

   **Текст запроса**

   ```JSON
    {  
       "location":"westus",
       "name":"myVM",
       "identity":{  
          "type":"UserAssigned",
          "identityIds":[  
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       },
       "properties":{  
          "hardwareProfile":{  
             "vmSize":"Standard_D2_v2"
          },
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
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[  
                {  
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {  
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{  
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{  
             "networkInterfaces":[  
                {  
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{  
                      "primary":true
                   }
                }
             ]
          }
       }
    }

   ```
  
   **API версии 2017-12-01**

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"UserAssigned","identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

   **Текст запроса**

   ```JSON
    {
       "location":"westus",
       "name":"myVM",
       "identity":{
          "type":"UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       },
       "properties":{
          "hardwareProfile":{
             "vmSize":"Standard_D2_v2"
          },
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
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{
             "networkInterfaces":[
                {
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{
                      "primary":true
                   }
                }
             ]
          }
       }
    }
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>Задание управляемого удостоверения, назначаемого пользователем, для имеющейся виртуальной машины Azure

Чтобы присвоить назначаемое пользователем удостоверение виртуальной машине, вашей учетной записи должны быть назначены роли [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) и [Оператор управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Назначать другие роли в каталоге Azure AD не требуется.

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания виртуальной машины с удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2.  Создайте управляемое удостоверение, назначаемое пользователем, как описано в разделе [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью вызовов REST API](how-to-manage-ua-identity-rest.md#create-a-user-assigned-managed-identity).

3. Чтобы предотвратить удаление существующих управляемых удостоверений, назначаемых пользователем или назначаемых системой, для виртуальной машины, нужно получить список назначенных ей типов удостоверений с помощью приведенной ниже команды CURL. Если у вас есть управляемые удостоверения, назначенные масштабируемому набору виртуальных машин, они перечислены в значении `identity`.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>" 
   ```

   ```HTTP
   GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01 HTTP/1.1
   ```
   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.

    Если для виртуальной машины заданы управляемые удостоверения, назначаемые пользователем или назначаемые системой, как указано в значении `identity` в ответе, перейдите к шагу 5, где описано, как сохранить управляемое удостоверение, назначаемое системой, при добавлении в виртуальную машину управляемого удостоверения, назначаемого пользователем.

4. Если для виртуальной машины не заданы какие-либо управляемые удостоверения, назначаемые пользователем, используйте приведенную ниже команду CURL для вызова конечной точки REST Azure Resource Manager, чтобы назначить виртуальной машине первое управляемое удостоверение, назначаемое пользователем.

   Следующий пример задает управляемое удостоверение, назначаемое пользователем `ID1` для виртуальной машины *myVM* в группе ресурсов *myResourceGroup*.  Замените `<ACCESS TOKEN>` значением, полученным на предыдущем шаге при запросе маркера доступа носителя, а вместо `<SUBSCRIPTION ID>` укажите значение, подходящее для вашей среды.

   **API версии 2018-06-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        |
 
   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
    
             }
          }
       }
    }
   ```

   **API версии 2017-12-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"userAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```
   
   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

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

5. Если для виртуальной машины заданы управляемые удостоверения, назначаемые пользователем или назначаемые системой, сделайте следующее.
   
   **API версии 2018-06-01**

   Добавьте управляемое удостоверение, назначаемое пользователем, в значение словаря `userAssignedIdentities`.
    
   Например, если для виртуальной машины заданы управляемое удостоверение, назначаемое системой, и управляемое удостоверение, назначаемое пользователем, `ID1` и вы хотите добавить для него управляемое удостоверение, назначаемое пользователем, `ID2`, сделайте следующее.

   ```bash
   curl  'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{},"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   
   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

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

   Например, если для виртуальной машины заданы управляемое удостоверение, назначаемое системой, и управляемое удостоверение, назначаемое пользователем, `ID1` и вы хотите добавить для него управляемое удостоверение, назначаемое пользователем, `ID2`, сделайте следующее. 

   ```bash
   curl  'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned,UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1","/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

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

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Удаление управляемого удостоверения, назначаемого пользователем, из виртуальной машины Azure

Чтобы удалить назначаемое пользователем удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

1. Получите маркер доступа носителя, который понадобится в следующем шаге в заголовке авторизации для создания виртуальной машины с удостоверением, назначаемым системой.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Чтобы предотвратить удаление существующих управляемых удостоверений, назначаемых пользователем, которые нужно сохранить для виртуальной машины, или удалить управляемое удостоверение, назначаемое системой, нужно получить список управляемых удостоверений с помощью следующей команды CURL. 

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"
   ```
 
   ```HTTP
   GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.
 
   Если у вас есть управляемые удостоверения, назначенные виртуальной машине, они перечислены в значении `identity` в ответе.

   Например, если для виртуальной машины заданы управляемые удостоверения, назначаемые пользователем, `ID1` и `ID2`, и вы хотите оставить назначенным только `ID1`, при этом сохранив удостоверение, назначаемое системой, сделайте следующее.
   
   **API версии 2018-06-01**

   К управляемому удостоверению, назначаемому пользователем, которое требуется удалить, необходимо добавить `null`.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":null}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

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
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **Заголовки запроса**

   |Заголовок запроса  |Описание  |
   |---------|---------|
   |*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
   |*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        | 

   **Текст запроса**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       }
    }
   ```

Если у виртуальной машины есть управляемые удостоверения, назначаемые системой и назначаемые пользователем, вы можете удалить все управляемые удостоверения, назначаемые пользователем, переключившись на использование только управляемого удостоверения, назначаемого системой, с помощью следующей команды.

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
```

```HTTP
PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
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
    
Если ваша виртуальная машина имеет только управляемые удостоверения, назначаемые пользователем, и вы хотите удалить их все, используйте следующую команду.

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"None"}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
```

```HTTP
PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
```

**Заголовки запроса**

|Заголовок запроса  |Описание  |
|---------|---------|
|*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
|*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.| 

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
