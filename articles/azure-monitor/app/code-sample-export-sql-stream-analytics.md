---
title: Экспорт в SQL из Azure Application Insights | Документация Майкрософт
description: Осуществляйте непрерывный экспорт данных Application Insights в SQL с использованием Stream Analytics.
ms.topic: conceptual
ms.date: 09/11/2017
ms.openlocfilehash: 25d28bb0fc35ef76231c085dc1d9d8a1234a264c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100587760"
---
# <a name="walkthrough-export-to-sql-from-application-insights-using-stream-analytics"></a>Пошаговое руководство. Экспорт в SQL из Application Insights с использованием Stream Analytics
В этой статье показано, как переместить данные телеметрии из [Application Insights Azure][start] в базу данных SQL Azure с помощью [непрерывного экспорта][export] и [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/). 

Непрерывный экспорт позволяет переместить данные телеметрии в службу хранилища Azure в формате JSON. Мы выполним анализ объектов JSON, используя Azure Stream Analytics, и создадим строки в таблице базы данных.

(В общих чертах непрерывный экспорт представляет собой способ для самостоятельного анализа телеметрии, которую приложения отправляют в Application Insights. Этот пример кода можно адаптировать для выполнения других задач с экспортированными данными телеметрии, таких как агрегирование данных.)

Для начала предположим, что у вас уже есть приложение, мониторинг которого нужно выполнить.

В этом примере мы используем данные просмотров страницы, но этот же шаблон можно легко использовать и для других типов данных, таких как пользовательские события и исключения. 

## <a name="add-application-insights-to-your-application"></a>Добавление Application Insights в приложение
Чтобы начать работу:

1. [Настройте Application Insights для веб-страниц](./javascript.md). 
   
    (В этом примере мы сосредоточимся на обработке данных просмотра страниц из клиентских браузеров. Но вы также можете настроить Application Insights для серверной части своего приложения [Java](./java-get-started.md) или [ASP.NET](./asp-net.md) и обрабатывать данные запросов, зависимостей и другие данные телеметрии сервера.)
2. Опубликуйте свое приложение и просмотрите данные телеметрии, появившиеся в ресурсе Application Insights.

## <a name="create-storage-in-azure"></a>Создание хранилища в Azure
В результате непрерывного экспорта происходит передача данных в учетную запись хранилища Azure, поэтому необходимо сначала создать хранилище.

1. Создайте учетную запись хранения в подписке на [портале Azure][portal].
   
    ![На портале Azure выберите «Создать», «Данные», «Хранилище». Выберите «Классический» и щелкните «Создать». Введите имя хранилища.](./media/code-sample-export-sql-stream-analytics/040-store.png)
2. Создание контейнера
   
    ![В новом хранилище выберите "Контейнеры", щелкните элемент "Контейнеры", а затем — "Добавить".](./media/code-sample-export-sql-stream-analytics/050-container.png)
3. Скопируйте ключ доступа к хранилищу.
   
    Он вскоре потребуется для настройки входных данных для службы Stream Analytics.
   
    ![В хранилище откройте "Параметры", "Ключи" и сделайте копию первичного ключа доступа.](./media/code-sample-export-sql-stream-analytics/21-storage-key.png)

## <a name="start-continuous-export-to-azure-storage"></a>Запуск непрерывного экспорта в службе хранилища Azure
1. На портале Azure перейдите к ресурсу Application Insights, созданному для приложения.
   
    ![Выберите "Обзор", Application Insights, а затем выберите свое приложение.](./media/code-sample-export-sql-stream-analytics/060-browse.png)
2. Создайте непрерывный экспорт.
   
    ![Выберите "Параметры", "Непрерывный экспорт", "Добавить".](./media/code-sample-export-sql-stream-analytics/070-export.png)

    Выберите учетную запись хранения, созданную ранее:

    ![Установите место назначения экспорта.](./media/code-sample-export-sql-stream-analytics/080-add.png)

    Задайте необходимые типы событий:

    ![Выберите типы событий.](./media/code-sample-export-sql-stream-analytics/085-types.png)


1. Пусть данные накопятся. Предоставьте пользователям возможность поработать с приложением на протяжении некоторого времени. После получения данных телеметрии в [обозревателе метрик](../essentials/metrics-charts.md) отобразятся статистические диаграммы, а в разделе [поиска по журналу диагностики](./diagnostic-search.md) — отдельные события. 
   
    Данные также будут экспортированы в хранилище. 
2. Проверьте экспортированные данные на портале (щелкните **Обзор**, выберите учетную запись хранения и щелкните **Контейнеры**) или в Visual Studio. В Visual Studio откройте меню **"Вид" или "Обозреватель облака"** и выберите элемент "Azure" или "Хранилище". Если эти пункты меню не отображаются, установите пакет SDK для Azure. Откройте диалоговое окно "Новый проект" и выберите Visual C# / Cloud / Get Microsoft Azure SDK for .NET. (Visual C# / Облако / Получить пакет Microsoft Azure SDK для .NET.)
   
    ![В Visual Studio откройте "Обозреватель сервера", "Azure", "Хранилище".](./media/code-sample-export-sql-stream-analytics/087-explorer.png)
   
    Запишите общую часть имени пути, которое образовано от имени приложения и ключа инструментирования. 

