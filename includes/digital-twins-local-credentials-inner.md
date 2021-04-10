---
author: baanders
description: Включаемый файл для настройки локальной проверки подлинности для DefaultAzureCredential в примерах Azure Digital Twins — без вводных сведений
ms.service: digital-twins
ms.topic: include
ms.date: 10/22/2020
ms.author: baanders
ms.openlocfilehash: fb148551db798207a52bd7aef629da79dd3341e1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102212826"
---
При использовании `DefaultAzureCredential` пример будет искать учетные данные в локальной среде, например имя для входа Azure в локальной версии [Azure CLI](/cli/azure/install-azure-cli), в Visual Studio либо Visual Studio Code. Поэтому вам нужно *войти в Azure локально* с помощью одного из этих механизмов, чтобы настроить учетные данные для примера.

Если вы используете для запуска примера кода Visual Studio или Visual Studio Code, убедитесь, что вошли в этот редактор с теми же учетными данными Azure, которые хотите использовать для доступа к своему экземпляру Azure Digital Twins.

Если это не так, можно [установить локальную версию Azure CLI](/cli/azure/install-azure-cli), запустить на компьютере командную строку и выполнить команду `az login`, чтобы войти в учетную запись Azure. После этого при запуске примера кода вход в систему должен выполняться автоматически.