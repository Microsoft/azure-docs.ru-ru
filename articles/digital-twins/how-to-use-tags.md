---
title: Добавление тегов в цифровые двойников
titleSuffix: Azure Digital Twins
description: См. статью как реализовать Теги на цифровой двойников
author: baanders
ms.author: baanders
ms.date: 7/22/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 70bf46de072a97eca810dda60a5331df14172ed6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100555147"
---
# <a name="add-tags-to-digital-twins"></a>Добавление тегов в цифровые двойников 

Вы можете использовать концепцию тегов, чтобы дополнительно найти и классифицировать свой цифровой двойников. В частности, пользователям может потребоваться реплицировать теги из существующих систем, таких как [теги иголка](https://project-haystack.org/doc/TagModel), в свои экземпляры Azure Digital двойников. 

В этом документе описываются шаблоны, которые можно использовать для реализации тегов в цифровом двойников.

Сначала теги добавляются в качестве свойств в [модели](concepts-models.md) , описывающей цифровой двойника. Затем это свойство задается в двойника, когда оно создается на основе модели. После этого Теги можно использовать в [запросах](concepts-query-language.md) для обнаружения и фильтрации двойников.

## <a name="marker-tags"></a>Теги маркера 

**Тег маркера** — это простая строка, используемая для пометки или классификации цифрового двойникаа, например "Blue" или "Red". Эта строка является именем тега, а теги маркера не имеют осмысленного значения — тег является значимым только его присутствием (или отсутствием). 

### <a name="add-marker-tags-to-model"></a>Добавление тегов маркера в модель 

Теги маркера моделируются в виде [дтдл](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) -карт из `string` в `boolean` . Логическое значение `mapValue` игнорируется, так как присутствие тега является важным. 

Ниже приведен фрагмент модели двойника, реализующей тег маркера в качестве свойства:

:::code language="json" source="~/digital-twins-docs-samples/models/tags.json" range="2-16":::

### <a name="add-marker-tags-to-digital-twins"></a>Добавление тегов маркеров в цифровые двойников

Когда `tags` свойство является частью модели цифрового двойника, можно установить тег маркера в цифровом двойника, задав значение этого свойства. 

Ниже приведен пример, который заполняет маркер `tags` для трех двойников:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="TagPropertiesMarker":::

### <a name="query-with-marker-tags"></a>Запрос с тегами маркеров

После добавления тегов в цифровые двойников Теги можно использовать для фильтрации двойников в запросах. 

Ниже приведен запрос для получения всех двойников, помеченных как "Red": 

:::code language="sql" source="~/digital-twins-docs-samples/queries/queries.sql" id="QueryMarkerTags1":::

Можно также комбинировать теги для более сложных запросов. Ниже приведен запрос для получения всех двойников, которые округляются, а не красного цвета. 

:::code language="sql" source="~/digital-twins-docs-samples/queries/queries.sql" id="QueryMarkerTags2":::

## <a name="value-tags"></a>Теги значений 

**Тег значения** — это пара "ключ-значение", которая используется для присвоения каждому тегу значения, например `"color": "blue"` или `"color": "red"` . После создания тега значения его можно также использовать в качестве тега маркера, игнорируя значение тега. 

### <a name="add-value-tags-to-model"></a>Добавить теги значений в модель 

Теги значений моделируются в виде [дтдл](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) -карт из `string` в `string` . И, `mapKey` и `mapValue` являются значимыми. 

Ниже приведен фрагмент модели двойника, реализующей тег value в качестве свойства:

:::code language="json" source="~/digital-twins-docs-samples/models/tags.json" range="17-31":::

### <a name="add-value-tags-to-digital-twins"></a>Добавление тегов значений в Digital двойников

Как и в случае с тегами маркеров, можно задать тег value в цифровом двойника, задав значение этого `tags` свойства из модели. Чтобы использовать тег значения в качестве тега маркера, можно присвоить `tagValue` полю пустое строковое значение ( `""` ). 

Ниже приведен пример, который заполняет значение `tags` для трех двойников:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="TagPropertiesValue":::

Обратите внимание, что `red` и `purple` используются в качестве тегов маркера в этом примере.

### <a name="query-with-value-tags"></a>Запрос с тегами значений

Как и в случае с тегами маркера, можно использовать теги значений для фильтрации двойников в запросах. Можно также использовать теги значений и теги меток вместе.

В приведенном выше примере используется `red` как тег маркера. Помните, что это запрос для получения всех двойников, помеченных как "Red": 

:::code language="sql" source="~/digital-twins-docs-samples/queries/queries.sql" id="QueryMarkerTags1":::

Ниже приведен запрос для получения всех мелких сущностей (тег значения), а не красного: 

:::code language="sql" source="~/digital-twins-docs-samples/queries/queries.sql" id="QueryMarkerValueTags":::

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о проектировании моделей цифровых двойника и управлении ими:
* [*Руководство. Управление моделями ДТДЛ*](how-to-manage-model.md)

Дополнительные сведения о запросах к графу двойника:
* [*Пошаговое руководство. запрос графа двойника*](how-to-query-graph.md)