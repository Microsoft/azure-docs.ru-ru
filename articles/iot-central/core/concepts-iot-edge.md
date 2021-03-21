---
title: Azure IoT Edge и Azure IoT Central | Документация Майкрософт
description: Узнайте, как использовать Azure IoT Edge с приложением IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 02/19/2021
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom:
- device-developer
- iot-edge
ms.openlocfilehash: e0f3464420c5cb429f780999bf5983b2ab142567
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102608637"
---
# <a name="connect-azure-iot-edge-devices-to-an-azure-iot-central-application"></a>Подключение устройств Azure IoT Edge к приложению Azure IoT Central

*Эта статья предназначена для создателей решений и разработчиков устройств.*

Azure IoT Edge перемещает облачную аналитику и настраиваемую бизнес-логику на устройства, чтобы ваша организация могла сосредоточиться на бизнес-аналитике вместо управления данными. Разворачивайте свое решение Интернета вещей, упаковывая бизнес-логику в стандартные контейнеры, развертывая эти контейнеры на устройствах и отслеживая их в облаке.

В этой статье рассматриваются следующие вопросы:

* Как IoT Edge устройства подключаются к приложению IoT Central.
* Использование IoT Central для управления устройствами IoT Edge.

Дополнительные сведения о IoT Edge см. в разделе [что такое Azure IOT Edge?](../../iot-edge/about-iot-edge.md)

## <a name="iot-edge"></a>IoT Edge

IoT Edge состоит из трех компонентов:

* *Модули IoT Edge* — это контейнеры, в которых выполняются службы Azure, партнерские службы или ваш собственный код. Модули развертываются на устройства IoT Edge и выполняются на них локально. Дополнительные сведения см. в разделе [Знакомство с Azure IOT Edge модулями](../../iot-edge/iot-edge-modules.md).
* *Среда выполнения IOT Edge* работает на каждом IOT Edge устройстве и управляет модулями, развернутыми на каждом устройстве. Среда выполнения состоит из двух модулей IoT Edge: *IOT Edge агента* и *IOT Edge концентратора*. Дополнительные сведения см. [в разделе Знакомство со средой выполнения Azure IOT EDGE и ее архитектурой](../../iot-edge/iot-edge-runtime.md).
* С помощью *облачного интерфейса* вы можете осуществлять мониторинг устройств IoT Edge и управлять ими удаленно. IoT Central является примером облачного интерфейса.

Устройство IoT Edge может быть следующим:

* Изолированное устройство, состоящее из модулей.
* *Устройство шлюза* с подключенными к нему подчиненными устройствами.

## <a name="iot-edge-as-a-gateway"></a>IoT Edge в качестве шлюза

Устройство IoT Edge может действовать в качестве шлюза, обеспечивающего подключение между другими нижестоящими устройствами в сети и приложением IoT Central.

Существует два шаблона шлюза:

* В шаблоне *прозрачного шлюза* модуль IOT Edge концентратора ведет себя как IOT Central и обрабатывает подключения от устройств, зарегистрированных в IOT Central. Сообщения передаются с подчиненных устройств в IoT Central, как если бы между ними нет шлюза.

* В шаблоне *шлюза перевода* устройства, которые не могут самостоятельно подключаться к IOT Central, подключаются к пользовательскому модулю IOT Edge. Модуль на IoT Edge устройстве обрабатывает входящие сообщения подчиненного устройства, а затем пересылает их IoT Centralу.

Прозрачные и преобразованные шаблоны шлюзов не являются взаимоисключающими. Одно устройство IoT Edge может работать как с прозрачным шлюзом, так и с шлюзом перевода.

Дополнительные сведения о шаблонах шлюза IoT Edge см. в разделе [как можно использовать IOT Edge устройство в качестве шлюза](../../iot-edge/iot-edge-as-gateway.md).

### <a name="downstream-device-relationships-with-a-gateway-and-modules"></a>Связи подчиненных устройств со шлюзом и модулями

Подчиненные устройства могут подключаться к устройству шлюза IoT Edge через модуль *IOT Edge концентратора*  . В этом сценарии IoT Edge устройство является прозрачным шлюзом:

:::image type="content" source="media/concepts-iot-edge/gateway-transparent.png" alt-text="Схема прозрачного шлюза" border="false":::

Подчиненные устройства также могут подключаться к устройству шлюза IoT Edge через настраиваемый модуль. В следующем сценарии подчиненные устройства подключаются через пользовательский модуль *Modbus* . В этом сценарии IoT Edge устройство является шлюзом перевода:

:::image type="content" source="media/concepts-iot-edge/gateway-module.png" alt-text="Схема подключения настраиваемого модуля" border="false":::

