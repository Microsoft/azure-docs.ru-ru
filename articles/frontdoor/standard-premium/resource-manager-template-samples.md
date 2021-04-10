---
title: Примеры шаблонов Resource Manager для Azure Front Door
description: Сведения о примерах шаблонов Azure Resource Manager для Azure Front Door.
services: frontdoor
author: johndowns
ms.author: jodowns
ms.service: frontdoor
ms.topic: sample
ms.date: 03/24/2021
ms.openlocfilehash: 929adb0be948339af033d85b0dabd7e1cedf353e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105561752"
---
# <a name="azure-resource-manager-templates-for-azure-front-door"></a>Шаблоны Azure Resource Manager для Azure Front Door

> [!Note]
> Эта документация предназначена для Azure Front Door категории "Стандартный" или "Премиум" (предварительная версия). Сведения об Azure Front Door см. [здесь](../front-door-overview.md).

В следующей таблице приведены ссылки на шаблоны Azure Resource Manager для Azure Front Door с эталонными архитектурами, а также другие службы Azure.

| Образец | Описание |
|-|-|
| [Набор правил](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-standard-premium-rule-set/) | Создает профиль и набор правил Front Door.  |
|**Источники Службы приложений**| **Описание** |
| [Служба приложений](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-standard-premium-app-service-public) | Создает приложение Службы приложений с общедоступной конечной точкой и профилем Front Door.  |
| [Служба приложений с Приватным каналом](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-premium-app-service-private-link) | Создает приложение Службы приложений с частной конечной точкой и профилем Front Door.  |
| [Среда службы приложений с Приватным каналом](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-premium-app-service-environment-internal-private-link) | Создает Среду службы приложений, приложение с частной конечной точкой и профилем Front Door.  |
|**Источники Функций Azure**| **Описание** |
| [Функции Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-standard-premium-function-public/) | Создает приложение Функции Azure с общедоступной конечной точкой и профилем Front Door.  |
| [Функции Azure с Приватным каналом](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-premium-function-private-link) | Создает приложение Функции Azure с частной конечной точкой и профилем Front Door.  |
|**Источники Управления API**| **Описание** |
| [Управление API (внешний)](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-standard-premium-api-management-external) | Создает экземпляр Управления API с интеграцией внешней виртуальной сети и профилем Front Door.  |
|**Источники службы хранилища**| **Описание** |
| [Статический веб-сайт хранилища](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-standard-premium-storage-static-website) | Создает учетную запись хранения Azure и статический веб-сайт с общедоступной конечной точкой и профилем Front Door.  |
| [Большие двоичные объекты хранилища с Приватным каналом](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-premium-storage-blobs-private-link) | Создает учетную запись хранения Azure и контейнер больших двоичных объектов с частной конечной точкой и профилем Front Door.  |
|**Источники Шлюза приложений**| **Описание** |
| [Шлюз приложений](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-standard-premium-application-gateway-public) | Создает Шлюз приложений и профиль Front Door. |
|**Источники виртуальной машины**| **Описание** |
| [Виртуальная машина со службой Приватного канала](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-premium-vm-private-link) | Создает виртуальную машину, службу Приватного канала и профиль Front Door. |
| | |
