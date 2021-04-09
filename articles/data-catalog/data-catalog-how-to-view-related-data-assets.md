---
title: Как просматривать связанные ресурсы данных в Каталоге данных Azure
description: В этой статье объясняется, как просматривать связанные ресурсы данных в каталоге данных Azure.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: c1c5ddcacfc529f8fc4ab9a70cea540c44da9fa6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "104674790"
---
# <a name="how-to-view-related-data-assets-in-azure-data-catalog"></a>Как просматривать связанные ресурсы данных в каталоге данных Azure

[!INCLUDE [Azure Purview redirect](../../includes/data-catalog-use-purview.md)]

Каталог данных Azure позволяет просматривать ресурсы данных, связанные с выбранным ресурсом, и связи между ними. 

## <a name="supported-data-sources"></a>Поддерживаемые источники данных 
При регистрации ресурсов данных из следующих источников данных каталог данных Azure автоматически регистрирует метаданные о связях соединения между выбранными ресурсами данных. 

- SQL Server
- База данных SQL Azure
- MySQL
- Oracle;

> [!NOTE]
> Чтобы в каталоге данных импортировать связь между двумя ресурсами данных, необходимо одновременно зарегистрировать оба ресурса. Если один из них добавлен отдельно, добавьте его и другой ресурс еще раз, чтобы импортировать связи между этими ресурсами.

## <a name="view-related-data-assets"></a>Просмотр связанных ресурсов данных
Для просмотра ресурсов данных, связанных с выбранным набором данных, используйте вкладку **Связи**, как показано на следующем рисунке: 

![Просмотр связанных ресурсов данных в каталоге данных Azure](media/data-catalog-how-to-view-related-data-assets/relationships-tab.png)

В этом примере у выбранного ресурса данных **ProductSubcategory** имеется две связи: 

- Столбец ProductSubcategoryID таблицы Product имеет связь по внешнему ключу со столбцом ProductSubcategoryID выбранной таблицы ProductSubcategory. 
- Столбец ProductCategoryID таблицы ProductSubCategory имеет связь по внешнему ключу со столбцом ProductCategoryID выбранной таблицы ProductCategory.

> [!NOTE]
> Обратите внимание на направление стрелки в древовидном представлении связей.  

Для получения дополнительных сведений, таких как полное имя столбца, наведите на него указатель мыши, и появится всплывающее окно, как на следующем рисунке: 

![Извлечение отношений в каталоге данных Azure](media/data-catalog-how-to-view-related-data-assets/relationship-popup.png)

Для включения связей между зарегистрированными ресурсами зарегистрируйте их повторно.

## <a name="next-steps"></a>Дальнейшие действия
- [Как управлять ресурсами данных](data-catalog-how-to-manage.md)
