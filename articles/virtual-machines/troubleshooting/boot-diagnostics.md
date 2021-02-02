---
title: Диагностика загрузки виртуальных машин в Azure | Документация Майкрософт
description: Общие сведения о двух функциях отладки виртуальных машин в Azure.
services: virtual-machines
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: delhan
ms.openlocfilehash: 9030adb9904095ac9b909e650ec6f11dcdf85ed3
ms.sourcegitcommit: 445ecb22233b75a829d0fcf1c9501ada2a4bdfa3
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/02/2021
ms.locfileid: "99475528"
---
# <a name="how-to-use-boot-diagnostics-to-troubleshoot-virtual-machines-in-azure"></a>Как устранять неполадки виртуальных машин в Azure с помощью диагностики загрузки

Существует множество причин, почему виртуальную машину невозможно загрузить. Чтобы устранить проблемы с виртуальными машинами, созданными с помощью модели развертывания диспетчер ресурсов, можно использовать следующие функции отладки: вывод на консоль и снимок экрана для виртуальных машин Azure. 

На портале вы можете просмотреть выходные данные журнала консоли для виртуальных машинах Linux. Azure позволяет просмотреть снимок экрана из гипервизора для виртуальных машин Windows и Linux. Обе эти функции поддерживаются на виртуальных машинах Azure во всех регионах. Обратите внимание, что в учетной записи хранения снимки экрана и вывод могут занимать до 10 минут.

Можно выбрать параметр **Диагностика загрузки**, чтобы просмотреть журнал и снимок экрана.

![Resource Manager](./media/virtual-machines-common-boot-diagnostics/screenshot1.png)

## <a name="common-boot-errors"></a>Распространенные ошибки загрузки

- [0xC000000E](https://support.microsoft.com/help/4010129)
- [0xC000000F](https://support.microsoft.com/help/4010130)
- [0xC0000011](https://support.microsoft.com/help/4010134)
- [0xC0000034](https://support.microsoft.com/help/4010140)
- [0xC0000098](https://support.microsoft.com/help/4010137)
- [0xC00000BA](https://support.microsoft.com/help/4010136)
- [0xC000014C](https://support.microsoft.com/help/4010141)
- [0xC0000221](https://support.microsoft.com/help/4010132)
- [0xC0000225](https://support.microsoft.com/help/4010138)
- [0xC0000359](https://support.microsoft.com/help/4010135)
- [0xC0000605](https://support.microsoft.com/help/4010131)
- [Операционная система не найдена](https://support.microsoft.com/help/4010142)
- [Сбой при загрузке или INACCESSIBLE_BOOT_DEVICE](https://support.microsoft.com/help/4010143)

## <a name="enable-diagnostics-on-a-virtual-machine-created-using-the-azure-portal"></a>Включение диагностики для виртуальных машин, созданных с помощью портала Azure

Следующие процедуры предназначены для виртуальной машины, созданной с помощью модели развертывания Resource Manager.

На вкладке **Управление** в разделе **Наблюдение** удостоверьтесь, что включена **Диагностика загрузки**. В раскрывающемся списке **Учетная запись хранения диагностики** выберите учетную запись хранения, в которой следует разместить файлы диагностики.
 
![Создание виртуальной машины](./media/virtual-machines-common-boot-diagnostics/enable-boot-diagnostics-vm.png)

> [!NOTE]
> Функция диагностики загрузки не поддерживает учетные записи хранения класса Premium или типы учетных записей хранения, избыточных в зонах. В случае использования учетной записи хранения уровня "Премиум" для диагностики загрузки при запуске виртуальной машины может произойти ошибка StorageAccountTypeNotSupported. 
>

### <a name="deploying-from-an-azure-resource-manager-template"></a>Развертывание из шаблона Azure Resource Manager

При развертывании на основе шаблона Azure Resource Manager перейдите к ресурсу виртуальной машины и добавьте раздел профиля диагностики. Установите заголовок версии API как "2015-06-15" или более поздний. Последней является версия 2018-10-01.

```json
{
  "apiVersion": "2018-10-01",
  "type": "Microsoft.Compute/virtualMachines",
  … 
```

Профиль диагностики позволяет выбрать учетную запись хранения, в которую нужно поместить эти журналы.

```json
    "diagnosticsProfile": {
    "bootDiagnostics": {
    "enabled": true,
    "storageUri": "[concat('https://', parameters('newStorageAccountName'), '.blob.core.windows.net')]"
    }
    }
    }
}
```

Дополнительные сведения о развертывании ресурсов с помощью шаблонов см. [в разделе Краткое руководство. Создание и развертывание шаблонов Azure Resource Manager с помощью портал Azure](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

## <a name="enable-boot-diagnostics-on-existing-virtual-machine"></a>Включение диагностики загрузки в существующей виртуальной машине 

Чтобы включить диагностику загрузки в существующей виртуальной машине, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com) и выберите виртуальную машину.
2. В разделе **Поддержка и устранение неполадок** выберите **Диагностика загрузки**, а затем — вкладку **Параметры**.
3. В параметрах **Диагностика загрузки** измените состояние на **Вкл.** и из раскрывающегося списка **Учетная запись хранения** выберите учетную запись. 
4. Сохраните изменения.

    ![Обновление имеющейся виртуальной машины](./media/virtual-machines-common-boot-diagnostics/enable-for-existing-vm.png)

### <a name="enable-boot-diagnostics-using-the-azure-cli"></a>Включение диагностики загрузки с помощью Azure CLI

Чтобы включить диагностику загрузки на существующей виртуальной машине Azure, вы можете воспользоваться Azure CLI. Дополнительные сведения см. в статье [az vm boot-diagnostics](/cli/azure/vm/boot-diagnostics).
