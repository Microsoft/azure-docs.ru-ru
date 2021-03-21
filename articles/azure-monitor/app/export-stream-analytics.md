---
title: Экспорт из Azure Application Insights с помощью Stream Analytics | Документация Майкрософт
description: Stream Analytics может непрерывно преобразовывать, фильтровать и маршрутизировать данные, экспортируемые из Application Insights.
ms.topic: conceptual
ms.date: 01/08/2019
ms.openlocfilehash: a517bddd8981554b7fb5044d33b6c6777df51e36
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101719803"
---
# <a name="use-stream-analytics-to-process-exported-data-from-application-insights"></a>Обработка данных, экспортированных из Application Insights, при помощи Stream Analytics

[Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/) — идеальное средство для обработки данных, [экспортированных из Application Insights](export-telemetry.md). Stream Analytics может извлекать данные из различных источников. Это средство может преобразовывать и фильтровать данные и затем отправлять их в различные приемники.

В этом примере мы создадим адаптер, который принимает данные из Application Insights, переименовывает и обрабатывает некоторые поля, а затем передает данные в Power BI.

> [!WARNING]
> Мы рекомендуем использовать более простые и удобные [способы отображения данных Application Insights в Power BI](./export-power-bi.md). Используемый в этой статье способ приведен только в качестве примера, чтобы показать, как можно обрабатывать экспортированные данные.

![Блок-схема экспорта в PBI через SA](./media/export-stream-analytics/020.png)

## <a name="create-storage-in-azure"></a>Создание хранилища в Azure
В результате непрерывного экспорта происходит передача данных в учетную запись хранилища Azure, поэтому необходимо сначала создать хранилище.

