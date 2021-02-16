---
title: Управление графом двойника со связями
titleSuffix: Azure Digital Twins
description: Узнайте, как управлять графом цифровых двойников, подключив их к связям.
author: baanders
ms.author: baanders
ms.date: 11/03/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: b713a19cbe572998bb6e5050ab2d7721a844f07a
ms.sourcegitcommit: 7ec45b7325e36debadb960bae4cf33164176bc24
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/16/2021
ms.locfileid: "100530456"
---
# <a name="manage-a-graph-of-digital-twins-using-relationships"></a>Управление графиком цифровых двойников с помощью связей

Основой Azure Digital двойников является [двойника граф](concepts-twins-graph.md) , представляющий всю среду. Граф двойника состоит из отдельных цифровых двойников, подключенных через **связи**. 

Получив рабочий [экземпляр Azure Digital двойников](how-to-set-up-instance-portal.md) и настроив код [проверки подлинности](how-to-authenticate-client.md) в клиентском приложении, вы можете использовать [**API дигиталтвинс**](/rest/api/digital-twins/dataplane/twins) для создания, изменения и удаления цифровых двойников и их отношений в экземпляре Azure Digital двойников. Вы также можете использовать [пакет SDK для .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true)или [Azure Digital двойников CLI](how-to-use-cli.md).

Эта статья посвящена управлению связями и графу в целом. для работы с отдельными цифровыми двойниковми см. раздел [*как управлять цифровыми двойников*](how-to-manage-twin.md).

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

## <a name="ways-to-manage-graph"></a>Способы управления Graph

[!INCLUDE [digital-twins-ways-to-manage.md](../../includes/digital-twins-ways-to-manage.md)]

Вы также можете вносить изменения в граф с помощью примера Azure Digital двойников (ADT) Explorer, который позволяет визуализировать двойников и Graph, а также использовать пакет SDK в фоновом режиме. В следующем разделе подробно описывается этот пример.

[!INCLUDE [visualizing with Azure Digital Twins explorer](../../includes/digital-twins-visualization.md)]

## <a name="create-relationships"></a>Создавать связи

Связи описывают, как разные цифровые двойников соединяются друг с другом, который образует базу двойника графа.

Связи создаются с помощью `CreateOrReplaceRelationshipAsync()` вызова. 

Чтобы создать связь, необходимо указать:
* Идентификатор источника двойника ( `srcId` в образце кода ниже): идентификатор двойника, в которой создается связь.
* Целевой идентификатор двойника ( `targetId` в примере кода ниже): идентификатор двойника, в которой получено отношение.
* Имя связи (в приведенном `relName` ниже примере кода): универсальный тип связи, что подобно _Contains_.
* Идентификатор отношения (в приведенном `relId` ниже примере кода): конкретное имя для этой связи, что вроде _Relationship1_.

ИДЕНТИФИКАТОР связи должен быть уникальным в пределах заданного исходного двойника. Оно не обязательно должно быть глобально уникальным.
Например, для двойника *foo* каждый конкретный идентификатор связи должен быть уникальным. Однако другая двойниканая *черта* может иметь исходящее отношение, совпадающее с идентификатором связи *foo* .

В следующем примере кода показано, как создать связь в вашем экземпляре Digital двойников для Azure. Он использует вызов пакета SDK (выделенный) в пользовательском методе, который может присутствовать в контексте более крупной программы.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="CreateRelationshipMethod" highlight="13":::

Теперь эту пользовательскую функцию можно вызвать для создания связи, которая _содержит_ такой объект: 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseCreateRelationship":::

Если вы хотите создать несколько связей, можно повторить вызовы одного и того же метода, передав разные типы отношений в аргумент. 

