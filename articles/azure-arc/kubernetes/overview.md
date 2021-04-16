---
title: Общие сведения о Kubernetes с поддержкой Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 03/03/2021
ms.topic: overview
author: mlearned
ms.author: mlearned
description: В этой статье представлен обзор Kubernetes с поддержкой Azure Arc.
keywords: Kubernetes, Arc, Azure, контейнеры
ms.custom: references_regions
ms.openlocfilehash: 69e9886f214d0076c8e66231fd6ad15bb060828f
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106449654"
---
# <a name="what-is-azure-arc-enabled-kubernetes"></a>Что представляет собой Azure Arc с поддержкой Kubernetes?

С помощью службы Kubernetes с поддержкой Azure Arc вы можете подключать и настраивать кластеры Kubernetes в среде Azure и за ее пределами. При подключении кластера Kubernetes К Azure Arc он:
* отображается на портале Azure с идентификатором Azure Resource Manager и управляемым удостоверение; 
* помещается в подписку и группу ресурсов Azure;
* получает теги, как и любой ресурс Azure. 

Чтобы подключить кластер Kubernetes к Azure, администратор кластера должен развернуть агенты. Эти агенты:
* выполняются в пространстве имен Kubernetes `azure-arc` в качестве стандартных развертываний Kubernetes;
* обслуживают подключения к Azure;
* собирают журналы и метрики Azure Arc;
* отслеживают запросы конфигураций. 

Kubernetes с поддержкой Azure Arc совместим со стандартным протоколом SSL, защищающим данные при передаче. Кроме того, для обеспечения конфиденциальности информация в базах данных Azure Cosmos DB хранится в зашифрованном виде.

## <a name="supported-kubernetes-distributions"></a>Поддерживаемые дистрибутивы Kubernetes

Kubernetes с поддержкой Azure Arc работает со всеми кластерами Kubernetes, сертифицированными для Cloud Native Computing Foundation. Команда Azure Arc вместе с [основными отраслевыми партнерами выполнила проверки соответствия](./validation-program.md) для своих дистрибутивов Kubernetes со службой Kubernetes с поддержкой Azure Arc.

## <a name="supported-scenarios"></a>Поддерживаемые сценарии 

Kubernetes с поддержкой Azure Arc используется в следующих сценариях: 

* Подключение кластера Kubernetes, работающего за пределами среды Azure, для учета ресурсов, их группировки и присвоения тегов.

* Развертывание приложений и применение конфигурации с помощью управления конфигурацией на основе GitOps. 

* Просмотр и мониторинг кластеров с помощью Azure Monitor для контейнеров.

* Принудительная защита от угроз с помощью Azure Defender для Kubernetes.

* Применение политик с помощью Политики Azure для Kubernetes.

[!INCLUDE [azure-lighthouse-supported-service](../../../includes/azure-lighthouse-supported-service.md)]

## <a name="supported-regions"></a>Поддерживаемые регионы 

Kubernetes с поддержкой Azure Arc в настоящее время доступен в следующих регионах: 

* Восточная часть США
* Западная Европа
* центрально-западная часть США
* Центрально-южная часть США
* Юго-Восточная Азия
* южная часть Соединенного Королевства
* Западная часть США 2
* Восточная Австралия
* восточная часть США 2
* Северная Европа

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как подключить кластер к Azure Arc.
> [!div class="nextstepaction"]
> [Подключение кластера к Azure Arc](./quickstart-connect-cluster.md)