На следующей схеме показаны подключения к устройству шлюза IoT Edge через оба типа модулей. В этом сценарии устройство IoT Edge является прозрачным и шлюзом перевода.

:::image type="content" source="media/concepts-iot-edge/gateway-module-transparent.png" alt-text="Схема подключения с использованием обоих модулей подключения" border="false":::

Подчиненные устройства могут подключаться к устройству шлюза IoT Edge через несколько пользовательских модулей. На следующей схеме показаны подчиненные устройства, подключающиеся через пользовательский модуль Modbus, пользовательский модуль BLE и модуль *IOT Edge Hub*  :

:::image type="content" source="media/concepts-iot-edge/gateway-two-modules-transparent.png" alt-text="Схема подключения с использованием нескольких пользовательских модулей" border="false":::

## <a name="iot-edge-devices-and-iot-central"></a>IoT Edge устройств и IoT Central

IoT Edge устройства могут использовать маркеры *подписи общего доступа* или сертификаты X. 509 для проверки подлинности в IOT Central. Вы можете вручную зарегистрировать устройства IoT Edge в IoT Central перед их первым подключением или использовать службу подготовки устройств для регистрации. Дополнительные сведения см. в статье [Get connected to Azure IoT Central](concepts-get-connected.md) (Подключение к Azure IoT Central).

IoT Central использует [шаблоны устройств](concepts-device-templates.md) , чтобы определить, как IOT Central взаимодействует с устройством. Например, в шаблоне устройства указано:

* Типы телеметрии и свойств, которые отправляет устройство, чтобы IoT Central может интерпретировать их и создавать визуализации.
* Команды, на которые реагирует устройство, чтобы IoT Central мог отобразить пользовательский интерфейс для оператора, который будет использоваться для вызова команд.

Устройство IoT Edge может отсылать данные телеметрии, синхронизировать значения свойств и реагировать на команды так же, как стандартное устройство. Таким образом, устройству IoT Edge требуется шаблон устройства в IoT Central.

### <a name="iot-edge-device-templates"></a>Шаблоны устройств IoT Edge

IoT Central шаблоны устройств используют модели для описания возможностей устройств. На следующей схеме показана структура модели для устройства IoT Edge:

:::image type="content" source="media/concepts-iot-edge/iot-edge-model.png" alt-text="Структура модели для IoT Edge устройства, подключенного к IoT Central" border="false":::

IoT Central моделирует IoT Edge устройство следующим образом:

* Каждый шаблон устройства IoT Edge имеет модель возможностей.
* Для каждого пользовательского модуля, указанного в манифесте развертывания, создается модель возможностей модуля.
* Связь устанавливается между каждой моделью возможностей модуля и моделью устройства.
* Модель возможностей модуля реализует один или несколько интерфейсов модуля.
* Каждый интерфейс модуля содержит данные телеметрии, свойства и команды.

### <a name="iot-edge-deployment-manifests-and-iot-central-device-templates"></a>IoT Edge манифесты развертывания и шаблоны устройств IoT Central

В IoT Edge вы развертываете бизнес-логику и управляете ими в форме модулей. IoT Edge модули — это наименьшая единица вычислений, управляемая IoT Edge, и может содержать службы Azure, такие как Azure Stream Analytics, или собственный код для конкретного решения.

*Манифест развертывания* IOT Edge содержит список модулей IOT Edge для развертывания на устройстве и способы их настройки. Дополнительные сведения см. в статьях о [развертывании модулей и установке маршрутов в IOT Edge](../../iot-edge/module-composition.md).

В Azure IoT Central импортируйте манифест развертывания, чтобы создать шаблон устройства для IoT Edge устройства.

В следующем фрагменте кода показан пример манифеста развертывания IoT Edge:

```json
{
  "modulesContent": {
    "$edgeAgent": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "runtime": {
          "type": "docker",
          "settings": {
            "minDockerVersion": "v1.25",
            "loggingOptions": "",
            "registryCredentials": {}
          }
        },
        "systemModules": {
          "edgeAgent": {
            "type": "docker",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-agent:1.0.9",
              "createOptions": "{}"
            }
          },
          "edgeHub": {
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-hub:1.0.9",
              "createOptions": "{}"
            }
          }
        },
        "modules": {
          "SimulatedTemperatureSensor": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
              "createOptions": "{}"
            }
          }
        }
      }
    },
    "$edgeHub": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "routes": {
            "route": "FROM /* INTO $upstream"
        },
        "storeAndForwardConfiguration": {
          "timeToLiveSecs": 7200
        }
      }
    },
    "SimulatedTemperatureSensor": {
      "properties.desired": {
           "SendData": true,
           "SendInterval": 10
      }
    }
  }
}
```

В предыдущем фрагменте кода можно увидеть следующее:

