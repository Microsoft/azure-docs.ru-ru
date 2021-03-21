---
title: Фильтрация подключений по протоколу IP в Azure IoT DPS | Документация Майкрософт
description: Узнайте, как использовать фильтрацию IP-адресов для блокирования подключений к экземпляру Azure IoT DPS с определенных IP-адресов. Вы можете блокировать подключения с отдельных IP-адресов или их диапазонов.
author: wesmc7777
ms.author: wesmc
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 12/14/2020
ms.openlocfilehash: e1b175a176255da465433b2db45cb3cb67d360d1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98934495"
---
# <a name="use-azure-iot-dps-ip-connection-filters"></a>Использование фильтрации подключений по протоколу IP в Azure IoT DPS

Безопасность является важной характеристикой любого решения Интернета вещей. Иногда при настройке системы безопасности необходимо явно указать IP-адреса, с которых могут подключаться устройства. Функция *фильтрации IP-адресов* для Подготовки устройств к добавлению в Центр Интернета вещей Azure (DPS) позволяет настроить правила для отклонения или приема трафика с определенных IPv4-адресов.

## <a name="when-to-use"></a>Назначение

Есть два варианта применения этой функции, когда рекомендуется блокировать подключения к конечной точке DPS с определенных IP-адресов:

* Служба DPS должна принимать трафик только с IP-адресов из определенного диапазона и отклонять остальной трафик. Допустим, что DPS используется в сочетании с [Azure ExpressRoute](../expressroute/expressroute-faqs.md#supported-services) для создания частных подключений между DPS и устройствами.

* В этом случае требуется отклонять трафик с IP-адресов, которые были определены администратором DPS как подозрительные.

## <a name="how-filter-rules-are-applied"></a>Применение правил фильтрации

Правила фильтрации IP-адресов применяются на уровне экземпляра DPS. Поэтому они действуют для всех подключений с устройств и внутренних приложений, использующих любые поддерживаемые протоколы.

При попытке подключения с IP-адреса, который соответствует правилу отклонения IP-адресов в экземпляре DPS, пользователь получает сообщение об ошибке с кодом состояния 401 (Не авторизовано) и описанием. В ответном сообщении не указывается правило фильтрации IP-адресов.

## <a name="default-setting"></a>Параметр по умолчанию

По умолчанию раздел **Фильтрация IP-адресов** на портале для DPS пуст. Этот означает, что по умолчанию DPS принимает подключения с любых IP-адресов. То есть, такое значение параметра по умолчанию равноценно правилу, которое принимает диапазон IP-адресов 0.0.0.0/0.

![Параметры фильтрации IP-адресов по умолчанию для IoT DPS](./media/iot-dps-ip-filtering/ip-filter-default.png)

## <a name="add-or-edit-an-ip-filter-rule"></a>Добавление или изменение правила фильтрации IP-адресов

Чтобы добавить правило фильтрации IP-адресов, выберите **+ Добавить правило фильтрации IP-адресов**.

![Добавление правила фильтрации IP-адресов в IoT DPS](./media/iot-dps-ip-filtering/ip-filter-add-rule.png)

После выбора параметра **Добавить правило фильтрации IP-адресов** заполните поля.

![После выбора параметра "Добавить правило фильтрации IP-адресов"](./media/iot-dps-ip-filtering/ip-filter-after-selecting-add.png)

* Укажите **имя** правила фильтрации IP-адресов. Оно должно быть уникальной строкой буквенно-цифровых символов длиной не более 128 символов без учета регистра. Допускаются только 7-разрядные буквы и цифры ASCII, а также `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}`.

* Укажите один IPv4-адрес или блок IP-адресов в нотации CIDR. Например, в нотации CIDR 192.168.100.0/22 обозначает диапазон из 1024 IPv4-адресов от 192.168.100.0 до 192.168.103.255.

* В качестве **действия** для правила фильтрации IP-адресов выберите **Разрешить** или **Блокировать**.

После заполнения полей выберите **Сохранить**, чтобы сохранить правило. Появится предупреждение о том, что выполняется обновление.

![Уведомление о сохранении правила фильтрации IP-адресов](./media/iot-dps-ip-filtering/ip-filter-save-new-rule.png)

Параметр **Добавить** недоступен, если достигнуто максимальное число правил фильтрации IP-адресов 100.

Чтобы изменить существующее правило, выберите данные, которые необходимо изменить, внесите изменения, а затем выберите **Сохранить**, чтобы сохранить изменения.

> [!NOTE]
> Отклонение IP-адресов может препятствовать взаимодействию других служб Azure с экземпляром DPS.

## <a name="delete-an-ip-filter-rule"></a>Удаление правила фильтрации IP-адресов

Чтобы удалить правило фильтрации IP-адресов, выберите значок корзины в этой строке, а затем выберите **Сохранить**. Правило удаляется, а изменения сохраняются.

![Удаление правила фильтрации IP-адресов в IoT DPS](./media/iot-dps-ip-filtering/ip-filter-delete-rule.png)


## <a name="update-ip-filter-rules-in-code"></a>Обновление правил фильтрации IP-адресов в коде

Вы можете получать и изменять правила фильтрации IP-адресов в DPS, используя конечную точку REST поставщика ресурсов Azure. Подробные сведения о `properties.ipFilterRules` см. в статье [Iot Hub Resource — Create Or Update](/rest/api/iot-dps/iotdpsresource/createorupdate) (Ресурс Центра Интернета вещей. Создание или обновление).

Обновление правил фильтрации IP-адресов в DPS сейчас не поддерживается для Azure CLI или Azure PowerShell, но может быть выполнено с помощью шаблонов Azure Resource Manager. Рекомендации по использованию шаблонов Azure Resource Manager см. в [этой статье](../azure-resource-manager/templates/overview.md). В приведенных ниже примерах шаблонов показано, как создавать, изменять и удалять правила фильтрации IP-адресов в DPS.

### <a name="add-an-ip-filter-rule"></a>Добавление правила фильтрации IP-адресов

В следующем примере шаблона создается новое правило фильтрации IP-адресов с именем AllowAll, которое принимает весь трафик.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#", 
    "contentVersion": "1.0.0.0", 
    "parameters": {
        "iotDpsName": {
            "type": "string",
            "defaultValue": "[resourceGroup().name]",
            "minLength": 3,
            "metadata": {
                "description": "Specifies the name of the IoT DPS service."
            }
        }, 
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for Iot DPS resource."
            }
        }        
    }, 
    "variables": {
        "iotDpsApiVersion": "2020-01-01"
    }, 
    "resources": [
        {
            "type": "Microsoft.Devices/provisioningServices",
            "apiVersion": "[variables('iotDpsApiVersion')]",
            "name": "[parameters('iotDpsName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "S1",
                "tier": "Standard",
                "capacity": 1
            },
            "properties": {
                "IpFilterRules": [
                    {
                        "FilterName": "AllowAll",
                        "Action": "Accept",
                        "ipMask": "0.0.0.0/0"
                    }
                ]
            }            
        }
    ]
}
```

Обновите атрибуты правила фильтрации IP-адресов для шаблона в соответствии со своими требованиями.

| attribute                | Описание |
| ------------------------ | ----------- |
| **FilterName**           | Укажите имя правила фильтрации IP-адресов. Оно должно быть уникальной строкой буквенно-цифровых символов длиной не более 128 символов без учета регистра. Принимаются только символы ASCII 7-разрядной буквы плюс  `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}`   . |
| **Действие**               | Допустимые действия для правила фильтрации IP-адресов: **Принять** или  **Отклонить** . |
| **ipMask**               | Укажите один IPv4-адрес или блок IP-адресов в нотации CIDR. Например, в нотации CIDR 192.168.100.0/22 обозначает диапазон из 1024 IPv4-адресов от 192.168.100.0 до 192.168.103.255. |


### <a name="update-an-ip-filter-rule"></a>Обновление правила фильтрации IP-адресов

В следующем примере шаблона выполняется обновление правила фильтрации IP-адресов с именем AllowAll (показано ранее) для отклонения всего трафика.

```json
{ 
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",  
    "contentVersion": "1.0.0.0",  
    "parameters": { 
        "iotDpsName": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().name]", 
            "minLength": 3, 
            "metadata": { 
                "description": "Specifies the name of the IoT DPS service." 
            } 
        },  
        "location": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().location]", 
            "metadata": { 
                "description": "Location for Iot DPS resource." 
            } 
        }        
    },  
    "variables": { 
        "iotDpsApiVersion": "2020-01-01" 
    },  
    "resources": [ 
        { 
            "type": "Microsoft.Devices/provisioningServices", 
            "apiVersion": "[variables('iotDpsApiVersion')]", 
            "name": "[parameters('iotDpsName')]", 
            "location": "[parameters('location')]", 
            "sku": { 
                "name": "S1", 
                "tier": "Standard", 
                "capacity": 1 
            }, 
            "properties": { 
                "IpFilterRules": [ 
                    { 
                        "FilterName": "AllowAll", 
                        "Action": "Reject", 
                        "ipMask": "0.0.0.0/0" 
                    } 
                ] 
            }             
        } 
    ] 
}
```

### <a name="delete-an-ip-filter-rule"></a>Удаление правила фильтрации IP-адресов

В следующем примере шаблона удаляются все правила фильтрации IP-адресов для экземпляра DPS.

```json
{ 
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",  
    "contentVersion": "1.0.0.0",  
    "parameters": { 
        "iotDpsName": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().name]", 
            "minLength": 3, 
            "metadata": { 
                "description": "Specifies the name of the IoT DPS service." 
            } 
        },  
        "location": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().location]", 
            "metadata": { 
                "description": "Location for Iot DPS resource." 
            } 
        }        
    },  
    "variables": { 
        "iotDpsApiVersion": "2020-01-01" 
    },  
    "resources": [ 
        { 
            "type": "Microsoft.Devices/provisioningServices", 
            "apiVersion": "[variables('iotDpsApiVersion')]", 
            "name": "[parameters('iotDpsName')]", 
            "location": "[parameters('location')]", 
            "sku": { 
                "name": "S1", 
                "tier": "Standard", 
                "capacity": 1 
            }, 
            "properties": { 
            }             
        } 
    ] 
}
```



## <a name="ip-filter-rule-evaluation"></a>Оценка правила фильтрации IP-адресов

Правила фильтрации IP-адресов применяются по порядку, поэтому первое правило, которое соответствует IP-адресу, определяет действие (принять или отклонить).

Например, если вы хотите, чтобы адреса в диапазоне 192.168.100.0/22 принимались, а все остальные отклонялись, то первым правилом в списке должно быть именно правило о приеме трафика из диапазона адресов 192.168.100.0/22. Следующим должно быть правило об отклонении всех адресов; для этого используется диапазон 0.0.0.0/0.

Последовательность правил фильтрации IP-адресов в списке можно изменить. Для этого щелкните три вертикальные точки в начале строки и воспользуйтесь функцией перетаскивания.

Чтобы сохранить новую последовательность, нажмите кнопку **Сохранить**.

![Изменение порядка правил фильтрации IP-адресов в DPS](./media/iot-dps-ip-filtering/ip-filter-rule-order.png)

## <a name="next-steps"></a>Дальнейшие действия

Сведения об управлении DPS см. в следующих статьях:

* [Основные сведения об IP-адресах в IoT DPS](iot-dps-understand-ip-address.md)
* [Настройка DPS с помощью Azure CLI](quick-setup-auto-provision-cli.md)
* [Управление доступом к DPS](how-to-control-access.md)
