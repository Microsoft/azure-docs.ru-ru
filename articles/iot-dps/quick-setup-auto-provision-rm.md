---
title: Краткое руководство. Создание Службы подготовки устройств к добавлению в Центр Интернета вещей Azure (DPS) с помощью шаблона Azure Resource Manager (шаблона ARM)
description: Краткое руководство по Azure. Сведения о том, как создать Службу подготовки устройств к добавлению в Центр Интернета вещей Azure (DPS) с помощью шаблона Azure Resource Manager (шаблона ARM).
author: wesmc7777
ms.author: wesmc
ms.date: 01/27/2021
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.custom: mvc, subject-armqs, devx-track-azurecli
ms.openlocfilehash: bcb37c624f53d961020de022569a621ca1dfaba1
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789003"
---
# <a name="quickstart-set-up-the-iot-hub-device-provisioning-service-dps-with-an-arm-template"></a>Краткое руководство. Настройка Службы подготовки устройств к добавлению в Центр Интернета вещей с помощью шаблона ARM

С помощью шаблона [Azure Resource Manager](../azure-resource-manager/management/overview.md) (шаблона ARM) вы можете программно настраивать облачные ресурсы Azure, необходимые для подготовки устройств. В этой процедуре показано, как создать центр Интернета вещей и новую Службу подготовки устройств к добавлению в Центр Интернета вещей с помощью шаблона ARM. Также этот шаблон связывает центр Интернета вещей с ресурсом DPS. Это связывание позволяет ресурсу DPS назначать устройства в центр по настроенным вами политикам распределения.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

В этом кратком руководстве для выполнения программных шагов, необходимых для создания группы ресурсов и развертывания шаблона, используются [портал Azure](../azure-resource-manager/templates/deploy-portal.md) и [Azure CLI](../azure-resource-manager/templates/deploy-cli.md). Однако для развертывания шаблона также можно применить [PowerShell](../azure-resource-manager/templates/deploy-powershell.md), .NET, Ruby или другие языки программирования. 

Если ваша среда соответствует всем предварительным требованиям и вы уже работали с шаблонами ARM, нажмите кнопку **Развернуть в Azure**, чтобы открыть на портале Azure шаблон развертывания.

[![Действие "Развернуть в Azure" на странице общих сведений](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-iothub-device-provisioning%2fazuredeploy.json)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]


## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-iothub-device-provisioning/).

> [!NOTE]
> В настоящее время шаблоны ARM не поддерживают создание регистраций с новыми ресурсами DPS. Наличие такой поддержки является востребованным и понятным, поэтому мы рассматриваем возможность ее добавления.

:::code language="json" source="~/quickstart-templates/101-iothub-device-provisioning/azuredeploy.json":::

В приведенном выше шаблоне определено два ресурса Azure:

* [**Microsoft.Devices/iothubs.**](/azure/templates/microsoft.devices/iothubs) Создает новый центр Интернета вещей Azure.
* [**Microsoft.Devices/provisioningservices.**](/azure/templates/microsoft.devices/provisioningservices) Создает новую Службу подготовки устройств к добавлению в центр Интернета вещей, которая уже связана с новым центром Интернета вещей.


## <a name="deploy-the-template"></a>Развертывание шаблона

#### <a name="deploy-with-the-portal"></a>Развертывание с помощью портала

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон развертывания. Этот шаблон создает новый центр Интернета вещей и ресурс DPS. Наш центр будет связан с этим ресурсом DPS.

    [![Этапы развертывания в Azure с помощью портала](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-iothub-device-provisioning%2fazuredeploy.json)

2. Выберите или введите следующие значения, а затем щелкните **Проверка и создание**.

    ![Параметры развертывания шаблона ARM на портале](./media/quick-setup-auto-provision-rm/arm-template-deployment-parameters-portal.png)    

    Сохраните все значения по умолчанию, для которых ниже не указано иное значение, чтобы создать центр Интернета вещей и ресурс DPS.

    | Поле | Описание |
    | :---- | :---------- |
    | **Подписка** | Выберите подписку Azure. |
    | **Группа ресурсов** | Щелкните **Создать**, введите уникальное имя группы ресурсов и нажмите кнопку **ОК**. |
    | **Регион** | Выберите регион для размещения ресурсов. Например, **Восточная часть США**. |
    | **Имя центра Интернета вещей** | Введите имя центра Интернета вещей, которое должно быть глобально уникальным в пределах пространства имен *.azure-devices.net*. Это имя центра вам потребуется в следующем разделе при проверке развертывания. |
    | **Имя службы подготовки** | Введите имя нового ресурса службы подготовки устройств (DPS). Это имя должно быть глобально уникальным в пределах пространства имен *.azure-devices-provisioning.net*. Это имя DPS вам потребуется в следующем разделе при проверке развертывания. |
    
3. На следующем экране ознакомьтесь с условиями. Если вы согласны со всеми условиями, щелкните **Создать**. 

    Развертывание займет несколько минут. 

    В дополнение к порталу Azure можно также использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-powershell.md).


#### <a name="deploy-with-the-azure-cli"></a>Развертывание с помощью Azure CLI