* Существует три модуля. *IOT Edge агент* и модули системы *концентратора IOT Edge* , которые имеются в каждом манифесте развертывания. Пользовательский модуль **симулатедтемпературесенсор** .
* Образы общедоступного модуля извлекаются из репозитория реестра контейнеров Azure, для подключения которого не требуются учетные данные. Для частных образов модулей задайте учетные данные реестра контейнеров для использования в `registryCredentials` параметре модуля *агента IOT Edge* .
* Пользовательский модуль **симулатедтемпературесенсор** имеет два свойства `"SendData": true` и `"SendInterval": 10` .

При импорте манифеста развертывания в приложение IoT Central создается следующий шаблон устройства:

:::image type="content" source="media/concepts-iot-edge/device-template.png" alt-text="Снимок экрана, показывающий шаблон устройства, созданный из манифеста развертывания.":::

На предыдущем снимке экрана можно увидеть следующее:

* Модуль с именем **симулатедтемпературесенсор**. Модули *IOT Edge агента* и *концентратора IOT Edge* не отображаются в шаблоне.
* Интерфейс под названием **Управление** , который содержит два записываемых свойства с именами **сенддата** и **сендинтервал**.

Манифест развертывания не содержит сведений о данных телеметрии, которые отправляет модуль **симулатедтемпературесенсор** , или о командах, на которые он отвечает. Перед публикацией добавьте эти определения в шаблон устройства вручную.

Дополнительные сведения см. в разделе [учебник. добавление Azure IOT Edge устройства в приложение IOT Central Azure](tutorial-add-edge-as-leaf-device.md).

### <a name="update-a-deployment-manifest"></a>Обновление манифеста развертывания

При создании новой [версии](howto-version-device-template.md) шаблона устройства можно заменить манифест развертывания новой версией:

При замене манифеста развертывания все подключенные IoT Edge устройства загружают новый манифест и обновляют модули. Однако IoT Central не обновляет интерфейсы в шаблоне устройства с учетом изменений в конфигурации модуля. Например, если заменить манифест, показанный в предыдущем фрагменте, следующим манифестом, вы не увидите свойство **сендунитс** в интерфейсе **управления** в шаблоне устройства автоматически. Вручную добавьте новое свойство в интерфейс **управления** , чтобы IOT Central распознать его:

```json
{
  "modulesContent": {
    "$edgeAgent": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "runtime": {
          "type": "docker",
          "settings": {
            "minDockerVersion": "v1.25",
            "loggingOptions": "",
            "registryCredentials": {}
          }
        },
        "systemModules": {
          "edgeAgent": {
            "type": "docker",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-agent:1.0.9",
              "createOptions": "{}"
            }
          },
          "edgeHub": {
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-hub:1.0.9",
              "createOptions": "{}"
            }
          }
        },
        "modules": {
          "SimulatedTemperatureSensor": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
              "createOptions": "{}"
            }
          }
        }
      }
    },
    "$edgeHub": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "routes": {
            "route": "FROM /* INTO $upstream"
        },
        "storeAndForwardConfiguration": {
          "timeToLiveSecs": 7200
        }
      }
    },
    "SimulatedTemperatureSensor": {
      "properties.desired": {
           "SendData": true,
           "SendInterval": 10,
           "SendUnits": "Celsius"
      }
    }
  }
}
```

## <a name="deploy-the-iot-edge-runtime"></a>Развертывание среды выполнения IoT Edge

Сведения о том, где можно запустить среду выполнения IoT Edge, см. в разделе [Azure IOT Edge Поддерживаемые системы](../../iot-edge/support.md).

Также можно установить среду выполнения IoT Edge в следующих средах:

* [Установка или удаление Azure IoT Edge для Linux](../../iot-edge/how-to-install-iot-edge.md)
* [Установка и инициализация Azure IoT Edge для Linux на устройстве Windows (предварительная версия)](../../iot-edge/how-to-install-iot-edge-on-windows.md)
* [Запуск Azure IoT Edge на виртуальных машинах Ubuntu в Azure](../../iot-edge/how-to-install-iot-edge-ubuntuvm.md)

## <a name="iot-edge-gateway-devices"></a>Устройства шлюза IoT Edge

Если в качестве устройства шлюза выбрано устройство IoT Edge, можно добавить подчиненные отношения к моделям устройств для устройств, которые нужно подключить к устройству шлюза.

Дополнительные сведения см. в статье [Подключение устройств с помощью IOT Edge прозрачного шлюза](how-to-connect-iot-edge-transparent-gateway.md).

## <a name="next-steps"></a>Следующие шаги

Если вы являетесь разработчиком устройств, предлагаем следующий шаг: Узнайте, как [разрабатывать собственные модули IOT Edge](../../iot-edge/module-development.md).
