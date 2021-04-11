---
title: Учебник по очистке ресурсов | Azure
description: Из этого учебника вы узнаете, как очистить ресурсы Azure, выделенные при создании веб-приложения.
services: storage, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 04/02/2021
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: cfb42b82943f03c3633ff69662ab841bf587ede4
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106221863"
---
# <a name="tutorial-clean-up-resources"></a>Руководство по Очистка ресурсов

Если вы выполнили все шаги из этого учебника, состоящего из нескольких частей, то уже создали службу приложений, план размещения службы приложений и учетную запись хранения в группе ресурсов. Вы также создали регистрацию приложения в Azure Active Directory. Если ресурсы и регистрация приложения больше не нужны, удалите их, чтобы за них не взималась плата.

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
>
> * удалять ресурсы Azure, созданные при работе с этим учебником.

## <a name="delete-the-resource-group"></a>удаление группы ресурсов.

В меню [портала Azure](https://portal.azure.com) щелкните **Группы ресурсов** и выберите группу ресурсов, содержащую службу приложений и план службы приложений.

Щелкните **Удалить группу ресурсов**. Одновременно с группой ресурсов удаляются все содержащиеся в ней ресурсы.

:::image type="content" alt-text="Снимок экрана: удаление группы ресурсов." source="./media/scenario-secure-app-clean-up-resources/delete-resource-group.png":::

Выполнение этой команды может занять несколько минут.

## <a name="delete-the-app-registration"></a>Удаление регистрации приложения

В меню портала выберите **Azure Active Directory** > **Регистрация приложений**. Затем выберите созданное вами приложение.
:::image type="content" alt-text="Снимок экрана: выбор регистрации приложения." source="./media/scenario-secure-app-clean-up-resources/select-app-registration.png":::

В разделе общих сведений регистрации приложения выберите **Удалить**.
:::image type="content" alt-text="Снимок экрана: удаление регистрации приложения." source="./media/scenario-secure-app-clean-up-resources/delete-app-registration.png":::

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как:

> [!div class="checklist"]
>
> * удалять ресурсы Azure, созданные при работе с этим учебником.

Узнайте, как подключить к базе данных приложения [.NET Core](tutorial-dotnetcore-sqldb-app.md), [Python](tutorial-python-postgresql-app.md), [Java](tutorial-java-spring-cosmosdb.md) или [Node.js](tutorial-nodejs-mongodb-app.md).