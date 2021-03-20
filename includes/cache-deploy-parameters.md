---
author: wesmc7777
ms.service: redis-cache
ms.topic: include
ms.date: 04/02/2019
ms.author: wesmc
ms.openlocfilehash: 498a7ee28b9404d0733e4615f4df635a8c904b51
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "67185096"
---
### <a name="cacheskuname"></a>Параметр cacheSKUName

Ценовая категория нового кэша Azure для Redis.

```json
    "cacheSKUName": {
      "type": "string",
      "allowedValues": [
        "Basic",
        "Standard",
        "Premium"
      ],
      "defaultValue": "Basic",
      "metadata": {
        "description": "The pricing tier of the new Azure Cache for Redis."
      }
    },
```

Шаблон определяет значения, разрешенные для этого параметра (Basic, Standard или Premium), и присваивает значение по умолчанию (Basic), если значение не указано. Уровень Basic предоставляет один узел с различными размерами (до 53 ГБ). Уровень Standard предоставляет два узла (основной и реплика) с различными размерами (до 53 ГБ) и соглашением об уровне обслуживания 99,9 %.

### <a name="cacheskufamily"></a>Параметр cacheSKUFamily

Семейство для SKU.

```json
    "cacheSKUFamily": {
      "type": "string",
      "allowedValue/s": [
        "C",
        "P"
      ],
      "defaultValue": "C",
      "metadata": {
        "description": "The family for the sku."
      }
    },
```

### <a name="cacheskucapacity"></a>Параметр cacheSKUCapacity

Создает экземпляр кэша Azure для Redis.

Для семейств уровня "базовый" и "Стандартный":

```json
    "cacheSKUCapacity": {
      "type": "int",
      "allowedValues": [
        0,
        1,
        2,
        3,
        4,
        5,
        6
      ],
      "defaultValue": 0,
      "metadata": {
        "description": "The size of the new Azure Cache for Redis instance. "
      }
    }
```

Емкость кэша значения Premium определяется одинаково, за исключением допустимых значений, выполняемых от 1 до 5, а не от 0 до 6.

Шаблон определяет целочисленные значения, разрешенные для этого параметра (от 0 до 6 для семейств уровня "базовый" и "Стандартный"; от 1 до 5 для семейства "Премиум"). Если значение не указано, шаблон присваивает значение по умолчанию 0 для Basic и Standard, 1 — для Premium.

Значения соответствуют следующим размерам кэша:

| Значение | Базовый и Стандартный<br>Размер кэша | Premium<br>Размер кэша |
| :---: | :------------------------------: | :-------------------: |
| 0     | 250 МБ (по умолчанию)                 | Н/Д                   |
| 1     | 1 ГБ                             | 6 ГБ (по умолчанию)        |
| 2     | 2,5 ГБ                           | 13 ГБ                 |
| 3     | 6 ГБ                             | 26 ГБ                 |
| 4     | 13 ГБ                            | 53 ГБ                 |
| 5     | 26 ГБ                            | 120 ГБ                |
| 6     | 53 ГБ                            | Н/Д                   |
