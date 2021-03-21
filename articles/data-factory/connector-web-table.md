---
title: Копирование данных из веб-таблицы с помощью фабрики данных Azure
description: Сведения о соединителе Веб-таблицы фабрики данных Azure, который позволяет копировать данные из Веб-таблицы в хранилища данных, поддерживаемые фабрикой данных в качестве приемников.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 08/01/2019
ms.author: jingwang
ms.openlocfilehash: 1ab45868afd8d1b7ba1f61f5eaacca283817e6d6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100367016"
---
# <a name="copy-data-from-web-table-by-using-azure-data-factory"></a>Копирование данных из Веб-таблицы с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-web-table-connector.md)
> * [Текущая версия](connector-web-table.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных веб-таблиц. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

Далее приводятся различия между соединителем веб-таблиц, [соединителем REST](connector-rest.md) и [соединителем HTTP](connector-http.md).

- **Соединитель веб-таблиц** извлекает содержимое таблицы со страницы HTML.
- **Соединитель RESTful** специально поддерживает копирование данных из интерфейсов API RESTful.
- **Соединитель HTTP** применяется для извлечения данных из любой конечной точки HTTP, например для скачивания файла. 

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель веб-таблиц поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Вы можете скопировать данные из базы данных веб-таблиц в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

Сейчас этот соединитель веб-таблиц поддерживает только **извлечение содержимого таблицы из HTML-страницы**.

## <a name="prerequisites"></a>Предварительные условия

Для использования этого соединителя веб-таблиц нужно настроить локальную среду выполнения интеграции. Дополнительные сведения см. в статье [Создание и настройка локальной среды выполнения интеграции](create-self-hosted-integration-runtime.md).

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю веб-таблиц.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы веб-таблиц поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **Web** |Да |
| url | URL-адрес источника Web |Да |
| authenticationType | Допустимое значение: **Anonymous**. |Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Требуется локальная среда IR, как упоминалось в разделе [Предварительные требования](#prerequisites). |Да |

**Пример**.

```json
{
    "name": "WebLinkedService",
    "properties": {
        "type": "Web",
        "typeProperties": {
            "url" : "https://en.wikipedia.org/wiki/",
            "authenticationType": "Anonymous"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). Этот раздел содержит список свойств, поддерживаемых набором данных веб-таблиц.

Чтобы скопировать данные из веб-таблиц, задайте для свойства type набора данных значение **WebTable**. Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство type для набора данных должно иметь значение **WebTable**. | Да |
| path |Относительный URL-адрес ресурса, который содержит таблицу. |Нет. Если путь не задан, используется только URL-адрес, указанный в определении связанной службы. |
| index |Индекс таблицы в ресурсе. Инструкции по получению индекса таблицы на странице HTML см. [в разделе Получение индекса таблицы в HTML-странице](#get-index-of-a-table-in-an-html-page) . |Да |

**Пример**.

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Web linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых веб-таблицами в качестве источника.

### <a name="web-table-as-source"></a>Веб-таблицы в качестве источника

Чтобы скопировать данные из веб-таблицы, задайте тип источника **WebSource** в действии копирования, дополнительные свойства не поддерживаются.

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromWebTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Web table input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "WebSource"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="get-index-of-a-table-in-an-html-page"></a>Получение индекса таблицы на HTML-странице

Для получения индекса таблицы, которую необходимо настроить в [свойствах набора данных](#dataset-properties), можно использовать, например, Excel 2016:

1. Запустите **Excel 2016** и перейдите на вкладку **Данные**.
2. На панели инструментов щелкните **Создать запрос**, выберите **Из других источников** и щелкните **Из Интернета**.

    ![Меню Power Query](./media/copy-data-from-web-table/PowerQuery-Menu.png)
3. В диалоговом окне **Из Интернета** введите **URL-адрес**, который будет использоваться в JSON связанной службы (например, https://en.wikipedia.org/wiki/)), вместе с указанным для набора данных путем (например, AFI%27s_100_Years…100_Movies), а затем нажмите кнопку **ОК**.

    ![Диалоговое окно "Из Интернета"](./media/copy-data-from-web-table/FromWeb-DialogBox.png)

    В этом примере используется URL-адрес https://en.wikipedia.org/wiki/AFI%27s_100_Years...100_Movies.
4. Если отображается диалоговое окно **Доступ к веб-содержимому**, выберите соответствующий **URL-адрес** и **тип аутентификации**, а затем нажмите кнопку **Подключить**.

   ![Диалоговое окно "Доступ к веб-содержимому"](./media/copy-data-from-web-table/AccessWebContentDialog.png)
5. В представлении дерева щелкните элемент **table**, чтобы просмотреть содержимое таблицы, а затем в нижней части экрана нажмите кнопку **Изменить**.  

   ![Диалоговое окно навигатора](./media/copy-data-from-web-table/Navigator-DialogBox.png)
6. В окне **Редактор запросов** на панели инструментов нажмите кнопку **Расширенный редактор**.

    ![Кнопка "Расширенный редактор"](./media/copy-data-from-web-table/QueryEditor-AdvancedEditorButton.png)
7. В диалоговом окне "Расширенный редактор" число, отображаемое рядом с полем "Источник", является индексом.

    ![Расширенный редактор — индекс](./media/copy-data-from-web-table/AdvancedEditor-Index.png)

Если вы работаете с Excel 2013, используйте [Microsoft Power Query для Excel](https://www.microsoft.com/download/details.aspx?id=39379), чтобы получить индекс. Дополнительные сведения см. в статье [Подключение к веб-странице](https://support.office.com/article/Connect-to-a-web-page-Power-Query-b2725d67-c9e8-43e6-a590-c0a175bd64d8). Точно так же можно использовать [Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/).


## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
