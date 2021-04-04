---
title: Учебник по добавлению образца модели (Azure Analysis Services) | Документация Майкрософт
description: Из этого руководства вы узнаете, как добавить образец модели в Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: f882a40940a5c7202e9cf1f5c8b8927f008f4a39
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92013616"
---
# <a name="tutorial-add-a-sample-model-from-the-portal"></a>Руководство. Добавление образца модели с портала

Из этого руководства вы научитесь добавлять на свой сервер образец табличного шаблона базы данных Adventure Works. Образец модели представляет собой готовую версию образца модели данных "Интернет-продажи Adventure Works" (1200). Образец модели полезен для тестирования управления моделью, соединения со средствами и клиентскими приложениями, а также для выполнения запросов к данным модели. В рамках этого руководства используется [портал Azure](https://portal.azure.com) и [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) для выполнения следующих действий. 

> [!div class="checklist"]
> * Добавление готового образца табличной модели данных на сервер 
> * Подключение к модели с помощью SSMS

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником необходимы указанные ниже компоненты.

- Сервер Azure Analysis Services. Для получения дополнительных сведений см. статью [Создание сервера с помощью портала](analysis-services-create-server.md).
- Разрешения администратора сервера
- [Среда SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)


## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал](https://portal.azure.com/).

## <a name="add-a-sample-model"></a>Добавление образца модели

1. В области **Обзор** сервера выберите **Новая модель**.

    ![Создание образца модели](./media/analysis-services-create-sample-model/aas-create-sample-new-model.png)

2. В разделе **Новая модель** > **Выбор источника данных** установите флажок **Пример данных**, а затем нажмите кнопку **Добавить**.

    ![Выбор элемента "Новая модель"](./media/analysis-services-create-sample-model/aas-create-sample-data.png)

3. В области **Обзор** проверьте, добавлен ли образец модели `adventureworks`.

    ![Выбор примеров данных](./media/analysis-services-create-sample-model/aas-create-sample-verify.png)


## <a name="clean-up-resources"></a>Очистка ресурсов

Образец модели использует ресурсы кэш-памяти. Если образец модели не используется для тестирования, следует удалить его с сервера.

В следующих шагах описано, как удалить модель с сервера с помощью среды SSMS.

1. В среде SSMS выберите **Обозреватель объектов** и щелкните **Подключиться** > **Analysis Services**.

2. В поле **Подключение к серверу** вставьте имя сервера, затем в поле **Проверка подлинности** выберите **Active Directory — универсальная с поддержкой многофакторной проверки подлинности**, введите имя пользователя и нажмите кнопку **Подключиться**.

    ![Вход](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-signin.png)

3. В **обозревателе объектов** щелкните правой кнопкой мыши пример базы данных `adventureworks`, затем выберите **Удалить**.

    ![Удаление примера базы данных](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-delete.png)

## <a name="next-steps"></a>Дальнейшие действия 

В этом руководстве было рассмотрено добавление базового образца модели на сервер. Когда есть шаблон базы данных, можно подключиться к нему из SQL Server Management Studio и добавить роли пользователей. Для получения дополнительных сведений перейдите к следующему руководству.

> [!div class="nextstepaction"]
> [Руководство по настройке ролей администратора сервера и пользователя](tutorials/analysis-services-tutorial-roles.md)