События записываются в JSON-файлы больших двоичных объектов. Каждый файл может содержать одно или несколько событий. Поэтому нам нужна возможность считывать данные событий и отфильтровывать необходимые поля. Есть все виды вещей, которые мы можем сделать с данными, но наш план уже сегодня — использовать Stream Analytics для перемещения данных в базу данных SQL. Это облегчит выполнение многих любопытных запросов.

## <a name="create-an-azure-sql-database"></a>Создание базы данных SQL Azure
В своей подписке на [портале Azure][portal] создайте базу данных (и сервер, если у вас его еще нет), куда будут записываться данные.

!["Создать", "Данные", SQL.](./media/code-sample-export-sql-stream-analytics/090-sql.png)

Убедитесь, что сервер разрешает доступ к службам Azure:

!["Обзор", "Серверы", ваш сервер, "Параметры", "Брандмауэр", "Разрешить доступ к Azure".](./media/code-sample-export-sql-stream-analytics/100-sqlaccess.png)

## <a name="create-a-table-in-azure-sql-database"></a>Создание таблицы в базе данных SQL Azure
Подключитесь к базе данных, созданной в предыдущем разделе, с помощью желаемого инструмента управления. В этом пошаговом руководстве мы используем [инструменты управления SQL Server](/sql/ssms/sql-server-management-studio-ssms) (SSMS).

![Подключение к Базе данных SQL Azure](./media/code-sample-export-sql-stream-analytics/31-sql-table.png)

Создайте новый запрос и выполните следующий сценарий T-SQL:

```SQL

CREATE TABLE [dbo].[PageViewsTable](
    [pageName] [nvarchar](max) NOT NULL,
    [viewCount] [int] NOT NULL,
    [url] [nvarchar](max) NULL,
    [urlDataPort] [int] NULL,
    [urlDataprotocol] [nvarchar](50) NULL,
    [urlDataHost] [nvarchar](50) NULL,
    [urlDataBase] [nvarchar](50) NULL,
    [urlDataHashTag] [nvarchar](max) NULL,
    [eventTime] [datetime] NOT NULL,
    [isSynthetic] [nvarchar](50) NULL,
    [deviceId] [nvarchar](50) NULL,
    [deviceType] [nvarchar](50) NULL,
    [os] [nvarchar](50) NULL,
    [osVersion] [nvarchar](50) NULL,
    [locale] [nvarchar](50) NULL,
    [userAgent] [nvarchar](max) NULL,
    [browser] [nvarchar](50) NULL,
    [browserVersion] [nvarchar](50) NULL,
    [screenResolution] [nvarchar](50) NULL,
    [sessionId] [nvarchar](max) NULL,
    [sessionIsFirst] [nvarchar](50) NULL,
    [clientIp] [nvarchar](50) NULL,
    [continent] [nvarchar](50) NULL,
    [country] [nvarchar](50) NULL,
    [province] [nvarchar](50) NULL,
    [city] [nvarchar](50) NULL
)

CREATE CLUSTERED INDEX [pvTblIdx] ON [dbo].[PageViewsTable]
(
    [eventTime] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)

```

![Создание Пажевиевстабле](./media/code-sample-export-sql-stream-analytics/34-create-table.png)

В этом примере мы используем данные из представлений страниц. Для просмотра других доступных данных проверьте выходные данные JSON и ознакомьтесь с разделом [Экспорт модели данных](./export-data-model.md).

