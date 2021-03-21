---
title: Функции шаблона — Any
description: Описывает функции, доступные в Бицеп для преобразования типов.
ms.topic: conceptual
author: tfitzmac
ms.author: tomfitz
ms.service: azure-resource-manager
ms.subservice: templates
ms.date: 03/02/2021
ms.openlocfilehash: b0cb51c9a79d1100de7f1ef32fe326eddcdd6dcc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746916"
---
# <a name="any-function-for-bicep"></a>Любая функция для Бицеп

Бицеп поддерживает функцию, вызываемую `any()` для разрешения ошибок типов в системе типов бицеп. Эта функция используется, если формат предоставленного значения не соответствует предполагаемой системе типов. Например, если свойству требуется число, но необходимо указать его как строку, например `'0.5'` . Используйте `any()` функцию, чтобы подавить ошибку, сообщаемую системой типов.

Эта функция не существует в среде выполнения шаблона Azure Resource Manager. Он используется только Бицеп и не создается в JSON для созданного шаблона.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="any"></a>any

`any(value)`

Возвращает значение, совместимое с любым типом данных.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| value | Да | все типы | Значение, которое необходимо преобразовать в совместимый тип. |

### <a name="return-value"></a>Возвращаемое значение

Значение в форме, совместимое с любым типом данных.

### <a name="examples"></a>Примеры

В следующем примере шаблона показано, как использовать `any()` функцию для предоставления числовых значений в виде строк.

```bicep
resource wpAci 'microsoft.containerInstance/containerGroups@2019-12-01' = {
  name: 'wordpress-containerinstance'
  location: location
  properties: {
    containers: [
      {
        name: 'wordpress'
        properties: {
          ...
          resources: {
            requests: {
              cpu: any('0.5')
              memoryInGB: any('0.7')
            }
          }
        }
      }
    ]
  }
}
```

Функция работает с любым назначенным значением в Бицеп. В следующем примере используется `any()` с выражением ternary в качестве аргумента.  

```bicep
publicIPAddress: any((pipId == '') ? null : {
  id: pipId
})
```

## <a name="next-steps"></a>Дальнейшие действия

Более сложные способы использования `any()` функции см. в следующих примерах:

* [Дочерние ресурсы, для которых требуются определенные имена](https://github.com/Azure/bicep/blob/main/docs/examples/201/api-management-create-all-resources/main.bicep#L246)
* [Свойство ресурса, не определенное в типе ресурса, несмотря на то, что оно существует](https://github.com/Azure/bicep/blob/main/docs/examples/201/log-analytics-with-solutions-and-diagnostics/main.bicep#L26)

