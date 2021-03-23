---
title: Сведения об устройствах в конкретных зонах
description: Использование локальной консоли управления для получения исчерпывающих сведений о представлении для конкретной зоны
author: shhazam-ms
manager: rkarlin
ms.author: shhazam
ms.date: 03/21/2021
ms.topic: how-to
ms.service: azure
ms.openlocfilehash: 685d7b1df4389c356ee7b531d179919025d298d0
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104776336"
---
# <a name="view-information-per-zone"></a>Просмотр сведений по зоне


## <a name="view-a-device-map-for-a-zone"></a>Просмотр схемы устройства для зоны

Просмотр схемы устройства для выбранной зоны на датчике. В этом представлении отображаются все сетевые элементы, связанные с выбранной зоной, включая датчики, подключенные к ним устройства и другие сведения.

:::image type="content" source="media/how-to-work-with-asset-inventory-information/zone-map-screenshot.png" alt-text="Снимок экрана: Схема зоны.":::


- В окне **Управление сайтом** выберите пункт **Просмотреть карту зоны** на панели, содержащей имя зоны.

  :::image type="content" source="media/how-to-work-with-asset-inventory-information/default-region-to-default-business-unit-v2.png" alt-text="Регион по умолчанию для подразделения по умолчанию.":::

Откроется окно " **схема устройства** ". Для просмотра устройств и сведений об устройстве на карте доступны следующие средства. Дополнительные сведения о каждой из этих функций см. в статье *"защитник для платформы IOT для пользователей"*.

- **Представления масштаба карт**: упрощенное представление, представление соединений и подробное представление. Отображаемое представление отображения зависит от уровня масштаба на карте. Переключение между представлениями карт осуществляется путем настройки уровней масштаба.

  :::image type="icon" source="media/how-to-work-with-asset-inventory-information/zoom-icon.png" border="false":::

- **Средства поиска и макетирования карт**: инструменты, используемые для отображения различных сегментов сети, устройств, групп устройств или слоев.

  :::image type="content" source="media/how-to-work-with-asset-inventory-information/search-and-layout-tools.png" alt-text="Снимок экрана: представление &quot;средства поиска и макета&quot;.":::

- **Метки и индикаторы на устройствах:** Например, число устройств, сгруппированных в подсети в сети ИТ. В этом примере это 8.

  :::image type="content" source="media/how-to-work-with-asset-inventory-information/labels-and-indicators.png" alt-text="Снимок экрана с метками и индикаторами.":::

- **Просмотр свойств устройства**. Например, датчик, отслеживающий свойства устройства и основного устройства. Щелкните устройство правой кнопкой мыши, чтобы просмотреть свойства устройства.

  :::image type="content" source="media/how-to-work-with-asset-inventory-information/asset-properties-v2.png" alt-text="Снимок экрана представления свойств устройства.":::

- **Оповещение, связанное с устройством:** Щелкните устройство правой кнопкой мыши, чтобы просмотреть связанные предупреждения.

  :::image type="content" source="media/how-to-work-with-asset-inventory-information/show-alerts.png" alt-text="Снимок экрана представления &quot;Показать предупреждения&quot;.":::

## <a name="view-alerts-associated-with-a-zone"></a>Просмотр оповещений, связанных с зоной

Чтобы просмотреть предупреждения, связанные с определенной зоной, выполните следующие действия.

- Щелкните значок предупреждения в окне **зона** . 

  :::image type="content" source="media/how-to-work-with-asset-inventory-information/business-unit-view-v2.png" alt-text="Представление бизнес-единицы по умолчанию с примерами.":::

Дополнительные сведения см. в разделе [Обзор: работа с предупреждениями](how-to-work-with-alerts-on-premises-management-console.md).

### <a name="view-the-device-inventory-of-a-zone"></a>Просмотр данных инвентаризации устройств зоны

Чтобы просмотреть сведения об инвентаризации устройств, связанных с определенной зоной, выполните следующие действия.

- Выберите **Просмотр описи устройств** в окне **зона** .

  :::image type="content" source="media/how-to-work-with-asset-inventory-information/default-business-unit.png" alt-text="Появится экран Инвентаризация устройства.":::

Дополнительные сведения см. [в разделе исследование всех обнаружений датчиков предприятия в инвентаризации устройства](how-to-investigate-all-enterprise-sensor-detections-in-a-device-inventory.md).

## <a name="view-additional-zone-information"></a>Просмотр дополнительных сведений о зоне

Доступны следующие дополнительные сведения о зоне:

- **Сведения о зоне**: Просмотр числа устройств, предупреждений и датчиков, связанных с зоной.

- **Сведения о датчике**: Просмотр имени, IP-адреса и версии каждого датчика, назначенного зоне.

- **Состояние подключения**: Если датчик отключен, подключитесь к датчику. См. раздел [датчики подключения к локальной консоли управления](how-to-activate-and-set-up-your-on-premises-management-console.md#connect-sensors-to-the-on-premises-management-console). 

- **Ход обновления**. при обновлении датчика подключения будут отображаться состояния обновления. Во время обновления локальная консоль управления не получает сведения об устройстве от датчика.

## <a name="next-steps"></a>Дальнейшие действия

[Анализ глобальных, региональных и локальных угроз](how-to-gain-insight-into-global-regional-and-local-threats.md)
