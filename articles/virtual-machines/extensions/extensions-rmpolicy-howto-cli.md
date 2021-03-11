---
title: Использование политики Azure для ограничения установки расширения виртуальной машины (Linux)
description: Служба "Политика Azure" позволяет ограничить развертывание расширений на виртуальных машинах.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: linux
ms.date: 03/23/2018
ms.openlocfilehash: 582f690c00ff8bdd06e4d2867724aa482bcacaf3
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102559976"
---
# <a name="use-azure-policy-to-restrict-extensions-installation-on-linux-vms"></a>Ограничение установки расширений на виртуальных машинах Linux с помощью службы "Политика Azure"

Если вы хотите предотвратить использование или установку определенных расширений на виртуальных машинах Linux, можно создать определение политики Azure с помощью интерфейса командной строки, чтобы ограничить расширения для виртуальных машин в группе ресурсов. 

В этом руководстве используется интерфейс командной строки в Azure Cloud Shell, который постоянно обновляется до последней версии. Если вы намерены запустить Azure CLI локально, установите версию не ниже 2.0.26. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli). 

## <a name="create-a-rules-file"></a>Создание файла правил

Чтобы ограничить допустимые для установки расширения, вам потребуется [правило](../../governance/policy/concepts/definition-structure.md#policy-rule) с логикой определения расширений.

В этом примере показано, как можно запретить установку расширений, опубликованных издателем Microsoft.OSTCExtensions, создав файл правил в Azure Cloud Shell. Если вы работаете с локальным интерфейсом командной строки, можете создать локальный файл и заменить путь ~/clouddrive фактическим расположением файла на компьютере.

В [оболочке Bash Cloud Shell](https://shell.azure.com/bash) введите такой текст:

```bash
vim ~/clouddrive/azurepolicy.rules.json
```

Скопируйте следующий код JSON и вставьте его в файл.

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.OSTCExtensions/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.OSTCExtensions/virtualMachines/extensions/publisher",
                "equals": "Microsoft.OSTCExtensions"
            },
            {
                "field": "Microsoft.OSTCExtensions/virtualMachines/extensions/type",
                "in": "[parameters('notAllowedExtensions')]"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

Закончив копирование, нажмите клавишу **Esc** и введите символы **:wq**, чтобы сохранить и закрыть файл.


## <a name="create-a-parameters-file"></a>Создание файла параметров

Вам понадобится также файл [параметров](../../governance/policy/concepts/definition-structure.md#parameters), который позволяет создать структуру для передачи списка блокируемых расширений. 

Этот пример демонстрирует, как создать в Cloud Shell файл параметров для виртуальных машин Linux. Если вы работаете с локальным интерфейсом командной строки, можете создать локальный файл и заменить путь ~/clouddrive фактическим расположением файла на компьютере.

В [оболочке Bash Cloud Shell](https://shell.azure.com/bash) введите такой текст:

```bash
vim ~/clouddrive/azurepolicy.parameters.json
```

Скопируйте следующий код JSON и вставьте его в файл.

```json
{
    "notAllowedExtensions": {
        "type": "Array",
        "metadata": {
            "description": "The list of extensions that will be denied. Example: CustomScriptForLinux, VMAccessForLinux etc.",
            "displayName": "Denied extension"
        }
    }
}
```

Закончив копирование, нажмите клавишу **Esc** и введите символы **:wq**, чтобы сохранить и закрыть файл.

## <a name="create-the-policy"></a>Создание политики

Определение политики — это объект для хранения конфигурации, которую нужно использовать. Для определения политики используются файлы правил и параметров. Создайте определение политики с помощью команды [az policy definition create](/cli/azure/role/assignment).

В этом примере правила и параметры представлены в виде файлов, которые вы создали в формате JSON и сохранили с помощью Cloud Shell.

```azurecli-interactive
az policy definition create \
   --name 'not-allowed-vmextension-linux' \
   --display-name 'Block VM Extensions' \
   --description 'This policy governs which VM extensions that are blocked.' \
   --rules '~/clouddrive/azurepolicy.rules.json' \
   --params '~/clouddrive/azurepolicy.parameters.json' \
   --mode All
```


## <a name="assign-the-policy"></a>Назначение политики

В этом примере назначается политика для группы ресурсов с помощью команды [az policy assignment create](/cli/azure/policy/assignment). На всех виртуальных машинах, созданных в группе ресурсов **myResourceGroup**, невозможно будет установить расширение доступа к виртуальной машине Linux и пользовательские расширения для Linux. Прежде чем назначить политику, нужно создать группу ресурсов.

Команда [az account list](/cli/azure/account) предоставит вам идентификатор подписки, который следует указать вместо образца в примере.


```azurecli-interactive
az policy assignment create \
   --name 'not-allowed-vmextension-linux' \
   --scope /subscriptions/<subscription Id>/resourceGroups/myResourceGroup \
   --policy "not-allowed-vmextension-linux" \
   --params '{
        "notAllowedExtensions": {
            "value": [
                "VMAccessForLinux",
                "CustomScriptForLinux"
            ]
        }
    }'
```

## <a name="test-the-policy"></a>Проверка политики

Для проверки политики создайте виртуальную машину и попробуйте добавить нового пользователя.


```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --generate-ssh-keys
```

Попытайтесь создать пользователя с именем **myNewUser** с помощью расширения доступа к виртуальной машине.

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --password 'mynewuserpwd123!'
```



## <a name="remove-the-assignment"></a>Удаление назначения

```azurecli-interactive
az policy assignment delete --name 'not-allowed-vmextension-linux' --resource-group myResourceGroup
```
## <a name="remove-the-policy"></a>Удаление политики

```azurecli-interactive
az policy definition delete --name 'not-allowed-vmextension-linux'
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Что такое служба "Политика Azure"?](../../governance/policy/overview.md)
