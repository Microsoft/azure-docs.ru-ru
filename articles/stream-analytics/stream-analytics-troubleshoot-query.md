---
title: Устранение неполадок з запросами в службе Azure Stream Analytics
description: В этой статье описаны методы устранения неполадок с запросами в заданиях Azure Stream Analytics.
author: sidramadoss
ms.author: sidram
ms.service: stream-analytics
ms.topic: troubleshooting
ms.date: 03/31/2020
ms.custom: seodec18
ms.openlocfilehash: 0dc36b817d5b5cdf731edecd64e1879c153d866a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98015136"
---
# <a name="troubleshoot-azure-stream-analytics-queries"></a>Устранение неполадок з запросами в службе Azure Stream Analytics

В этой статье описаны распространенные проблемы с разработкой запросов Stream Analytics и способы их устранения.

В этой статье описаны распространенные проблемы, возникающие при разработке запросов Stream Analytics, способы диагностики и устранения этих проблем. Для многих действий по устранению неполадок должны быть включены журналы ресурсов для задания Stream Analytics. Если журналы ресурсов еще не включены, воспользуйтесь статьей [Устранение неполадок в Azure Stream Analytics с помощью журналов ресурсов](stream-analytics-job-diagnostic-logs.md)

## <a name="query-is-not-producing-expected-output"></a>Запрос не создает ожидаемых выходных данных

1.  Изучите ошибки с помощью локального тестирования:

    - На портале Azure откройте вкладку **Запрос** и выберите **Тест**. Для [проверки запроса](stream-analytics-test-query.md) используйте загруженные демонстрационные данные. Проанализируйте все ошибки и попытайтесь исправить их.   
    - Кроме того, вы можете [проверить запрос локально](stream-analytics-live-data-local-testing.md) с помощью инструментов Stream Analytics для Visual Studio или [Visual Studio Code](visual-studio-code-local-run-live-input.md). 

2.  Пошаговая [Отладка запросов с помощью схемы заданий](debug-locally-using-job-diagram-vs-code.md) Azure Stream Analytics средств для Visual Studio Code. На схеме заданий показано, как потоки данных из источников входных данных (концентратор событий, центр Интернета вещей и т. д.) проходят через несколько шагов запроса и попадают в приемники выходных данных. Каждый шаг запроса сопоставляется с временным результирующим набором, который определяется в скрипте с помощью инструкции WITH. Чтобы найти источник проблемы, вы можете просматривать данные и метрики по каждому из промежуточных результирующих наборов.

    ![Просмотр результатов в схеме заданий](./media/debug-locally-using-job-diagram-vs-code/preview-result.png)

3.  Если используются [**метки времени**](/stream-analytics-query/timestamp-by-azure-stream-analytics), убедитесь, что они зарегистрированы для событий после [начала выполнения задания](./stream-analytics-time-handling.md).

4.  Исключите типичные проблемы, например:
    - Предложение [**WHERE**](/stream-analytics-query/where-azure-stream-analytics) в запросе фильтрует все события, предотвращая создание выходных данных.
    - Выполнение функции [**CAST**](/stream-analytics-query/cast-azure-stream-analytics) завершается ошибкой, что приводит к сбою всего задания. Чтобы избежать сбоев приведения типов, используйте функцию [**TRY_CAST**](/stream-analytics-query/try-cast-azure-stream-analytics).
    - При использовании оконных функций подождите до завершения окна для вывода выходных данных из запроса.
    - Метка времени для событий предшествует времени запуска задания, и события отбрасываются.
    - Условия [**JOIN**](/stream-analytics-query/join-azure-stream-analytics) не имеют совпадений. Если совпадений нет, возвращается нулевой набор результатов.

5.  Убедитесь, что политики упорядочения событий настроены надлежащим образом. Перейдите к разделу **Параметры** и выберите [**Упорядочение событий**](./stream-analytics-time-handling.md). Эта политика *НЕ* применяется при проверке запроса по нажатию кнопки **Тест**. В результате тестирования в браузере и при выполнении задания в рабочей среде отличаются одним компонентом. 

6. Для отладки по журналам действий и ресурсов:
    - в [журнале действий](../azure-resource-manager/management/view-activity-logs.md) отфильтруйте результаты для поиска и диагностики ошибок;
    - примените [журнал ресурсов для заданий](stream-analytics-job-diagnostic-logs.md) для поиска и диагностики ошибок.

## <a name="resource-utilization-is-high"></a>Интенсивное использование ресурсов

