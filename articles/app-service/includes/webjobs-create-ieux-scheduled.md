---
author: ggailey777
ms.assetid: af01771e-54eb-4aea-af5f-f883ff39572b
ms.topic: include
ms.date: 10/16/2018
ms.title: include
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: ef0aa8ba1983ca30fd44c27fe570b6b5f51733a5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746686"
---
## <a name="create-a-scheduled-webjob"></a><a name="CreateScheduledCRON"></a> Создание запланированного веб-задания


1. Получено [портал Azure](https://portal.azure.com).
1. Перейдите в **службу приложений** вашего <abbr title="Ресурс приложения может быть веб-приложением, приложением API или мобильным приложением.">Ресурс приложения</abbr>.
1. Выберите **Веб-задания**.

   ![Выбор веб-заданий](../media/web-sites-create-web-jobs/select-webjobs.png)

1. На странице **Веб-задания** выберите **Добавить**.

    ![Страница веб-задания](../media/web-sites-create-web-jobs/wjblade.png)

1. В окне **Добавление веб-задания** задайте значения параметров, указанные в таблице.

    ![Страница добавления веб-задания](../media/web-sites-create-web-jobs/addwjscheduled.png)
    
    | Параметр      | Образец значения   |
    | ------------ | ----------------- | 
    | <abbr title="Имя, которое является уникальным в пределах приложения службы приложений. Должно начинаться с буквы или цифры и не может содержать специальные символы, отличные от `-` и `_` .">Имя</a> | myScheduledWebJob |  |
    | <abbr title="*ZIP*-файл, который содержит исполняемый файл или файл сценария, а также все вспомогательные файлы, необходимые для запуска программы или сценария.">Передача файла</abbr> | ConsoleApp.zip |
    | <abbr title="Типы включают в себя непрерывные, активируемые.">Тип</abbr> | Активируемые |
    | <abbr title="Для надежной работы расписания включите функцию Always On. Она доступна только в ценовых категориях "Базовый", "Стандартный" и "Премиум".">Триггеры</a> | Запланировано |
    | Выражение CRON</a> | 0 0/20 * * * * | 
    
    <br>
    
    <details>
     <summary>Дополнительные сведения о выражениях CRON</summary>
     <a name="#ncrontab-expressions"></a>
    
     Можно ввести [выражение нкронтаб](../../azure-functions/functions-bindings-timer.md#ncrontab-expressions) на портале или включить `settings.job` файл в корень *ZIP* -файла веб-задания, как показано в следующем примере:
     
     ```json
     {
         "schedule": "0 */15 * * * *"
     }
     ```
     
     Дополнительные сведения см. в разделе [Планирование запуска веб-задания](../webjobs-dotnet-deploy-vs.md#scheduling-a-triggered-webjob).
     
     [!INCLUDE [webjobs-cron-timezone-note](../../../includes/webjobs-cron-timezone-note.md)]
     </details>
     <br>

1. Нажмите кнопку **OK**.

    Новое веб-задание появится на странице **Веб-задания**.
    
    ![Список веб-заданий](../media/web-sites-create-web-jobs/listallwebjobs.png)
