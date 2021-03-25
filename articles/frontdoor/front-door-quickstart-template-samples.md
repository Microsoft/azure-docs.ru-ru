---
title: Примеры шаблонов Azure Resource Manager для Azure Front Door
description: Ознакомьтесь с примерами шаблонов Resource Manager для Azure Front Door, в том числе шаблонами для создания базовой конфигурации Front Door и настройки ограничения скорости для Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2020
ms.author: duau
ms.openlocfilehash: bac1df020bf2a683fc04a4d05ae73311e149f70c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97511779"
---
# <a name="azure-resource-manager-deployment-model-templates-for-front-door"></a>Шаблоны модели развертывания Azure Resource Manager для Front Door

В приведенной ниже таблице представлены ссылки на шаблоны модели развертывания с помощью Azure Resource Manager для службы Azure Front Door. 

| Шаблон | Описание |
| ---| ---|
| [Создание базовой конфигурации Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-create-basic)| Создает базовую конфигурацию Front Door с одной серверной системой. |
| [Создание Front Door с несколькими серверными системами, серверных пулами и маршрутизацией на основе URL-адресов](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-create-multiple-backends)| Создает Front Door с балансировкой нагрузки, настроенный для нескольких серверных систем в одном или нескольких серверных пулах на основе URL-пути. |
| [Подключение личного домена с помощью Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-custom-domain)| Добавьте личный домен в службу Front Door. |
| [Создание Front Door с геофильтрацией](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-geo-filtering)| Создайте конфигурацию Front Door, которая разрешает и блокирует трафик из определенных стран и регионов. |
| [Управление пробами работоспособности для серверных систем в конфигурации Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-health-probes)| Обновите Front Door, чтобы изменить параметры проб работоспособности, обновив путь к пробам, а также периодичность отправки проб. |
| [Создание Front Door с конфигурацией серверной системы "активный/резервный"](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-priority-lb)| Создает Front Door, который демонстрирует маршрутизацию на основе приоритетов для топологии приложений "активный/резервный", т. е. по умолчанию отправляет весь трафик в основную (наиболее приоритетную) серверную систему, пока она не станет недоступной. |
| [Создание Front Door с кэшированием определенных маршрутов](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-create-caching)| Создает Front Door с кэшированием определенной конфигурации маршрутизации, что позволяет кэшировать все статические активы для вашей рабочей нагрузки. |
| [Настройка сходства сеансов для имен узлов Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-session-affinity) | Обновляет Front Door, включая сходство сеанса для интерфейсного узла и, таким образом, отправляя последующий трафик из одного и того же сеанса пользователя в одну и ту же серверную часть. |
| [Настройка Front Door для добавления IP-адресов клиентов в список разрешений или список блокировок](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-clientip)| Настраивает Front Door на ограничение трафика с определенных IP-адресов, используя настраиваемое управление доступом и IP-адреса клиентов. |
| [Настройка Front Door на работу с определенными параметрами HTTP](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-http-params)| Настраивает Front Door на разрешение или блокировку определенного трафика согласно параметрам HTTP во входящих запросах с использованием пользовательских правил для управления доступом на основе параметров HTTP. |
| [Настройка ограничения скорости для Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-rate-limiting)| Настраивает Front Door на ограничение скорости входящего трафика для заданного интерфейсного узла. |
| | |

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [создании Front Door](quickstart-create-front-door.md).
- Дополнительные сведения о том, [как работает Front Door](front-door-routing-architecture.md).