1. Создайте "классическую" учетную запись хранения в подписке на [портале Azure](https://portal.azure.com).
   
   ![На портале Azure выберите "Создать", "Данные", "Хранилище".](./media/export-stream-analytics/030.png)
2. Создание контейнера
   
    ![В новом хранилище выберите "Контейнеры", щелкните элемент "Контейнеры", а затем — "Добавить".](./media/export-stream-analytics/040.png)
3. Скопируйте ключ доступа к хранилищу.
   
    Он вскоре потребуется для настройки входных данных для службы Stream Analytics.
   
    ![В хранилище откройте "Параметры", "Ключи" и сделайте копию первичного ключа доступа.](./media/export-stream-analytics/045.png)

## <a name="start-continuous-export-to-azure-storage"></a>Запуск непрерывного экспорта в службе хранилища Azure

[Непрерывный экспорт](export-telemetry.md) перемещает данные из Application Insights в хранилище Azure.

1. На портале Azure перейдите к ресурсу Application Insights, созданному для приложения.
   
    ![Выберите "Обзор", Application Insights, а затем выберите свое приложение.](./media/export-stream-analytics/050.png)
2. Создайте непрерывный экспорт.
   
    ![Выберите "Параметры", "Непрерывный экспорт", "Добавить".](./media/export-stream-analytics/060.png)

    Выберите учетную запись хранения, созданную ранее:

    ![Установите место назначения экспорта.](./media/export-stream-analytics/070.png)

    Задайте необходимые типы событий:

    ![Выберите типы событий.](./media/export-stream-analytics/080.png)

1. Пусть данные накопятся. Предоставьте пользователям возможность поработать с приложением на протяжении некоторого времени. После получения данных телеметрии в [обозревателе метрик](../essentials/metrics-charts.md) отобразятся статистические диаграммы, а в разделе [поиска по журналу диагностики](./diagnostic-search.md) — отдельные события.
   
    Данные также будут экспортированы в хранилище. 
2. Проверьте экспортированные данные. В Visual Studio откройте меню **"Вид" или "Обозреватель облака"** и выберите элемент "Azure" или "Хранилище". Если эти пункты меню не отображаются, установите пакет SDK для Azure. Откройте диалоговое окно "Новый проект" и выберите Visual C# / Cloud / Get Microsoft Azure SDK for .NET. (Visual C# / Облако / Получить пакет Microsoft Azure SDK для .NET.)
   
    ![Снимок экрана, показывающий, как задать типы событий, которые требуется просмотреть.](./media/export-stream-analytics/04-data.png)
   
    Запишите общую часть имени пути, которое образовано от имени приложения и ключа инструментирования.

События записываются в JSON-файлы больших двоичных объектов. Каждый файл может содержать одно или несколько событий. Поэтому нам нужна возможность считывать данные событий и отфильтровывать необходимые поля. С данными можно выполнять любые действия, но сейчас мы будем использовать Stream Analytics для перемещения данных в Power BI.

## <a name="create-an-azure-stream-analytics-instance"></a>Создание экземпляра Azure Stream Analytics

На [портале Azure](https://portal.azure.com/) выберите службу Azure Stream Analytics и создайте новое задание Stream Analytics:

![Снимок экрана, на котором показана Главная страница для создания Stream Analytics задания в портал Azure.](./media/export-stream-analytics/SA001.png)

![Снимок экрана, на котором отображаются сведения, необходимые при создании нового задания Stream Analytics.](./media/export-stream-analytics/SA002.png)

Создав новое задание, выберите **Перейти к ресурсу**.

![Снимок экрана, на котором показано сообщение, полученное при успешном развертывании нового задания Stream Analytics.](./media/export-stream-analytics/SA003.png)

### <a name="add-a-new-input"></a>Добавление новых входных данных

![Снимок экрана, показывающий, как добавить входные данные в задание Stream Analytics.](./media/export-stream-analytics/SA004.png)

Задайте расположение для приема входных данных из большого двоичного объекта непрерывного экспорта:

![Снимок экрана, показывающий настройку задания Stream Analytics для получения входных данных из большого двоичного объекта непрерывного экспорта.](./media/export-stream-analytics/SA0005.png)

Теперь потребуется первичный ключ доступа из вашей учетной записи хранения, указанной ранее. Задайте его в качестве ключа учетной записи хранения.

### <a name="set-path-prefix-pattern"></a>Установка шаблона префикса пути

**Задайте в поле "Формат даты" значение в формате ГГГГ-ММ-ДД (с дефисами).**

Шаблон префикса пути указывает, где Stream Analytics находит входные файлы в хранилище. Вам необходимо настроить это поле в соответствии с тем, как функция непрерывного экспорта сохраняет данные. Задайте следующее значение:

`webapplication27_12345678123412341234123456789abcdef0/PageViews/{date}/{time}`

В данном примере:

* `webapplication27` — имя ресурса Application Insights ( **только строчные буквы**).
* `1234...` — ключ инструментирования ресурса Application Insights с **удаленными дефисами**. 
* `PageViews` — тип данных, которые необходимо проанализировать. Доступные типы зависят от фильтра, установленного в функции непрерывного экспорта. Изучите экспортированные данные, чтобы увидеть другие доступные типы, а также ознакомьтесь с [моделью экспорта данных](export-data-model.md).
* `/{date}/{time}` – шаблон, записанный буквально.

> [!NOTE]
> Проверьте хранилище и убедитесь, что вы получаете нужный путь.

## <a name="add-new-output"></a>Добавление новых выходных данных

Теперь выберите свое задание и последовательно нажмите элементы **Выходные данные** > **Добавить**.

![Снимок экрана, на котором показано, как выбрать Stream Analytics задание для добавления новых выходных данных.](./media/export-stream-analytics/SA006.png)


![Выбор нового канала и элементов "Выходные данные", "Добавить", "Power BI".](./media/export-stream-analytics/SA010.png)

Предоставьте свою **рабочую или учебную учетную запись** для обеспечения доступа Stream Analytics к ресурсу Power BI. Затем укажите имя для выходных данных, а также для целевого набора данных и таблицы Power BI.

## <a name="set-the-query"></a>Определение запроса

Запрос определяет преобразование входных данных в выходные.

Используйте функцию "Проверить" для проверки выходных данных. Предоставьте ей образец данных, полученный на странице входных данных.

### <a name="query-to-display-counts-of-events"></a>Запрос для отображения количества событий

Вставьте следующий запрос.

```SQL
SELECT
  flat.ArrayValue.name,
  count(*)
INTO
  [pbi-output]
FROM
  [export-input] A
OUTER APPLY GetElements(A.[event]) as flat
GROUP BY TumblingWindow(minute, 1), flat.ArrayValue.name
```

* export-input — это псевдоним, присвоенный входным данным потока.
* pbi-output — это определенный псевдоним выходных данных.
* Мы используем [OUTER APPLY GetElements](/stream-analytics-query/apply-azure-stream-analytics), так как имя события находится во вложенном массиве JSON. Затем элемент Select выбирает имя события, а также количество экземпляров с этим именем за определенный период времени. Предложение [Group By](/stream-analytics-query/group-by-azure-stream-analytics) группирует элементы по периодам времени в одну минуту.

### <a name="query-to-display-metric-values"></a>Запрос для отображения значений метрики

```SQL
SELECT
  A.context.data.eventtime,
  avg(CASE WHEN flat.arrayvalue.myMetric.value IS NULL THEN 0 ELSE  flat.arrayvalue.myMetric.value END) as myValue
INTO
  [pbi-output]
FROM
  [export-input] A
OUTER APPLY GetElements(A.context.custom.metrics) as flat
GROUP BY TumblingWindow(minute, 1), A.context.data.eventtime
```

* Этот запрос позволяет получить время события и значение метрики по телеметрии метрики. Значения метрик находятся в массиве, поэтому мы используем шаблон OUTER APPLY GetElements для извлечения строк. myMetric — это имя метрики.

### <a name="query-to-include-values-of-dimension-properties"></a>Запрос для добавления значений свойств измерения

```SQL
WITH flat AS (
SELECT
  MySource.context.data.eventTime as eventTime,
  InstanceId = MyDimension.ArrayValue.InstanceId.value,
  BusinessUnitId = MyDimension.ArrayValue.BusinessUnitId.value
FROM MySource
OUTER APPLY GetArrayElements(MySource.context.custom.dimensions) MyDimension
)
SELECT
  eventTime,
  InstanceId,
  BusinessUnitId
INTO AIOutput
FROM flat
```

* Этот запрос включает в себя значения свойств измерения вне зависимости от того, соответствует ли конкретному измерению фиксированный индекс в массиве.

## <a name="run-the-job"></a>Запуск задания

Дату запуска задания можно выбрать в прошлом.

![Выбор задания и элемента "Запрос". Вставка следующего примера.](./media/export-stream-analytics/SA008.png)

Подождите, пока задание находится в статусе выполнения.

## <a name="see-results-in-power-bi"></a>Просмотр результатов в Power BI

> [!WARNING]
> Мы рекомендуем использовать более простые и удобные [способы отображения данных Application Insights в Power BI](./export-power-bi.md). Используемый в этой статье способ приведен только в качестве примера, чтобы показать, как можно обрабатывать экспортированные данные.


Откройте Power BI с рабочей или учебной учетной записью и выберите набор данных и таблицу, которые были определены как выходные данные задания Stream Analytics.

![Выбор набора данных и полей в Power BI.](./media/export-stream-analytics/200.png)

Теперь этот набор данных можно использовать в отчетах и панелях мониторинга в [Power BI](https://powerbi.microsoft.com).

![На снимке экрана показан пример отчета из набора данных в Power BI.](./media/export-stream-analytics/210.png)

## <a name="no-data"></a>Данные отсутствуют?

* Проверьте правильность [формата даты](#set-path-prefix-pattern) — ГГГГ-ММ-ДД (с дефисами).

## <a name="next-steps"></a>Дальнейшие действия
* [непрерывный экспорт.](export-telemetry.md)
* [Подробный справочник по модели данных типов и значений свойств.](export-data-model.md)
* [Application Insights](./app-insights-overview.md)
