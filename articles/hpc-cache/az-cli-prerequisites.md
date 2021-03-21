---
title: Azure CLI необходимых компонентов для кэша HPC Azure
description: Шаги настройки перед использованием Azure CLI для создания или изменения кэша Azure HPC
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 07/08/2020
ms.author: v-erkel
ms.openlocfilehash: 13f45c96a830110bd0f4a2d4a2b422921d7a2e31
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94654462"
---
# <a name="set-up-azure-cli-for-azure-hpc-cache"></a>Настройка Azure CLI для Azure HPC Cache

Выполните следующие действия, чтобы подготовить среду перед использованием Azure CLI для создания кэша Azure HPC или управления им.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для кэша HPC Azure требуется версия 2,7 или более поздняя Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="set-default-resource-group-optional"></a>Задать группу ресурсов по умолчанию (необязательно)

Большая часть команд для работы с кэшем HPC требует передачи группы ресурсов кэша. Группу ресурсов по умолчанию можно задать с помощью команды [AZ configure](/cli/azure/reference-index#az-configure).

## <a name="next-steps"></a>Дальнейшие действия

После установки расширения Azure CLI и входа в систему можно использовать Azure CLI для создания систем кэша Azure HPC и управления ими.

* [Создание Azure HPC Cache](hpc-cache-create.md)
* [Azure CLI документация по кэшу HPC](/cli/azure/ext/hpc-cache/hpc-cache)
