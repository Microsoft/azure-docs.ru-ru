---
title: Руководство. Подключение Azure Analysis Services к Power BI Desktop | Документация Майкрософт
author: minewiskan
description: Из этого руководства вы узнаете, как получить имя сервера Analysis Services на портале Azure и подключиться к этому серверу с помощью Power BI Desktop.
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: owend
ms.reviewer: owend
ms.openlocfilehash: 46805427e6d00e858cf869eef68bf68549f57fb9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92018123"
---
# <a name="tutorial-connect-with-power-bi-desktop"></a>Руководство. Подключение с помощью Power BI Desktop

В этом руководстве с помощью Power BI Desktop вы подключитесь к базе данных модели из примера adventureworks на своем сервере. В предложенных вам задачах моделируются типичные действия пользователя при подключении к модели и создании простого отчета на основе данных модели.

> [!div class="checklist"]
> * Получение имени сервера с помощью портала
> * Подключение с помощью Power BI Desktop
> * Создание простого отчета

## <a name="prerequisites"></a>предварительные требования

- [База данных модели из примера adventureworks,](../analysis-services-create-sample-model.md) размещенная на сервере.
- Разрешения на [*чтение*](../analysis-services-server-admins.md) для базы данных модели из примера adventureworks.
- [Последняя версия Power BI Desktop](https://powerbi.microsoft.com/desktop).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure
В этом руководстве вы входите на портал только для того, чтобы узнать имя сервера. Как правило, пользователи получают имя сервера от администратора сервера.

Войдите на [портал](https://portal.azure.com/).

## <a name="get-server-name"></a>Получение имени сервера
Чтобы подключиться к серверу из Power BI Desktop, вам нужно знать имя сервера. Имя сервера можно получить на портале.

На **портале Azure** выберите сервер, а затем щелкните **Обзор** > **Имя сервера** и скопируйте имя сервера.
   
   ![Получение имени сервера в Azure](./media/analysis-services-tutorial-pbid/aas-copy-server-name.png)

## <a name="connect-in-power-bi-desktop"></a>Подключение в Power BI Desktop

1. В Power BI Desktop выберите **Получить данные** > **Azure** > **База данных Azure Analysis Services**.

   ![Подключение с помощью раздела "Получить данные"](./media/analysis-services-tutorial-pbid/aas-pbid-connect-aasserver.png)

2. Вставьте имя сервера в поле **Сервер**, затем в поле **База данных** введите имя **adventureworks** и щелкните **ОК**.

   ![Ввод имени сервера и имени базы данных модели](./media/analysis-services-tutorial-pbid/aas-pbid-connect-aas-servername.png)

3. При появлении запроса введите свои учетные данные. Используемая учетная запись должна иметь по меньшей мере разрешения на чтение для базы данных модели из примера adventureworks.

    Модель adventureworks откроется в Power BI Desktop с пустым отчетом в представлении "Отчет". Список **Поля** содержит все нескрытые объекты модели. Состояние подключения отображается в правом нижнем углу.

4. В разделе **Визуализации** выберите **Линейчатая диаграмма с группировкой**, затем щелкните **значок форматирования** (валик) и включите параметр **Метки данных**. 

   ![Визуализации](./media/analysis-services-tutorial-pbid/aas-pbid-visualizations-report.png)

5. В таблице **Поля** > **Продажи через Интернет** выберите меры **Internet Sales Total** (Итого продаж через Интернет) и **Margin** (Маржа). В таблице **Product Category** (Категория продуктов) выберите **Product Category Name** (Имя категории продуктов).

   ![Полный отчет](./media/analysis-services-tutorial-pbid/aas-pbid-complete-report.png)

    Уделите несколько минут изучению модели из примера adventureworks, например создавая разные визуализации и срезы по данным и метрикам. Когда вы будете довольны своим отчетом, не забудьте его сохранить.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если этот отчет вам не нужен, не сохраняйте его, или удалите файл, если вы его уже сохранили.

## <a name="next-steps"></a>Дальнейшие действия
Из этого руководства вы узнали, как с помощью Power BI Desktop подключиться к модели данных на сервере и создать по ней простой отчет. Если вы еще не умеете создавать модель данных, см. [руководство по моделированию табличных данных на примере Adventure Works Internet Sales](/analysis-services/tutorial-tabular-1400/as-adventure-works-tutorial) в документации по SQL Server Analysis Services.