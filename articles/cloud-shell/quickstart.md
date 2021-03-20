---
title: Краткое руководство по Azure Cloud Shell-bash
description: Узнайте, как использовать командную строку Bash в браузере с Azure Cloud Shell.
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 03/12/2018
ms.author: damaerte
ms.openlocfilehash: 91b7c58890518559c046023bd78c9248e9840f9f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89468755"
---
# <a name="quickstart-for-bash-in-azure-cloud-shell"></a>Краткое руководство по Bash в Azure Cloud Shell

В этом документе объясняется, как использовать Bash в Azure Cloud Shell на [портале Azure](https://ms.portal.azure.com/).

> [!NOTE]
> Также вы можете ознакомиться с кратким руководством по использованию [PowerShell в Azure Cloud Shell](quickstart-powershell.md).

## <a name="start-cloud-shell"></a>Запуск Cloud Shell
1. Запустите **Cloud Shell** в верхней области навигации портал Azure. <br>
![Снимок экрана, показывающий, как запустить Azure Cloud Shell в портал Azure.](media/quickstart/shell-icon.png)

2. Выберите подписку для создания учетной записи хранения и общей папки для службы файлов Microsoft Azure.
3. Нажмите кнопку "Создать хранилище".

> [!TIP]
> Вы автоматически проходите проверку подлинности для Azure CLI в каждом сеансе.

### <a name="select-the-bash-environment"></a>Выбор среды Bash
Убедитесь, что в раскрывающемся списке сред в левой части окна оболочки отображается пункт `Bash`. <br>
![Снимок экрана, показывающий, как выбрать среду bash для Azure Cloud Shell.](media/quickstart/env-selector.png)

### <a name="set-your-subscription"></a>Настройка подписки
1. Выведите список подписок, к которым у вас есть доступ.
   ```azurecli-interactive
   az account list
   ```

2. Задайте предпочитаемую подписку:

   ```azurecli-interactive
   az account set --subscription 'my-subscription-name'
   ```

> [!TIP]
> Подписка будет сохранена для последующих сеансов в файле `/home/<user>/.azure/azureProfile.json`.

### <a name="create-a-resource-group"></a>Создание группы ресурсов
Создайте группу ресурсов в западной части США с именем MyRG:
```azurecli-interactive
az group create --location westus --name MyRG
```

### <a name="create-a-linux-vm"></a>Создание виртуальной машины Linux
Создайте виртуальную машину Ubuntu в новой группе ресурсов. Azure CLI создаст ключи SSH и настроит с их помощью виртуальную машину. <br>

```azurecli-interactive
az vm create -n myVM -g MyRG --image UbuntuLTS --generate-ssh-keys
```

> [!NOTE]
> Инструкция `--generate-ssh-keys` указывает Azure CLI создать и настроить открытый и закрытый ключи на виртуальной машине и в каталоге `$Home`. По умолчанию ключи размещаются в Cloud Shell в папках `/home/<user>/.ssh/id_rsa` и `/home/<user>/.ssh/id_rsa.pub`. Папка `.ssh` сохраняется в образе размером 5 ГБ, размещенном в подключенной общей папке, который используется для хранения `$Home`.

Имя пользователя на этой виртуальной машине будет использоваться в Cloud Shell ($User@Azure:).

### <a name="ssh-into-your-linux-vm"></a>SSH-подключение к виртуальной машине Linux
1. Найдите имя виртуальной машины на панели поиска портала Azure.
2. Щелкните "Подключиться", чтобы получить имя и общедоступный IP-адрес виртуальной машины. <br>
   ![Снимок экрана, показывающий, как подключиться к Linux V с помощью S H.](media/quickstart/sshcmd-copy.png)

3. Подключитесь к виртуальной машине по протоколу SSH с помощью команды `ssh`.
   ```
   ssh username@ipaddress
   ```

После установки SSH-подключения отобразится строка приветствия Ubuntu. <br>
![Снимок экрана, показывающий инициализацию Ubuntu и приветствие после установки подключения S H.](media/quickstart/ubuntu-welcome.png)

## <a name="cleaning-up"></a>Очистка 
1. Закройте сеанс SSH.
   ```
   exit
   ```

2. Удалите группу ресурсов и все входящие в нее ресурсы.
   ```azurecli-interactive
   az group delete -n MyRG
   ```

## <a name="next-steps"></a>Дальнейшие действия
[Дополнительные сведения о сохранении файлов для Bash в Cloud Shell.](persisting-shell-storage.md) <br>
[Справочник команд Azure CLI](/cli/azure/) <br>
[Общие сведения о службе файлов Azure](../storage/files/storage-files-introduction.md) <br>