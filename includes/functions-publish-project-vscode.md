---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/28/2020
ms.author: glenga
ms.openlocfilehash: 2517f132578b5de6b062b38ce94581f118327a13
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99493602"
---
## <a name="publish-the-project-to-azure"></a>Публикация проекта в Azure

В этом разделе показано создание приложения-функции и сопутствующих ресурсов в подписке Azure и последующее развертывание кода.

> [!IMPORTANT]
> Публикация в существующее приложение-функцию перезаписывает содержимое этого приложения в Azure.


1. Щелкните значок Azure на панели действий, а затем в области **Azure: Функции** выберите кнопку **Deploy to function app...** (Развертывание в приложение-функцию).

    ![Публикация проекта в Azure](./media/functions-publish-project-vscode/function-app-publish-project.png)

1. Введите следующие сведения по соответствующим запросам:

    - **Выбор папки**. Выберите папку из рабочей области или перейдите к папке, содержащей приложение-функцию. Этот элемент не отобразится, если вы уже открыли допустимое приложение-функцию.

    - **Выбрать подписку**. Выберите подписку, которую нужно использовать. Если у вас только одна подписка, вы не увидите этот параметр.

    - **Select Function App in Azure** (Выбор приложения-функции в Azure). Выберите `- Create new Function App`. (Не выбирайте параметр `Advanced`, так как он не рассматривается в этой статье.)
      
    - **Enter a globally unique name for the function app** (Ввод глобально уникального имени для приложения-функции). Введите имя, допустимое в пути URL-адреса. Имя, которое вы вводите, проверяется, чтобы убедиться, что оно уникально в функциях Azure.
    
    - **Select a location for new resources** (Выбор расположения для новых ресурсов).  Для повышения производительности выберите [регион](https://azure.microsoft.com/regions/) рядом с вами. 
    
    Расширение показывает в области уведомлений состояние отдельных ресурсов по мере их создания в Azure.

    :::image type="content" source="media/functions-publish-project-vscode/resource-notification.png" alt-text="Уведомление о создании ресурса Azure":::
    
1.  После завершения в вашей подписке создаются следующие ресурсы Azure с именами, производными от имени приложения-функции:
    
    [!INCLUDE [functions-vs-code-created-resources](functions-vs-code-created-resources.md)]

    После создания приложения-функции и применения пакета развертывания отобразится уведомление. 

    [!INCLUDE [functions-vs-code-create-tip](functions-vs-code-create-tip.md)]

4. Выберите **View Output** (Просмотреть выходные данные) в уведомлении, чтобы просмотреть результаты создания и развертывания ресурсов Azure. Если вы пропустили уведомление, щелкните значок колокольчика в правом нижнем углу, чтобы снова просмотреть его.

    ![Создание уведомления о завершении](media/functions-publish-project-vscode/function-create-notifications.png)
