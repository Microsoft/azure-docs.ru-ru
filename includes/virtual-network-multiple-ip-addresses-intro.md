---
title: Включить файл
description: включить файл
services: virtual-network
author: jimdial
ms.service: virtual-network
ms.topic: include
ms.date: 04/09/2018
ms.author: jdial
ms.custom: include file
ms.openlocfilehash: 68ce927c05f7179cca0aa2833460b9550f0a82d2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96027788"
---
> [!div class="op_single_selector"]
> * [Портал Azure](../articles/virtual-network/virtual-network-multiple-ip-addresses-portal.md)
> * [PowerShell](../articles/virtual-network/virtual-network-multiple-ip-addresses-powershell.md)
> * [Azure CLI](../articles/virtual-network/virtual-network-multiple-ip-addresses-cli.md)
>

К виртуальной машине Azure подключены один или несколько сетевых интерфейсов. Каждому такому адаптеру статически или динамически назначается один или несколько общедоступных или частных IP-адресов. Назначение нескольких IP-адресов виртуальной машине дает следующие возможности:

* Возможность размещать на одном сервере несколько веб-сайтов или служб с разными IP-адресами и SSL-сертификатами.
* возможность использовать виртуальную машину в качестве виртуального сетевого устройства, такого как брандмауэр или балансировщик нагрузки.
* Возможность добавления любых частных IP-адресов любых сетевых карт во внутренний пул Azure Load Balancer. Раньше во внутренний пул можно было добавить только основной IP-адрес для основной сетевой карты. Дополнительные сведения о балансировке нагрузки в конфигурациях с несколькими IP-адресами см. в [этой статье](../articles/load-balancer/load-balancer-multiple-ip.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Каждому сетевому адаптеру, подключенному к виртуальной машине, присвоена одна или несколько конфигураций IP-адресов. Каждая конфигурация получает один статический или динамический частный IP-адрес. Кроме того, каждой конфигурации также может быть присвоен один ресурс общедоступного IP-адреса. Ресурсу общедоступного IP-адреса назначен один динамический или статический общедоступный IP-адрес. Дополнительные сведения об IP-адресах в Azure см. в [этой статье](../articles/virtual-network/public-ip-addresses.md). 

Сетевой карте может быть назначено ограниченное число частных IP-адресов. Число общедоступных IP-адресов, которые можно использовать в одной подписке Azure, также ограничено. См. дополнительные сведения об [ограничениях в Azure](../articles/azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).