Для работы с Azure CLI потребуется версия 2.6 или более поздняя. Если вы используете Azure CLI локально, версию можно проверить командой `az --version`.

Войдите в учетную запись Azure и выберите подписку.

1. Если Azure CLI выполняется локально, а не на портале, необходимо выполнить вход. Для этого в командной строке запустите команду [login](/cli/azure/get-started-with-az-cli2):
    
    ```azurecli
    az login
    ```

    Следуйте инструкциям, чтобы выполнить аутентификацию с использованием кода и войти в учетную запись Azure через веб-браузер.

2. Если у вас есть несколько подписок Azure, то при выполнении входа в Azure вы получаете доступ ко всем учетным записям Azure, связанным с вашими учетными данными. Используйте следующую [команду для вывода учетных записей Azure](/cli/azure/account), доступных для использования:
    
    ```azurecli
    az account list -o table
    ```

    Используйте следующую команду, чтобы выбрать подписку, которая будет использоваться для выполнения команд, позволяющих создать ресурсы центра Интернета вещей и DPS. Вы можете использовать имя подписки или идентификатор из выходных данных предыдущей команды:

    ```azurecli
    az account set --subscription {your subscription name or id}
    ```

3. Скопируйте и вставьте в командную строку CLI приведенные ниже команды. Затем выполните эти команды, нажав клавишу **ВВОД**.
   
    > [!TIP]
    > При выполнении команд вам будет предложено указать расположение группы ресурсов. Список доступных расположений можно получить, выполнив такую команду:
    >
    > `az account list-locations -o table`
    >
    >
    
    ```azurecli-interactive
    read -p "Enter a project name that is used for generating resource names:" projectName &&
    read -p "Enter the location (i.e. centralus):" location &&
    templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-iothub-device-provisioning/azuredeploy.json" &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
    echo "Press [ENTER] to continue ..." &&
    read
    ```

4. При выполнении команд вам будет предложено ввести следующую информацию. Укажите эти значения, подтверждая каждое клавишей **ВВОД**.

    | Параметр | Описание |
    | :-------- | :---------- |
    | **Имя проекта** | Значение этого параметра будет использоваться для создания группы ресурсов, в которой будут храниться все ресурсы. В конец имени группы ресурсов будет добавлена строка `rg`. |
    | **расположение** | Это значение обозначает регион, в котором будут размещаться все ресурсы. |
    | **iotHubName** | Введите имя центра Интернета вещей, которое должно быть глобально уникальным в пределах пространства имен *.azure-devices.net*. Это имя центра вам потребуется в следующем разделе при проверке развертывания. |
    | **provisioningServiceName** | Введите имя нового ресурса службы подготовки устройств (DPS). Это имя должно быть глобально уникальным в пределах пространства имен *.azure-devices-provisioning.net*. Это имя DPS вам потребуется в следующем разделе при проверке развертывания. |

    Для развертывания шаблона используется Azure CLI. Помимо Azure CLI, вы можете использовать Azure PowerShell, портал Azure и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-powershell.md).


## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

1. Чтобы проверить развертывание, выполните следующую [команду получения списка ресурсов](/cli/azure/resource#az_resource_list), после чего найдите в выходных данных новую службу подготовки и центр Интернета вещей:

    ```azurecli
     az resource list -g "${projectName}rg"
    ```

2. Чтобы убедиться, что центр Интернета вещей связан с ресурсом DPS, выполните следующую команду [для отображения расширения DPS](/cli/azure/iot/dps#az_iot_dps_show).

    ```azurecli
     az iot dps show --name <Your provisioningServiceName>
    ```

    Найдите здесь центры, которые связаны с элементом `iotHubs`.


## <a name="clean-up-resources"></a>Очистка ресурсов

Другие краткие руководства в этой коллекции созданы на основе этого документа. Если вы планируете продолжать работу с этими руководствами по быстрому запуску или обычными руководствами, не удаляйте созданные ресурсы. Если вы не планируете продолжать работу, можно удалить группу ресурсов и все ее ресурсы с помощью портала Azure или Azure CLI.

Чтобы удалить группу ресурсов и все ее ресурсы с помощью портала Azure, просто откройте эту группу ресурсов и щелкните **Удалить группу ресурсов** в верхней части страницы.

Чтобы удалить развернутую группу ресурсов с помощью Azure CLI, сделайте следующее:

```azurecli
az group delete --name "${projectName}rg"
```

Группы ресурсов и отдельные ресурсы также можно удалять с помощью портала Azure, PowerShell, REST API и поддерживаемых пакетов SDK платформ, опубликованных для Azure Resource Manager или Службы подготовки устройств к добавлению в Центр Интернета вещей.

## <a name="next-steps"></a>Дальнейшие действия

Вы развернули центр Интернета вещей и экземпляр службы подготовки устройств, а затем связали эти два ресурса. Чтобы узнать, как в этой конфигурации подготовить устройство, см. краткое руководство по созданию устройства.

> [!div class="nextstepaction"]
> [Краткое руководство по подготовке устройства](./quick-create-simulated-device-symm-key.md)
