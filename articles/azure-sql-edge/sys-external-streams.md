---
title: sys.external_streams (Transact-SQL) — Azure SQL ребро
description: Дополнительные сведения об использовании sys.external_streams в Azure SQL ребр
keywords: sys.external_streams, SQL для пограничных вычислений
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2019
ms.openlocfilehash: 04950f01c06bc3c8ed3bb11a790310c2319a0579
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90900307"
---
# <a name="sysexternal_streams-transact-sql"></a>sys.external_streams (Transact-SQL)

Возвращает строку для каждого объекта внешнего потока, созданного в области базы данных.

|Имя столбца|Тип данных|Описание|  
|-----------------|---------------|-----------------|
|**name**|**sysname**|Имя потока. Уникален в пределах базы данных.|
|**object_id**|**int**|Идентификационный номер объекта потока. Уникален в пределах базы данных.|
|**principal_id**|**int**|Идентификатор участника, который является владельцем этой сборки.|
|**schema_id**|**int**| Идентификатор схемы, содержащей объект.|
|**parent_object_id**|**идентификатор**| Идентификатор родительского объекта для этого потока. В текущей реализации всегда возвращает значение NULL.|
|**type**|**char(2)**|Тип объекта. Для объектов потока тип всегда имеет значение ES.|
|**type_desc**|**nvarchar(60)**| Описание типа объекта. Для объектов потока тип всегда имеет значение EXTERNAL_STREAM.|
|**create_date**|**datetime**| Дата создания объекта.|
|**modify_date**|**datetime**| Дата последнего изменения объекта с помощью инструкции ALTER.|
|**is_ms_shipped**|**bit**| Объект, созданный с помощью внутреннего компонента.|  
|**is_published**|**bit**|Объект опубликован.|  
|**is_schema_published**|**bit**|Опубликована только схема объекта.|
|**max_column_id_used**|**bit**| Этот столбец используется для внутренних задач и в будущем будет удален.|  
|**uses_ansi_nulls**|**bit**| Объект потока был создан с параметром SET ANSI_NULLS ON.|
|**data_source_id**|**int**| Идентификатор объекта для внешнего источника данных, представленного объектом потока. |  
|**file_format_id**|**int**| Идентификатор объекта для формата внешнего файла, используемого объектом потока. Формат внешнего файла необходим, чтобы указать фактическую структуру данных, на которые ссылается внешний поток.| 
|**расположение**|**varchar(max)**| Целевой объект для объекта внешнего потока. Дополнительные сведения см. в разделе [Создание внешнего потока](overview.md). |
|**input_option**|**varchar(max)**| Входные параметры, используемые при создании внешнего потока. Дополнительные сведения см. в разделе [Создание внешнего потока](overview.md). |
|**output_option**|**varchar(max)**| Выходные параметры, используемые при создании внешнего потока. Дополнительные сведения см. в разделе [Создание внешнего потока](overview.md). | 

## <a name="permissions"></a>Разрешения

Видимость метаданных в представлениях каталогов ограничивается защищаемыми объектами, которыми пользователь владеет или на которые ему были предоставлены разрешения. Дополнительные сведения см. в разделе [Metadata Visibility Configuration](/sql/relational-databases/security/metadata-visibility-configuration/).

## <a name="see-also"></a>См. также раздел

- [Представления каталога (Transact-SQL)](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql/)
- [Системные представления (Transact-SQL)](/sql/t-sql/language-reference/)
