---
title: Создание плана "функции Azure" Premium на портале
description: Узнайте, как использовать портал Azure для создания приложения-функции, которое выполняется в плане Premium.
ms.topic: how-to
ms.date: 10/30/2020
ms.openlocfilehash: 9cab67f096665c9333fa40bcb790896fcbebd8d5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98676592"
---
# <a name="create-a-premium-plan-function-app-in-the-azure-portal"></a>Создание приложения-функции плана Premium в портал Azure

Функции Azure предлагают масштабируемый план Premium, обеспечивающий подключение к виртуальной сети, без холодного запуска и оборудования уровня "Премиум". Дополнительные сведения см. в статье [план функций Azure Premium](functions-premium-plan.md). 

Из этой статьи вы узнаете, как использовать портал Azure для создания приложения-функции в плане Premium. 

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com) с помощью своей учетной записи Azure.

## <a name="create-a-function-app"></a>Создание приложения-функции

Для выполнения функций вам понадобится приложение-функция, позволяющее группировать функции в логические единицы и упростить развертывание, масштабирование и совместное использование ресурсов, а также управление ими.

[!INCLUDE [functions-premium-create](../../includes/functions-premium-create.md)]

На этом этапе можно создавать функции в новом приложении функции. Эти функции могут воспользоваться преимуществами [плана Premium](functions-premium-plan.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Clean-up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавить функцию, активируемую HTTP] (./functions-get-started.md
