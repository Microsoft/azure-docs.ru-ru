---
title: Системные требования для шлюза Microsoft Azure Data Box | Документация Майкрософт
description: Дополнительные сведения о требованиях к программному обеспечению и сети для шлюза Microsoft Azure Data Box
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 03/01/2021
ms.author: alkohli
ms.openlocfilehash: e7c8653b39a3e0333ff6e98783a6e9a1437dba22
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101739217"
---
# <a name="azure-data-box-gateway-system-requirements"></a>Шлюз Azure Data Box требования к системе

В этой статье описаны важные требования к системе и рекомендации для решения на основе шлюза Microsoft Azure Data Box и для клиентов, подключающихся к нему. Прежде чем развертывать шлюз Data Box, внимательно ознакомьтесь с приведенной ниже информацией и по мере необходимости возвращайтесь к ней во время развертывания и последующих действий. 

Системные требования для виртуального устройства шлюза Data Box включают.

- **Требования к программному обеспечению для узлов**. Описание поддерживаемых платформ, браузеров для локальной конфигурации пользовательского интерфейса, клиентов SMB и любых дополнительных требований для хостов, которые подключаются к устройству.
- **Требования к сети для устройства**. Предоставление информации о любых сетевых требованиях к работе виртуального устройства.


## <a name="specifications-for-the-virtual-device"></a>Спецификации виртуального устройства

Чтобы подготовить виртуальное устройство Data Box, базовая главная система Шлюза Azure Data Box должна быть в состоянии выделить указанный ниже объем ресурсов.

| Спецификации                                          | Описание              |
|---------------------------------------------------------|--------------------------|
| Виртуальные процессоры (ядра)   | Минимум 4 |
| Память  | Минимум 8 ГБ. Мы настоятельно рекомендуем не менее 16 ГБ. |
| Доступность|Один узел|
| Диски| Диск ОС: 250 ГБ <br> Диск данных: минимум 2 ТБ, с тонкой подготовкой, с использованием дисков SSD|
| Сетевые интерфейсы|Один или несколько виртуальных сетевых интерфейсов|


## <a name="supported-os-for-clients-connected-to-device"></a>Поддерживаемая ОС для клиентов, подключенных к устройству

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-supported-client-os.md)]

## <a name="supported-protocols-for-clients-accessing-device"></a>Поддерживаемые протоколы для клиентов, обращающихся к устройству

[!INCLUDE [Supported protocols for clients accessing device](../../includes/data-box-edge-gateway-supported-client-protocols.md)]

## <a name="supported-virtualization-platforms-for-device"></a>Поддерживаемые платформы виртуализации для устройства

| **Операционная система и (или) платформа**  |**Версии**   |**Примечания**  |
|---------|---------|---------|
|Hyper-V  |  2012 R2 <br> 2016 <br> 2019 |         |
|VMware ESXi     | 6.0 <br> 6,5 <br> 6.7       |Средства VMware не поддерживаются.         |


## <a name="supported-storage-accounts"></a>Учетные записи хранилища BLOB-объектов

[!INCLUDE [Supported storage accounts](../../includes/data-box-edge-gateway-supported-storage-accounts.md)]


## <a name="supported-storage-types"></a>Поддерживаемые типы хранилища

[!INCLUDE [Supported storage types](../../includes/data-box-edge-gateway-supported-storage-types.md)]

## <a name="supported-browsers-for-local-web-ui"></a>Поддерживаемые браузеры для локального пользовательского веб-интерфейса

[!INCLUDE [Supported browsers for local web UI](../../includes/data-box-edge-gateway-supported-browsers.md)]

## <a name="networking-port-requirements"></a>Обязательные сетевые порты

В таблице ниже приведены порты, которые должны быть открытыми в брандмауэре для разрешения использования SMB, облака или управления трафиком. В этой таблице значение *в* или *входящий* относится к направлению, из которого клиент запрашивает доступ к вашему устройству. Значение *исходящий* указывает на *направление*, в котором устройство шлюза Data Box отправляет данные за пределами развертывания, например в Интернет.

[!INCLUDE [Port configuration for device](../../includes/data-box-edge-gateway-port-config.md)]

## <a name="url-patterns-for-firewall-rules"></a>Шаблоны URL-адресов для правил брандмауэра

Нередко сетевые администраторы могут настраивать правила брандмауэра на основе шаблонов URL-адресов, чтобы фильтровать входящий и исходящий трафик. Устройство шлюза Data Box и служба шлюза Data Box зависят от других приложений Майкрософт, в том числе от служебной шины Microsoft Azure, контроля доступа Azure Active Directory, учетных записей хранения и серверов центра обновления Майкрософт. Шаблоны URL-адресов, связанных с этими приложениями, можно использовать для настройки правил брандмауэра. Важно понимать, что шаблоны URL-адресов, связанных с этими приложениями, могут меняться. Поэтому сетевому администратору потребуется отслеживать правила брандмауэра для шлюза Data Box и обновлять их по мере необходимости.

В большинстве случаев мы рекомендуем настраивать правила брандмауэра для исходящего трафика на основе фиксированных IP-адресов шлюза Data Box. Тем не менее, чтобы задать расширенные правила брандмауэра, необходимые для создания безопасных сред, можно использовать приведенные ниже сведения.

> [!NOTE]
> - В качестве IP-адресов устройств (источников) всегда должны устанавливаться все сетевые интерфейсы с поддержкой облака.
> - Конечные IP-адреса должны быть установлены в [диапазоны адресов центра обработки данных Azure](https://www.microsoft.com/download/confirmation.aspx?id=41653).

[!INCLUDE [URL patterns for firewall](../../includes/data-box-edge-gateway-url-patterns-firewall.md)]

### <a name="url-patterns-for-azure-government"></a>Шаблоны URL-адресов для Azure для государственных организаций

[!INCLUDE [Azure Government URL patterns for firewall](../../includes/data-box-edge-gateway-gov-url-patterns-firewall.md)]

## <a name="internet-bandwidth"></a>Пропускная способность Интернета

[!INCLUDE [Internet bandwidth](../../includes/data-box-edge-gateway-internet-bandwidth.md)]

## <a name="next-step"></a>Следующий шаг

* [Tutorial: Prepare to deploy Azure Data Box Gateway (Preview)](data-box-gateway-deploy-prep.md) (Руководство. Подготовка к развертыванию шлюза Azure Data Box)