## <a name="create-an-azure-stream-analytics-instance"></a>Создание экземпляра Azure Stream Analytics
На [портале Azure](https://portal.azure.com/) выберите службу Azure Stream Analytics и создайте новое задание Stream Analytics:

![На снимке экрана показана страница задания Stream Analytics с выделенной кнопкой "создать".](./media/code-sample-export-sql-stream-analytics/SA001.png)

![Новое задание Stream Analytics](./media/code-sample-export-sql-stream-analytics/SA002.png)

Создав новое задание, выберите **Перейти к ресурсу**.

![На снимке экрана показано сообщение Развертывание прошло, и нажмите кнопку ресурс.](./media/code-sample-export-sql-stream-analytics/SA003.png)

#### <a name="add-a-new-input"></a>Добавление новых входных данных

![На снимке экрана показана страница входные данные с выбранной кнопкой "Добавить".](./media/code-sample-export-sql-stream-analytics/SA004.png)

Задайте расположение для приема входных данных из большого двоичного объекта непрерывного экспорта:

![На снимке экрана показано новое окно ввода с выбранными параметрами "псевдоним входа", "источник" и "учетная запись хранения".](./media/code-sample-export-sql-stream-analytics/SA0005.png)

Теперь потребуется первичный ключ доступа из вашей учетной записи хранения, указанной ранее. Задайте его в качестве ключа учетной записи хранения.

#### <a name="set-path-prefix-pattern"></a>Установка шаблона префикса пути

**Задайте в поле "Формат даты" значение в формате ГГГГ-ММ-ДД (с дефисами).**

Шаблон префикса пути указывает, как Stream Analytics находит входные файлы в хранилище. Вам необходимо настроить это поле в соответствии с тем, как функция непрерывного экспорта сохраняет данные. Задайте следующее значение:

```sql
webapplication27_12345678123412341234123456789abcdef0/PageViews/{date}/{time}
```

В данном примере:

* `webapplication27` — имя ресурса Application Insights ( **только строчные буквы**). 
* `1234...` — ключ инструментирования ресурса Application Insights с **удаленными дефисами**. 
* `PageViews` – тип данных, которые необходимо проанализировать. Доступные типы зависят от фильтра, установленного в функции непрерывного экспорта. Изучите экспортированные данные, чтобы увидеть другие доступные типы, а также ознакомьтесь с [моделью экспорта данных](./export-data-model.md).
* `/{date}/{time}` – шаблон, записанный буквально.

Чтобы получить имя и ключ инструментирования Application Insights, откройте раздел "Основные компоненты" на странице обзора или вкладку "Параметры".

> [!TIP]
> Используйте функцию Sample для проверки правильности входного пути. В случае сбоя убедитесь, что в хранилище для выбранного диапазона времени есть данные. Измените определение ввода и убедитесь, что учетная запись хранения, префикс пути и формат даты указаны правильно.

 
## <a name="set-query"></a>Настройка запроса
Откройте раздел запроса:

Замените запрос по умолчанию следующим:

```SQL

    SELECT flat.ArrayValue.name as pageName
    , flat.ArrayValue.count as viewCount
    , flat.ArrayValue.url as url
    , flat.ArrayValue.urlData.port as urlDataPort
    , flat.ArrayValue.urlData.protocol as urlDataprotocol
    , flat.ArrayValue.urlData.host as urlDataHost
    , flat.ArrayValue.urlData.base as urlDataBase
    , flat.ArrayValue.urlData.hashTag as urlDataHashTag
      ,A.context.data.eventTime as eventTime
      ,A.context.data.isSynthetic as isSynthetic
      ,A.context.device.id as deviceId
      ,A.context.device.type as deviceType
      ,A.context.device.os as os
      ,A.context.device.osVersion as osVersion
      ,A.context.device.locale as locale
      ,A.context.device.userAgent as userAgent
      ,A.context.device.browser as browser
      ,A.context.device.browserVersion as browserVersion
      ,A.context.device.screenResolution.value as screenResolution
      ,A.context.session.id as sessionId
      ,A.context.session.isFirst as sessionIsFirst
      ,A.context.location.clientip as clientIp
      ,A.context.location.continent as continent
      ,A.context.location.country as country
      ,A.context.location.province as province
      ,A.context.location.city as city
    INTO
      AIOutput
    FROM AIinput A
    CROSS APPLY GetElements(A.[view]) as flat


```

Обратите внимание, что первые несколько свойств соответствуют данным просмотров страницы. При экспорте телеметрии других типов будут получены другие свойства. См. [значения и типы свойств в подробном руководстве по модели данных](./export-data-model.md).

## <a name="set-up-output-to-database"></a>Настройка выходных данных в базе данных
Выберите SQL в качестве типа выходных данных.

![В Stream Analytics выберите "Выходные данные".](./media/code-sample-export-sql-stream-analytics/SA006.png)

Укажите базу данных.

![Введите информацию о базе данных.](./media/code-sample-export-sql-stream-analytics/SA007.png)

Закройте окно мастера и дождитесь появления уведомления о том, что выходные данные были настроены.

## <a name="start-processing"></a>Начало обработки
Запустите задание на панели действий:

![В Stream Analytics щелкните "Запуск".](./media/code-sample-export-sql-stream-analytics/SA008.png)

Вы можете выбрать тип обработки данных – обработка текущих данных или данных, полученных ранее. Последний вариант удобен, если непрерывный экспорт уже выполняется в течение некоторого времени.

Через несколько минут вернитесь к инструментам управления SQL Server и просмотрите передачу данных. Например, используйте запрос наподобие этого:

```sql
SELECT TOP 100 *
FROM [dbo].[PageViewsTable]
```

## <a name="related-articles"></a>Похожие статьи
* [Экспорт в Power BI с использованием Stream Analytics](./export-power-bi.md)
* [подробная ссылка на модель данных для типов и значений свойств.](./export-data-model.md)
* [Непрерывный экспорт в Application Insights](./export-telemetry.md)
* [Application Insights](https://azure.microsoft.com/services/application-insights/)

<!--Link references-->

[diagnostic]: ./diagnostic-search.md
[export]: ./export-telemetry.md
[metrics]: ../essentials/metrics-charts.md
[portal]: https://portal.azure.com/
[start]: ./app-insights-overview.md

