---
title: включить файл
description: включить файл
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 10/06/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 134f1dc7cb6e53c181b2f518055e5cb758fccf31
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91812713"
---
1. В разделе виртуальной глобальной сети выберите "Концентраторы" и щелкните **+ Создать концентратор**.

   :::image type="content" source="media/virtual-wan-p2s-hub/new-hub.jpg" alt-text="Новый концентратор":::

1. На странице "Создание виртуального концентратора" заполните следующие поля.

   * **Регион** — выберите регион, в котором требуется развернуть виртуальный концентратор.
   * **Имя** — введите имя, которое требуется вызвать для виртуального концентратора.
   * **Пространство частных адресов концентратора** — диапазон адресов концентратора в нотации CIDR.

   :::image type="content" source="media/virtual-wan-p2s-hub/create-hub.jpg" alt-text="Создание виртуального концентратора":::

1. На вкладке "Точка — сеть" заполните следующие поля:

   * **Единицы масштабирования шлюза** — совокупная емкость шлюза VPN пользователя.
   * **Конфигурация "точка — сеть"**  — создана на предыдущем шаге.
   * **Пул адресов клиента** — для удаленных пользователей.
   * **IP-адрес пользовательского DNS-сервера**.

   :::image type="content" source="media/virtual-wan-p2s-hub/hub-with-p2s.png" alt-text="Концентратор с подключением точка — сеть":::

1. Выберите **Review + create** (Просмотреть и создать).
1. На странице **Проверка пройдена** щелкните **Создать**.