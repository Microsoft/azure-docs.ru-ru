---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 07/21/2020
ms.openlocfilehash: 27df83552b450b7657c0595d1d419001df98148c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87040233"
---
## <a name="prerequisites"></a>Предварительные требования

* Учетная запись [OneDrive](https://www.microsoft.com/store/apps/onedrive/9wzdncrfj1p3). 

Прежде чем использовать учетную запись OneDrive с Logic Apps, необходимо авторизовать Logic Apps, чтобы подключиться к учетной записи OneDrive в портал Azure.

Выполните следующие действия, чтобы авторизовать Logic Apps для подключения к учетной записи OneDrive.  

1. Войдите на портал Azure. 

1. В разделе **службы Azure** выберите **Logic Apps**. Затем выберите имя приложения логики из списка.

1. В меню приложения логики выберите **Конструктор приложений логики** в разделе **средства разработки**.

1. В конструкторе Logic Apps выберите в раскрывающемся списке параметр **отобразить API, управляемые Майкрософт** , а затем введите в поле поиска *OneDrive* . Выберите триггер или действие для использования:

   ![Снимок экрана конструктора Logic Apps, отображающий список действий API OneDrive для добавления.](./media/connectors-create-api-onedrive/onedrive-1.png)

2. Если вы ранее не создавали подключения к OneDrive, выполните запрос на вход с использованием учетных данных OneDrive:  

   ![Снимок экрана конструктора Logic Apps, в котором отображается запрос на вход для API OneDrive.](./media/connectors-create-api-onedrive/onedrive-2.png)

3. Выберите **Войти** и введите имя пользователя и пароль. Выберите **Вход**: 

   ![Снимок экрана со страницей входа учетная запись Майкрософт для авторизации API OneDrive.](./media/connectors-create-api-onedrive/onedrive-3.png)   

    Эти учетные данные используются для авторизации приложения логики для доступа к данным в вашей учетной записи OneDrive. 

4. Выберите **Да**, чтобы авторизовать приложение логики для использования учетной записи OneDrive.  

   ![Снимок экрана учетная запись Майкрософт авторизации для Logic Apps с отображением разрешенных действий.](./media/connectors-create-api-onedrive/onedrive-4.png)   
   
5. Теперь подключение будет указано на шаге. Щелкните Сохранить, а затем продолжите создание приложения логики. 

   ![Снимок экрана: конструктор Logic Apps, отображающий редактор действий с подключением к API OneDrive.](./media/connectors-create-api-onedrive/onedrive-5.png)
