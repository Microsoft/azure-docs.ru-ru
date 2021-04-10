---
title: Учебник по созданию виртуальных машин Linux и управлению ими с помощью Azure CLI
description: В этом руководстве описано, как с помощью Azure CLI создать и администрировать виртуальные машины в Azure.
services: virtual-machines
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines
ms.collection: linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/23/2018
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 77213fe83a56a33f4d10658f85465349949be792
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102555607"
---
# <a name="tutorial-create-and-manage-linux-vms-with-the-azure-cli"></a>Руководство по Создание виртуальных машин Linux и управление ими с помощью Azure CLI

Виртуальные машины Azure предоставляют полностью настраиваемую и гибкую вычислительную среду. В этом руководстве рассматриваются основные элементы развертывания виртуальной машины Azure, например выбор ее размера, образа и ее развертывание. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины и подключение к ней
> * Выбор и использование образов виртуальных машин
> * Просмотр и использование определенных размеров виртуальных машин
> * Изменение размера виртуальной машины
> * Просмотр виртуальной машины и оценка ее состояния

При работе с этим руководством используется интерфейс командной строки (CLI) в [Azure Cloud Shell](../../cloud-shell/overview.md), который всегда обновлен до последней версии. Чтобы открыть Cloud Shell, выберите **Попробовать** в верхнем углу любого блока кода.

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.0.30 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). 

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Группу ресурсов следует создавать до виртуальной машины. В этом примере создается группа ресурсов с именем *myResourceGroupVM* в регионе *eastus*. 

```azurecli-interactive
az group create --name myResourceGroupVM --location eastus
```

Группа ресурсов указывается при создании или изменении виртуальной машины, что показывается в этом руководстве.

## <a name="create-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину, выполнив команду [az vm create](/cli/azure/vm). 

При создании виртуальной машины доступно несколько вариантов, таких как образ операционной системы, определение размера диска и учетные данные администратора. В следующем примере создается виртуальная машина *myVM* под управлением Ubuntu Server. На виртуальной машине создается учетная запись пользователя с именем *azureuser*, а также ключи SSH, если их еще нет в расположении для ключей по умолчанию ( *~/.ssh*):

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupVM \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Создание виртуальной машины может занять несколько минут. После создания виртуальной машины Azure CLI выводит информацию о ней. Запишите `publicIpAddress`. Этот адрес может использоваться для доступа к виртуальной машине. 

```output
{
  "fqdns": "",
  "id": "/subscriptions/d5b9d4b7-6fc1-0000-0000-000000000000/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroupVM"
}
```

## <a name="connect-to-vm"></a>Подключение к виртуальной машине

Теперь вы можете подключиться к виртуальной машине с помощью SSH из Azure Cloud Shell или с локального компьютера. Замените IP-адрес в примере адресом `publicIpAddress`, записанным на предыдущем шаге.

```bash
ssh azureuser@52.174.34.95
```

После входа на виртуальную машину можно установить и настроить приложения. По окончании работы закройте сеанс SSH, как обычно:

```bash
exit
```

## <a name="understand-vm-images"></a>Описание образов виртуальных машин

Azure Marketplace содержит множество образов, которые можно использовать для создания виртуальных машин. На предыдущих шагах виртуальная машина создавалась с помощью образа Ubuntu. На этом шаге Azure CLI используется для поиска на сайте Marketplace образа CentOS, который затем используется для развертывания второй виртуальной машины. 

Чтобы просмотреть список наиболее часто используемых образов, используйте команду [az vm image list](/cli/azure/vm/image).

```azurecli-interactive 
az vm image list --output table
```

Она отобразит наиболее популярные образы виртуальных машин в Azure.

