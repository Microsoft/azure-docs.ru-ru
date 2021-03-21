---
title: Расширение виртуальной машины агента наблюдателя за сетями Azure для Linux
description: Развертывание агента Наблюдателя за сетями на виртуальной машине Linux с помощью расширения виртуальной машины.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: linux
ms.date: 02/14/2017
ms.openlocfilehash: bc252e560df782625d795b30c6688a34f5c2bd79
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102563597"
---
# <a name="network-watcher-agent-virtual-machine-extension-for-linux"></a>Расширение виртуальной машины агента Наблюдателя за сетями для Linux

## <a name="overview"></a>Обзор

[Наблюдатель за сетями Azure](../../network-watcher/index.yml) — это служба мониторинга производительности, диагностики и анализа сети, позволяющая наблюдать за сетями Azure. Расширение виртуальной машины для агента службы "Наблюдатель за сетями" необходимо для некоторых функций этой службы для виртуальных машин Azure, например, для записи трафика по требованию и других дополнительных функций.

В этой статье подробно описаны поддерживаемые платформы и параметры развертывания для расширения виртуальной машины для агента службы "Наблюдатель за сетями" для Linux. Установка агента не прерывает работу и не требует перезагрузки виртуальной машины. Расширение можно развернуть на развертываемой виртуальной машине. Если виртуальная машина развернута службой Azure, необходимо проверить документацию службы на наличие разрешения на установку расширения для виртуальной машины.

## <a name="prerequisites"></a>Предварительные требования

### <a name="operating-system"></a>Операционная система

Расширение виртуальной машины для агента службы "Наблюдатель за сетями" можно настроить для следующих дистрибутивов Linux.

| Distribution | Версия |
|---|---|
| Ubuntu | 12+ |
| Debian | 7 и 8 |
| Red Hat | 6 и 7 |
| Oracle Linux | 6.8 и выше и 7 |
| SUSE Linux Enterprise Server | 11 и 12 |
| OpenSUSE Leap | 42.3 или выше |
| CentOS | 6.5 и выше и 7 |
| CoreOS | 899.17.0 и выше |


### <a name="internet-connectivity"></a>Подключение к Интернету

Для некоторых функциональных возможностей агента службы "Наблюдатель за сетями" требуется, чтобы виртуальная машина была подключена к Интернету. Без возможности установить исходящие подключения некоторые возможности агента службы "Наблюдатель за сетями" могут работать неправильно или стать недоступными. Дополнительные сведения о функциях службы "Наблюдатель за сетями", для которых нужен агент, приведены в [документации по службе "Наблюдатель за сетями"](../../network-watcher/index.yml).

## <a name="extension-schema"></a>Схема расширения

В следующем коде JSON показана схема для расширения агента Наблюдателя за сетями. Расширение не требует и не поддерживает какие-либо параметры, предоставляемые пользователем. Оно использует собственную конфигурацию по умолчанию.

```json
{
    "type": "extensions",
    "name": "Microsoft.Azure.NetworkWatcher",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Azure.NetworkWatcher",
        "type": "NetworkWatcherAgentLinux",
        "typeHandlerVersion": "1.4",
        "autoUpgradeMinorVersion": true
    }
}
```

### <a name="property-values"></a>Значения свойств

| Имя | Значение и пример |
| ---- | ---- |
| версия_API | 2015-06-15 |
| publisher | Microsoft.Azure.NetworkWatcher |
| тип | NetworkWatcherAgentLinux |
| typeHandlerVersion | 1.4 |

## <a name="template-deployment"></a>Развертывание шаблона

Вы можете развернуть расширения виртуальной машины Azure с помощью шаблона Azure Resource Manager. Чтобы развернуть расширение агента службы "Наблюдатель за сетями", используйте в шаблоне предыдущую схему JSON.

## <a name="azure-classic-cli-deployment"></a>Развертывание классического Azure CLI

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Следующий пример развертывает расширение виртуальной машины для агента службы "Наблюдатель за сетями" на существующей виртуальной машине, развернутой с помощью классической модели.

```console
azure config mode asm
azure vm extension set myVM1 NetworkWatcherAgentLinux Microsoft.Azure.NetworkWatcher 1.4
```

## <a name="azure-cli-deployment"></a>Развертывание с помощью Azure CLI

Следующий пример развертывает расширение виртуальной машины для агента службы "Наблюдатель за сетями" на существующей виртуальной машине, развернутой с помощью Resource Manager.

```azurecli
az vm extension set --resource-group myResourceGroup1 --vm-name myVM1 --name NetworkWatcherAgentLinux --publisher Microsoft.Azure.NetworkWatcher --version 1.4
```

## <a name="troubleshooting-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshooting"></a>Устранение неполадок

Сведения о состоянии развертываний расширения можно получить на портале Azure или при помощи Azure CLI.

В следующем примере показано состояние развертывания расширения NetworkWatcherAgentLinux для виртуальной машины, развернутой с помощью Resource Manager и Azure CLI.

```azurecli
az vm extension show --name NetworkWatcherAgentLinux --resource-group myResourceGroup1 --vm-name myVM1
```

### <a name="support"></a>Поддержка

Если вам нужна дополнительная помощь в любой момент в этой статье, см. [документацию к наблюдателю за сетями](../../network-watcher/index.yml)или обратитесь к экспертам по Azure на [форумах MSDN Azure и Stack overflow](https://azure.microsoft.com/support/forums/). Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните **Получить поддержку**. Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).
