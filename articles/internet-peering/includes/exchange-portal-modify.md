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
ms.openlocfilehash: e06b5261ca6923e158c818d236a30cf6ebff189b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81680925"
---
В этом разделе описывается выполнение следующих операций изменения для прямого пиринга.

### <a name="add-exchange-peering-connections"></a>Добавление подключений пиринга Exchange

1. Нажмите кнопку **+ добавить подключения** и настройте новое соединение пиринга.
    > [!div class="mx-imgBorder"]
    > ![Представление ресурсов пиринга](../media/setup-exchange-modify-addconnection.png)
1. Заполните форму **Подключение пиринга Exchange** и нажмите кнопку **сохранить**. Чтобы получить справку по настройке подключения пиринга, ознакомьтесь с инструкциями в разделе "Создание и инициализация прямого пиринга".
    > [!div class="mx-imgBorder"]
    > ![Форма подключения пиринга Exchange](../media/setup-exchange-modify-savenewconnection.png)

### <a name="remove-exchange-peering-connections"></a>Удаление подключений пиринга Exchange

1. Выберите подключение пиринга, которое нужно удалить, а затем выберите **...**  >  **Удаление подключения**.
    > [!div class="mx-imgBorder"]
    > ![Кнопка удаления соединения](../media/setup-exchange-modify-deleteconnection.png)
1. Введите идентификатор ресурса в поле **Подтверждение удаления** и выберите **Удалить**.
    > [!div class="mx-imgBorder"]
    > ![Подтверждение удаления](../media/setup-exchange-modify-deleteconnectionconfirm.png)

### <a name="add-an-ipv4-or-ipv6-session-on-active-connections"></a>Добавление сеанса IPv4 или IPv6 для активных соединений

1. Выберите подключение пиринга, которое необходимо изменить, а затем выберите **...**  >  **Изменить соединение**.
    > [!div class="mx-imgBorder"]
    > ![Кнопка изменения соединения](../media/setup-exchange-modify-editconnection.png)
1. Добавьте сведения об **адресе IPv4** или **IPv6** и нажмите кнопку **сохранить**.
    > [!div class="mx-imgBorder"]
    > ![Изменение соединения пиринга](../media/setup-exchange-modify-editconnectionsettings.png)

### <a name="remove-an-ipv4-or-ipv6-session-on-active-connections"></a>Удаление сеанса IPv4 или IPv6 на активных подключениях

Удаление сеанса IPv4 или IPv6 из существующего подключения сейчас не поддерживается на портале. Для получения дополнительных сведений обратитесь в службу [пиринга Майкрософт](mailto:peeringexperience@microsoft.com).