```output
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
WindowsServer  MicrosoftWindowsServer  2016-Datacenter     MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest     Win2016Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2012-R2-Datacenter  MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest  Win2012R2Datacenter  latest
WindowsServer  MicrosoftWindowsServer  2008-R2-SP1         MicrosoftWindowsServer:WindowsServer:2008-R2-SP1:latest         Win2008R2SP1         latest
WindowsServer  MicrosoftWindowsServer  2012-Datacenter     MicrosoftWindowsServer:WindowsServer:2012-Datacenter:latest     Win2012Datacenter    latest
UbuntuServer   Canonical               16.04-LTS           Canonical:UbuntuServer:16.04-LTS:latest                         UbuntuLTS            latest
CentOS         OpenLogic               7.3                 OpenLogic:CentOS:7.3:latest                                     CentOS               latest
openSUSE-Leap  SUSE                    42.2                SUSE:openSUSE-Leap:42.2:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7.3                 RedHat:RHEL:7.3:latest                                          RHEL                 latest
SLES           SUSE                    12-SP2              SUSE:SLES:12-SP2:latest                                         SLES                 latest
Debian         credativ                8                   credativ:Debian:8:latest                                        Debian               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
```

Получить полный список можно, добавив аргумент `--all`. Кроме того, список образов можно отфильтровать по издателю или предложению с помощью аргумента `--publisher` или `–-offer` соответственно. В этом примере список образов отфильтрован по предложению *CentOS*. 

```azurecli-interactive 
az vm image list --offer CentOS --all --output table
```

Частичные выходные данные приведены ниже.

```output
Offer             Publisher         Sku   Urn                                     Version
----------------  ----------------  ----  --------------------------------------  -----------
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201501         6.5.201501
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201503         6.5.201503
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201506         6.5.201506
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20150904       6.5.20150904
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20160309       6.5.20160309
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20170207       6.5.20170207
```

Чтобы развернуть виртуальную машину с помощью определенного образа, запишите значение в столбце *Urn*, которое состоит из сведений об издателе, предложении, номера SKU и (необязательно) номера версии для [идентификации](cli-ps-findimage.md#terminology) образа. При указании образа его номер версии можно заменить ключевым словом latest. В этом случае будет выбрана последняя версия дистрибутива. В данном примере добавлен аргумент `--image`, чтобы указать последнюю версию образа CentOS 6.5.  

```azurecli-interactive
az vm create --resource-group myResourceGroupVM --name myVM2 --image OpenLogic:CentOS:6.5:latest --generate-ssh-keys
```

## <a name="understand-vm-sizes"></a>Описание размеров виртуальных машин

Размер виртуальной машины определяет количество выделяемых ей вычислительных ресурсов, таких как ЦП, GPU и память. Размеры виртуальных машин должны соответствовать ожидаемой рабочей нагрузке. При увеличении рабочей нагрузки размер существующей виртуальной машины может быть изменен.

### <a name="vm-sizes"></a>Размеры виртуальных машин

В приведенной ниже таблицы указаны категории размеров и примеры использования.  

| Тип                     | Распространенные размеры           |    Описание       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Универсальные](../sizes-general.md)         |B, Dsv3, Dv3, DSv2, Dv2, Av2, DC| Сбалансированное соотношение ресурсов ЦП и памяти. Идеально подходят для разработки и тестирования малых и средних приложений и решений для обработки данных.  |
| [Оптимизированные для вычислений](../sizes-compute.md)   | Fsv2          | Высокое соотношение ресурсов ЦП и памяти. Подходят для приложений со средним объемом трафика, сетевых устройств и пакетных процессов.        |
| [Оптимизированные для памяти](../sizes-memory.md)    | Esv3, Ev3, M, DSv2, Dv2  | Высокое соотношение ресурсов памяти и числа ядер. Отлично подходят для реляционных баз данных, кэша среднего и большого объема, а также выполняющейся в памяти аналитики.                 |
| [Оптимизированные для хранилища](../sizes-storage.md)      | Lsv2, Ls              | Высокая пропускная способность дисков и количество операций ввода-вывода. Идеальный вариант для работы с большими данными, а также с базами данных SQL и NoSQL.                                                         |
| [GPU](../sizes-gpu.md)          | NV, NVv2, NC, NCv2, NCv3, ND            | Специализированные виртуальные машины, предназначенные для ресурсоемкой отрисовки изображений и редактирования видео.       |
| [Высокопроизводительные](../sizes-hpc.md) | H        | Виртуальные машины с самыми мощными ЦП, для которых можно настроить сетевые интерфейсы с высокой пропускной способностью (RDMA). |


