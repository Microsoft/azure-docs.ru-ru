---
title: Настройка Azure Analysis Services для источников данных виртуальной сети | Документация Майкрософт
description: Узнайте, как настроить сервер Azure Analysis Services для подключения к источникам данным через шлюз в виртуальной сети Azure.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 912dab16cd854ae940b7bbdfe88a8da7adf1c5e0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84197193"
---
# <a name="use-gateway-for-data-sources-on-an-azure-virtual-network-vnet"></a>Использование шлюза для источников данных в виртуальной сети Azure

В этой статье описывается свойство сервера Azure Analysis Services **AlwaysUseGateway**, которое следует использовать при размещении источников данных в [виртуальной сети Azure](../virtual-network/virtual-networks-overview.md).

## <a name="server-access-to-vnet-data-sources"></a>Доступ сервера к источникам данных виртуальной сети

Если доступ к источникам данных осуществляется через виртуальную сеть, сервер служб Azure Analysis Services должен подключаться к этим источникам данных так, как если бы они размещались в вашей локальной среде. Свойство сервера **AlwaysUseGateway** позволяет указать сервер для доступа ко всем источникам данных через [локальный шлюз](analysis-services-gateway.md). 

Источники данных Управляемый экземпляр Azure SQL выполняются в виртуальной сети Azure с частным IP-адресом. Если на экземпляре включена общедоступная конечная точка, дополнительный шлюз не требуется. Если общедоступная конечная точка не используется, нужен локальный шлюз данных, а свойство AlwaysUseGateway должно иметь значение true.

> [!NOTE]
> Это свойство действует только в тех случаях, если установлен и настроен [локальный шлюз данных](analysis-services-gateway.md). Шлюз может находиться в виртуальной сети.

## <a name="configure-alwaysusegateway-property"></a>Настройка свойства AlwaysUseGateway

1. Последовательно выберите SSMS > сервер > **Свойства** > **Общие** > **Показать дополнительные (все) свойства**.
2. Для параметра **ASPaaS\AlwaysUseGateway** выберите **true**.

    ![Свойство AlwaysUseGateway](media/analysis-services-vnet-gateway/aas-ssms-always-property.png)


## <a name="see-also"></a>См. также раздел
[Подключение к локальным источникам данных](analysis-services-gateway.md)   
[Установка и настройка локального шлюза данных](analysis-services-gateway-install.md)   
[Виртуальная сеть Azure](../virtual-network/virtual-networks-overview.md)   

