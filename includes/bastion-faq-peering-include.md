---
title: включить файл
description: включить файл
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: include
ms.date: 11/05/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 57b0bb9ab8ceb34021a38db0d0abf2c9c919e808
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94356658"
---
### <a name="can-i-still-deploy-multiple-bastion-hosts-across-peered-virtual-networks"></a>Можно ли по-прежнему развертывать несколько узлов бастиона между одноранговыми виртуальными сетями?

Да. По умолчанию пользователь видит узел бастиона, развернутый в той же виртуальной сети, в которой находится виртуальная машина. Однако в меню **Подключение** пользователь может видеть несколько узлов бастиона, обнаруженных в одноранговых сетях. Они могут выбрать узел бастиона, который предпочитает использовать для подключения к виртуальной машине, развернутой в виртуальной сети.

### <a name="if-my-peered-vnets-are-deployed-in-different-subscriptions-will-connectivity-via-bastion-work"></a>Если мой пиринг виртуальных сетей развертывается в разных подписках, будет ли подключение через бастиона работать?

Да, подключение через бастиона будет продолжать работать для одноранговых виртуальных сетей в разных подписках для одного клиента. Подписки в двух разных клиентах не поддерживаются. Чтобы увидеть бастиона в раскрывающемся меню **подключить** , пользователь должен выбрать подсети, к которым у них есть доступ, в разделе **Подписка > Глобальная подписка**.

:::image type="content" source="./media/bastion-faq-peering-include/global-subscriptions.png" alt-text="Фильтр глобальных подписок" lightbox="./media/bastion-faq-peering-include/global-subscriptions.png":::

### <a name="i-have-access-to-the-peered-vnet-but-i-cant-see-the-vm-deployed-there"></a>У меня есть доступ к одноранговой виртуальной сети, но виртуальная машина, развернутая там, не отображается.

Убедитесь, что у пользователя есть доступ на **Чтение** как к виртуальной машине, так и к одноранговой виртуальной сети. Кроме того, проверьте наличие IAM, что пользователь имеет доступ для **чтения** к следующим ресурсам:

* роль читателя на виртуальной машине;
* роль читателя на сетевом адаптере с частным IP-адресом для виртуальной машины;
* роль читателя для ресурса "Бастион Azure".
* Роль читателя в виртуальной сети (не требуется, если нет одноранговой виртуальной сети).

|Разрешения|Описание|Тип разрешения|
|---|---| ---|
|Microsoft.Network/bastionHosts/read |Получение узла-бастиона|Действие|
|Microsoft.Network/virtualNetworks/BastionHosts/action |Получение ссылок узла-бастиона в виртуальной сети.|Действие|
|Microsoft. Network/virtualNetworks/Бастионхостс/значение по умолчанию/действие|Получение ссылок узла-бастиона в виртуальной сети.|Действие|
|Microsoft.Network/networkInterfaces/read|Возвращает определение сетевого интерфейса.|Действие|
|Microsoft.Network/networkInterfaces/ipconfigurations/read|Возвращает определение IP-конфигурации сетевого интерфейса.|Действие|
|Microsoft.Network/virtualNetworks/read|Возвращает определение виртуальной сети.|Действие|
|Microsoft.Network/virtualNetworks/subnets/virtualMachines/read|Возвращает ссылки на все виртуальные машины в подсети виртуальной сети.|Действие|
|Microsoft.Network/virtualNetworks/virtualMachines/read|Возвращает ссылки на все виртуальные машины в виртуальной сети.|Действие|