---
title: Управление сертификатами устройств — Azure IoT Edge | Документация Майкрософт
description: Создание тестовых сертификатов, установка и управление ими на Azure IoT Edge устройстве для подготовки к развертыванию в рабочей среде.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/01/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: e5c85d2c3049ea8718d0a9e0e574c13d0d99394c
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103200278"
---
# <a name="manage-certificates-on-an-iot-edge-device"></a>Управление сертификатами на IoT Edge устройстве

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

Все устройства IoT Edge используют сертификаты для создания безопасных соединений между средой выполнения и любыми модулями, работающими на устройстве. IoT Edge устройства работают, как шлюзы, используют те же сертификаты для подключения к их нижестоящим устройствам.

## <a name="install-production-certificates"></a>Установка рабочих сертификатов

При первой установке IoT Edge и подготовке устройства устройство настраивается с помощью временных сертификатов, что позволяет протестировать службу.
Срок действия этих временных сертификатов истекает через 90 дней, или их можно сбросить, перезапустив компьютер.
Когда вы перейдете в рабочий сценарий или хотите создать устройство шлюза, необходимо предоставить собственные сертификаты.
В этой статье описываются действия по установке сертификатов на устройства IoT Edge.

Дополнительные сведения о различных типах сертификатов и их ролях см. в разделе [понимание того, как Azure IOT Edge использует сертификаты](iot-edge-certs.md).

>[!NOTE]
>Термин "корневой ЦС", используемый в этой статье, относится к общедоступному центру сертификации цепочки сертификатов для вашего решения IoT. Не нужно использовать корневой каталог сертификатов в центре сертификации, или корневой центр сертификации вашей организации. Во многих случаях это фактический открытый сертификат промежуточного ЦС.

### <a name="prerequisites"></a>Предварительные требования

* Устройство IoT Edge.

  Если устройство IoT Edge не настроено, вы можете создать его на виртуальной машине Azure. Выполните действия, описанные в одной из кратких руководств, чтобы [создать виртуальное устройство Linux](quickstart-linux.md) или [создать виртуальное устройство Windows](quickstart.md).

* Наличие сертификата корневого центра сертификации (CA), самозаверяющего или приобретенного в доверенном коммерческом центре сертификации, например Baltimore, VeriSign, DigiCert или Глобалсигн.

  Если у вас еще нет корневого центра сертификации, но вы хотите испытать IoT Edge функции, для которых требуются производственные сертификаты (например, сценарии шлюза), можно [создать демонстрационные сертификаты для тестирования IOT Edgeных функций устройства](how-to-create-test-certificates.md).

### <a name="create-production-certificates"></a>Создание рабочих сертификатов

Для создания следующих файлов следует использовать собственный центр сертификации:

* Корневой ЦС
* Сертификат ЦС устройства
* Закрытый ключ ЦС устройства

В этой статье мы будем называть *корневой центр сертификации* , который не является самым верхним центром сертификации для Организации. Это самый верхний центр сертификации для сценария IoT Edge, который модуль центра IoT Edge, пользовательские модули и все подчиненные устройства используют для установления отношения доверия между собой.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

> [!NOTE]
> В настоящее время ограничение в либиоссм предотвращает использование сертификатов, срок действия которых истекает 1 января 2038.

:::moniker-end

