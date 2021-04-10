---
title: Управление IoT Central программным способом | Документация Майкрософт
description: В этой статье показано, как создавать приложения IoT Central и управлять ими программным способом. Вы можете просматривать, изменять и удалять приложения с помощью пакетов средств разработки для разных языков, включая JavaScript, Python, C#, Ruby и Go.
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 12/23/2020
ms.topic: how-to
ms.openlocfilehash: 2feb4d85e7be4076eef8b2796d15b622f6b17c3a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "104658307"
---
# <a name="manage-iot-central-programmatically"></a>Управление IoT Central программным способом

[!INCLUDE [iot-central-selector-manage](../../../includes/iot-central-selector-manage.md)]

Приложения IoT Central можно создавать и администрировать не только через веб-сайт [диспетчера приложений Azure IoT Central](https://aka.ms/iotcentral), но и программным способом с применением специальных пакетов средств разработки для Azure. Поддерживаются языки JavaScript, Python, C#, Ruby и Go.

## <a name="install-the-sdk"></a>Установка пакета SDK

В следующей таблице перечислены репозитории пакетов средств разработки и команды для их установки.

| Репозиторий пакета средств разработки | Установка пакета |
| -------------- | ------------ |
| [Пакет SDK Azure IotCentralClient для JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/iotcentral/arm-iotcentral) | `npm install @azure/arm-iotcentral` |
| [Пакет SDK Microsoft Azure для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/iothub/azure-mgmt-iotcentral/azure/mgmt/iotcentral) | `pip install azure-mgmt-iotcentral` |
| [Пакет Azure SDK для .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/iotcentral/Microsoft.Azure.Management.IotCentral) | `dotnet add package Microsoft.Azure.Management.IotCentral` |
| [Пакет SDK Microsoft Azure для Ruby — управление ресурсами (предварительная версия)](https://github.com/Azure/azure-sdk-for-ruby/tree/master/management/azure_mgmt_iot_central/lib/2018-09-01/generated/azure_mgmt_iot_central) | `gem install azure_mgmt_iot_central` |
| [Пакет Azure SDK для Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/iotcentral) | [Пакет Maven](https://search.maven.org/search?q=a:azure-mgmt-iotcentral) |
| [Пакет Azure SDK для Go](https://github.com/Azure/azure-sdk-for-go/tree/master/services/iotcentral/mgmt/2018-09-01/iotcentral) | [Выпуски пакетов](https://github.com/Azure/azure-sdk-for-go/releases) |

## <a name="samples"></a>Примеры

В [репозитории примеров использования пакета средств разработки для Azure IoT Central ARM](/samples/azure-samples/azure-iot-central-arm-sdk-samples/azure-iot-central-arm-sdk-samples/) собраны примеры кода для нескольких языков программирования, которые демонстрируют, как создавать, обновлять, перечислять и удалять приложения Azure IoT Central.

[!INCLUDE [Warning About Access Required](../../../includes/iot-central-warning-contribitorrequireaccess.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали, как управлять приложениями Azure IoT Central программным способом. Рекомендуем перейти к изучению службы [Azure Resource Manager](../../azure-resource-manager/management/overview.md).