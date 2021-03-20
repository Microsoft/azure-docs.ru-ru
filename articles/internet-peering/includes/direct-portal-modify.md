---
title: включить файл
titleSuffix: Azure
description: включить файл
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: c3212acd5015edbb639b8904b885c718d654e5b4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96356267"
---
В этом разделе описывается выполнение следующих операций изменения для прямого пиринга.

### <a name="add-direct-peering-connections"></a>Добавление соединений с прямыми пиринга
1. Нажмите кнопку **+ добавить подключения** и настройте новое соединение пиринга.
    > [!div class="mx-imgBorder"]
    > ![На странице Ашбурнпиринг-Connections перечисляются подключения. Кнопка + Добавить соединения выделяется.](../media/setup-direct-modify-addconnection.png)

1. Заполните форму **подключения прямого пиринга** и нажмите кнопку **сохранить**. Чтобы получить справку по настройке подключения пиринга, ознакомьтесь с инструкциями в разделе "Создание и инициализация прямого пиринга".
    > [!div class="mx-imgBorder"]
    > ![Форма подключения Direct пиринга](../media/setup-direct-modify-savenewconnection.png)

### <a name="remove-direct-peering-connections"></a>Удаление подключений прямого пиринга

Удаление подключения в настоящее время не поддерживается в портал Azure. Для получения дополнительных сведений обратитесь в службу [пиринга Майкрософт](mailto:peeringexperience@microsoft.com).

### <a name="upgrade-or-downgrade-bandwidth-on-active-connections"></a>Обновление или понижение пропускной способности для активных соединений
1. Выберите подключение пиринга, которое необходимо изменить, а затем выберите **...**  >  **Изменить соединение**.
    > [!div class="mx-imgBorder"]
    > ![Изменение подключения](../media/setup-direct-modify-editconnection.png)

1. Измените пропускную способность, переместив ползунок, а затем нажмите кнопку **сохранить**.
    > [!div class="mx-imgBorder"]
    > ![Изменить пропускную способность](../media/setup-direct-modify-editconnectionsettings.png)

### <a name="add-ipv4-or-ipv6-session-information-on-active-connections"></a>Добавление сведений о сеансе IPv4 или IPv6 в активных соединениях
1. Выберите подключение пиринга, которое необходимо изменить, а затем выберите **...**  >  **Измените подключение** , как показано на шаге 1.
1. Введите **префикс IPv4 для сеанса** или сведения о **префиксе сеанса IPv6** и нажмите кнопку **сохранить**.

### <a name="remove-ipv4-or-ipv6-session-information-on-active-connections"></a>Удаление сведений о сеансе IPv4 или IPv6 на активных подключениях
Удаление **префикса IPv4** или префикса **сеанса IPv6** в настоящее время не поддерживается на портале. Для получения дополнительных сведений обратитесь в службу [пиринга Майкрософт](mailto:peeringexperience@microsoft.com).
