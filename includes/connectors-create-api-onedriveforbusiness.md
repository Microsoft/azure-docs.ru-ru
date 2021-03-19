---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 07/21/2020
ms.openlocfilehash: 06a503e800309d8689735417931820d9f16a6e75
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87102958"
---
## <a name="prerequisites"></a>Предварительные требования

* Учетная запись [OneDrive для бизнеса](https://OneDrive.com) 

Прежде чем использовать учетную запись OneDrive для бизнеса с Logic Apps, необходимо авторизовать Logic Apps, чтобы подключиться к учетной записи OneDrive для бизнеса в портал Azure.

Выполните следующие действия, чтобы авторизовать Logic Apps для подключения к учетной записи OneDrive для бизнеса:  

1. Войдите на портал Azure. 

1. В разделе **службы Azure** выберите **Logic Apps**. Затем выберите имя приложения логики из списка.

1. В меню приложения логики выберите **Конструктор приложений логики** в разделе **средства разработки**.

1. В конструкторе Logic Apps выберите в раскрывающемся списке параметр **отобразить API, управляемые Майкрософт** , а затем введите в поле поиска *OneDrive для бизнеса* . Выберите триггер или действие для использования:  

   ![Снимок экрана конструктора Logic Apps, показывающий триггер повторения с действиями API OneDrive для бизнеса.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-1.png)

2. Если вы ранее не создавали подключения к OneDrive для бизнеса, выполните запрос, чтобы указать учетные данные OneDrive для бизнеса. Эти учетные данные используются для авторизации приложения логики для доступа к данным вашей учетной записи OneDrive для бизнеса:  

   ![Снимок экрана: конструктор Logic Apps, отображающий запрос на вход в OneDrive для бизнеса.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-2.png)

3. Укажите имя пользователя и пароль OneDrive для бизнеса, чтобы авторизовать приложение логики:  

   ![Снимок экрана: страница входа в OneDrive для бизнеса, в которой отображается запрос на вход.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-3.png)   

4. Теперь подключение будет указано на шаге. Щелкните Сохранить, а затем продолжите создание приложения логики. 

   ![Снимок экрана конструктора Logic Apps, в котором отображается триггер с подключением OneDrive для бизнеса.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-4.png)   