Дополнительные сведения о вспомогательном классе `BasicRelationship` см. в разделе Практические руководства. [*Использование интерфейсов API и пакетов SDK для цифровых двойников Azure*](how-to-use-apis-sdks.md#serialization-helpers).

### <a name="create-multiple-relationships-between-twins"></a>Создание нескольких связей между двойников

Отношения можно классифицировать следующим образом: 

* Исходящие связи: отношения, принадлежащие этой двойника, которые, в своюмся, должны соединяться с другими двойников. `GetRelationshipsAsync()`Метод используется для получения исходящих отношений двойника.
* Входящие отношения: отношения, принадлежащие другим двойников, которые указывают на этот двойника для создания ссылки "входящий". `GetIncomingRelationshipsAsync()`Метод используется для получения входящих отношений двойника.

На число связей, которые можно использовать между двумя двойниковами, не накладывается никаких ограничений. Вы можете иметь любое количество отношений между двойников. 

Это означает, что можно выразить несколько различных типов отношений между двумя двойников одновременно. Например, *двойника A* может иметь как *хранимую* связь, так и *производимую* связь с *двойника B*.

При необходимости можно даже создать несколько экземпляров одного типа связи между одними и теми же двумя двойников. В этом примере *двойника A* может иметь два разных *хранимых* отношения с *двойника B*, если связи имеют разные идентификаторы отношений.

## <a name="list-relationships"></a>Список связей

Чтобы получить доступ к списку **исходящих** связей для заданного двойника в графе, можно использовать `GetRelationships()` метод следующим образом:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="GetRelationshipsCall":::

Он возвращает `Azure.Pageable<T>` или `Azure.AsyncPageable<T>` , в зависимости от того, используется ли синхронная или асинхронная версия вызова.

Ниже приведен пример, в котором извлекается список связей. Он использует вызов пакета SDK (выделенный) в пользовательском методе, который может присутствовать в контексте более крупной программы.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FindOutgoingRelationshipsMethod" highlight="8":::

Теперь можно вызвать этот пользовательский метод, чтобы увидеть исходящие отношения двойников следующим образом:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFindOutgoingRelationships":::

Полученные связи можно использовать для перехода к другим двойников в графе. Для этого прочтите `target` поле из возвращаемой связи и используйте его в качестве идентификатора для следующего вызова `GetDigitalTwin()` .

### <a name="find-incoming-relationships-to-a-digital-twin"></a>Поиск входящих отношений для цифрового двойника

В Azure Digital двойников также есть API для поиска всех **входящих** отношений с заданным двойника. Это часто полезно для обратных переходов или при удалении двойника.

>[!NOTE]
> `IncomingRelationship` вызовы не возвращают полный текст связи. Дополнительные сведения о `IncomingRelationship` классе см. в его [справочной документации](/dotnet/api/azure.digitaltwins.core.incomingrelationship?view=azure-dotnet&preserve-view=true).

Пример кода в предыдущем разделе посвящен поиску исходящих связей от двойника. Следующий пример структурирован аналогично, но находит *Входящие* связи с двойника. В этом примере также используется вызов пакета SDK (выделено) внутри пользовательского метода, который может появиться в контексте более крупной программы.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FindIncomingRelationshipsMethod" highlight="8":::

Теперь можно вызвать этот пользовательский метод, чтобы увидеть входящие отношения двойников следующим образом:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFindIncomingRelationships":::

### <a name="list-all-twin-properties-and-relationships"></a>Вывод списка всех свойств и отношений двойника

Используя описанные выше методы для перечисления входящих и исходящих отношений с двойника, можно создать метод, который выводит полные сведения двойника, включая свойства двойника и оба типа связей. Ниже приведен пример пользовательского метода, который показывает, как объединить приведенные выше пользовательские методы для этой цели.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FetchAndPrintMethod":::

Теперь можно вызвать эту пользовательскую функцию следующим образом: 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFetchAndPrint":::

## <a name="update-relationships"></a>Обновить связи

Связи обновляются с помощью `UpdateRelationship` метода. 

>[!NOTE]
>Этот метод предназначен для обновления **свойств** связи. Если необходимо изменить исходный двойника или целевой двойника связи, необходимо [Удалить связь](#delete-relationships) и [создать ее повторно](#create-relationships) с помощью нового двойников.

Обязательные параметры для клиентского вызова — это идентификатор источника двойника (двойника, где исходит связь), идентификатор связи для обновления и документ [исправления JSON](http://jsonpatch.com/) , содержащий свойства и новые значения, которые необходимо обновить.

Ниже приведен пример кода, демонстрирующий использование этого метода. В этом примере используется вызов пакета SDK (выделено) внутри пользовательского метода, который может появиться в контексте более крупной программы.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UpdateRelationshipMethod" highlight="6":::

Ниже приведен пример вызова этого пользовательского метода, который передает документ исправления JSON со сведениями для обновления свойства.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseUpdateRelationship":::

## <a name="delete-relationships"></a>Удаление связей

Первый параметр указывает исходный двойника (двойника, где исходит связь). Другой параметр — идентификатор связи. Вам потребуется идентификатор двойника и идентификатор связи, так как идентификаторы связей уникальны только в пределах области двойника.

Ниже приведен пример кода, демонстрирующий использование этого метода. В этом примере используется вызов пакета SDK (выделено) внутри пользовательского метода, который может появиться в контексте более крупной программы.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="DeleteRelationshipMethod" highlight="5":::

Теперь можно вызвать этот пользовательский метод, чтобы удалить связь следующим образом:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseDeleteRelationship":::

## <a name="runnable-twin-graph-sample"></a>Пример готового графа двойника

Следующий готовый к запуску фрагмент кода использует операции связи из этой статьи для создания графа двойника из цифровых двойников и связей.

### <a name="set-up-the-runnable-sample"></a>Настройка готового к запуску примера

В фрагменте кода используется [*Room.jsна*](https://github.com/Azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Room.json) определениях моделей и [*Floor.js*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) из [*учебника: изучение Azure Digital двойников с примером клиентского приложения*](tutorial-command-line-app.md). Вы можете использовать эти ссылки, чтобы перейти непосредственно к файлам или загрузить их как часть полного [примера проекта.](/samples/azure-samples/digital-twins-samples/digital-twins-samples/) 

Перед запуском образца выполните следующие действия.
1. Скачайте файлы модели, поместите их в свой проект и замените `<path-to>` заполнители в приведенном ниже коде, чтобы сообщить программе, где их можно найти.
2. Замените заполнитель `<your-instance-hostname>` именем узла своего экземпляра цифрового двойников Azure.
3. Добавьте в проект две зависимости, которые понадобятся для работы с Azure Digital двойников. Первая — это пакет для [пакета SDK Azure Digital Twins для .NET](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true), вторая содержит средства для проверки подлинности в Azure.

      ```cmd/sh
      dotnet add package Azure.DigitalTwins.Core
      dotnet add package Azure.Identity
      ```

Кроме того, необходимо настроить локальные учетные данные, если вы хотите выполнить пример напрямую. В следующем разделе приведено пошаговое руководство.
[!INCLUDE [Azure Digital Twins: local credentials prereq (outer)](../../includes/digital-twins-local-credentials-outer.md)]

### <a name="run-the-sample"></a>Запуск примера

После выполнения описанных выше действий можно запустить следующий пример кода.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs":::

Ниже приведены выходные данные консоли для приведенной выше программы. 

:::image type="content" source="./media/how-to-manage-graph/console-output-twin-graph.png" alt-text="Выходные данные консоли, отображающие сведения о двойника, входящие и исходящие связи двойников." lightbox="./media/how-to-manage-graph/console-output-twin-graph.png":::

> [!TIP]
> Граф двойника является концепцией создания связей между двойников. Если вы хотите просмотреть визуальное представление графа двойника, см. раздел [*визуализация*](how-to-manage-graph.md#visualization) в этой статье. 

## <a name="create-graph-from-a-csv-file"></a>Создание графа из CSV-файла

В практических случаях двойника иерархии часто создаются на основе данных, хранящихся в другой базе данных или в виде CSV-файла. В этом разделе показано, как считать данные из CSV-файла и создать граф двойника из него.

Рассмотрим следующую таблицу данных, описывающую набор цифровых двойников и связей.

|  Идентификатор модели    | Идентификатор двойника (должен быть уникальным) | Имя связи  | Идентификатор целевого двойника  | Данные инициализации двойника |
| --- | --- | --- | --- | --- |
| дтми: пример: Floor; 1    | Floor1 | содержит | Room1 | |
| дтми: пример: Floor; 1    | Floor0 | содержит | Room0 | |
| дтми: пример: комната; 1    | Room1 | | | {"Температура": 80} |
| дтми: пример: комната; 1    | Room0 | | | {"Температура": 70} |

Одним из способов получения этих данных в Azure Digital двойников является преобразование таблицы в CSV-файл и написание кода для интерпретации файла в команды для создания двойников и связей. В следующем примере кода показано чтение данных из CSV-файла и создание графа двойника в Azure Digital двойников.

В приведенном ниже коде CSV-файл называется *data.csv*, а также есть заполнитель, представляющий **имя узла** для своего экземпляра Digital двойников Azure. В примере также используется несколько пакетов, которые можно добавить в проект, чтобы помочь в этом процессе.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graphFromCSV.cs":::

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о запросах к графу двойников для Azure Digital двойника:
* [*Основные понятия: язык запросов*](concepts-query-language.md)
* [*Пошаговое руководство. запрос графа двойника*](how-to-query-graph.md)