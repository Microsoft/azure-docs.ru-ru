---
title: Развертывание расширений виртуальной машины с помощью шаблона
description: Сведения о том, как развернуть расширения виртуальной машины с помощью шаблонов Resource Manager.
author: mumian
ms.date: 03/26/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 320908ab4b24c43e8bd5209d4f32ef0211036958
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105628430"
---
# <a name="tutorial-deploy-virtual-machine-extensions-with-arm-templates"></a>Руководство по развертыванию расширений виртуальных машин с помощью шаблонов Resource Manager

Узнайте, как использовать [расширения виртуальных машин Azure](../../virtual-machines/extensions/features-windows.md), чтобы выполнять задачи настройки и автоматизации после развертывания виртуальных машин Azure. Существует множество разных расширений виртуальных машин, которые можно использовать с виртуальными машинами Azure. При работе с этим учебником вы развернете расширение пользовательского скрипта из шаблона Azure Resource Manager для выполнения скрипта PowerShell на виртуальной машине Windows. Скрипт устанавливает веб-сервер на виртуальной машине.

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Подготовка скрипта PowerShell
> * Открытие шаблона быстрого запуска
> * Изменение шаблона
> * Развертывание шаблона

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей необходимо иметь следующее.

* Visual Studio Code с расширением средств Resource Manager. См. [Краткое руководство. Создание шаблонов ARM с помощью Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).
* Для повышения уровня безопасности используйте пароль, созданный для учетной записи администратора виртуальной машины. Ниже приведен пример создания пароля.

    ```console
    openssl rand -base64 32
    ```

    Для защиты криптографических ключей и других секретов используйте Azure Key Vault. Дополнительные сведения см. в статье [Учебник. Интеграция Azure Key Vault в развертывание шаблона Resource Manager](./template-tutorial-use-key-vault.md). Мы также советуем обновлять пароль каждые три месяца.

## <a name="prepare-a-powershell-script"></a>Подготовка скрипта PowerShell

Вы можете использовать встроенный скрипт PowerShell или файл скрипта. В этом руководстве описано, как использовать файл скрипта. Используется доступный на [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-vm-extension/installWebServer.ps1) скрипт PowerShell со следующим содержимым:

```azurepowershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Если вы решили опубликовать файл в своем расположении, обновите элемент `fileUri` в шаблоне, как описано в этом руководстве.

## <a name="open-a-quickstart-template"></a>Открытие шаблона быстрого запуска

Шаблоны быстрого запуска Azure — это репозиторий для шаблонов ARM. Вместо создания шаблона с нуля можно найти пример шаблона и настроить его. Шаблон, используемый в этом руководстве, называется [Deploy a simple Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/) (Развертывание простой виртуальной машины Windows).

1. В Visual Studio Code выберите **Файл** > **Открыть файл**.
1. В поле **Имя файла** вставьте следующий URL-адрес:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```

1. Чтобы открыть файл, выберите **Открыть**.
    Шаблон определяет пять ресурсов:

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

     Прежде чем настраивать шаблон, рекомендуется получить основные сведения о нем.

1. Сохраните файл под именем *azuredeploy.json* на локальный компьютер, выбрав **Файл** > **Сохранить как**.

## <a name="edit-the-template"></a>Изменение шаблона

Добавьте ресурс расширения виртуальной машины в существующий шаблон со следующим содержимым:

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "apiVersion": "2020-12-01",
  "name": "[concat(variables('vmName'),'/', 'InstallWebServer')]",
  "location": "[parameters('location')]",
  "dependsOn": [
      "[concat('Microsoft.Compute/virtualMachines/',variables('vmName'))]"
  ],
  "properties": {
      "publisher": "Microsoft.Compute",
      "type": "CustomScriptExtension",
      "typeHandlerVersion": "1.7",
      "autoUpgradeMinorVersion":true,
      "settings": {
        "fileUris": [
          "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-vm-extension/installWebServer.ps1"
        ],
        "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File installWebServer.ps1"
      }
  }
}
```

Дополнительные сведения об определении этого ресурса см. в [справочнике по расширению](/azure/templates/microsoft.compute/virtualmachines/extensions). Ниже приведены некоторые важные элементы.

* `name`. Так как ресурс расширения является дочерним ресурсом объекта виртуальной машины, у имени должен быть префикс имени виртуальной машины. См. о [настройке имени и типа дочернего ресурса](child-resource-name-type.md).
* `dependsOn`. Создайте ресурс расширения после создания виртуальной машины.
* `fileUris`. Это расположение, в котором хранятся файлы скриптов. Если вы решили не использовать указанное расположение, необходимо обновить значения.
* `commandToExecute`. Эта команда запускает скрипт.

Чтобы использовать встроенный скрипт, удалите `fileUris` и обновите `commandToExecute` следующим образом:

```powershell
powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools && powershell.exe remove-item 'C:\\inetpub\\wwwroot\\iisstart.htm' && powershell.exe Add-Content -Path 'C:\\inetpub\\wwwroot\\iisstart.htm' -Value $('Hello World from ' + $env:computername)
```

Этот встроенный скрипт также обновляет содержимое файла _iisstart.html_.

Кроме того, необходимо открыть HTTP-порт, чтобы вы могли получить доступ к веб-серверу.

1. Найдите `securityRules` в шаблоне.
1. Добавьте следующее правило рядом с **default-allow-3389**.

    ```json
    {
      "name": "AllowHTTPInBound",
      "properties": {
        "priority": 1010,
        "access": "Allow",
        "direction": "Inbound",
        "destinationPortRange": "80",
        "protocol": "Tcp",
        "sourcePortRange": "*",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "*"
      }
    }
    ```

## <a name="deploy-the-template"></a>Развертывание шаблона

Дополнительные сведения о процедуре развертывания см. в разделе **Развертывание шаблона** статьи [Руководство. Создание шаблонов Resource Manager с зависимыми ресурсами](./template-tutorial-create-templates-with-dependent-resources.md#deploy-the-template). Мы советуем использовать пароль, созданный для учетной записи администратора виртуальной машины. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites) этой статьи.

В Cloud Shell выполните следующую команду, чтобы получить общедоступный IP-адрес виртуальной машины.

```azurepowershell
(Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName).IpAddress
```

Вставьте URL-адрес в веб-браузер. Откроется страница приветствия IIS по умолчанию, которая будет выглядеть следующим образом:

![Страница приветствия служб IIS](./media/template-tutorial-deploy-vm-extensions/resource-manager-template-deploy-extensions-customer-script-web-server.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если развернутые ресурсы Azure больше не нужны, вы можете их очистить. Для этого необходимо удалить группу ресурсов.

1. На портале Azure на панели слева выберите **Группа ресурсов**.
2. В поле **Фильтровать по имени** введите имя группы ресурсов.
3. Выберите имя группы ресурсов.
    В группе ресурсов будет отображаться шесть ресурсов.
4. В главном меню выберите **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы развернули виртуальную машину и расширение виртуальной машины. Расширение установило веб-сервер IIS на виртуальной машине. Чтобы узнать, как использовать расширение Базы данных SQL Azure для импорта BACPAC-файла, перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Развертывание расширений SQL](./template-tutorial-deploy-sql-extensions-bacpac.md)
