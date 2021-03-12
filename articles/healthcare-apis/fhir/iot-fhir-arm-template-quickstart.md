---
title: Краткое руководство. Развертывание соединителя "Azure IoT для FHIR" (предварительная версия) с использованием шаблона ARM
description: В этом кратком руководстве описано, как развернуть соединитель Azure IoT для FHIR (предварительная версия) с помощью шаблона Azure Resource Manager (шаблон ARM).
author: rbhaiya
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: quickstart
ms.author: rabhaiya
ms.date: 01/21/2021
ms.openlocfilehash: b241484b8e8d981036fff4998d475b8b80fae42d
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020580"
---
# <a name="quickstart-use-an-azure-resource-manager-arm-template-to-deploy-azure-iot-connector-for-fhir-preview"></a>Краткое руководство. Использование шаблона Azure Resource Manager (шаблона ARM) для развертывания соединителя Azure IoT для FHIR (предварительная версия)

Из этого краткого руководства вы узнаете, как с помощью шаблона Azure Resource Manager (шаблона ARM) развернуть соединитель Azure IoT для FHIR (стандарт "Ресурсы быстрого взаимодействия в сфере здравоохранения"), который предоставляется в рамках Azure API для FHIR. Чтобы развернуть рабочий экземпляр соединителя Azure IoT для FHIR, этот шаблон дополнительно развертывает родительскую службу Azure API для FHIR и приложение Azure IoT Central, которое экспортирует данные телеметрии из симулятора устройств в соединитель Azure IoT для FHIR. Вы можете выполнить шаблон ARM, который развертывает соединитель Azure IoT для FHIR, с помощью портала Azure, PowerShell или CLI.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. После входа шаблон откроется на портале Azure.

[:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание соединителя Azure IoT для FHIR в Azure с помощью шаблона ARM на портале Azure.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fiomt-fhir%2Fmaster%2Fdeploy%2Ftemplates%2Fmanaged%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

# <a name="portal"></a>[Портал](#tab/azure-portal)

Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/).

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

* Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/).
* Если вы хотите выполнить код локально, используйте [Azure PowerShell](/powershell/azure/install-az-ps).

# <a name="cli"></a>[CLI](#tab/CLI)

* Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/).
* Если вы хотите выполнить код локально, сделайте следующее.
    * Оболочка Bash (например, Git Bash, которая включена в [Git для Windows](https://gitforwindows.org)).
    * [Azure CLI](/cli/azure/install-azure-cli).

---

## <a name="review-the-template"></a>Изучение шаблона

Этот [шаблон](https://raw.githubusercontent.com/microsoft/iomt-fhir/master/deploy/templates/managed/azuredeploy.json) определяет следующие ресурсы Azure:

* **Microsoft.HealthcareApis/services**
* **Microsoft.HealthcareApis/services/iomtconnectors**;
* **Microsoft.IoTCentral/IoTApps**.

## <a name="deploy-the-template"></a>Развертывание шаблона

# <a name="portal"></a>[Портал](#tab/azure-portal)

Щелкните следующую ссылку, чтобы развернуть соединитель Azure IoT для FHIR с помощью шаблона ARM на портале Azure:

[:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание соединителя Azure IoT для FHIR в Azure с помощью шаблона ARM на портале Azure.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fiomt-fhir%2Fmaster%2Fdeploy%2Ftemplates%2Fmanaged%2Fazuredeploy.json)

На странице **Развертывание Azure API для FHIR** выполните следующие действия:

1. При необходимости замените **подписку** по умолчанию на другую подписку.

2. В разделе **Группа ресурсов** щелкните **Создать новую**, введите имя новой группы ресурсов и нажмите **ОК**.

3. Если вы создаете новую группу ресурсов, выберите для нее **Регион**.

4. Введите имя нового экземпляра API Azure для FHIR в поле **FHIR Service Name** (Имя службы FHIR).

5. Выберите **расположение** для API Azure для FHIR. Оно может отличаться от региона группы ресурсов или совпадать с ним.

6. Укажите имя для экземпляра соединителя Azure IoT для FHIR в поле **Iot Connector Name** (Имя соединителя IoT).

7. Укажите имя для подключения, созданного в соединителе Azure IoT для FHIR, в поле **Connection Name** (Имя подключения). Это подключение используется приложением Azure IoT Central, чтобы отправлять данные телеметрии с имитированного устройства в соединитель Azure IoT для FHIR.

8. Введите имя нового приложения Azure IoT Central в поле **Iot Central Name** (Имя Центра Интернета вещей). Это приложение будет использовать шаблон *непрерывного мониторинга пациента* для имитации устройства.

9. Выберите расположение для приложения IoT Central из раскрывающегося списка **IoT Central Location** (Расположение Центра Интернета вещей). 

10. Выберите **Review + create** (Просмотреть и создать).

11. Ознакомьтесь с условиями, а затем выберите **Создать**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> Если вы хотите запускать скрипты PowerShell локально, сначала введите `Connect-AzAccount`, чтобы настроить учетные данные Azure.

Если поставщик ресурсов `Microsoft.HealthcareApis` еще не зарегистрирован для вашей подписки, его можно зарегистрировать с помощью следующего интерактивного кода. Чтобы выполнить этот код в Azure Cloud Shell, щелкните **Попробовать** в правом верхнем углу блока кода.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.HealthcareApis
```

Если поставщик ресурсов `Microsoft.IoTCentral` еще не зарегистрирован для вашей подписки, его можно зарегистрировать с помощью следующего интерактивного кода. Чтобы выполнить этот код в Azure Cloud Shell, щелкните **Попробовать** в правом верхнем углу блока кода.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.IoTCentral
```

Используйте следующий код для развертывания службы соединителя Azure IoT для FHIR в Azure с помощью шаблона ARM.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"
$fhirServiceName = Read-Host -Prompt "Enter a name for the new Azure API for FHIR service"
$location = Read-Host -Prompt "Enter an Azure region (for example, westus2) for the service"
$iotConnectorName = Read-Host -Prompt "Enter a name for the new Azure IoT Connector for FHIR"
$connectionName = Read-Host -Prompt "Enter a name for the connection with Azure IoT Connector for FHIR"
$iotCentralName = Read-Host -Prompt "Enter a name for the new Azure IoT Central Application"
$iotCentralLocation = Read-Host -Prompt "Enter a location for the new Azure IoT Central Application"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion
Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure IoT Connector for FHIR service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri https://raw.githubusercontent.com/microsoft/iomt-fhir/master/deploy/templates/managed/azuredeploy.json `
    -fhirServiceName $fhirServiceName `
    -location $location
    -iotConnectorName $iotConnectorName
    -connectionName $connectionName
    -iotCentralName $iotCentralName
    -iotCentralLocation $iotCentralLocation
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Если поставщик ресурсов `Microsoft.HealthcareApis` еще не зарегистрирован для вашей подписки, его можно зарегистрировать с помощью следующего интерактивного кода. Чтобы выполнить этот код в Azure Cloud Shell, щелкните **Попробовать** в правом верхнем углу блока кода.

```azurecli-interactive
az extension add --name healthcareapis
```

Если поставщик ресурсов `Microsoft.IoTCentral` еще не зарегистрирован для вашей подписки, его можно зарегистрировать с помощью следующего интерактивного кода.

```azurecli-interactive
az extension add --name azure-iot
```

Используйте следующий код для развертывания службы соединителя Azure IoT для FHIR в Azure с помощью шаблона ARM.

```azurecli-interactive
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
read -p "Enter a name for the new Azure API for FHIR service: " fhirServiceName &&
read -p "Enter an Azure region (for example, westus2) for the service: " location &&
read -p "Enter a name for the new Azure IoT Connector for FHIR: " iotConnectorName &&
read -p "Enter a name for the connection with Azure IoT Connector for FHIR: " connectionName &&
read -p "Enter a name for the new Azure IoT Central Application: " iotCentralName &&
read -p "Enter a location for the new Azure IoT Central Application: " iotCentralLocation &&

params='fhirServiceName='$fhirServiceName' location='$location' iotConnectorName='$iotConnectorName' connectionName='$connectionName' iotCentralName='$iotCentralName' iotCentralLocation='$iotCentralLocation &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure IoT Connector for FHIR service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/microsoft/iomt-fhir/master/deploy/templates/managed/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> Развертывание занимает несколько минут. Запишите имена службы Azure API для FHIR, приложения Azure IoT Central и группы ресурсов. Они будут использоваться для просмотра развернутых ресурсов в дальнейшем.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы просмотреть обзор новой службы Azure API для FHIR, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com) найдите и выберите **Azure API для FHIR**.

2. В списке FHIR выберите свою новую службу. Откроется страница **Обзор** для новой службы Azure API для FHIR.

3. Убедитесь, что новая учетная запись API FHIR подготовлена. Для этого щелкните ссылку рядом с **конечной точкой метаданных FHIR**, чтобы получить отчет о возможностях API FHIR. Ссылка имеет формат `https://<service-name>.azurehealthcareapis.com/metadata`. Если учетная запись подготовлена, отображается большой JSON-файл.

4. Чтобы убедиться, что новый соединитель Azure IoT для FHIR успешно подготовлен, щелкните элемент **Соединитель IoT (предварительная версия)** в меню навигации слева, чтобы открыть страницу **Соединители IoT**. На этой странице должен отображаться подготовленный соединитель Azure IoT для FHIR, для которого параметр *Состояние* имеет значение *Подключено*, параметр *Подключения* имеет значение *1*, а в областях *Сопоставление устройств* и *Сопоставление FHIR* указан значок *Успешно*.

5. На [портале Azure](https://portal.azure.com) найдите и выберите **Приложения IoT Central**.

6. В списке приложений IoT Central выберите созданную вами службу. Откроется страница **Обзор** для нового приложения IoT Central.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Выполните следующий интерактивный код, чтобы просмотреть сведения о службе Azure API для FHIR. Вам нужно будет ввести имя новой службы FHIR и группы ресурсов.

```azurepowershell-interactive
$fhirServiceName = Read-Host -Prompt "Enter the name of your Azure API for FHIR service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new FHIR service has been provisioned"

$requestUri="https://" + $fhirServiceName + ".azurehealthcareapis.com/metadata"
$metadata = Invoke-WebRequest -Uri $requestUri
$metadata.RawContent
Read-Host "Press [ENTER] to continue"
```

> [!NOTE]
> Сейчас соединитель Azure IoT для FHIR не предоставляет команды PowerShell. Чтобы проверить правильность подготовки соединителя Azure IoT для FHIR, используйте процесс проверки на вкладке **Portal** (Портал).

Выполните приведенный ниже интерактивный код, чтобы просмотреть сведения о приложении Azure IoT Central. Вам нужно будет ввести имя нового приложения Azure IoT Central и группы ресурсов.

```azurepowershell-interactive
$iotCentralName = Read-Host -Prompt "Enter the name of your Azure IoT Central application"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzIotCentralApp -ResourceGroupName $resourceGroupName -Name $iotCentralName" -Verbose
Get-AzIotCentralApp -ResourceGroupName $resourceGroupName -Name $iotCentralName
```

# <a name="cli"></a>[CLI](#tab/CLI)

Выполните следующий интерактивный код, чтобы просмотреть сведения о службе Azure API для FHIR. Вам нужно будет ввести имя новой службы FHIR и группы ресурсов.

```azurecli-interactive
read -p "Enter the name of your Azure API for FHIR service: " fhirServiceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az healthcareapis service show --resource-group $resourceGroupName --resource-name $fhirServiceName" &&
az healthcareapis service show --resource-group $resourceGroupName --resource-name $fhirServiceName &&
read -p "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new service has been provisioned: " &&
requestUrl='https://'$fhirServiceName'.azurehealthcareapis.com/metadata' &&
curl --url $requestUrl &&
read -p "Press [ENTER] to continue: "
```

> [!NOTE]
> Сейчас соединитель Azure IoT для FHIR не предоставляет команды CLI. Чтобы проверить правильность подготовки соединителя Azure IoT для FHIR, используйте процесс проверки на вкладке **Portal** (Портал).

Выполните приведенный ниже интерактивный код, чтобы просмотреть сведения о приложении Azure IoT Central. Вам нужно будет ввести имя нового приложения Azure IoT Central и группы ресурсов.

```azurecli-interactive
read -p "Enter the name of your IoT Central application: " iotCentralName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az iot central app show -g $resourceGroupName -n $iotCentralName" &&
az iot central app show -g $resourceGroupName -n $iotCentralName &&
```

---

## <a name="connect-your-iot-data-with-the-azure-iot-connector-for-fhir-preview"></a>Подключение данных Интернета вещей Azure к соединителю "Azure IoT для FHIR" (предварительная версия)
> [!IMPORTANT]
> Шаблон сопоставления устройств, предоставленный в этом руководстве, предназначен для работы с функцией экспорта данных (устарела) в IoT Central.

Сейчас приложение IoT Central не предоставляет шаблон ARM, команды PowerShell или CLI для настройки экспорта данных. Поэтому выполните приведенные ниже инструкции на портале Azure.  

После развертывания приложения IoT Central два готовых виртуальных устройства начнут создавать данные телеметрии. В рамках этого учебника мы будем получать данные телеметрии из симулятора *Smart Vitals Patch* в FHIR через соединитель "Azure IoT для FHIR". Чтобы экспортировать данные Интернета вещей в соединитель "Azure IoT для FHIR", необходимо [настроить экспорт данных (прежняя версия) в IoT Central](../../iot-central/core/howto-export-data-legacy.md). На странице "Экспорт данных (прежняя версия)" сделайте следующее.
- Выберите *Центры событий Azure* в качестве назначения экспорта.
- Выберите значение *Использовать строку подключения* в поле **Пространство имен Центров событий**.
- Укажите строку подключения к соединителю "Azure IoT для FHIR", полученную на предыдущем шаге, в поле **Строка подключения**.
- В поле **Данные для экспорта** для параметра **Телеметрия** оставьте значение *Вкл.* .

---

## <a name="view-device-data-in-azure-api-for-fhir"></a>Просмотр данных устройства в Azure API для FHIR

Вы можете просматривать ресурсы наблюдения на основе FHIR, созданные соединителем Azure IoT для FHIR для сервера FHIR, используя Postman. Настройте [доступ к Azure API для FHIR в Postman](access-fhir-postman-tutorial.md) и выполните запрос `GET` к `https://your-fhir-server-url/Observation?code=http://loinc.org|8867-4`, чтобы просмотреть ресурсы наблюдения FHIR с информацией о частоте пульса.

> [!TIP]
> Убедитесь, что пользователь имеет необходимые права доступа к плоскости данных Azure API для FHIR. Для назначения ролей в плоскости данных используйте [управление доступом на основе ролей Azure (Azure RBAC)](configure-azure-rbac.md).

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите ненужную группу ресурсов, когда надобность в ней отпадет. Ресурсы в ней также будут удалены.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. На [портале Azure](https://portal.azure.com) найдите и выберите элемент **Группы ресурсов**.

2. В списке групп ресурсов выберите имя нужной группы ресурсов.

3. На странице **Обзор** для этой группы ресурсов выберите **Удалить группу ресурсов**.

4. В диалоговом окне подтверждения введите имя группы ресурсов и щелкните **Удалить**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the name of the resource group to delete"
Write-Verbose "Remove-AzResourceGroup -Name $resourceGroupName" -Verbose
Remove-AzResourceGroup -Name $resourceGroupName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
read -p "Enter the name of the resource group to delete: " resourceGroupName &&
echo "DELETE A RESOURCE GROUP (AND ITS RESOURCES):  az group delete --name $resourceGroupName" &&
az group delete --name $resourceGroupName &&
read -p "Press [ENTER] to continue: "
```

---

Пошаговые инструкции по созданию шаблона Resource Manager см. в [этом учебнике](../../azure-resource-manager/templates/template-tutorial-create-first-template.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы развернули функцию соединителя "Azure IoT для FHIR" в ресурсе Azure API для FHIR. Чтобы получить дополнительные сведения о соединителе "Azure IoT для FHIR", просмотрите следующие ресурсы.

Сведения о разных стадиях потока данных в соединителе "Azure IoT для FHIR".

>[!div class="nextstepaction"]
>[Поток данных для соединителя "Azure IoT для FHIR"](iot-data-flow.md)

Сведения о настройке соединителя Интернета вещей с помощью шаблонов устройства и сопоставления FHIR.

>[!div class="nextstepaction"]
>[Шаблоны сопоставления для соединителя "Azure IoT для FHIR"](iot-mapping-templates.md)

*На портале Azure соединитель Azure IoT для FHIR называется соединителем IoT (предварительная версия). FHIR — это зарегистрированная торговая марка организации HL7, которая используется с разрешения HL7.
