---
title: Выполнение команд Azure CLI
description: Основные команды Azure CLI для управления виртуальными машинами в режиме Azure Resource Manager
author: RicksterCDN
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 05/12/2017
ms.author: rclaus
ms.openlocfilehash: 2084d79ecbbc53ef9e3c75bae0664eae7de0eccb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102559636"
---
# <a name="common-azure-cli-commands-for-managing-azure-resources"></a>Основные команды Azure CLI для управления ресурсами Azure

Azure CLI позволяет создавать ресурсы Azure в macOS, Linux и Windows и управлять ими. В этой статье описаны некоторые из наиболее часто используемых команд для создания и управления виртуальными машинами.

Для этой статьи требуется Azure CLI версии 2.0.4 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli). Вы также можете использовать [Cloud Shell](../../cloud-shell/quickstart.md) из своего браузера.

## <a name="basic-azure-resource-manager-commands-in-azure-cli"></a>Основные команды Azure Resource Manager в Azure CLI
Чтобы получить подробные сведения о конкретных параметрах командной строки, можно использовать интерактивную справку по командам, к которой можно получить доступ с помощью команды `az <command> <subcommand> --help`.

### <a name="create-vms"></a>Создание виртуальных машин
| Задача | Команды Azure CLI |
| --- | --- |
| Создание группы ресурсов | `az group create --name myResourceGroup --location eastus` |
| Создание виртуальной машины Linux | `az vm create --resource-group myResourceGroup --name myVM --image ubuntults` |
| Создание виртуальной машины Windows | `az vm create --resource-group myResourceGroup --name myVM --image win2016datacenter` |

### <a name="manage-vm-state"></a>Управление состоянием виртуальной машины
| Задача | Команды Azure CLI |
| --- | --- |
| Запуск виртуальной машины | `az vm start --resource-group myResourceGroup --name myVM` |
| Остановка виртуальной машины | `az vm stop --resource-group myResourceGroup --name myVM` |
| Освобождение виртуальной машины | `az vm deallocate --resource-group myResourceGroup --name myVM` |
| Перезапуск виртуальной машины | `az vm restart --resource-group myResourceGroup --name myVM` |
| Повторное развертывание виртуальной машины | `az vm redeploy --resource-group myResourceGroup --name myVM` |
| Удаление виртуальной машины | `az vm delete --resource-group myResourceGroup --name myVM` |

### <a name="get-vm-info"></a>Получение сведений о виртуальной машине
| Задача | Команды Azure CLI |
| --- | --- |
| Вывод списка виртуальных машин | `az vm list` |
| Получение информации о виртуальной машине | `az vm show --resource-group myResourceGroup --name myVM` |
| Получение сведений об использовании ресурсов виртуальной машины | `az vm list-usage --location eastus` |
| Получить все доступные размеры виртуальной машины | `az vm list-sizes --location eastus` |

## <a name="disks-and-images"></a>Диски и образы
| Задача | Команды Azure CLI |
| --- | --- |
| Добавление диска данных в виртуальную машину | `az vm disk attach --resource-group myResourceGroup --vm-name myVM --disk myDataDisk --size-gb 128 --new` |
| Удаление диска данных от виртуальной машины | `az vm disk detach --resource-group myResourceGroup --vm-name myVM --disk myDataDisk` |
| Изменение размера диска | `az disk update --resource-group myResourceGroup --name myDataDisk --size-gb 256` |
| Моментальный снимок диска | `az snapshot create --resource-group myResourceGroup --name mySnapshot --source myDataDisk` |
| Создание образа виртуальной машины | `az image create --resource-group myResourceGroup --source myVM --name myImage` |
| Создание виртуальной машины на основе образа | `az vm create --resource-group myResourceGroup --name myNewVM --image myImage` |


## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о примерах команд интерфейса командной строки см. в руководстве по [созданию виртуальных машин Linux и управлению ими с помощью Azure CLI](tutorial-manage-vm.md).
