---
title: Руководство по Получение данных устройства через Центр Интернета вещей Azure
description: В этом учебнике показано, как включить маршрутизацию данных устройства из Центра Интернета вещей в Azure API для FHIR с помощью соединителя "Azure IoT для FHIR".
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: tutorial
ms.date: 11/13/2020
ms.author: punagpal
ms.openlocfilehash: 77d54ef0c9bef40af47c2fc48b9d4b5d6315b119
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780401"
---
# <a name="tutorial-receive-device-data-through-azure-iot-hub"></a>Руководство по Получение данных устройства через Центр Интернета вещей Azure

Соединитель Azure IoT для Ресурсов быстрого взаимодействия в сфере здравоохранения (FHIR&#174;) позволяет принимать данные из Интернета медицинских вещей (Internet of Medical Things, IoMT) в Azure API для FHIR. В кратком руководстве [Развертывание соединителя "Azure IoT для FHIR"(предварительная версия) с помощью портала Azure](iot-fhir-portal-quickstart.md) показан пример устройства под управлением Azure IoT Central, которое [отправляет данные телеметрии](iot-fhir-portal-quickstart.md#connect-your-devices-to-iot) в соединитель "Azure IoT для FHIR". Соединитель "Azure IoT для FHIR" также может работать с устройствами, подготовленными и управляемыми через Центр Интернета вещей Azure. В этом учебнике показано, как подключить Центр Интернета вещей Azure к соединителю "Azure IoT для FHIR" и настроить маршрутизацию данных устройства.

## <a name="prerequisites"></a>Предварительные требования

- Активная подписка Azure — [создайте подписку бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure API для FHIR хотя бы с одним соединителем "Azure IoT для FHIR" — [разверните соединитель "Azure IoT для FHIR" с помощью портала Azure (предварительная версия)](iot-fhir-portal-quickstart.md).
- Ресурс Центра Интернета вещей Azure, подключенный к реальным или имитированным устройствам — [создайте Центр Интернета вещей с помощью портала Azure](../../iot-hub/quickstart-send-telemetry-dotnet.md).

> [!TIP]
> Если вы используете приложение для имитированного устройства в Центре Интернета вещей Azure, вы можете выбрать любое приложение из разных поддерживаемых языков и систем.

## <a name="get-connection-string-for-azure-iot-connector-for-fhir-preview"></a>Получение строки подключения для соединителя "Azure IoT для FHIR" (предварительная версия)

Для безопасного подключения Центра Интернета вещей к соединителю "Azure IoT для FHIR" вещей требуется строка подключения. Просмотрите сведения о [создании строки подключения для соединителя "Azure IoT для FHIR"](iot-fhir-portal-quickstart.md#generate-a-connection-string). Эта строка подключения будет использоваться на следующем шаге.

Соединитель "Azure IoT для FHIR" использует для получения сообщений устройства экземпляр концентратора событий Azure. Созданная выше строка подключения по сути устанавливает подключение к этому базовому концентратору событий.

## <a name="connect-azure-iot-hub-with-the-azure-iot-connector-for-fhir-preview"></a>Подключение Центра Интернета вещей Azure к соединителю "Azure IoT для FHIR" (предварительная версия)

Центр Интернета вещей Azure поддерживает [маршрутизацию сообщений](../../iot-hub/iot-hub-devguide-messages-d2c.md). Эта возможность позволяет отправлять данные устройства в разные службы Azure, например в концентратор событий, учетную запись хранения или служебную шину. Соединитель "Azure IoT для FHIR" использует эту возможность для подключения и отправки данных устройства из Центра Интернета вещей Azure в конечную точку концентратора событий.

> [!NOTE] 
> Сейчас [создать маршрутизацию сообщений](../../iot-hub/tutorial-routing.md) можно только с помощью команд PowerShell или CLI, так как концентратор событий соединителя "Azure IoT для FHIR" не размещается в подписке клиента и не доступен через портал Azure. Когда объекты маршрута добавляются с помощью PowerShell или CLI, они появляются на портале Azure, и вы можете управлять ими там.

Настройка маршрутизации сообщений состоит из двух этапов.

### <a name="add-an-endpoint"></a>Добавление конечной точки
Этот шаг определяет конечную точку, к которой Центр Интернета вещей будет направлять данные. Для создания конечной точки выполните команду PowerShell [Add-AzIotHubRoutingEndpoint](/powershell/module/az.iothub/Add-AzIotHubRoutingEndpoint) или команду CLI [az iot hub routing-endpoint create](/cli/azure/iot/hub/routing-endpoint#az_iot_hub_routing_endpoint_create) (в зависимости от предпочтений).

Ниже приведен список параметров, которые можно использовать с командой для создания конечной точки.

|Параметр PowerShell|Параметр CLI|Описание|
|---|---|---|
|ResourceGroupName|resource-group|Имя группы ресурсов для ресурса Центра Интернета вещей.|
|Имя|hub-name|Имя ресурса Центра Интернета вещей.|
|EndpointName|endpoint-name|Укажите имя, которое вы хотите назначить создаваемой конечной точке.|
|EndpointType|endpoint-type|Тип конечной точки, к которой будет подключаться Центр Интернета вещей. Используйте литеральное значение EventHub для PowerShell и eventhub для CLI.|
|EndpointResourceGroup|endpoint-resource-group|Имя группы ресурсов для ресурса Azure API для FHIR соединителя "Azure IoT для FHIR". Это значение можно узнать на странице обзора для Azure API для FHIR.|
|EndpointSubscriptionId|endpoint-subscription-id|Идентификатор подписки для ресурса Azure API для FHIR соединителя "Azure IoT для FHIR". Это значение можно узнать на странице обзора для Azure API для FHIR.|
|ConnectionString|Строка подключения|Строка подключения к соединителю "Azure IoT для FHIR". Используйте значение, полученное на предыдущем шаге.|

### <a name="add-a-message-route"></a>Добавление маршрута сообщений
На этом шаге определяется маршрут сообщений на основе созданной выше конечной точки. Создайте маршрут с помощью команды PowerShell [Add-AzIotHubRoute](/powershell/module/az.iothub/Add-AzIoTHubRoute) или команды CLI [az iot hub route create](/cli/azure/iot/hub/route#az_iot_hub_route_create) (в зависимости от предпочтений).

Ниже приведен список параметров, которые можно использовать с командой для добавления маршрута сообщений.

|Параметр PowerShell|Параметр CLI|Описание|
|---|---|---|
|ResourceGroupName|g|Имя группы ресурсов для ресурса Центра Интернета вещей.|
|Имя|hub-name|Имя ресурса Центра Интернета вещей.|
|EndpointName|endpoint-name|Имя ранее созданной конечной точки.|
|RouteName|route-name|Имя, которое вы хотите присвоить создаваемому маршруту сообщений.|
|Источник|source-type|Тип данных для отправки в конечную точку. Используйте литеральное значение DeviceMessages для PowerShell и devicemessages для CLI.|

## <a name="send-device-message-to-iot-hub"></a>Отправка сообщения в Центр Интернета вещей

Используйте реальное или имитированное устройство, чтобы отправить в центр Интернета вещей Azure тестовое сообщение пульса, показанное ниже. Это сообщение будет перенаправлено в соединитель "Azure IoT для FHIR", где оно будет преобразовано в ресурс наблюдения FHIR и сохранено в Azure API для FHIR.

```json
{
  "HeartRate": 80,
  "RespiratoryRate": 12,
  "HeartRateVariability": 64,
  "BodyTemperature": 99.08839032397609,
  "BloodPressure": {
    "Systolic": 23,
    "Diastolic": 34
  },
  "Activity": "walking"
}
```
> [!IMPORTANT]
> Не забудьте, что отправляемое сообщение устройства должно соответствовать [шаблонам сопоставления](iot-mapping-templates.md), которые настроены для вашего соединителя "Azure IoT для FHIR".

## <a name="view-device-data-in-azure-api-for-fhir"></a>Просмотр данных устройства в Azure API для FHIR

Вы можете просматривать ресурсы наблюдения FHIR, созданные соединителем "Azure IoT для FHIR" в Azure API для FHIR, используя Postman. Настройте [доступ к Azure API для FHIR в Postman](access-fhir-postman-tutorial.md) и выполните запрос `GET` к `https://your-fhir-server-url/Observation?code=http://loinc.org|8867-4`, чтобы просмотреть ресурсы наблюдения FHIR с информацией о частоте пульса, отправленной ранее в тестовом сообщении.

> [!TIP]
> Убедитесь, что пользователь имеет необходимые права доступа к плоскости данных Azure API для FHIR. Для назначения ролей в плоскости данных используйте [управление доступом на основе ролей Azure (Azure RBAC)](configure-azure-rbac.md).


## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как настроить Центр Интернета вещей Azure для маршрутизации данных устройства в соединитель "Azure IoT для FHIR". Чтобы получить дополнительные сведения о соединителе "Azure IoT для FHIR", просмотрите следующие ресурсы.

Сведения о разных стадиях потока данных в соединителе "Azure IoT для FHIR".

>[!div class="nextstepaction"]
>[Поток данных для соединителя "Azure IoT для FHIR"](iot-data-flow.md)

Сведения о настройке соединителя Интернета вещей с помощью шаблонов устройства и сопоставления FHIR.

>[!div class="nextstepaction"]
>[Шаблоны сопоставления для соединителя "Azure IoT для FHIR"](iot-mapping-templates.md)

*На портале Azure соединитель Azure IoT для FHIR называется соединителем IoT (предварительная версия). FHIR — это зарегистрированная торговая марка организации HL7, которая используется с разрешения HL7.