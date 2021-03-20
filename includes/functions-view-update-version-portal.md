---
title: включить файл
description: включить файл
services: functions
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 11/26/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: aa9a715fdafc143a116458691965087b016dec1f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83343360"
---
Выполните приведенные ниже инструкции, чтобы узнать, какая версия среды выполнения сейчас используется в приложении-функции, и обновить ее при необходимости.

1. На [портале Azure](https://portal.azure.com) перейдите к приложению-функции.

1. В разделе **Параметры** выберите **Конфигурация**. На вкладке **Параметры среды выполнения функции** выберите **версию среды выполнения**. Запишите конкретную версию среды выполнения. В приведенном ниже примере задана версия `~3`.

    :::image type="content" source="./media/functions-view-update-version-portal/functions-view-runtime-version.png" alt-text="Просмотр версии среды выполнения." border="true":::

1. Чтобы закрепить приложение-функцию на версии среды выполнения 1.x, в пункте **версии среды выполнения** выберите **~1**. Этот параметр неактивен, если в приложении уже есть функции.

1. После изменения версии среды выполнения вернитесь на вкладку **Обзор** и щелкните **Перезапустить**, чтобы перезапустить приложение.  Приложение-функция перезапускается в среде выполнения версии 1.x, и при создании функций используются шаблоны версии 1.x.

    :::image type="content" source="./media/functions-view-update-version-portal/functions-restart-function-app.png" alt-text="Перезапустите приложение функции." border="true":::
