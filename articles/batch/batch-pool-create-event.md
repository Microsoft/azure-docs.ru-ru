---
title: Событие создания пула пакетной службы Azure
description: Ссылка на событие создания пула пакетной службы, которая формируется после создания пула. Содержимое журнала предоставляет общие сведения о пуле.
ms.topic: reference
ms.date: 10/08/2020
ms.openlocfilehash: f8b020401443b2a9e80837599d6fd4b2a3001d27
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97609009"
---
# <a name="pool-create-event"></a>Событие создания пула

 Это событие возникает при создании пула. Содержимое журнала предоставляет общие сведения о пуле. Обратите внимание, что если целевой размер пула больше 0 вычислительных узлов, сразу после этого события начинается событие начала изменения размера пула.

 В следующем примере приведен текст события создания для пула, сформированного с помощью свойства `CloudServiceConfiguration`.

```
{
    "id": "myPool1",
    "displayName": "Production Pool",
    "vmSize": "Standard_F1s",
    "imageType": "VirtualMachineConfiguration",
    "cloudServiceConfiguration": {
        "osFamily": "3",
        "targetOsVersion": "*"
    },
    "networkConfiguration": {
        "subnetId": " "
    },
    "virtualMachineConfiguration": {
          "imageReference": {
            "publisher": " ",
            "offer": " ",
            "sku": " ",
            "version": " "
          },
          "nodeAgentId": " "
        },
    "resizeTimeout": "300000",
    "targetDedicatedNodes": 2,
    "targetLowPriorityNodes": 2,
    "taskSlotsPerNode": 1,
    "vmFillType": "Spread",
    "enableAutoScale": false,
    "enableInterNodeCommunication": false,
    "isAutoPool": false
}
```

|Элемент|Type|Примечания|
|-------------|----------|-----------|
|`id`|Строка|Идентификатор пула.|
|`displayName`|Строка|Отображаемое имя пула.|
|`vmSize`|Строка|Размер виртуальных машин в пуле. Все виртуальные машины в пуле имеют одинаковый размер. <br/><br/> Сведения о доступных размерах виртуальных машин для пулов облачных служб (пулы, созданные с помощью cloudServiceConfiguration), см. в статье [Размеры для облачных служб](../cloud-services/cloud-services-sizes-specs.md). Пакетная служба поддерживает все размеры виртуальных машин для облачных служб, кроме `ExtraSmall`.<br/><br/> Сведения о доступных размерах виртуальных машин для пулов при использовании образов из магазина виртуальных машин (пулы, созданные с помощью virtualMachineConfiguration) см. в статье [Размеры виртуальных машин](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Linux) или [Размеры виртуальных машин](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Windows). Пакетная служба поддерживает все размеры виртуальных машин Azure, кроме `STANDARD_A0`. Для хранилища класса Premium также не поддерживаются размеры таких серий: `STANDARD_GS`, `STANDARD_DS` и `STANDARD_DSV2`.|
|`imageType`|Строка|Метод развертывания для образа. Поддерживаются такие значения: `virtualMachineConfiguration` или `cloudServiceConfiguration`.|
|[`cloudServiceConfiguration`](#bk_csconf)|Сложный тип|Конфигурация облачной службы для пула.|
|[`virtualMachineConfiguration`](#bk_vmconf)|Сложный тип|Конфигурация виртуальной машины для пула.|
|[`networkConfiguration`](#bk_netconf)|Сложный тип|Конфигурация сети для пула.|
|`resizeTimeout`|Time|Время ожидания при выделении вычислительных узлов для пула, указанное для последней операции изменения размера в этом пуле.  (Исходная регулировка размера при создании пула считается изменением размера.)|
|`targetDedicatedNodes`|Int32|Число выделенных вычислительных узлов, запрошенных для пула.|
|`targetLowPriorityNodes`|Int32|Число вычислительных узлов с низким приоритетом, запрошенных для пула.|
|`enableAutoScale`|Bool|Указывает, корректируется ли размер пула автоматически с течением времени.|
|`enableInterNodeCommunication`|Bool|Указывает, настроен ли пул для прямой связи между узлами.|
|`isAutoPool`|Bool|Определяет, создан ли пул с помощью механизма AutoPool задания.|
|`taskSlotsPerNode`|Int32|Максимальное число задач, которые могут быть запущены одновременно на одном вычислительном узле в пуле.|
|`vmFillType`|Строка|Определяет, каким образом пакетная служба распределяет задачи между вычислительными узлами в пуле. Допустимые значения: "Spread" (Распределение) или "Pack" (Упаковка).|

###  <a name="cloudserviceconfiguration"></a><a name="bk_csconf"></a> cloudServiceConfiguration

|Имя элемента|Тип|Примечания|
|------------------|----------|-----------|
|`osFamily`|Строка|Семейство гостевой ОС Azure для установки на виртуальных машинах в пуле.<br /><br /> Возможны следующие значения:<br /><br /> **2** — семейство ОС 2, эквивалентное Windows Server 2008 R2 с пакетом обновления 1 (SP1).<br /><br /> **3** — семейство ОС 3, эквивалентное Windows Server 2012.<br /><br /> **4** — семейство ОС 4, эквивалентное Windows Server 2012 R2.<br /><br /> Дополнительные сведения см. в статье [Выпуски гостевой ОС Azure](../cloud-services/cloud-services-guestos-update-matrix.md#releases).|
|`targetOSVersion`|Строка|Версия гостевой ОС Azure для установки на виртуальных машинах в пуле.<br /><br /> Значение по умолчанию — **\*** , что означает последнюю версию операционной системы для заданного семейства.<br /><br /> Другие допустимые значения см. в статье [Выпуски гостевой ОС Azure](../cloud-services/cloud-services-guestos-update-matrix.md#releases).|

###  <a name="virtualmachineconfiguration"></a><a name="bk_vmconf"></a> virtualMachineConfiguration

|Имя элемента|Тип|Примечания|
|------------------|----------|-----------|
|[`imageReference`](#bk_imgref)|Сложный тип|Указывает информацию об используемой платформе или образе Marketplace.|
|`nodeAgentId`|Строка|Номер SKU для агента узла пакетной службы, подготовленного на вычислительном узле.|
|[`windowsConfiguration`](#bk_winconf)|Сложный тип|Указывает параметры операционной системы Windows на виртуальной машине. Это свойство не нужно задавать, если imageReference ссылается на образ операционной системы Linux.|

###  <a name="imagereference"></a><a name="bk_imgref"></a> imageReference

|Имя элемента|Тип|Примечания|
|------------------|----------|-----------|
|`publisher`|Строка|Издатель образа.|
|`offer`|Строка|Предложение образа.|
|`sku`|Строка|Номер SKU образа.|
|`version`|Строка|Версия образа.|

###  <a name="windowsconfiguration"></a><a name="bk_winconf"></a> windowsConfiguration

|Имя элемента|Тип|Примечания|
|------------------|----------|-----------|
|`enableAutomaticUpdates`|Логическое|Указывает, включено ли для виртуальной машины автоматическое обновление. Если это свойство не задано, используется значение по умолчанию true.|

###  <a name="networkconfiguration"></a><a name="bk_netconf"></a> networkConfiguration

|Имя элемента|Тип|Примечания|
|------------------|--------------|----------|
|`subnetId`|Строка|Указывает идентификатор ресурса для подсети, в которой создаются вычислительные узлы пула.|