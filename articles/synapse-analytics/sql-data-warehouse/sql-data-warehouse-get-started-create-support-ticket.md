---
title: Запросы увеличений квоты и получение поддержки
description: Как создать запрос на поддержку для Azure Synapse Analytics на портале Azure. Запрашивайте увеличение квоты или получайте поддержку в решении проблем.
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/10/2020
author: gaursa
ms.author: gaursa
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: b5b7563b49c6f0eb155223026797f8306e642f5a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104585661"
---
# <a name="request-quota-increases-and-get-support-for-azure-synapse-analytics"></a>Запросы увеличений квоты и получение поддержки для Azure Synapse Analytics

В этой статье описывается, как отправить запрос в службу поддержки портала Azure для Azure Synapse Analytics. Этот процесс позволяет запросить увеличение квоты или отправить запрос на техническую поддержку группе технических специалистов поддержки.

## <a name="create-a-support-ticket"></a>Создание запроса в службу поддержки

Выполните следующие действия, чтобы создать новый запрос на поддержку для Azure Synapse Analytics из портала Azure.

1. В главном меню на [портале Azure](https://portal.azure.com) выберите **Справка и поддержка**.

   ![Ссылка "Справка и поддержка"](./media/sql-data-warehouse-get-started-create-support-ticket/help-plus-support.png)


1. В разделе **Справка и поддержка** выберите **Новый запрос на получение поддержки**.

    ![Создание нового запроса на поддержку](./media/sql-data-warehouse-get-started-create-support-ticket/new-support-request.png)

1. Выберите свой [план поддержки Azure](https://azure.microsoft.com/support/plans/?WT.mc_id=Support_Plan_510979/).

   * Поддержка по **вопросам, связанным с выставлением счетов, квотами и управлением подпиской**, доступна на всех уровнях.
   * Поддержка по **замене или ремонту** обеспечивается на уровнях поддержки [Разработка](https://azure.microsoft.com/support/plans/developer/), [Стандартный](https://azure.microsoft.com/support/plans/standard/), [Professional Direct](https://azure.microsoft.com/support/plans/prodirect/) и [Premier](https://azure.microsoft.com/support/plans/premier/). Проблемы характера «замена или ремонт» — это возникающие при использовании Azure проблемы, относительно которых имеются обоснованные предположения, что их причиной стала корпорация Майкрософт.
   * Услуги **обучения** и **консультирования** разработчиков доступны на уровнях поддержки [Professional Direct](https://azure.microsoft.com/support/plans/prodirect/) и [Premier](https://azure.microsoft.com/support/plans/premier/).

   При наличии плана поддержки Premier можно также создавать отчеты по проблемам с Azure Synapse Analytics на [портале Microsoft Premier](https://premier.microsoft.com/). Дополнительные сведения о доступных планах поддержки, включая области действия, время ответа, цены и т. д., см. на странице [планов поддержки Azure](https://azure.microsoft.com/support/plans/?WT.mc_id=Support_Plan_510979/).  Часто задаваемые вопросы о поддержке Azure см. на [Странице вопросов и ответов поддержки Azure](https://azure.microsoft.com/support/faq/).

1. Выберите соответствующий тип проблемы в разделе **Тип проблемы**. Чтобы устранить неполадки, выберите раздел **Техническая**. Чтобы запросить повышение квоты, выберите **Ограничения службы и подписки (квоты)** .

   ![Выбор типа проблемы](./media/sql-data-warehouse-get-started-create-support-ticket/select-quota-issue-type.png)  

   > [!NOTE]
   > Оставшаяся часть этой статьи посвящена запросам на увеличение квот. Но можно также выбрать **Техническая** для запросов о разрешении проблем в службу поддержки. Если выбрать **Техническая**, вам будет предложено ввести сводку проблемы, а затем указать ее тип в разделе **Выбрать тип проблемы**. Вы можете увидеть решения, которые помогут решить проблему. Если представленные решения не помогли устранить проблему, выберите **Далее: подробные сведения** и заполните форму, чтобы отправить запрос в службу поддержки.

1. Для запросов на увеличение квоты выберите **Azure Synapse Analytics** в разделе **Тип квоты**. Затем щелкните **Next: Решения >>** .

   ![Выберите тип квоты](./media/sql-data-warehouse-get-started-create-support-ticket/select-quota-type.png)

1. В окне **сведения** выберите **Введите сведения** , чтобы ввести дополнительные сведения.

   ![Ссылка "Укажите сведения"](./media/sql-data-warehouse-get-started-create-support-ticket/provide-details-link.png)

## <a name="quota-request-types"></a>Типы запросов квоты

При нажатии кнопки **Ввод сведений** отображается окно **сведения о квоте** , в котором можно добавить дополнительные сведения. В следующих разделах описаны различные запросы квот, доступные для Azure Synapse Analytics.

### <a name="synapse-sql-pool-data-warehouse-units-dwus-per-server"></a>Синапсе единиц хранилища данных пула SQL (Dwu) на сервер

Чтобы запросить увеличение количества DWU на сервер, выполните следующие действия.

1. Выберите тип квоты **синапсе пул SQL DWU для каждого сервера** .

1. Выберите **ресурс** , для которого требуется увеличить квоту, с помощью раскрывающегося списка.

1. Введите новую квоту в раздел **Квота запросов** .

1. Выберите **Сохранить и продолжить**.

   ![Сведения о квотах DWU](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-dwus.png)


### <a name="servers-per-subscription"></a>Серверов на подписку

Чтобы запросить увеличение количества серверов на подписку, необходимо выполнить следующие действия.

1. Выберите **серверы SQL Server на подписку** в качестве типа квоты.

1. В списке **Расположение** выберите целевой регион Azure. Квота для каждой подписки в каждом регионе.

1. В поле **запрос квоты** введите запрос на максимальное число серверов в этом регионе.

   ![Сведения о квотах серверов](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-servers.png)



1. Выберите **Сохранить и продолжить**.

Некоторые типы предложений доступны не в каждом регионе. Вы можете получать следующую ошибку:

![Ошибка доступа к региону](./media/sql-data-warehouse-get-started-create-support-ticket/region-access-error.png)

### <a name="enable-subscription-access-to-a-region"></a>Включение доступа к подписке для региона

Чтобы включить доступ к регионам для подписки, необходимо выполнить следующие действия.  

1. Выберите тип квоты **доступа к региону пула SQL синапсе (хранилище данных)** .

1. Выберите регион, выбрав **Расположение** из раскрывающегося списка.

1. Укажите требования к производительности DWU в разделе **DWU Required (обязательный** ).

1. Введите **Описание бизнес-требований**. 

1. Выберите **Сохранить и продолжить**.

![Доступ к региону](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-region.png)


### <a name="for-other-quota-requests"></a>Для других запросов квот

Выберите **другой запрос квоты** в раскрывающемся меню тип квоты для других типов запросов квоты:

![Другие сведения о квотах](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details.png)

## <a name="submit-your-request"></a>Отправка запроса

Последний шаг — указать остальные сведения в запросе поддержки для базы данных SQL. Затем нажмите кнопку **Далее: Проверка и создание>>**.

![Проверить сведения о создании](./media/sql-data-warehouse-get-started-create-support-ticket/review-create-details.png)

После просмотра сведений о запросе выберите **создать** , чтобы отправить запрос.

![Создание билета](./media/sql-data-warehouse-get-started-create-support-ticket/create-ticket.png)

## <a name="monitor-a-support-ticket"></a>Мониторинг запроса в службу поддержки

После отправки запроса с вами свяжется представитель службы поддержки Azure. Чтобы проверить состояние запроса и сведения, выберите **все запросы на поддержку** на панели мониторинга.

![Проверка состояния](./media/sql-data-warehouse-get-started-create-support-ticket/monitor-ticket.png)

## <a name="other-resources"></a>Другие ресурсы

Вы также можете подключиться к сообществу Azure синапсе Analytics на [Stack overflow](https://stackoverflow.com/questions/tagged/azure-synapse+or+azure-sql-data-warehouse) или на [странице Microsoft Q&A вопрос для Azure синапсе Analytics](/answers/topics/azure-synapse-analytics.html).