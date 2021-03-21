---
title: Как развернуть модуль двойника OPC для Azure с нуля | Документация Майкрософт
description: В этой статье описывается, как развернуть OPC двойника с нуля с помощью колонки IoT Edge портал Azure, а также с помощью команды AZ CLI.
author: dominicbetts
ms.author: dobett
ms.date: 11/26/2018
ms.topic: conceptual
ms.service: industrial-iot
ms.custom: devx-track-azurecli
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: 38235f9b01b321e27664ee837763732971f0b85c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102201504"
---
# <a name="deploy-opc-twin-module-and-dependencies-from-scratch"></a>Развертывание модуля и зависимостей OPC двойника с нуля

> [!IMPORTANT]
> Актуальную информацию по этой теме см. в статье [Промышленный Интернет вещей в Azure](https://azure.github.io/Industrial-IoT/).

Модуль OPC двойника выполняется на IoT Edge и предоставляет несколько служб пограничных услуг для двойникаов устройств OPC и служб реестра. 

Существует несколько вариантов развертывания модулей в шлюзе [Azure IOT Edge](https://azure.microsoft.com/services/iot-edge/) .

- [Развертывание из колонки IoT Edge портал Azure](../iot-edge/how-to-deploy-modules-portal.md)
- [Развертывание с помощью AZ CLI](../iot-edge/how-to-deploy-cli-at-scale.md)

> [!NOTE]
> Дополнительные сведения о развертывании и инструкциях см. в [репозитории](https://github.com/Azure/azure-iiot-components)GitHub.

## <a name="deployment-manifest"></a>Манифест развертывания

Все модули развертываются с помощью манифеста развертывания.  Ниже приведен пример манифеста для развертывания как [издателя OPC](https://github.com/Azure/iot-edge-opc-publisher) , так и [OPC двойника](https://github.com/Azure/azure-iiot-opc-twin-module) .

```json
{
  "content": {
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
                "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
                "createOptions": ""
              }
            },
            "edgeHub": {
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}], \"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
              }
            }
          },
          "modules": {
            "opctwin": {
              "version": "1.0",
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/iotedge/opc-twin:latest",
                "createOptions": "{\"NetworkingConfig\": {\"EndpointsConfig\": {\"host\": {}}}, \"HostConfig\": {\"NetworkMode\": \"host\" }}"
              }
            },
            "opcpublisher": {
              "version": "2.0",
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/iotedge/opc-publisher:latest",
                "createOptions": "{\"Hostname\":\"publisher\",\"Cmd\":[\"publisher\",\"--pf=./pn.json\",\"--di=60\",\"--tm\",\"--aa\",\"--si=0\",\"--ms=0\"],\"ExposedPorts\":{\"62222/tcp\":{}},\"NetworkingConfig\":{\"EndpointsConfig\":{\"host\":{}}},\"HostConfig\":{\"NetworkMode\":\"host\",\"PortBindings\":{\"62222/tcp\":[{\"HostPort\":\"62222\"}]}}}"
              }
            }
          }
        }
      },
      "$edgeHub": {
        "properties.desired": {
          "schemaVersion": "1.0",
          "routes": {
            "opctwinToIoTHub": "FROM /messages/modules/opctwin/* INTO $upstream",
            "opcpublisherToIoTHub": "FROM /messages/modules/opcpublisher/* INTO $upstream"
          },
          "storeAndForwardConfiguration": {
            "timeToLiveSecs": 7200
          }
        }
      }
    }
  }
}
```

## <a name="deploying-from-azure-portal"></a>Развертывание из портал Azure

Самый простой способ развертывания модулей на Azure IoT Edge устройстве шлюза — это портал Azure.  

### <a name="prerequisites"></a>Предварительные условия

1. Развернуть [зависимости](howto-opc-twin-deploy-dependencies.md) OPC двойника и получить получившийся `.env` файл. Обратите внимание на развернутую `hub name` `PCS_IOTHUBREACT_HUB_NAME` переменную в результирующем `.env` файле.

2. Зарегистрируйте и запустите шлюз [Linux](../iot-edge/how-to-install-iot-edge.md) или [Windows](../iot-edge/how-to-install-iot-edge.md) IOT EDGE и запишите его `device id` .

### <a name="deploy-to-an-edge-device"></a>Развертывание на пограничном устройстве

1. Войдите на [портал Azure](https://portal.azure.com/) и перейдите к своему Центру Интернета вещей.

2. Выберите **IOT Edge** в меню слева.

3. Щелкните идентификатор целевого устройства в списке устройств.

4. Выберите **задать модули**.

5. В разделе **модули развертывания** страницы выберите **Добавить** и **IOT Edge модуль.**

6. В диалоговом окне **IOT Edge пользовательский модуль** используйте в `opctwin` качестве имени для модуля, а затем укажите *универсальный код ресурса (URI) образа* контейнера.

   ```bash
   mcr.microsoft.com/iotedge/opc-twin:latest
   ```

   В качестве *параметров создания контейнера* используйте следующий код JSON:

   ```json
   {"NetworkingConfig": {"EndpointsConfig": {"host": {}}}, "HostConfig": {"NetworkMode": "host" }}
   ```

   При желании заполните необязательные поля. Дополнительные сведения о возможностях при создании контейнера, политике перезапуска и требуемом состоянии см. в разделе [Требуемые свойства EdgeAgent](../iot-edge/module-edgeagent-edgehub.md#edgeagent-desired-properties). Подробные сведения о двойниках модулей см. в разделе [Определение или обновление требуемых свойств](../iot-edge/module-composition.md#define-or-update-desired-properties).

7. Выберите **сохранить** и повторите шаг **5**.  

8. В диалоговом окне IoT Edge настраиваемый модуль используйте в `opcpublisher` качестве имени модуля, а *URI образа* контейнера — как 

   ```bash
   mcr.microsoft.com/iotedge/opc-publisher:latest
   ```

   В качестве *параметров создания контейнера* используйте следующий код JSON:

   ```json
   {"Hostname":"publisher","Cmd":["publisher","--pf=./pn.json","--di=60","--tm","--aa","--si=0","--ms=0"],"ExposedPorts":{"62222/tcp":{}},"HostConfig":{"PortBindings":{"62222/tcp":[{"HostPort":"62222"}] }}}
   ```

9. Щелкните **сохранить** , а затем **Далее** , чтобы перейти к разделу маршруты.

10. На вкладке Routes (маршруты) Вставьте следующий текст 

    ```json
    {
      "routes": {
        "opctwinToIoTHub": "FROM /messages/modules/opctwin/* INTO $upstream",
        "opcpublisherToIoTHub": "FROM /messages/modules/opcpublisher/* INTO $upstream"
      }
    }
    ```

    и нажмите кнопку **Далее** .

11. Проверьте сведения о развертывании и манифест.  Он должен выглядеть, как в приведенном выше манифесте развертывания.  Нажмите кнопку **Submit** (Отправить).

12. Завершив развертывание модулей на устройстве, вы можете просмотреть их на странице портала **Сведения об устройствах**. Эта страница отображает имя каждого развернутого модуля и полезную информацию о нем, включая состояние развертывания и код завершения.

## <a name="deploying-using-azure-cli"></a>Развертывание с помощью Azure CLI

### <a name="prerequisites"></a>Предварительные условия

1. Установите последнюю версию [интерфейса командной строки Azure (AZ)](/cli/azure/) [отсюда](/cli/azure/install-azure-cli).

### <a name="quickstart"></a>Краткое руководство

1. Сохраните приведенный выше манифест развертывания в `deployment.json` файл.  

2. Следующая команда применяет конфигурацию к устройству IoT Edge:

   ```azurecli
   az iot edge set-modules --device-id [device id] --hub-name [hub name] --content ./deployment.json
   ```

   `device id`Параметр учитывает регистр. Параметр content указывает на сохраненный ранее файл манифеста развертывания. 
    ![AZ IoT Edge Set-modules Output](/azure/iot-edge/media/how-to-deploy-cli/set-modules.png)

3. Завершив развертывание модулей на устройстве, вы можете просмотреть их список с помощью следующей команды:

   ```azurecli
   az iot hub module-identity list --device-id [device id] --hub-name [hub name]
   ```

   В параметре идентификатора устройства учитывается регистр. ![Выходные данные команды az iot hub module-identity list](/azure/iot-edge/media/how-to-deploy-cli/list-modules.png)

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как развернуть OPC двойника с нуля, предлагаем следующий шаг:

> [!div class="nextstepaction"]
> [Развертывание OPC двойника в существующем проекте](howto-opc-twin-deploy-existing.md)