### <a name="find-available-vm-sizes"></a>Поиск всех доступных размеров виртуальных машин

Чтобы просмотреть список доступных размеров виртуальных машин в определенном регионе, используйте команду [az vm list-sizes](/cli/azure/vm). 

```azurecli-interactive 
az vm list-sizes --location eastus --output table
```

Частичные выходные данные приведены ниже.

```output
  MaxDataDiskCount    MemoryInMb  Name                      NumberOfCores    OsDiskSizeInMb    ResourceDiskSizeInMb
------------------  ------------  ----------------------  ---------------  ----------------  ----------------------
                 2          3584  Standard_DS1                          1           1047552                    7168
                 4          7168  Standard_DS2                          2           1047552                   14336
                 8         14336  Standard_DS3                          4           1047552                   28672
                16         28672  Standard_DS4                          8           1047552                   57344
                 4         14336  Standard_DS11                         2           1047552                   28672
                 8         28672  Standard_DS12                         4           1047552                   57344
                16         57344  Standard_DS13                         8           1047552                  114688
                32        114688  Standard_DS14                        16           1047552                  229376
                 1           768  Standard_A0                           1           1047552                   20480
                 2          1792  Standard_A1                           1           1047552                   71680
                 4          3584  Standard_A2                           2           1047552                  138240
                 8          7168  Standard_A3                           4           1047552                  291840
                 4         14336  Standard_A5                           2           1047552                  138240
                16         14336  Standard_A4                           8           1047552                  619520
                 8         28672  Standard_A6                           4           1047552                  291840
                16         57344  Standard_A7                           8           1047552                  619520
```

### <a name="create-vm-with-specific-size"></a>Создание виртуальной машины с определенным размером

В предыдущем примере создания виртуальной машины размер не был указан, что привело к использованию размера по умолчанию. Размер виртуальной машины можно выбрать во время ее создания с помощью команды [az vm create](/cli/azure/vm) и аргумента `--size`. 

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupVM \
    --name myVM3 \
    --image UbuntuLTS \
    --size Standard_F4s \
    --generate-ssh-keys
