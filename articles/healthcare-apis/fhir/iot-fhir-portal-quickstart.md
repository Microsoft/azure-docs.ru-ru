---
title: Краткое руководство. Развертывание соединителя "Azure IoT для FHIR" (предварительная версия) на портале Azure
description: В этом кратком руководстве показано, как развернуть, настроить и использовать функцию соединителя "Azure IoT для FHIR" в Azure API для FHIR с помощью портала Azure.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: quickstart
ms.date: 04/05/2021
ms.author: punagpal
ms.openlocfilehash: 0a382ad948e7fd2efc6ab59eb94da3d6cbb210d7
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106443783"
---
# <a name="quickstart-deploy-azure-iot-connector-for-fhir-preview-using-azure-portal"></a>Краткое руководство. Развертывание соединителя "Azure IoT для FHIR" (предварительная версия) на портале Azure

Соединитель Azure IoT для Ресурсов быстрого взаимодействия в сфере здравоохранения (FHIR&#174;)* — это дополнительная функция Azure API для FHIR, которая позволяет принимать данные из Интернета медицинских вещей (Internet of Medical Things, IoMT). На этапе предварительной версии функция соединителя "Azure IoT для FHIR" доступна бесплатно. В этом кратком руководстве описано следующее:
- Развертывание и настройка соединителя "Azure IoT для FHIR" с помощью портала Azure.
- Использование имитированного устройства для отправки данных в соединитель "Azure IoT для FHIR".
- Просмотр ресурсов, созданных соединителем "Azure IoT для FHIR" в Azure API для FHIR.

## <a name="prerequisites"></a>Предварительные требования

- Активная подписка Azure — [создайте подписку бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Ресурс Azure API для FHIR — [разверните Azure API для FHIR с помощью портала Azure](fhir-paas-portal-quickstart.md).

## <a name="go-to-azure-api-for-fhir-resource"></a>Переход к ресурсу Azure API для FHIR.

Откройте [портал Azure](https://portal.azure.com) и перейдите к ресурсу **Azure API для FHIR**, для которого вы хотите создать функцию соединителя "Azure IoT для FHIR".

[![Ресурс Azure API для FHIR](media/quickstart-iot-fhir-portal/portal-azure-api-fhir.jpg)](media/quickstart-iot-fhir-portal/portal-azure-api-fhir.jpg#lightbox)

В меню навигации слева щелкните **соединитель IoT (предварительная версия)** в разделе **Надстройки**, чтобы открыть страницу **Соединители IoT**.

[![Функция соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-connectors.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connectors.jpg#lightbox)

## <a name="create-new-azure-iot-connector-for-fhir-preview"></a>Создание соединителя "Azure IoT для FHIR" (предварительная версия)

Нажмите кнопку **Добавить**, чтобы открыть страницу **создания соединителя IoT**.

[![Добавление соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-connectors-add.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connectors-add.jpg#lightbox)

Введите параметры нового соединителя "Azure IoT для FHIR". Нажмите кнопку **Создать** и дождитесь развертывания соединителя "Azure IoT для FHIR".

> [!NOTE]
> Выберите **Создать** в качестве значения в раскрывающемся списке **Тип разрешения** для этой установки. 

[![Создание соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-create.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-create.jpg#lightbox)

|Параметр|Значение|Описание |
|---|---|---|
|Имя соединителя|Уникальное имя|Введите имя, чтобы определить соединитель "Azure IoT для FHIR". Это имя должно быть уникальным в пределах ресурса Azure API для FHIR. Имя может содержать только строчные буквы, цифры и знак дефиса (-). Оно должно начинаться и заканчиваться буквой или цифрой и содержать от 3 до 24 символов.|
|Тип разрешения|Подстановка или создание|Выберите **Подстановка**, если вы используете отдельный процесс для создания ресурсов FHIR [Устройство](https://www.hl7.org/fhir/device.html) или [Пациент](https://www.hl7.org/fhir/patient.html) в Azure API для FHIR. Соединитель "Azure IoT для FHIR" будет использовать ссылки на эти ресурсы при создании ресурса FHIR [Наблюдение](https://www.hl7.org/fhir/observation.html) для представления данных устройства. Выберите **Создать**, если вы хотите, чтобы соединитель "Azure IoT для FHIR" создал простые ресурсы "Устройство" и "Пациент" в Azure API для FHIR, используя соответствующие значения идентификатора, имеющиеся в данных устройствах.|

После завершения установки вновь созданный соединитель "Azure IoT для FHIR" будет отображаться на странице **Соединители IoT**.

[![Созданный соединитель IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-created.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-created.jpg#lightbox)

## <a name="configure-azure-iot-connector-for-fhir-preview"></a>Настройка соединителя "Azure IoT для FHIR" (предварительная версия)

Соединителю "Azure IoT для FHIR" требуется два шаблона сопоставления, чтобы преобразовать сообщения устройства в ресурсы наблюдения FHIR: **сопоставление устройств** и **сопоставление FHIR**. Соединитель "Azure IoT для FHIR" не будет полностью работоспособным, пока эти сопоставления не будут отправлены.

[![Соединитель IoT с отсутствующими сопоставлениями](media/quickstart-iot-fhir-portal/portal-iot-connector-missing-mappings.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-missing-mappings.jpg#lightbox)

Чтобы отправить шаблоны сопоставления, щелкните только что развернутый соединитель "Azure IoT для FHIR", чтобы открыть страницу **соединителя IoT**.

[![Выбор соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-click.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-click.jpg#lightbox)

#### <a name="device-mapping"></a>Сопоставление устройств

Шаблон сопоставления устройств преобразует данные устройства в нормализованную схему. На странице **соединителя IoT** выберите **Configure device mapping** (Настройка сопоставления устройств), чтобы открыть страницу **сопоставления устройств**. 

[![Выбор параметра настройки сопоставления устройств на странице соединителе IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-click-device-mapping.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-click-device-mapping.jpg#lightbox)

На странице **сопоставление устройств** добавьте следующий скрипт в редактор JSON и нажмите кнопку **Сохранить**.

```json
{
  "templateType": "CollectionContent",
  "template": [
    {
      "templateType": "IotJsonPathContent",
      "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@Body.telemetry.HeartRate)]",
        "patientIdExpression": "$.Properties.iotcentral-device-id",
        "values": [
          {
            "required": "true",
            "valueExpression": "$.Body.telemetry.HeartRate",
            "valueName": "hr"
          }
        ]
      }
    }
  ]
}
```

[![Сопоставление устройств соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-device-mapping.jpg)](media/quickstart-iot-fhir-portal/portal-iot-device-mapping.jpg#lightbox)

#### <a name="fhir-mapping"></a>Сопоставление FHIR

Шаблон сопоставления FHIR преобразует нормализованное сообщение в ресурс наблюдения FHIR. На странице **соединителя IoT** щелкните **Configure FHIR mapping** (Настройка сопоставления FHIR), чтобы открыть страницу **сопоставления FHIR**.  

[![Выбор параметра настройки сопоставления FHIR на странице соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-click-fhir-mapping.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-click-fhir-mapping.jpg#lightbox)

На странице **FHIR mapping** (Сопоставление FHIR) добавьте следующий скрипт в редактор JSON и нажмите кнопку **Сохранить**.

```json
{
  "templateType": "CollectionFhir",
  "template": [
    {
      "templateType": "CodeValueFhir",
      "template": {
        "codes": [
          {
            "code": "8867-4",
            "system": "http://loinc.org",
            "display": "Heart rate"
          }
        ],
        "periodInterval": 0,
        "typeName": "heartrate",
        "value": {
          "unit": "count/min",
          "valueName": "hr",
          "valueType": "Quantity"
        }
      }
    }
  ]
}
```

[![Сопоставление FHIR на странице соединителе IoT](media/quickstart-iot-fhir-portal/portal-iot-fhir-mapping.jpg)](media/quickstart-iot-fhir-portal/portal-iot-fhir-mapping.jpg#lightbox)

## <a name="generate-a-connection-string"></a>Создание строки подключения

Устройству IoMT для подключения и отправки сообщений в соединитель "Azure IoT для FHIR" требуется строка подключения. На странице **соединителя IoT** для вновь развернутого соединителя "Azure IoT для FHIR" нажмите кнопку **Manage client connections** (Управление клиентскими подключениями). 

[![Выбор параметра управления клиентскими подключениями в соединителе IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-click-client-connections.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-click-client-connections.jpg#lightbox)

На странице **Подключения** нажмите кнопку **Добавить**, чтобы создать соединение. 

[![Подключения соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-connections.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connections.jpg#lightbox)

Укажите понятное имя для этого подключения в окне наложения и нажмите кнопку **Создать**.

[![Новое подключение соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-new-connection.jpg)](media/quickstart-iot-fhir-portal/portal-iot-new-connection.jpg#lightbox)

Выберите только что созданное подключение на странице **Подключения** и скопируйте значение поля **Первичная строка подключения** из окна наложения справа.

[![Строка подключения соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-connection-string.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connection-string.jpg#lightbox)

Эта строка подключения будет использоваться на следующем шаге. 

## <a name="connect-your-devices-to-iot"></a>Подключение устройств к Интернету вещей

Azure предоставляет обширный набор продуктов Интернета вещей для подключения устройств Интернета вещей и управления ими. Вы можете создать собственное решение на основе PaaS с помощью Центра Интернета вещей Azure или начать использовать платформу управления приложениями Интернета вещей с помощью Azure IoT Central. В этом учебнике мы будем использовать Azure IoT Central, содержащий отраслевые шаблоны решений, которые помогут вам приступить к работе.

Разверните [шаблон приложения для непрерывного мониторинга состояния пациентов](../../iot-central/healthcare/tutorial-continuous-patient-monitoring.md#create-an-application-template). Этот шаблон включает в себя два имитированных устройства, которые создают данные в режиме реального времени, чтобы помочь вам приступить к работе: **Smart Vitals Patch** и **Smart Knee Brace**.

> [!NOTE]
> Когда реальные устройства готовы, вы можете использовать одно приложение IoT Central для [подключения устройств](../../iot-central/core/howto-set-up-template.md) и замены симуляторов устройств. Данные устройства будут автоматически поступать во FHIR. 

## <a name="connect-your-iot-data-with-the-azure-iot-connector-for-fhir-preview"></a>Подключение данных Интернета вещей Azure к соединителю "Azure IoT для FHIR" (предварительная версия)

После развертывания приложения IoT Central два готовых виртуальных устройства начнут создавать данные телеметрии. В рамках этого учебника мы будем получать данные телеметрии из симулятора *Smart Vitals Patch* в FHIR через соединитель "Azure IoT для FHIR". Чтобы экспортировать данные Интернета вещей в соединитель "Azure IoT для FHIR", необходимо [настроить непрерывный экспорт данных в IoT Central](../../iot-central/core/howto-export-data.md). Сначала необходимо создать подключение к назначению, а затем создать задание экспорта данных для непрерывного запуска: 

> [!NOTE]
> Вы хотите выбрать **Экспорт данных** и **Экспорт данных (прежние версии)** в IOT Central параметров приложения для этого раздела.

[![IoT Central параметры экспорта данных](media/quickstart-iot-fhir-portal/iot-central-data-export-dashboard.png)](media/quickstart-iot-fhir-portal/iot-central-data-export-dashboard.png#lightbox)

Создайте новое назначение:
- Перейдите на вкладку **назначения** и создайте новое место назначения.
- Начните с присвоения целевому объекту уникального имени.
- Выберите *концентраторы событий Azure* в качестве типа назначения.
- Укажите соединитель Azure IoT для строки подключения FHIR, полученной на предыдущем шаге, для поля **строки подключения** .

Создать новый экспорт данных:
- После создания назначения перейдите на вкладку **EXPORTS (экспорты** ) и создайте новый экспорт данных. 
- Начните с присвоения экспорту данных уникального имени.
- В разделе **данных выберите данные** *телеметрии* в качестве *типа экспортируемых данных*.
- В разделе **назначение** выберите имя назначения, созданное в предыдущем имени.

## <a name="view-device-data-in-azure-api-for-fhir"></a>Просмотр данных устройства в Azure API для FHIR

Вы можете просматривать ресурсы наблюдения FHIR, созданные соединителем "Azure IoT для FHIR" в Azure API для FHIR, используя Postman. Настройте [доступ к Azure API для FHIR в Postman](access-fhir-postman-tutorial.md) и выполните запрос `GET` к `https://your-fhir-server-url/Observation?code=http://loinc.org|8867-4`, чтобы просмотреть ресурсы наблюдения FHIR с информацией о частоте пульса. 

> [!TIP]
> Убедитесь, что пользователь имеет необходимые права доступа к плоскости данных Azure API для FHIR. Для назначения ролей в плоскости данных используйте [управление доступом на основе ролей Azure (Azure RBAC)](configure-azure-rbac.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если экземпляр соединителя "Azure IoT для FHIR" больше не нужен, его можно удалить, удалив связанную группу ресурсов, соответствующую службу Azure API для FHIR или сам экземпляр соединителя "Azure IoT для FHIR". 

Чтобы напрямую удалить экземпляр соединителя "Azure IoT для FHIR", выберите экземпляр на странице **соединителей IoT**, чтобы перейти на страницу **соединителя IoT**, и нажмите кнопку **Удалить**. Нажмите кнопку **Да** при запросе на подтверждение. 

[![Удаление экземпляра соединителя IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-delete.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-delete.jpg#lightbox)

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы развернули функцию соединителя "Azure IoT для FHIR" в ресурсе Azure API для FHIR. Чтобы получить дополнительные сведения о соединителе "Azure IoT для FHIR", просмотрите следующие ресурсы.

Сведения о разных стадиях потока данных в соединителе "Azure IoT для FHIR".

>[!div class="nextstepaction"]
>[Поток данных для соединителя "Azure IoT для FHIR"](iot-data-flow.md)

Сведения о настройке соединителя Интернета вещей с помощью шаблонов устройства и сопоставления FHIR.

>[!div class="nextstepaction"]
>[Шаблоны сопоставления для соединителя "Azure IoT для FHIR"](iot-mapping-templates.md)

*На портале Azure соединитель Azure IoT для FHIR называется соединителем IoT (предварительная версия). FHIR — это зарегистрированная торговая марка организации HL7, которая используется с разрешения HL7.
