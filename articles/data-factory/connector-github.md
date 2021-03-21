---
title: Подключение к GitHub
description: Использование GitHub для указания ссылок на сущности общих моделей данных
author: djpmsft
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/03/2020
ms.author: daperlov
ms.openlocfilehash: 5d555d7bc4d3aae9c016cacbe17b68c30859d99a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100372286"
---
# <a name="use-github-to-read-common-data-model-entity-references"></a>Использование GitHub для чтения ссылок на сущности общих моделей данных

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Соединитель GitHub в фабрике данных Azure используется только для получения схемы ссылки на сущность для формата [общей модели данных](format-common-data-model.md) в потоке сопоставления данных.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы GitHub поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства Type необходимо задать значение **GitHub**. | да
| userName | Имя пользователя GitHub | да |
| password | Пароль GitHub | да |

## <a name="next-steps"></a>Дальнейшие действия

Создайте [исходный набор данных](data-flow-source.md) в сопоставлении потока данных.