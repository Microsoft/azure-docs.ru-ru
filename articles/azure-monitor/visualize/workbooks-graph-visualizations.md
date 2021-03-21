---
title: Azure Monitor визуализаций графов книг
description: Сведения о всех Azure Monitor визуализациях графов книг.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: lagayhar
ms.openlocfilehash: a538eaf06013dcce6529c91816b419159a2600a1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100625650"
---
# <a name="graph-visualizations"></a>Визуализации графов

Книги поддерживают визуализацию произвольных графов на основе данных из журналов для отображения отношений между объектами мониторинга.

На приведенном ниже графе показаны данные, передаваемые из компьютера или из него через различные порты на внешние компьютеры. Он окрашивается по типу (компьютер или порт или внешний IP-адрес), а размеры краев соответствуют объему передаваемых данных. Базовые данные поступают из ККЛ запросов, предназначенных для подключений виртуальных машин.

[![Снимок экрана: представление сводки плиток](./media/workbooks-graph-visualizations/graph.png)](./media/workbooks-graph-visualizations/graph.png#lightbox)

## <a name="adding-a-graph"></a>Добавление диаграммы
1. Переключите книгу в режим редактирования, щелкнув элемент панели инструментов _изменить_ .
2. Используйте ссылку **Добавить запрос** , чтобы добавить в книгу элемент управления "запрос журнала".
3. Выберите тип запроса в поле **Журнал**, тип ресурса (например, Application Insights) и ресурсы для целевого объекта.
4. Используйте редактор запросов, чтобы ввести ККЛ для анализа.

    ```kusto
    let data = dependencies
    | summarize Calls = count() by App = appName, Request = operation_Name, Dependency = name
    | extend RequestId = strcat(App, '::', Request);
    let links = data
    | summarize Calls = sum(Calls) by App, RequestId
    | project SourceId = App, TargetId = RequestId, Calls, Kind = 'App -> Request'
    | union (data
        | project SourceId = RequestId, TargetId = Dependency, Calls, Kind = 'Request -> Dependency');
    let nodes = data
    | summarize Calls = sum(Calls) by App
    | project Id = App, Name = App, Calls, Kind = 'App'
    | union (data
        | summarize Calls = sum(Calls) by RequestId, Request
        | project Id = RequestId, Name = Request, Calls, Kind = 'Request')
    | union (data
        | summarize Calls = sum(Calls) by Dependency
        | project Id = Dependency, Name = Dependency, Calls, Kind = 'Dependency');
    nodes
    | union (links)
    ```

5. Установка **графа** визуализации
6. Нажмите кнопку **Параметры диаграммы** , чтобы открыть панель Параметры.
7. В _полях макет_ в нижней части задайте:
    * Идентификатор узла: `Id`
    * Идентификатор источника: `SourceId`
    * Идентификатор целевого объекта: `TargetId`
    * Метка ребра: `None`
    * Размер ребра: `Calls`
    * Размер узла: `None`
    * Тип цвета: `Categorical`
    * Поле цвета узла: `Kind`
    * Цветовая палитра: `Pastel`
8. В верхней части окна _Параметры формата узла_ задайте:
    * _Верхнее содержимое_— столбец использования: `Name` , модуль подготовки столбцов: `Text`
    * _Центрировать содержимое_— используйте столбец:, модуль `Calls` подготовки столбцов: `Big Number` , цветовая палитра: `None`
    * _Содержимое снизу_— используйте столбец: `Kind` , модуль подготовки столбцов: `Text`
9. Нажмите кнопку _сохранить и закрыть_ в нижней части панели.

[![Снимок экрана: представление сводки плиток с указанными выше запросом и параметрами.](./media/workbooks-graph-visualizations/graph-settings.png)](./media/workbooks-graph-visualizations/graph-settings.png#lightbox)

## <a name="graph-settings"></a>Параметры графа

| Параметр         | Объяснение                                                                                                        |
|:----------------|:-------------------------------------------------------------------------------------------------------------------|
| `Node Id`       | Выбирает столбец, предоставляющий уникальный идентификатор узлов на графе. Значение столбца может быть строкой или числом. |
| `Source Id`     | Выбирает столбец, предоставляющий идентификаторы исходных узлов для ребер на графе. Значения должны сопоставляться со значением в столбце _идентификатор узла_ . |
| `Target Id`     | Выбирает столбец, предоставляющий идентификаторы целевых узлов для ребер на графе. Значения должны сопоставляться со значением в столбце _идентификатор узла_ . |
| `Edge Label`    | Выбирает столбец, предоставляющий метки ребра на диаграмме.                                                            |
| `Edge Size`     | Выбирает столбец, предоставляющий метрику, на основе которой будут основываться значения ширины границ.                                |
| `Node Size`     | Выбирает столбец, предоставляющий метрику, на которой будут основываться области узлов.                                 |
| `Coloring Type` | Используется для выбора схемы цветового выделения узла.                                                                            |

## <a name="node-coloring-types"></a>Типы цветов узлов

| Тип цвета | Пояснение |
|:------------- |:------------|
| `None`        | Все узлы имеют одинаковый цвет. |
| `Categorical` | Узлам присваиваются цвета на основе значения или категории из столбца в результирующем наборе. В приведенном выше примере цветовое выделение основано на _типе_ столбца результирующего набора. Поддерживаемые палитры: `Default` , `Pastel` и `Cool tone` .  |
| `Field Based` | В этом типе столбец содержит определенные значения RGB, используемые для узла. Предоставляет наибольшую гибкость, но обычно требует больше работы для включения.  |

## <a name="node-format-settings"></a>Параметры формата узла

Авторы диаграмм могут указать, какое содержимое будет передано в различные части узла: сверху, слева, по центру, справа и снизу. Графы могут использовать любые книги модулей подготовки отчетов (текстовые, большие числа, строки Spark, значки и т. д.).

## <a name="field-based-node-coloring"></a>Цветовое выделение узлов на основе полей

1. Переключите книгу в режим редактирования, щелкнув элемент панели инструментов _изменить_ .
2. Используйте ссылку **Добавить запрос** , чтобы добавить в книгу элемент управления "запрос журнала".
3. Выберите тип запроса в поле **Журнал**, тип ресурса (например, Application Insights) и ресурсы для целевого объекта.
4. Используйте редактор запросов, чтобы ввести ККЛ для анализа.

     ```kusto
    let data = dependencies
    | summarize Calls = count() by App = appName, Request = operation_Name, Dependency = name
    | extend RequestId = strcat(App, '::', Request);
    let links = data
    | summarize Calls = sum(Calls) by App, RequestId
    | project SourceId = App, TargetId = RequestId, Calls, Kind = 'App -> Request'
    | union (data
        | project SourceId = RequestId, TargetId = Dependency, Calls, Kind = 'Request -> Dependency');
    let nodes = data
    | summarize Calls = sum(Calls) by App
    | project Id = App, Name = App, Calls, Color = 'FD7F23'
    | union (data
        | summarize Calls = sum(Calls) by RequestId, Request
        | project Id = RequestId, Name = Request, Calls, Color = 'B3DE8E')
    | union (data
        | summarize Calls = sum(Calls) by Dependency
        | project Id = Dependency, Name = Dependency, Calls, Color = 'C9B3D5');
    nodes
    | union (links)
    ```
5. Установка *графа* визуализации
6. Нажмите кнопку **Graph Settings (параметры диаграммы**  ), чтобы открыть панель Параметры.
7. В *полях макет* в нижней части задайте:
    * Идентификатор узла:`Id`
    * Идентификатор источника: `SourceId`
    * Идентификатор целевого объекта: `TargetId`
    * Метка ребра: `None`
    * Размер ребра: `Calls`
    * Размер узла: `Node`
    * Тип цвета: `Field Based`
    * Поле цвета узла: `Color`
8. В верхней части окна *Параметры формата узла* введите следующее.
    * В *верхнем содержимом* задайте:
        * Использовать столбец: `Name` .
        * Модуль подготовки столбцов: `Text` .
    * В *центре содержимого* задайте:
        * Использовать столбец: `Calls`
        * Модуль подготовки столбцов: `Big Number`
        * Цветовая палитра: `None`
    * В *нижнем содержимом* задайте:
        * Использовать столбец: `Kind`
        * Модуль подготовки столбцов: `Text` .
9. Нажмите **кнопку Сохранить и закрыть** в нижней части панели.

[![Снимок экрана, показывающий создание визуализации графа с цветом базового узла поля.](./media/workbooks-graph-visualizations/graph-field-based.png)](./media/workbooks-graph-visualizations/graph-field-based.png#lightbox)

## <a name="next-steps"></a>Дальнейшие действия

* Графы также поддерживают рендеринг составных линейчатых диаграмм. Дополнительные сведения см. в [документации по составной панели](workbooks-composite-bar.md).
* Дополнительные сведения об [источниках данных](workbooks-data-sources.md) , которые можно использовать в книгах.
