---
title: Преобразование онтологий
titleSuffix: Azure Digital Twins
description: Изучите процесс преобразования стандартных отраслевых моделей в ДТДЛ для Azure Digital двойников
author: baanders
ms.author: baanders
ms.date: 2/12/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: aa4dde51c077152dd5c8a938ad64ad0a051f89ad
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100561750"
---
# <a name="convert-industry-standard-ontologies-to-dtdl-for-azure-digital-twins"></a>Преобразование стандартных отраслевых онтологиес в ДТДЛ для цифрового двойников Azure

Большинство [онтологиес](concepts-ontologies.md) основаны на семантических веб-стандартах, таких как [OWL](https://www.w3.org/OWL/), [RDF](https://www.w3.org/2001/sw/wiki/RDF)и [рдфс](https://www.w3.org/2001/sw/wiki/RDFS). 

Чтобы использовать модель с цифровым двойников Azure, она должна быть в формате ДТДЛ. В этой статье описываются общие рекомендации по проектированию в виде **шаблона преобразования** для преобразования моделей на основе RDF в дтдл, чтобы их можно было использовать с цифровым двойников Azure. 

В статье также содержится [**пример кода преобразователя**](#converter-samples) для преобразователей RDF и OWL, который можно расширить для других схем в здании отрасли.

## <a name="conversion-pattern"></a>Шаблон преобразования

Существует несколько сторонних библиотек, которые можно использовать при преобразовании моделей на основе RDF в ДТДЛ. Некоторые из этих библиотек позволяют загрузить файл модели в граф. Вы можете выполнить цикл по графу для поиска конкретных конструкций РДФС и OWL и преобразовать их в ДТДЛ.   

В следующей таблице показано, как можно сопоставить конструкции РДФС и OWL с ДТДЛ. 

| Концепция РДФС и OWL | Конструкция РДФС/OWL | Концепция ДТДЛ | Конструкция ДТДЛ |
| --- | --- | --- | --- |
| Классы | `owl:Class`<br>Суффикс IRI<br>``rdfs:label``<br>``rdfs:comment`` | Интерфейс | `@type:Interface`<br>`@id`<br>`displayName`<br>`comment` 
| используются подклассы ; | `owl:Class`<br>Суффикс IRI<br>`rdfs:label`<br>`rdfs:comment`<br>`rdfs:subClassOf` | Интерфейс | `@type:Interface`<br>`@id`<br>`displayName`<br>`comment`<br>`extends` 
| Свойства DataType | `owl:DatatypeProperty`<br>`rdfs:label` или `INode`<br>`rdfs:label`<br>`rdfs:range` | Свойства интерфейса | `@type:Property`<br>`name`<br>`displayName`<br>`schema` 
| Свойства объекта | `owl:ObjectProperty`<br>`rdfs:label` или `INode`<br>`rdfs:range`<br>`rdfs:comment`<br>`rdfs:label` | Связь | `type:Relationship`<br>`name`<br>`target` (или пропущено, если нет `rdfs:range` )<br>`comment`<br>`displayName`<br>

В следующем фрагменте кода C# показано, как файл модели RDF загружается в граф и преобразуется в ДТДЛ с помощью библиотеки [**дотнетрдф**](https://www.dotnetrdf.org/) . 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/convertRDF.cs":::

## <a name="converter-samples"></a>Примеры конвертеров

### <a name="rdf-converter-application"></a>Приложение преобразователя RDF 

Существует пример приложения, которое преобразует файл модели на основе RDF в [дтдл (версия 2)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) для использования службой Digital двойников Azure. Она была проверена для схемы [модуля](https://brickschema.org/ontology/) и может быть расширена для других схем в здании отрасли (например, для [создания ТОПОЛОГИИ онтологи (Bot)](https://w3c-lbd-cg.github.io/bot/), [сети семантического датчика](https://www.w3.org/TR/vocab-ssn/)или [классов буилдингсмарт Industry Foundation (ИФК)](https://technical.buildingsmart.org/standards/ifc/ifc-schema-specifications/)).

Пример — это приложение командной строки .NET Core с именем **рдфтодтдлконвертер**.

Пример можно получить здесь: [**рдфтодтдлконвертер**](/samples/azure-samples/rdftodtdlconverter/digital-twins-model-conversion-samples/). 

Чтобы загрузить код на компьютер, нажмите кнопку *скачать ZIP-файл* под заголовком на целевой странице примера. Будет загружен *ZIP* -файл с именем *RdfToDtdlConverter_sample_application_to_convert_RDF_to_DTDL.zip*, который затем можно распаковать и исследовать.

Этот пример можно использовать для просмотра шаблонов преобразования в контексте и для использования в качестве стандартного блока для собственных приложений, выполняющих преобразования модели в соответствии с конкретными потребностями.

### <a name="owl2dtdl-converter"></a>Преобразователь OWL2DTDL 

[**Преобразователь OWL2DTDL**](https://github.com/Azure/opendigitaltwins-building-tools/tree/master/OWL2DTDL) — это пример, который преобразует OWL онтологи в набор объявлений интерфейса дтдл, который можно использовать в службе Digital двойников. Она также работает для онтологи сетей, сделанных одним корневым онтологи, повторно используя другие онтологиес с помощью `owl:imports` объявлений.

Этот преобразователь использовался для преобразования [ядра онтологи для реальных площадок](https://doc.realestatecore.io/3.1/full.html) в дтдл и может использоваться для любых онтологи на основе OWL.

## <a name="next-steps"></a>Дальнейшие действия 

* Узнайте больше о расширении отраслевых онтологиес в соответствии со своими спецификациями: [*Основные понятия: расширение онтологиес отрасли*](concepts-ontologies-extend.md).

* Или перейдите по пути для разработки моделей на основе онтологиес: [*использование стратегий онтологи в пути разработки модели*](concepts-ontologies.md#using-ontology-strategies-in-a-model-development-path).