Воспользуйтесь преимуществами параллелизма в Azure Stream Analytics. Сведения о масштабировании с использованием заданий параллелизации запросов Stream Analytics с помощью настройки входных разделов и определения запроса аналитики см. в статье [Использование параллелизации запросов в Azure Stream Analytics](stream-analytics-parallelization.md).

Если использование ресурсов постоянно превышает 80%, задержка водяного знака увеличивается, а количество событий, регистрируемых в журнале, увеличивается, рекомендуется увеличить число единиц потоковой передачи. Высокий уровень использования указывает, что задание использует близко к максимальному выделенному ресурсу.

## <a name="debug-queries-progressively"></a>Последовательная отладка запросов

При обработке данных в режиме реального времени важно знать, как выглядят данные во время запроса. Это можно увидеть на схеме заданий в Visual Studio. Если у вас нет Visual Studio, выполните дополнительные действия для вывода промежуточных данных.

Так как входные данные или шаги задания Azure Stream Analytics можно считать несколько раз, вы можете написать дополнительные инструкции SELECT INTO. Таким образом вы получаете промежуточные данные в хранилище, на основе которых можно проверить правильность данных, так же, как и с помощью *переменных просмотра* при отладке программы.

В следующем примере запроса в задании Azure Stream Analytics содержится один поток входных данных, два ссылочных набора входных данных и выходные данные для Хранилища таблиц Azure. Запрос объединяет данные из концентратора событий и два ссылочных больших двоичных объекта, чтобы получить сведения об имени и категории:

![Пример запроса SELECT INTO Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-query1.png)

Обратите внимание, что задание выполняется, но события не создаются в выходных данных. На плитке **Мониторинг**, показанной здесь, можно увидеть, что данные выводятся, но невозможно определить, на каком шаге операции **JOIN** были удалены все события.

![Плитка мониторинга Azure Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-monitor.png)

В этом случае можно добавить несколько дополнительных инструкций SELECT INTO в журнал для регистрации промежуточных результатов операции JOIN и данных, считываемых из входных данных.

В этом примере мы добавили два новых временных набора выходных данных. Они могут представлять собой любой приемник. Ниже в качестве примера используется служба хранилища Azure:

![Добавление дополнительных инструкций SELECT INTO в запрос Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-outputs.png)

Затем можно переписать запрос следующим образом:

![Перезаписанный запрос SELECT INTO Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-query2.png)

Теперь снова запустите задание, которое должно выполняться в течение нескольких минут. Затем запросите temp1 и temp2 с помощью Visual Studio Cloud Explorer для создания следующих таблиц:

**Таблица temp1**
![запрос SELECT INTO Stream Analytics таблицы temp1 ](./media/stream-analytics-select-into/stream-analytics-select-into-temp-table-1.png)

**Таблица temp2**
![запрос SELECT INTO Stream Analytics таблицы temp2](./media/stream-analytics-select-into/stream-analytics-select-into-temp-table-2.png)

Как вы видите, обе таблицы содержат данные, а столбец имени в temp2 заполнен правильно. Тем не менее, так как данные еще не выведены, произошла какая-то проблема:

![Таблица output1 запроса SELECT INTO Stream Analytics без данных](./media/stream-analytics-select-into/stream-analytics-select-into-out-table-1.png)

Выполнив выборку данных, можно почти определенно сказать, что проблема связана со второй операцией JOIN. Вы можете скачать ссылочные данные из большого двоичного объекта и убедиться в этом:

![Запрос SELECT INTO Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-ref-table-1.png)

Как видно, формат GUID в этих справочных данных отличается от формата столбца в таблице temp2. Именно поэтому данные не поступили в output1, как положено.

Вы можете исправить формат данных, передать их в ссылочный большой двоичный объект и повторить попытку:

![Запрос временной таблицы SELECT INTO Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-ref-table-2.png)

На этот раз выходные данные форматируются и заполняются требуемым образом.

![Запрос итоговой таблицы SELECT INTO Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-final-table.png)

## <a name="get-help"></a>Получить справку

Для получения дополнительной помощи воспользуйтесь [страницей вопросов и ответов по Azure Stream Analytics на сайте Майкрософт](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Дальнейшие действия

* [Введение в Azure Stream Analytics](stream-analytics-introduction.md)
* [Приступая к работе с Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Масштабирование заданий в службе Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Справочник по языку запросов Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Справочник по API-интерфейсу REST управления Stream Analytics](/rest/api/streamanalytics/)
