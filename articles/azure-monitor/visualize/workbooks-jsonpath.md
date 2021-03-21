---
title: Azure Monitor книги — преобразование данных JSON с помощью JSONPath
description: Использование JSONPath в книгах Azure Monitor для преобразования результатов JSON-данных, возвращаемых запрашиваемой конечной точкой, в требуемый формат.
services: azure-monitor
author: lgayhardt
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: lagayhar
ms.openlocfilehash: 142853f0d5ed787d0b7aaee1bb118a001f144227
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101722931"
---
# <a name="how-to-use-jsonpath-to-transform-json-data-in-workbooks"></a>Как использовать JSONPath для преобразования данных JSON в книги

Книги могут запрашивать данные из многих источников. Некоторые конечные точки, например [Azure Resource Manager](../../azure-resource-manager/management/overview.md) или Пользовательская конечная точка, могут возвращать результаты в формате JSON. Если данные JSON, возвращенные запрашиваемой конечной точкой, не настроены в нужном формате, JSONPath можно использовать для преобразования результатов.

JSONPath — это язык запросов для JSON, похожий на XPath for XML. Как и XPath, JSONPath позволяет извлечь и фильтрация данные из структуры JSON.

С помощью преобразования JSONPath авторы книг могут преобразовать JSON в табличную структуру. Затем таблицу можно использовать для построения [визуализации книг](./workbooks-overview.md#visualizations).

## <a name="using-jsonpath"></a>Использование JSONPath

1. Переключите книгу в режим редактирования, щелкнув элемент панели инструментов *изменить* .
2. Используйте ссылку *Добавить*  >  *запрос* на добавление запроса, чтобы добавить в книгу элемент управления.
3. Выберите источник данных в формате *JSON*.
4. Используйте редактор JSON для ввода следующего фрагмента кода JSON
    ```json
    { "store": {
        "books": [ 
          { "category": "reference",
            "author": "Nigel Rees",
            "title": "Sayings of the Century",
            "price": 8.95
          },
          { "category": "fiction",
            "author": "Evelyn Waugh",
            "title": "Sword of Honour",
            "price": 12.99
          },
          { "category": "fiction",
            "author": "Herman Melville",
            "title": "Moby Dick",
            "isbn": "0-553-21311-3",
            "price": 8.99
          },
          { "category": "fiction",
            "author": "J. R. R. Tolkien",
            "title": "The Lord of the Rings",
            "isbn": "0-395-19395-8",
            "price": 22.99
          }
        ],
        "bicycle": {
          "color": "red",
          "price": 19.95
        }
      }
    }
    ```  

Предположим, что мы предоставили приведенный выше объект JSON как представление инвентаризации магазина. Наша задача — создать таблицу доступных книг по магазинам, указав их названия, авторов и цены.

1. Перейдите на вкладку *Параметры результатов* и измените формат результата на *путь JSON*.
2. Примените следующие параметры пути JSON:

    Таблица путей JSON: `$.store.books` . Это поле представляет путь к корню таблицы. В этом случае мы будем заботиться об инвентаризации книг в магазине. Путь к таблице фильтрует JSON в сведениях о книге.

   | Идентификаторы столбцов | Путь JSON столбца |
   |:-----------|:-----------------|
   | Заголовок      | `$.title`        |
   | Автор     | `$.author`       |
   | Цена      | `$.price`        |

    Идентификаторами столбцов будут заголовки столбцов. Поля путей JSON в столбцах представляют путь от корня таблицы к значению столбца.

1. Примените указанные выше параметры, нажав кнопку *выполнить запрос* .

![ Изменение элемента запроса с помощью источника данных JSON и формата результата JSON Path](./media/workbooks-jsonpath/query-jsonpath.png)

## <a name="next-steps"></a>Дальнейшие действия
- [Общие сведения о книгах](./workbooks-overview.md)
- [Группы в Azure Monitor книгах](workbooks-groups.md)