```

### <a name="resize-a-vm"></a>Изменение размера виртуальной машины

После развертывания виртуальной машины ее размер можно изменить, чтобы увеличить или уменьшить выделенные ей ресурсы. Текущий размер виртуальной машины можно просмотреть с помощью команды [az vm show](/cli/azure/vm):

```azurecli-interactive
az vm show --resource-group myResourceGroupVM --name myVM --query hardwareProfile.vmSize
```

Перед изменением размера виртуальной машины проверьте, доступен ли желаемый размер в текущем кластере Azure. Команда [az vm list-vm-resize-options](/cli/azure/vm) отображает список всех размеров. 

```azurecli-interactive 
az vm list-vm-resize-options --resource-group myResourceGroupVM --name myVM --query [].name
```

Если желаемый размер доступен, то размер виртуальной машины можно изменить во включенном состоянии, однако виртуальную машину нужно будет перезагрузить. Используйте команду [az vm resize]( /cli/azure/vm) для изменения размера.

```azurecli-interactive 
az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_DS4_v2
```

Если желаемый размер в текущем кластере недоступен, то перед изменением размера виртуальную машину нужно освободить. Используйте команду [az vm deallocate]( /cli/azure/vm), чтобы остановить и освободить виртуальную машину. Обратите внимание на то, что после повторного включения виртуальной машины все данные на временном диске могут быть удалены. Кроме того, изменится общедоступный IP-адрес, если только не используется статический IP-адрес. 

```azurecli-interactive
az vm deallocate --resource-group myResourceGroupVM --name myVM
```

После освобождения виртуальной машины ее размер можно изменить. 

```azurecli-interactive
az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_GS1
```

После изменения размера можно запустить будет виртуальную машину.

```azurecli-interactive
az vm start --resource-group myResourceGroupVM --name myVM
```

## <a name="vm-power-states"></a>Состояния включенной виртуальной машины

Включенная виртуальная машина Azure может находиться в одном из многих состояний. Это состояние отражает текущее состояние виртуальной машины с точки зрения гипервизора. 

### <a name="power-states"></a>Состояния включения

| Состояние включения | Описание
|----|----|
| Запуск | Указывает, что виртуальная машина запущена. |
| Запущен | Указывает, что виртуальная машина работает. |
| Остановка | Указывает, что виртуальная машина останавливается. | 
| Остановлена | Указывает, что виртуальная машина остановлена. За виртуальные машины в остановленном состоянии по-прежнему взимается плата за вычислительные операции.  |
| Отмена выделения | Указывает, что виртуальная машина освобождается. |
| Освобождено | Указывает, что виртуальная машина удалена из гипервизора, но по-прежнему доступна в плоскости управления. За виртуальные машины в освобожденном состоянии не взимается плата за вычислительные операции. |
| - | Указывает, что состояние включенной виртуальной машины неизвестно. |

### <a name="find-the-power-state"></a>Поиск состояния электропитания

Чтобы получить сведения о состоянии конкретной виртуальной машины, используйте команду [az vm get-instance-view](/cli/azure/vm). Необходимо указать допустимое имя виртуальной машины и группы ресурсов. 

```azurecli-interactive
az vm get-instance-view \
    --name myVM \
    --resource-group myResourceGroupVM \
    --query instanceView.statuses[1] --output table
```

Выходные данные:

```output
ode                DisplayStatus    Level
------------------  ---------------  -------
PowerState/running  VM running       Info
```

Чтобы узнать рабочее состояние всех виртуальных машин в подписке, используйте [API "Виртуальные машины — показать все"](/rest/api/compute/virtualmachines/listall) с параметром **statusOnly**, которому задано значение *true*.

## <a name="management-tasks"></a>Задачи управления

В течение жизненного цикла виртуальной машины можно выполнять задачи управления, такие как запуск, остановка или удаление виртуальной машины. Кроме того, можно создавать скрипты для автоматизации повторяющихся или сложных задач. С помощью Azure CLI в командной строке или в скриптах можно выполнять множество распространенных задач управления. 

### <a name="get-ip-address"></a>Получение IP-адреса

Эта команда возвращает частный и общедоступный IP-адрес виртуальной машины.  

```azurecli-interactive
az vm list-ip-addresses --resource-group myResourceGroupVM --name myVM --output table
```

### <a name="stop-virtual-machine"></a>Прекращение работы виртуальной машины

```azurecli-interactive
az vm stop --resource-group myResourceGroupVM --name myVM
```

### <a name="start-virtual-machine"></a>Запуск виртуальной машины

```azurecli-interactive
az vm start --resource-group myResourceGroupVM --name myVM
```

### <a name="delete-resource-group"></a>Удалить группу ресурсов

При удалении группы ресурсов будут также удалены все ресурсы, содержащиеся в ней: виртуальная машина, виртуальная сеть и диск. При использовании параметра `--no-wait` управление возвращается в командную строку без ожидания завершения операции. Параметр `--yes` подтверждает, что вы хотите удалить ресурсы без дополнительного запроса.

```azurecli-interactive
az group delete --name myResourceGroupVM --no-wait --yes
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы изучили основы создания виртуальной машины и управления ею. Вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины и подключение к ней
> * Выбор и использование образов виртуальных машин
> * Просмотр и использование определенных размеров виртуальных машин
> * Изменение размера виртуальной машины
> * Просмотр виртуальной машины и оценка ее состояния

Перейдите к следующему руководству, чтобы узнать о дисках виртуальных машин.  

> [!div class="nextstepaction"]
> [Управление дисками Azure с помощью Azure CLI](./tutorial-manage-disks.md)