Чтобы просмотреть пример этих сертификатов, просмотрите сценарии, создающие демонстрационные сертификаты в разделе [Управление сертификатами ЦС для примеров и учебников](https://github.com/Azure/iotedge/tree/master/tools/CACertificates).

### <a name="install-certificates-on-the-device"></a>Установка сертификатов на устройстве

Установите цепочку сертификатов на устройстве IoT Edge и настройте среду выполнения IoT Edge для ссылки на новые сертификаты.

Скопируйте три файла сертификата и ключей на устройство IoT Edge. Для перемещения файлов сертификатов можно использовать службу, например [хранилище ключей Azure](../key-vault/index.yml), или функцию, такую как [протокол защищенного копирования](https://www.ssh.com/ssh/scp/).  Если сертификаты созданы на самом устройстве IoT Edge, этот шаг можно пропустить и использовать путь к рабочему каталогу.

Например, если вы использовали примеры сценариев для [создания демонстрационных сертификатов](how-to-create-test-certificates.md), скопируйте следующие файлы на устройство IoT-Edge:

* Сертификат ЦС устройства: `<WRKDIR>\certs\iot-edge-device-MyEdgeDeviceCA-full-chain.cert.pem`
* Закрытый ключ ЦС устройства: `<WRKDIR>\private\iot-edge-device-MyEdgeDeviceCA.key.pem`
* Корневой ЦС: `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

1. Откройте файл конфигурации управляющей программы безопасности IoT Edge.

   * Windows: `C:\ProgramData\iotedge\config.yaml`
   * Linux: `/etc/iotedge/config.yaml`

1. Задайте для свойств **сертификата** в файле config. YAML путь универсального кода ресурса (URI) к файлам сертификата и ключа на устройстве IOT Edge. Удалите `#` символ перед свойствами сертификата, чтобы раскомментировать четыре строки. Убедитесь, что **сертификаты:** строка не имеет предшествующих пробелов, а вложенные элементы имеют отступ в два пробела. Например:

   * Windows:

      ```yaml
      certificates:
        device_ca_cert: "file:///C:/<path>/<device CA cert>"
        device_ca_pk: "file:///C:/<path>/<device CA key>"
        trusted_ca_certs: "file:///C:/<path>/<root CA cert>"
      ```

   * Linux:

      ```yaml
      certificates:
        device_ca_cert: "file:///<path>/<device CA cert>"
        device_ca_pk: "file:///<path>/<device CA key>"
        trusted_ca_certs: "file:///<path>/<root CA cert>"
      ```

1. На устройствах Linux убедитесь, что у пользователя **iotedge** есть разрешения на чтение каталога, в котором хранятся сертификаты.

1. Если вы использовали другие сертификаты для IoT Edge на устройстве, перед запуском или перезапуском IoT Edge удалите эти файлы в следующих двух каталогах:

   * Windows: `C:\ProgramData\iotedge\hsm\certs` и `C:\ProgramData\iotedge\hsm\cert_keys`

   * Linux: `/var/lib/iotedge/hsm/certs` и `/var/lib/iotedge/hsm/cert_keys`
:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

1. Откройте файл конфигурации управляющей программы IoT Edge Security: `/etc/aziot/config.toml`

1. Найдите `trust_bundle_cert` параметр в начале файла. Раскомментируйте эту строку и укажите URI файла для сертификата корневого ЦС на устройстве.

   ```toml
   trust_bundle_cert = "file:///<path>/<root CA cert>"
   ```

1. Найдите `[edge_ca]` раздел в файле config. томл. Раскомментируйте строки в этом разделе и укажите пути URI файлов для сертификата и файлов ключей на устройстве IoT Edge.

   ```toml
   [edge_ca]
   cert = "file:///<path>/<device CA cert>"
   pk = "file:///<path>/<device CA key>"
   ```

1. Убедитесь, что у пользователя **iotedge** есть разрешения на чтение для каталога, в котором хранятся сертификаты.

1. Если вы использовали другие сертификаты для IoT Edge на устройстве, перед запуском или перезапуском IoT Edge удалите эти файлы в следующих двух каталогах:

   * `/var/lib/aziot/certd/certs`
   * `/var/lib/aziot/keyd/keys`

:::moniker-end
<!-- end 1.2 -->

<!-- 1.1. -->
<!-- Temporarily, customizable certificate lifetime not available in 1.2. Update before GA. -->
:::moniker range="iotedge-2018-06"

## <a name="customize-certificate-lifetime"></a>Настроить время существования сертификата

IoT Edge автоматически создает сертификаты на устройстве в нескольких случаях, в том числе:

* Если вы не предоставляете собственные рабочие сертификаты при установке и подготовке IoT Edge, диспетчер безопасности IoT Edge автоматически создает **сертификат ЦС устройства**. Этот самозаверяющий сертификат предназначен только для сценариев разработки и тестирования, а не для рабочей среды. Срок действия этого сертификата истекает через 90 дней.
* Диспетчер IoT Edge безопасности также создает **сертификат ЦС рабочей нагрузки** , подписанный сертификатом ЦС устройств.

Дополнительные сведения о функции различных сертификатов на IoT Edge устройстве см. в разделе [понимание того, как Azure IOT Edge использует сертификаты](iot-edge-certs.md).

Для этих двух автоматически создаваемых сертификатов имеется возможность установки флага **auto_generated_ca_lifetime_days** в файле конфигурации для настройки количества дней существования сертификатов.

>[!NOTE]
>Существует третий автоматически созданный сертификат, создаваемый диспетчером IoT Edge безопасности, **сертификатом сервера центра IOT Edge**. Этот сертификат всегда имеет время жизни 90 дня, но автоматически обновляется до истечения срока действия. Значение **auto_generated_ca_lifetime_days** не влияет на этот сертификат.

После истечения срока действия через указанное число дней IoT Edge необходимо перезапустить, чтобы повторно создать сертификат ЦС устройства. Сертификат ЦС устройства не будет продлен автоматически.

1. Чтобы настроить истечение срока действия сертификата, отличное от 90 дней по умолчанию, добавьте значение в поле "дни" в раздел " **Сертификаты** " файла конфигурации.

   ```yaml
   certificates:
     device_ca_cert: "<ADD URI TO DEVICE CA CERTIFICATE HERE>"
     device_ca_pk: "<ADD URI TO DEVICE CA PRIVATE KEY HERE>"
     trusted_ca_certs: "<ADD URI TO TRUSTED CA CERTIFICATES HERE>"
     auto_generated_ca_lifetime_days: <value>
   ```

   > [!NOTE]
   > В настоящее время ограничение в либиоссм предотвращает использование сертификатов, срок действия которых истекает 1 января 2038.

1. Удалите содержимое `hsm` папки, чтобы удалить ранее созданные сертификаты.

   Windows: `C:\ProgramData\iotedge\hsm\certs` и `C:\ProgramData\iotedge\hsm\cert_keys` Linux: `/var/lib/iotedge/hsm/certs` и `/var/lib/iotedge/hsm/cert_keys`

1. Перезапустите службу IoT Edge.

   Windows:

   ```powershell
   Restart-Service iotedge
   ```

   Linux:

   ```bash
   sudo systemctl restart iotedge
   ```

1. Подтвердите настройку времени существования.

   Windows:

   ```powershell
   iotedge check --verbose
   ```

   Linux:

   ```bash
   sudo iotedge check --verbose
   ```

   Проверьте выходные данные в окне " **готовность к рабочей среде": Проверка сертификатов** , в которой указывается количество дней до истечения срока действия сертификатов автоматически созданного ЦС.

:::moniker-end
<!-- end 1.1 -->

<!-- 
<!-- 1.2 --
:::moniker range=">=iotedge-2020-11"

1. To configure the certificate expiration to something other than the default 90 days, add the value in days to the **certificates** section of the config file.

   ```toml
   [certificates]
   device_ca_cert = "<ADD URI TO DEVICE CA CERTIFICATE HERE>"
   device_ca_pk = "<ADD URI TO DEVICE CA PRIVATE KEY HERE>"
   trusted_ca_certs = "<ADD URI TO TRUSTED CA CERTIFICATES HERE>"
   auto_generated_ca_lifetime_days = <value>
   ```

1. Delete the contents of the `certd` and `keyd` folders to remove any previously generated certificates: `/var/lib/aziot/certd/certs` `/var/lib/aziot/keyd/keys`

1. Restart IoT Edge.

   ```bash
   sudo iotedge system restart
   ```

1. Confirm the new lifetime setting.

   ```bash
   sudo iotedge check --verbose
   ```

   Check the output of the **production readiness: certificates** check, which lists the number of days until the automatically generated device CA certificates expire.
:::moniker-end
<!-- end 1.2 --
-->

## <a name="next-steps"></a>Дальнейшие действия

Установка сертификатов на устройстве IoT Edge является необходимым этапом перед развертыванием решения в рабочей среде. Узнайте больше о том, как [подготовиться к развертыванию решения IOT EDGE в рабочей среде](production-checklist.md).
