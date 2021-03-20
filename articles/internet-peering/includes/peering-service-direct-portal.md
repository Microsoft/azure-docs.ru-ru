---
title: включить файл
titleSuffix: Azure
description: включить файл
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: include
ms.date: 3/18/2020
ms.author: derekol
ms.openlocfilehash: e5804aa1b005e670d8b430b1c0a3bd62efd0bb06
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81687041"
---
1. Выберите подключение пиринга, которое вы хотите включить для службы пиринга Azure. Затем выберите **...**  >  **Изменить соединение**.
    > [!div class="mx-imgBorder"]
    > ![Соединение для изменения соединения с пирингом](../media/setup-direct-modify-editconnection.png)
1. В разделе **использовать для службы пиринга** выберите **включено** и нажмите кнопку **сохранить**.
    > [!div class="mx-imgBorder"]
    > ![Подключение пиринга включение службы пиринга](../media/setup-direct-modify-editconnectionsettings-peering-service.png)
1. На экране **обзора** отображаются сведения о развертывании. После завершения развертывания выберите **Переход к ресурсу**.
    > [!div class="mx-imgBorder"]
    > ![Развертывание завершено](../media/setup-direct-modify-overview-deployment-complete.png)

1. На панели " **зарегистрированные** префиксы" выберите **Добавить зарегистрированный префикс**.
    > [!div class="mx-imgBorder"]
    > ![Добавить зарегистрированный префикс](../media/setup-direct-modify-add-registered-prefix.png)
1. Зарегистрируйте префикс, выбрав **имя** и **префикс** и нажав кнопку **сохранить**.
    > [!div class="mx-imgBorder"]
    >  ![Регистрация префикса](../media/setup-direct-modify-register-a-prefix.png) 

1. После создания префикса его можно увидеть в списке **зарегистрированных префиксов**. Чтобы просмотреть дополнительные сведения, выберите **имя** префикса.
    > [!div class="mx-imgBorder"]
    > ![Зарегистрированные префиксы и подключения](../media/setup-direct-modify-registered-prefixes.png)
1. На странице зарегистрированный префикс отображаются полные сведения, включая **ключ префикса** для каждого префикса. Этот ключ должен быть предоставлен клиенту, выделенному этому префиксу от поставщика услуг Интернета. Затем клиент может зарегистрировать свой префикс в подписке с помощью этого ключа.
    > [!div class="mx-imgBorder"]
    > ![Префикс с ключом префикса](../media/setup-direct-modify-registered-prefix-detail.png)