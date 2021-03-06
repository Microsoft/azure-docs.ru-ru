---
title: Отправка журнала действий Azure в рабочую область Log Analytics с помощью портала Azure
description: Используйте портал Azure, чтобы создать параметр диагностики и рабочую область Log Analytics для отправки журнала действий в журналы Azure Monitor.
ms.topic: quickstart
author: bwren
ms.author: bwren
ms.date: 06/25/2020
ms.openlocfilehash: fec1f4f3ae13f6c9ed5fdd7ffbcd143e5c5e5f52
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102033254"
---
# <a name="send-azure-activity-log-to-log-analytics-workspace-using-azure-portal"></a>Отправка журнала действий Azure в рабочую область Log Analytics с помощью портала Azure
Журнал действий — это журнал платформы в Azure, который предоставляет аналитические сведения о событиях уровня подписки, например об изменении ресурса или запуске виртуальной машины. Вы можете просмотреть журнал действий на портале Azure или получить записи с помощью PowerShell и CLI. В этом кратком руководстве показано, как применить портал Azure для создания параметра диагностики и рабочей области Log Analytics, а также для отправки журнала действий в журналы Azure Monitor, где можно их анализировать с помощью [запросов по журналам](../logs/log-query-overview.md) и применять другие возможности, такие как [оповещения журнала](../alerts/alerts-log-query.md) и [книги](../visualize/workbooks-overview.md). 

## <a name="sign-in-to-azure-portal"></a>Вход на портал Azure
Войдите на портал Azure по адресу [https://portal.azure.com](https://portal.azure.com). 



## <a name="create-a-log-analytics-workspace"></a>Создание рабочей области Log Analytics
На портале Azure найдите и выберите **Рабочие области Log Analytics**. 

![Снимок экрана: портал Azure, на котором отображается поле поиска с запросом "рабочие области Log Analytics" и выделена надпись "Рабочие области Log Analytics" в разделе "Службы".](../logs/media/quick-create-workspace/azure-portal-01.png)
  
Щелкните **Добавить**, а затем укажите значения для параметров **Группа ресурсов**, **Имя** и **Расположение** рабочей области. Имя рабочей области должно быть уникальным в пределах всех подписок Azure.

![Создание рабочей области](media/quick-collect-activity-log/create-workspace.png)

Щелкните **Проверка и создание**, чтобы проверить параметры, а затем щелкните **Создать**, чтобы создать рабочую область. По умолчанию будет выбрана ценовая категория **с оплатой по мере использования**, по которой вы не будете нести расходов, пока не соберете достаточно большой объем данных. За сбор журнала действий плата не взимается.


## <a name="create-diagnostic-setting"></a>Создание параметра диагностики
На портале Azure найдите и выберите **Монитор**. 

![Снимок экрана: портал Azure, на котором отображается поле поиска с запросом "монитор" и выделена надпись "Монитор" в разделе "Службы".](media/quick-collect-activity-log/azure-portal-monitor.png)

Выберите **Журнал действий**. Здесь вы увидите последние события для текущей подписки. Щелкните **Параметры диагностики**, чтобы просмотреть параметры диагностики для подписки.

![Журнал действий](media/quick-collect-activity-log/activity-log.png)

Щелкните **Добавить параметр диагностики**, чтобы создать новый параметр. 

![Создание параметра диагностики](media/quick-collect-activity-log/create-diagnostic-setting.png)

Введите имя, например *Отправка журнала действий в рабочую область*. Выберите каждую из категорий. Выберите **Отправить в Log Analytics** как единственное место назначения, а затем укажите только что созданную рабочую область. Щелкните **Сохранить**, чтобы создать параметр диагностики, а затем закройте страницу.

![Новый параметр диагностики](media/quick-collect-activity-log/new-diagnostic-setting.png)

## <a name="generate-log-data"></a>Создание данных журнала
В рабочую область Log Analytics будут отправляться только новые записи журнала действий, поэтому выполните в подписке действия, которые будут регистрироваться, например запустите или остановите виртуальную машину либо создайте или измените другой ресурс. Создание параметра диагностики и запись данных в рабочую область может занять несколько минут. После этого все события, записанные в журнал действий, будут отправлены в рабочую область в течение нескольких секунд.

## <a name="retrieve-data-with-a-log-query"></a>Получение данных с помощью запросов по журналам

Выберите **Журналы** в меню **Azure Monitor**. Закройте страницу **Примеры запросов**. Если для созданной рабочей области не задана область, щелкните **Выбрать область** и найдите ее.

![Область Log Analytics](media/quick-collect-activity-log/log-analytics-scope.png)

В окне запроса введите `AzureActivity` и щелкните **Выполнить**. Этот простой запрос возвращает все записи в таблице *AzureActivity*, где хранятся все записи, отправленные из журнала действий.

![Простой запрос](media/quick-collect-activity-log/query-01.png)

Разверните одну из записей, чтобы просмотреть ее подробные свойства.

![Развертывание свойств](media/quick-collect-activity-log/expand-properties.png)

Попробуйте отправить более сложный запрос, например `AzureActivity | summarize count() by CategoryValue`, который возвращает количество событий, суммированных по категориям.

![Сложный запрос](media/quick-collect-activity-log/query-02.png)


## <a name="next-steps"></a>Дальнейшие действия
Из этого краткого руководства вы узнали, как настроить отправку журнала действий в рабочую область Log Analytics. Теперь вы можете настроить сбор других данных в рабочую область, где их можно анализировать с помощью [запросов к журналам](../logs/log-query-overview.md) в Azure Monitor, и применять другие возможности, например [оповещения журналов](../alerts/alerts-log-query.md) и [книги](../visualize/workbooks-overview.md). Затем вам нужно собрать [журналы ресурсов](../essentials/resource-logs.md) из ресурсов Azure, которые дополняют данные в журнале действий. Так вы сможете получить представление об операциях, выполненных в пределах каждого ресурса.


> [!div class="nextstepaction"]
> [Получение и анализ журналов ресурсов с помощью Azure Monitor](../essentials/tutorial-resource-logs.md)