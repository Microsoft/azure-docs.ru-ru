---
title: Преобразование JSON и XML с шаблонами жидкостей
description: Преобразование JSON и XML с помощью шаблонов жидкостей в виде карт в Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, daviburg, logicappspm
ms.topic: article
ms.date: 07/31/2020
ms.openlocfilehash: 0362c9ed4f736474dbd49e1bfaf1373e0f48acd6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94992715"
---
# <a name="transform-json-and-xml-using-liquid-templates-as-maps-in-azure-logic-apps"></a>Преобразование JSON и XML с помощью шаблонов жидкостей в виде карт в Azure Logic Apps

Если вы хотите выполнять базовые преобразования JSON в приложениях логики, можно использовать собственные [операции с данными](../logic-apps/logic-apps-perform-data-operations.md) , такие как **Создание** или **анализ JSON**. Для расширенных и сложных преобразований JSON в JSON, имеющих такие элементы, как итерации, потоки управления и переменные, создание и использование шаблонов, описывающих эти преобразования, с использованием языка шаблона с открытым исходным кодом [жидкости](https://shopify.github.io/liquid/) . Можно также [выполнить другие преобразования](#other-transformations), например JSON в текст, XML в JSON и XML, в текст.

Прежде чем можно будет выполнить преобразование «ликвидность» в приложении логики, необходимо сначала создать шаблон жидкости, определяющий нужное сопоставление. Затем вы [отправляете шаблон как карту](../logic-apps/logic-apps-enterprise-integration-maps.md) в [учетную запись интеграции](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md). При добавлении в приложение логики действия **преобразование JSON в формат JSON-жидкость** можно выбрать шаблон ликвидности в качестве схемы для использования действия.

В этой статье показано, как выполнить следующие задачи:

* Создайте шаблон ликвидности.
* Добавьте шаблон в учетную запись интеграции.
* Добавьте действие преобразование ликвидности в приложение логики.
* Выберите шаблон в качестве схемы, которую вы хотите использовать.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас нет ее, вы можете [зарегистрироваться для получения бесплатной учетной записи Azure](https://azure.microsoft.com/free/).

* Базовые знания [создания приложений логики](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* [Учетная запись интеграции](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)

* Основные сведения о [языке шаблона жидкости](https://shopify.github.io/liquid/)

  > [!NOTE]
  > Действие **преобразование JSON в JSON-жидкость** следует за [реализацией DotLiquid для жидкостьа](https://github.com/dotliquid/dotliquid), которая отличается в конкретных случаях от [реализации Shopify для жидкостей](https://shopify.github.io/liquid). Дополнительные сведения см. в разделе [рекомендации по шаблонам жидкости](#liquid-template-considerations).

## <a name="create-the-template"></a>Создание шаблона

1. Создайте шаблон жидкости, который будет использоваться в качестве Map для преобразования JSON. Вы можете использовать любое нужное средство редактирования.

   В этом примере создайте шаблон ликвидности, как описано в этом разделе:

   ```json
   {%- assign deviceList = content.devices | Split: ', ' -%}

   {
      "fullName": "{{content.firstName | Append: ' ' | Append: content.lastName}}",
      "firstNameUpperCase": "{{content.firstName | Upcase}}",
      "phoneAreaCode": "{{content.phone | Slice: 1, 3}}",
      "devices" : [
         {%- for device in deviceList -%}
            {%- if forloop.Last == true -%}
            "{{device}}"
            {%- else -%}
            "{{device}}",
            {%- endif -%}
         {%- endfor -%}
      ]
   }
   ```

1. Сохраните шаблон с помощью `.liquid` расширения. В этом примере используется `SimpleJsonToJsonTemplate.liquid`.

## <a name="upload-the-template"></a>Отправка шаблона

1. Войдите на [портал Azure](https://portal.azure.com) с помощью учетных данных учетной записи Azure.

1. В поле поиска портал Azure введите `integration accounts` и выберите **учетные записи интеграции**.

   ![Найти "учетные записи интеграции"](./media/logic-apps-enterprise-integration-liquid-transform/find-integration-accounts.png)

1. Найдите и выберите учетную запись интеграции.

   ![Выбор учетной записи интеграции](./media/logic-apps-enterprise-integration-liquid-transform/select-integration-account.png)

1. В области **Обзор** в разделе **компоненты** выберите **Maps**.

    ![Выберите плитку "карты"](./media/logic-apps-enterprise-integration-liquid-transform/select-maps-tile.png)

1. На панели " **карты** " выберите **Добавить** и укажите следующие сведения для карты:

   | Свойство. | Значение | Описание |
   |----------|-------|-------------|
   | **имя**; | `JsonToJsonTemplate` | Это имя сопоставления, в нашем примере это "JsontoJsonTemplate" |
   | **Тип схемы** | **liquid** | Обозначает тип сопоставления. Для преобразования JSON в JSON следует выбрать **Liquid**. |
   | **Схема** | `SimpleJsonToJsonTemplate.liquid` | Это существующий файл шаблона или сопоставления Liquid, который будет использован для преобразования. В нашем примере это SimpleJsonToJsonTemplate.liquid. Чтобы найти этот файл, используйте средство выбора файлов. Ограничения размера карт см. в разделе [ограничения и конфигурация](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits). |
   |||

   ![Добавить шаблон ликвидности](./media/logic-apps-enterprise-integration-liquid-transform/add-liquid-template.png)

## <a name="add-the-liquid-transformation-action"></a>Добавление действия преобразования «ликвидность»

1. Чтобы [создать пустое приложение логики](../logic-apps/quickstart-create-first-logic-app-workflow.md) на портале Azure, сделайте следующее:

1. В конструкторе приложений логики добавьте [триггер запроса](../connectors/connectors-native-reqres.md#add-request) в приложение логики.

1. В разделе триггера выберите **Добавить шаг**. В поле поиска введите `liquid` Фильтр и выберите это действие: **преобразовать JSON в JSON-жидкость**

   ![Поиск и выбор действия Liquid](./media/logic-apps-enterprise-integration-liquid-transform/search-action-liquid.png)

1. Откройте список **карт** и выберите шаблон ликвидности, который в этом примере — «JsonToJsonTemplate».

   ![Выбор сопоставления](./media/logic-apps-enterprise-integration-liquid-transform/select-map.png)

   Если список Maps пуст, скорее всего, приложение логики не связано с учетной записью интеграции. 
   Чтобы настроить связь с учетной записью интеграции, в которой хранится шаблон или сопоставление Liquid, выполните следующие действия.

   1. В меню приложения логики выберите **Параметры рабочего процесса**.

   1. В списке **выберите учетную запись интеграции** выберите учетную запись интеграции и нажмите кнопку **сохранить**.

      ![Привязка приложения логики к учетной записи интеграции](./media/logic-apps-enterprise-integration-liquid-transform/link-integration-account.png)

1. Теперь добавьте свойство **Content** в это действие. Откройте список **Добавить новый параметр** и выберите **содержимое**.

   ![Добавление свойства "Content" в действие](./media/logic-apps-enterprise-integration-liquid-transform/add-content-property-to-action.png)

1. Чтобы задать значение свойства **содержимого** , щелкните внутри поля **содержимого** , чтобы появился список динамического содержимого. Выберите маркер **тела** , который представляет выходные данные основного содержимого триггера.

   ![Выберите маркер "Body" для значения свойства "Content"](./media/logic-apps-enterprise-integration-liquid-transform/select-body.png)

   В итоге действие должно выглядеть приблизительно так, как показано в примере ниже:

   ![Действие "преобразование JSON в JSON" завершено](./media/logic-apps-enterprise-integration-liquid-transform/finished-transform-action.png)

## <a name="test-your-logic-app"></a>Тестирование приложения логики

Используя [POST](https://www.getpostman.com/postman) или аналогичное средство, опубликуйте входные данные JSON в приложении логики. Из приложения логики вы получите преобразованные выходные данные JSON примерно такого вида:

![Пример выходных данных](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontojson.png)

<a name="template-considerations"></a>

## <a name="liquid-template-considerations"></a>Рекомендации по шаблонам ликвидности

* Шаблоны ликвидности соответствуют [предельным размерам файлов для карт](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits) в Azure Logic Apps.

* Действие **преобразование JSON в JSON-жидкость** следует за [реализацией DotLiquid для жидкости](https://github.com/dotliquid/dotliquid). Эта реализация является портом платформа .NET Framework из [реализации Shopify для жидкостей](https://shopify.github.io/liquid/) и отличается в [конкретных случаях](https://github.com/dotliquid/dotliquid/issues).

  Ниже приведены известные отличия.

  * Действие **преобразование JSON в JSON-жидкость** в собственном коде выводит строку, которая может включать в себя JSON, XML, HTML и т. д. Действие жидкость указывает, что ожидаемый текстовый вывод из шаблона ликвидности — это строка JSON. Это действие указывает приложению логики выполнить синтаксический анализ входных данных в виде объекта JSON и применяет оболочку, чтобы жидкость могла интерпретировать структуру JSON. После преобразования действие указывает приложению логики выполнить синтаксический анализ текстового вывода из жидкости обратно в JSON.

    DotLiquid не имеет встроенного понимания JSON, поэтому убедитесь, что обратная косая черта ( `\` ) и все другие зарезервированные символы JSON являются escape-символом.

  * Если шаблон использует [фильтры жидкостей](https://shopify.github.io/liquid/basics/introduction/#filters), убедитесь, что соблюдены [соглашения об именовании DotLiquid и C#](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers#filter-and-output-casing), в которых используется *регистр предложений*. Для всех преобразований жидкость убедитесь, что имена фильтров в шаблоне также используют регистр предложений. В противном случае фильтры не будут работать.

    Например, при использовании `replace` фильтра используйте `Replace` , а не `replace` . То же правило применяется при попытке испытать примеры в [DotLiquid Online](http://dotliquidmarkup.org/try-online). Дополнительные сведения см. в разделе [Shopify жидкость Filters](https://shopify.dev/docs/themes/liquid/reference/filters) and [DotLiquid жидкость Filters](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Developers#create-your-own-filters). Спецификация Shopify включает примеры для каждого фильтра, поэтому для сравнения можно попробовать эти примеры в [DotLiquid-Try в Интернете](http://dotliquidmarkup.org/try-online).

  * `json`Фильтр из фильтров расширений Shopify в настоящее время [не реализован в DotLiquid](https://github.com/dotliquid/dotliquid/issues/384). Как правило, этот фильтр можно использовать для подготовки текстового вывода для синтаксического анализа строк JSON, но вместо этого необходимо использовать `Replace` фильтр.

  * Стандартный `Replace` фильтр в [реализации DotLiquid](https://github.com/dotliquid/dotliquid/blob/b6a7d992bf47e7d7dcec36fb402f2e0d70819388/src/DotLiquid/StandardFilters.cs#L425) использует [сопоставление регулярных выражений (Regex)](/dotnet/standard/base-types/regular-expression-language-quick-reference), а [Реализация Shopify](https://shopify.github.io/liquid/filters/replace/) использует [простое сопоставление строк](https://github.com/Shopify/liquid/issues/202). Обе реализации работают одинаково, пока не будет использоваться зарезервированный символ регулярного выражения или escape-символ в параметре Match.

    Например, чтобы экранировать escape-символ обратной косой черты ( `\` ), используйте `| Replace: '\\', '\\'` , а не `| Replace: '\', '\\'` . В этих примерах показано, как `Replace` фильтр ведет себя по-разному при попытке экранирования символа обратной косой черты. Хотя эта версия успешно работает:

    `{ "SampleText": "{{ 'The quick brown fox "jumped" over the sleeping dog\\' | Replace: '\\', '\\' | Replace: '"', '\"'}}"}`

    С этим результатом:

    `{ "SampleText": "The quick brown fox \"jumped\" over the sleeping dog\\\\"}`

    Эта версия завершается ошибкой:

    `{ "SampleText": "{{ 'The quick brown fox "jumped" over the sleeping dog\\' | Replace: '\', '\\' | Replace: '"', '\"'}}"}`

    С этой ошибкой:

    `{ "SampleText": "Liquid error: parsing "\" - Illegal \ at end of pattern."}`

    Дополнительные сведения см. в разделе [Замена стандартного фильтра с использованием сопоставления шаблона регулярного выражения...](https://github.com/dotliquid/dotliquid/issues/385).

  * `Sort`Фильтр в [реализации DotLiquid](https://github.com/dotliquid/dotliquid/blob/b6a7d992bf47e7d7dcec36fb402f2e0d70819388/src/DotLiquid/StandardFilters.cs#L326) сортирует элементы в массиве или коллекции по свойству, но с этими отличиями:<p>

    * Соответствует поведению [Sort_natural Shopify](https://shopify.github.io/liquid/filters/sort_natural/), а не правилам [сортировки Shopify](https://shopify.github.io/liquid/filters/sort/).

    * Выполняет сортировку только в порядке строкового алфавитно-цифрового порядка. Дополнительные сведения см. в разделе [числовая сортировка](https://github.com/Shopify/liquid/issues/980).

    * Использует порядок без *учета регистра* , а не порядок учета регистра. Дополнительные сведения см. [в разделе Фильтр сортировки не соответствует поведению регистра в спецификации Shopify]( https://github.com/dotliquid/dotliquid/issues/393).

<a name="other-transformations"></a>

## <a name="other-transformations-using-liquid"></a>Другие преобразования с использованием жидкостей

Жидкость не ограничена только преобразованиями JSON. Кроме того, жидкость можно использовать для выполнения других преобразований, например:

* [JSON в текст](#json-text)
* [XML в JSON](#xml-json)
* [XML в текст](#xml-text)

<a name="json-text"></a>

### <a name="transform-json-to-text"></a>Преобразование JSON в текст.

Вот шаблон ликвидности, который используется в этом примере:

```json
{{content.firstName | Append: ' ' | Append: content.lastName}}
```

Ниже приведены примеры входных и выходных данных.

![Пример выходных данных преобразования JSON в текст](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontotext.png)

<a name="xml-json"></a>

### <a name="transform-xml-to-json"></a>Преобразование XML в JSON.

Вот шаблон ликвидности, который используется в этом примере:

``` json
[{% JSONArrayFor item in content -%}
      {{item}}
  {% endJSONArrayFor -%}]
```

`JSONArrayFor`Цикл является настраиваемым механизмом циклов для входных данных XML, что позволяет создавать полезные данные JSON, что позволяет избежать завершающей запятой. Кроме того, `where` условие для этого настраиваемого механизма циклического цикла использует имя XML-элемента для сравнения, а не значение элемента, как другие фильтры жидкости. Дополнительные сведения см. [в разделе детальный обзор политики Set-Body. коллекции объектов](https://azure.microsoft.com/blog/deep-dive-on-set-body-policy).

Ниже приведены примеры входных и выходных данных.

![Пример выходных данных преобразования XML в JSON](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltojson.png)

<a name="xml-text"></a>

### <a name="transform-xml-to-text"></a>Преобразование XML в текст.

Вот шаблон ликвидности, который используется в этом примере:

``` json
{{content.firstName | Append: ' ' | Append: content.lastName}}
```

Ниже приведены примеры входных и выходных данных.

![Пример выходных данных преобразования XML в текст](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltotext.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Язык и примеры Shopify жидкости](https://shopify.github.io/liquid/basics/introduction/)
* [DotLiquid](http://dotliquidmarkup.org/)
* [DotLiquid — попробуйте в Интернете](http://dotliquidmarkup.org/try-online)
* [DotLiquid GitHub](https://github.com/dotliquid/dotliquid)
* [Проблемы GitHub DotLiquid](https://github.com/dotliquid/dotliquid/issues/)
* Дополнительные сведения о [картах](../logic-apps/logic-apps-enterprise-integration-maps.md)
