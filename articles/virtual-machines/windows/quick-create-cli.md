---
title: Краткое руководство. Создание виртуальной машины Windows с помощью Azure CLI
description: Из этого краткого руководства вы узнаете, как с помощью Azure CLI создать виртуальную машину Windows.
author: cynthn
ms.service: virtual-machines
ms.collection: windows
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 07/02/2019
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 769b42133fb894c916ecaa3c42dd7de85206c765
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102556253"
---
# <a name="quickstart-create-a-windows-virtual-machine-with-the-azure-cli"></a>Краткое руководство. Создание виртуальной машины Windows с помощью Azure CLI

Azure CLI используется для создания ресурсов Azure и управления ими из командной строки или с помощью скриптов. В этом кратком руководстве показано, как с помощью Azure CLI развернуть в Azure виртуальную машину под управлением Windows Server 2016. Чтобы проверить работу виртуальной машины, вы подключитесь к ней по протоколу удаленного рабочего стола (RDP) и установите веб-сервер IIS.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/bash](https://shell.azure.com/bash). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу **ВВОД**, чтобы выполнить его.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm). В следующем примере создается виртуальная машина с именем *myVM*. В этом примере используется *azureuser* для имени пользователя с правами администратора. 

Потребуется указать пароль, отвечающий [требованиям к паролям для виртуальных машин Azure](./faq.md#what-are-the-password-requirements-when-creating-a-vm
). Когда вы примените приведенный ниже пример, появится запрос на ввод пароля в командной строке. Можете также добавить параметр `--admin-password` со значением пароля. Имя пользователя и пароль будут использоваться позже при подключении к виртуальной машине.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser 
```

Создание виртуальной машины и вспомогательных ресурсов занимает несколько минут. В следующем примере выходных данных показано, что виртуальная машина успешно создана.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

Запишите значение `publicIpAddress`, которое появится в выходных данных виртуальной машины. Этот адрес будет использоваться для доступа к виртуальной машине на следующих шагах.

## <a name="open-port-80-for-web-traffic"></a>Открытие порта 80 для веб-трафика

По умолчанию при создании виртуальной машины Windows в Azure открыты только подключения по протоколу RDP. Откройте TCP-порт 80, который понадобится для веб-сервера IIS, с помощью команды [az vm open-port](/cli/azure/vm).

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-virtual-machine"></a>Подключение к виртуальной машине

На локальном компьютере выполните следующую команду, чтобы создать сеанс удаленного рабочего стола. Замените указанный IP-адрес общедоступным IP-адресом своей виртуальной машины. Когда появится запрос, введите учетные данные, указанные при создании виртуальной машины.

```powershell
mstsc /v:publicIpAddress
```

## <a name="install-web-server"></a>Установка веб-сервера

Чтобы проверить работу виртуальной машины, установите веб-сервер IIS. На виртуальной машине откройте командную строку PowerShell и выполните следующую команду:

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

После этого закройте RDP-подключение к виртуальной машине.

## <a name="view-the-web-server-in-action"></a>Проверка работы веб-сервера

Установив IIS и открыв через Интернет порт 80 на виртуальной машине, откройте страницу приветствия IIS по умолчанию в любом браузере. Используйте общедоступный IP-адрес виртуальной машины, полученный на предыдущем шаге. В следующем примере показан веб-сайт IIS по умолчанию:

![Сайт IIS по умолчанию](./media/quick-create-powershell/default-iis-website.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда группа ресурсов, виртуальная машина и все связанные с ней ресурсы станут ненужны, их можно удалить с помощью команды [az group delete](/cli/azure/group).

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы развернули простую виртуальную машину, открыли сетевой порт для веб-трафика и установили базовый веб-сервер. Дополнительные сведения о виртуальных машинах Azure см. в руководстве по работе с виртуальными машинами Windows.

> [!div class="nextstepaction"]
> [Создание виртуальных машин Windows и управление ими с помощью модуля Azure PowerShell](./tutorial-manage-vm.md)
