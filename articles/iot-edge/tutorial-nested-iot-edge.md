---
title: Учебник. Создание иерархии устройств IoT Edge — Azure IoT Edge
description: В этом учебнике показано, как создать иерархическую структуру устройств IoT Edge с помощью шлюзов.
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 2/26/2021
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
monikerRange: '>=iotedge-2020-11'
ms.openlocfilehash: c1b30a1eafe9af92c1ef3f81773d213ccf96555c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103462037"
---
# <a name="tutorial-create-a-hierarchy-of-iot-edge-devices-preview"></a>Руководство по созданию иерархии устройств IoT Edge (предварительная версия)

[!INCLUDE [iot-edge-version-202011](../../includes/iot-edge-version-202011.md)]

Разверните узлы Azure IoT Edge по сетям, организованным в виде иерархических слоев. Каждый слой в иерархии — это устройство шлюза, которое обрабатывает сообщения и запросы от устройств в слое под ним.

>[!NOTE]
>Для этого компонента требуется служба IoT Edge 1.2, которая предоставляется в общедоступной предварительной версии, в которой выполняются контейнеры Linux.

Структурировать иерархию устройств можно так, чтобы только верхний слой имел возможность подключения к облаку, а нижние слои могли взаимодействовать только с соседними северным и южным слоями. Такое сетевое наложение — это основа большинства промышленных сетей, соответствующих стандарту [ISA-95](https://en.wikipedia.org/wiki/ANSI/ISA-95).

Цель этого учебника — создать иерархию устройств IoT Edge, имитирующих рабочую среду. В итоге вы развернете [моделируемый модуль датчика температуры](https://azuremarketplace.microsoft.com/marketplace/apps/azure-iot.simulated-temperature-sensor) на устройстве нижнего слоя без доступа к Интернету, загрузив образы контейнеров в иерархию.

Для этого в этом учебнике рассматривается создание иерархии устройств IoT Edge, развертывание контейнеров среды выполнения IoT Edge на устройствах и настройка локальных устройств. В этом руководстве вы узнаете, как:

> [!div class="checklist"]
>
> * создавать и определять связи в иерархии устройств IoT Edge;
> * настраивать среду выполнения IoT Edge на устройствах в иерархии;
> * устанавливать согласованные сертификаты в иерархии устройств;
> * добавлять рабочие нагрузки на устройства в иерархии;
> * использовать [модуль прокси-сервера API IoT Edge](https://azuremarketplace.microsoft.com/marketplace/apps/azure-iot.azureiotedge-api-proxy?tab=Overview) для безопасной маршрутизации трафика HTTP через один порт на устройствах нижнего слоя.

В этом учебнике определены следующие слои сети.

* **Верхний слой**. Устройства IoT Edge на этом слое могут подключаться непосредственно к облаку.

* **Нижние слои**. Устройства IoT Edge на слоях, расположенных под верхним слоем, не могут подключаться к облаку напрямую. Для отправки и получения данных им нужно миновать одно или несколько промежуточных устройств IoT Edge.

Для простоты в этом учебнике используется иерархия двух устройств, показанная ниже. Первое **устройство верхнего слоя** представляет устройство на верхнем слое иерархии, которое может подключаться непосредственно к облаку. Это устройство также будет называться **родительским устройством**. Второе **устройство нижнего слоя** представляет устройство на нижнем слое иерархии, которое не может подключаться непосредственно к облаку. При необходимости для представления рабочей среды можно добавить дополнительные устройства нижнего слоя. Устройства на более низких слоях также называются **дочерними устройствами**. Конфигурация дополнительных устройств нижнего слоя будет соответствовать конфигурации **устройства нижнего слоя**.

![Структура иерархии учебника, содержащая два устройства: устройство верхнего слоя и устройство нижнего слоя](./media/tutorial-nested-iot-edge/tutorial-hierarchy-diagram.png)

## <a name="prerequisites"></a>Обязательные условия

Чтобы создать иерархию устройств IoT Edge, потребуются следующие ресурсы:

* Компьютер (Windows или Linux) с подключением к Интернету.
* Учетная запись Azure с действительной подпиской. Если у вас еще нет [подписки Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начать работу.
* [Центр Интернета вещей](../iot-hub/iot-hub-create-through-portal.md) ценовой категории "Бесплатный" или "Стандартный" в Azure.
* Azure CLI версии 2.3.1 с установленным расширением Azure IoT версии 0.10.6 или более поздней версии. В этом учебнике используется [Azure Cloud Shell](../cloud-shell/overview.md). Если вы не работали с Azure Cloud Shell, [ознакомьтесь с этим кратким руководством](./quickstart-linux.md#prerequisites).
  * Чтобы просмотреть текущие версии расширений и модулей Azure CLI, выполните команду [az version](/cli/azure/reference-index?#az_version).
* Устройство Linux для настройки в качестве устройства IoT Edge для каждого устройства в иерархии. В этом учебнике используются два устройства. Если у вас нет доступных устройств, можно создать виртуальные машины Azure для каждого устройства в иерархии, заменив текст заполнителя в следующей команде и выполнив ее:

   ```azurecli-interactive
   az vm create \
    --resource-group <REPLACE_WITH_RESOURCE_GROUP> \
    --name <REPLACE_WITH_UNIQUE_NAMES_FOR_EACH_VM> \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password <REPLACE_WITH_PASSWORD>
   ```

* Убедитесь, что для входящего трафика открыты порты 8000, 443, 5671, 8883.
  * 8000: используется для извлечения образов контейнеров Docker через прокси-сервер API.
  * 443: используется между родительским и дочерним центрами Edge для вызовов REST API.
  * 5671, 8883: используются протоколами AMQP и MQTT.

  Дополнительные сведения см. в статье [Как открыть порты для виртуальной машины Windows с помощью портала Azure](../virtual-machines/windows/nsg-quickstart-portal.md).

>[!TIP]
>Если вы хотите автоматически взглянуть на настройку иерархии устройств IoT Edge, можно обратиться к основанному на скриптах примеру [Azure IoT Edge для промышленного Интернета вещей](https://aka.ms/iotedge-nested-sample). Этот основанный на скриптах сценарий развертывает виртуальные машины Azure в качестве предварительно настроенных устройств для имитации заводской среды.
>
>Если вы хотите продолжить пошаговое создание примера иерархии, перейдите к следующим шагам учебника.

## <a name="configure-your-iot-edge-device-hierarchy"></a>Настройка иерархии устройств IoT Edge

### <a name="create-a-hierarchy-of-iot-edge-devices"></a>Создание иерархии устройств IoT Edge

Устройства IoT Edge составляют слои вашей иерархии. В этом учебнике будет создана иерархия из двух устройств IoT Edge: **устройства верхнего слоя** и его дочернего **устройства нижнего слоя**. При необходимости вы можете создать дополнительные дочерние устройства.

Чтобы создать устройства IoT Edge, можно использовать портал Azure или Azure CLI.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Найдите нужный Центр Интернета вещей на [портале Azure](https://ms.portal.azure.com/).

1. В меню слева в разделе **Автоматическое управление устройствами** выберите **IoT Edge**.

1. Выберите **Добавить устройство IoT Edge**. Это устройство будет являться устройством верхнего слоя, поэтому введите соответствующий уникальный идентификатор устройства. Нажмите кнопку **Сохранить**.

1. Выберите **Добавить устройство IoT Edge** еще раз. Это устройство будет являться устройством нижнего слоя, поэтому введите соответствующий уникальный идентификатор устройства.

1. Щелкните **Задать родительское устройство**, выберите устройство верхнего слоя в списке устройств и нажмите кнопку **ОК**. Нажмите кнопку **Сохранить**.

   ![Настройка родительского элемента для устройства нижнего слоя](./media/tutorial-nested-iot-edge/set-parent-device.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Чтобы создать устройство IoT Edge в своем центре, введите следующую команду в [Azure Cloud Shell](https://shell.azure.com/). Это устройство будет являться устройством верхнего слоя, поэтому введите соответствующий уникальный идентификатор устройства.

   ```azurecli-interactive
   az iot hub device-identity create --device-id {top_layer_device_id} --edge-enabled --hub-name {hub_name}
   ```

   При успешном создании устройства будет выведена конфигурация JSON устройства.

   ![Выходные данные JSON при успешном создании устройства](./media/tutorial-nested-iot-edge/json-success-output.png)

1. Введите следующую команду, чтобы создать устройство IoT Edge нижнего слоя Edge. Если вам нужен дополнительный слой в иерархии, можно создать более одного устройства нижнего слоя. Для каждого из них обязательно укажите уникальные удостоверения устройства.

   ```azurecli-interactive
   az iot hub device-identity create --device-id {lower_layer_device_id} --edge-enabled --hub-name {hub_name}
   ```

1. Введите следующую команду, чтобы определить каждую связь "родители-потомки" между **устройством верхнего слоя** и каждым **устройством нижнего слоя**. Не забудьте выполнить эту команду для каждого устройства нижнего слоя в иерархии.

   ```azurecli-interactive
   az iot hub device-identity parent set --device-id {lower_layer_device_id} --parent-device-id {top_layer_device_id} --hub-name {hub_name}
   ```

   Эта команда не имеет явных выходных данных.

---

Затем запишите строку подключения каждого устройства IoT Edge. Они будут использоваться позже.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. На [портале Azure](https://ms.portal.azure.com/) перейдите в раздел **IoT Edge** Центра Интернета вещей.

1. Щелкните идентификатор одного из устройств в списке устройств.

1. Нажмите кнопку **Копировать** в поле **Первичная строка подключения** и запишите строку.

1. Повторите эти действия для всех остальных устройств.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. В [Azure Cloud Shell](https://shell.azure.com/) для каждого устройства введите следующую команду, чтобы получить строку подключения устройства и записать ее:

   ```azurecli-interactive
   az iot hub device-identity connection-string show --device-id {device_id} --hub-name {hub_name}
   ```

---

Если вы правильно выполнили указанные выше действия, можете выполнить следующие действия, чтобы проверить правильность связей типа "родители-потомки" на портале Azure или в Azure CLI.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. На [портале Azure](https://ms.portal.azure.com/) перейдите в раздел **IoT Edge** Центра Интернета вещей.

1. Щелкните идентификатор **устройства нижнего слоя** в списке устройств.

1. На странице сведений об устройстве вы увидите удостоверение **устройства верхнего слоя**, указанное рядом с полем **Родительское устройство**.

   [Родительское устройство, подтвержденное дочерним устройством](./media/tutorial-nested-iot-edge/lower-layer-device-parent.png)

Вы также можете установить дочерние связи иерархии с помощью **устройства верхнего слоя**.

1. Щелкните идентификатор **устройства верхнего слоя** в списке устройств.

1. Выберите вкладку **Управление дочерними устройствами** вверху экрана.

1. В списке устройств должно появится **устройство нижнего слоя**.

   [Дочернее устройство, подтвержденное родительским устройством](./media/tutorial-nested-iot-edge/top-layer-device-child.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. В [Azure Cloud Shell](https://shell.azure.com/) можно проверить, что все дочерние устройства успешно установили связь с родительским устройством, получив удостоверение подтвержденного родительского устройства дочернего устройства. Будет выведена конфигурация JSON родительского устройства, представленного выше:

   ```azurecli-interactive
   az iot hub device-identity parent show --device-id {lower_layer_device_id} --hub-name {hub_name}
   ```

Вы также можете установить дочерние связи иерархии посредством запроса к **устройству верхнего слоя**.

1. Перечислите дочерние устройства, известные родительскому устройству:

    ```azurecli-interactive
    az iot hub device-identity children list --device-id {top_layer_device_id} --hub-name {hub_name}
    ```

---

После того, как иерархия будет должным образом структурирована, вы можете продолжить работу.

### <a name="create-certificates"></a>Создание сертификатов

Всем устройствам в [сценарии шлюза](iot-edge-as-gateway.md) требуется общий сертификат для настройки безопасных соединений между ними. Чтобы создать демонстрационные сертификаты для обоих устройств в этом сценарии, выполните следующие действия.

Чтобы создать демонстрационные сертификаты на устройстве Linux, необходимо клонировать скрипты создания и настроить их для локального запуска в bash.

1. Клонируйте репозиторий Git службы IoT Edge, который содержит скрипты для создания демонстрационных сертификатов.

   ```bash
   git clone https://github.com/Azure/iotedge.git
   ```

1. Перейдите в каталог, в котором вы планируете работать. Все файлы сертификатов и ключей будут созданы в этом каталоге.

1. Скопируйте файлы конфигурации и скриптов из клонированного репозитория IoT Edge в рабочий каталог.

   ```bash
   cp <path>/iotedge/tools/CACertificates/*.cnf .
   cp <path>/iotedge/tools/CACertificates/certGen.sh .
   ```

1. Создайте корневой сертификат ЦС и один промежуточный сертификат.

   ```bash
   ./certGen.sh create_root_and_intermediate
   ```

   Эта команда скрипта создает несколько файлов сертификатов и ключей, но в качестве **корневого сертификата ЦС** для иерархии шлюза используется следующий файл:

   * `<WRKDIR>/certs/azure-iot-test-only.root.ca.cert.pem`  

1. Создайте два набора сертификатов ЦС устройств IoT Edge и закрытых ключей с помощью следующей команды: один набор для устройства верхнего слоя и один набор для устройства нижнего слоя. Укажите запоминающиеся имена сертификатов ЦС, чтобы отличать их друг от друга.

   ```bash
   ./certGen.sh create_edge_device_ca_certificate "{top_layer_certificate_name}"
   ./certGen.sh create_edge_device_ca_certificate "{lower_layer_certificate_name}"
   ```

   Эта команда скрипта создает несколько файлов сертификатов и ключей, но мы используем следующий сертификат и пару ключей на каждом устройстве IoT Edge, на которое указывает ссылка в файле конфигурации:

   * `<WRKDIR>/certs/iot-edge-device-<CA cert name>-full-chain.cert.pem`
   * `<WRKDIR>/private/iot-edge-device-<CA cert name>.key.pem`

Каждому устройству требуется копия корневого сертификата ЦС и копия его собственного сертификата ЦС устройства и закрытого ключа. Для перемещения сертификатов на каждое устройство можно использовать USB-накопитель или [безопасное копирование файлов](https://www.ssh.com/ssh/scp/).

1. После передачи сертификатов установите корневой ЦС для каждого устройства.

   ```bash
   sudo cp <path>/azure-iot-test-only.root.ca.cert.pem /usr/local/share/ca-certificates/azure-iot-test-only.root.ca.cert.pem.crt
   sudo update-ca-certificates
   ```

   Эта команда должна вывести данные о том, что один сертификат добавлен в каталог `/etc/ssl/certs`.

   [Сообщение об успешной установке сертификата](./media/tutorial-nested-iot-edge/certificates-success-output.png)

Если вы выполнили указанные выше действия правильно, можете проверить, установлены ли сертификаты в `/etc/ssl/certs`, перейдя в этот каталог и выполнив поиск установленных сертификатов.

1. Перейдите к `/etc/ssl/certs`:

   ```bash
   cd /etc/ssl/certs
   ```

1. Перечислите установленные сертификаты и `grep` для `azure`:

   ```bash
   ll | grep azure
   ```

   Вы должны увидеть хэш сертификата, связанный с сертификатом корневого ЦС, и сертификат корневого ЦС, связанный с копией в вашем каталоге `usr/local/share`.

   [Результаты поиска сертификатов Azure](./media/tutorial-nested-iot-edge/certificate-list-results.png)

Когда сертификаты будут установлены на каждом устройстве должным образом, вы сможете продолжить работу.

### <a name="install-iot-edge-on-the-devices"></a>Установка IoT Edge на устройства

Установка образов среды выполнения IoT Edge версии 1.2 предоставляет устройствам доступ к функциям, необходимым для работы в качестве иерархии устройств.

Чтобы установить IoT Edge, необходимо установить соответствующую конфигурацию репозитория, необходимые компоненты и необходимые ресурсы выпуска.

1. Установите конфигурацию репозитория, соответствующую операционной системе устройства.

   * **Ubuntu Server 18.04**:

     ```bash
     curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
     ```

   * **Raspberry Pi OS Stretch**:

     ```bash
     curl https://packages.microsoft.com/config/debian/stretch/multiarch/prod.list > ./microsoft-prod.list
     ```

1. Скопируйте созданный список в каталог sources.list.d.

   ```bash
   sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   ```

1. Установите открытый ключ Microsoft GPG.

   ```bash
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

1. Обновите списки пакетов на устройстве.

   ```bash
   sudo apt-get update
   ```

1. Установите модуль Moby.

   ```bash
   sudo apt-get install moby-engine
   ```

1. Установите IoT Edge и службу идентификации Интернета вещей.

   ```bash
   sudo apt-get install aziot-edge
   ```

Если вы выполнили описанные выше действия правильно, то увидели сообщение на баннере Azure IoT Edge, предлагающее изменить файл конфигурации Azure IoT Edge `/etc/aziot/config.toml` на каждом устройстве в иерархии. Если это так, можно двигаться дальше.

### <a name="configure-the-iot-edge-runtime"></a>Настройка среды выполнения IoT Edge

Помимо подготовки устройств, шаги настройки позволяет установить доверенный обмен данными между устройствами в иерархии с использованием ранее созданных сертификатов. Эти шаги также начнут формировать сетевую структуру иерархии. Устройство верхнего слоя будет поддерживать подключение к Интернету, что позволит ему получать образы для своей среды выполнения из облака, тогда как устройства нижнего слоя для доступа к этим образам будут осуществлять маршрутизацию через устройство верхнего слоя.

Чтобы настроить среду выполнения IoT Edge, нужно изменить несколько компонентов файла конфигурации. Эти конфигурации немного различаются у **устройства верхнего слоя** и **устройства нижнего слоя**, поэтому учитывайте, файл конфигурации какого устройства вы редактируете на каждом шаге. Настройка среды выполнения IoT Edge для устройств состоит из четырех шагов, которые выполняются путем изменения файла конфигурации IoT Edge:

* Вручную предоставьте каждое устройство, добавив строку подключения этого устройства в файл конфигурации.

* Завершите настройку сертификатов устройства, указав файл конфигурации для сертификата ЦС устройства, закрытый ключ ЦС устройства и корневой сертификат ЦС.

* Выполните начальную загрузку системы с помощью агента IoT Edge.

* Обновите параметр **hostname** для устройства **верхнего слоя** и параметры **hostname** и **parent_hostname** для устройств **нижнего слоя**.

Выполните следующие действия и перезапустите службу IoT Edge, чтобы настроить устройства.

>[!TIP]
>При навигации по файлу конфигурации в Nano вы можете использовать сочетание клавиш **CTRL+W** для поиска ключевых слов.

1. На каждом устройстве создайте файл конфигурации на основе приложенного шаблона.

   ```bash
   sudo cp /etc/aziot/config.toml.edge.template /etc/aziot/config.toml

1. On each device, open the IoT Edge configuration file.

   ```bash
   sudo nano /etc/aziot/config.toml
   ```

1. Для устройства **верхнего слоя** найдите раздел **имени узла**. Раскомментируйте строку с параметром `hostname` и измените значение на полное доменное имя (FQDN) или IP-адрес устройства IoT Edge. Используйте любое значение, которое будет согласованно выбрано на всех устройствах в иерархии.

   ```toml
   hostname = "<device fqdn or IP>"
   ```

   >[!TIP]
   >Чтобы вставить содержимое буфера обмена в Nano, нажмите клавиши `Shift+Right Click` или `Shift+Insert`.

1. Для каждого устройства IoT Edge на **нижних слоях** найдите раздел **имени узла родительского устройства**. Раскомментируйте строку с параметром `parent_hostname` и измените его значение, чтобы оно указывало на полное доменное имя (FQDN) или IP-адрес родительского устройства. Используйте точное значение, которое вы поместили в поле **имени узла** родительского устройства. Для устройства IoT Edge **верхнего слоя** оставьте этот параметр закомментированным.

   ```toml
   parent_hostname = "<parent device fqdn or IP>"
   ```

   > [!NOTE]
   > Для иерархий с более чем одним нижним слоем обновите поле *parent_hostname* с полным доменным именем устройства в слое, расположенном выше.

1. Найдите в файле раздел **доверенного сертификата пакета**. Раскомментируйте строку с параметром `trust_bundle_cert` и измените его значение, указав путь URI к корневому сертификату ЦС, совместно используемому всеми устройствами в иерархии шлюза.

   ```toml
   trust_bundle_cert = "<root CA certificate>"
   ```

1. Найдите в файле раздел **подготовки**. Раскомментируйте строки **ручной подготовки с использованием строки подключения**. Для каждого устройства в иерархии измените значение **connection_string** строкой подключения для устройства IoT Edge.

   ```toml
   # Manual provisioning with connection string
   [provisioning]
   source = "manual"
   connection_string: "<Device connection string>"
   ```

1. Найдите раздел **агента Edge по умолчанию**.

   * Для устройства **верхнего слоя** измените значение образа edgeAgent на общедоступную предварительную версию модуля в Реестре контейнеров Azure.
   
     ```toml
     [agent.config]
     image = "mcr.microsoft.com/azureiotedge-agent:1.2.0-rc4"
     ```

   * Для каждого устройства IoT Edge на **нижних слоях** измените образ edgeAgent, чтобы он указывал на родительское устройство, за которым следует порт, прослушиваемый прокси-сервером API. Модуль прокси-сервера API будет развернут на родительском устройстве в следующем разделе.
   
     ```toml
     [agent.config]
     image = "<parent hostname value>:8000/azureiotedge-agent:1.2.0-rc4"
     ```

1. Найдите раздел **сертификата ЦС Edge** . Раскомментируйте первые три строки этого раздела. Затем измените эти два параметра, чтобы они указывали на сертификат ЦС устройства и файлы закрытых ключей ЦС устройства, созданные в предыдущем разделе и перемещенные на устройство IoT Edge. Укажите пути URI файла, которые принимают формат `file:///<path>/<filename>`, например `file:///certs/iot-edge-device-ca-top-layer-device.key.pem`.

   ```yml
   [edge_ca]
   cert = "<File URI path to the device full chain CA certificate unique to this device.>"
   pk = "<File URI path to the device CA private key unique to this device.>"
   ```

   >[!NOTE]
   >Обязательно используйте путь и имя файла сертификата ЦС **полной цепочки** для заполнения поля `device_ca_cert`.

1. Сохраните файл и закройте его.

   `CTRL + X`, `Y`, `Enter`

1. Когда введете сведения о подготовке в файл конфигурации, примените изменения:

   ```bash
   sudo iotedge config apply
   ```

Прежде чем продолжить, убедитесь, что изменили файл конфигурации каждого устройства в иерархии. В зависимости от структуры иерархии вы настроили одно **устройство верхнего слоя** и одно или несколько **устройств нижнего слоя**.

Если вы выполнили указанные выше действия должным образом, можете проверить правильность настройки устройств.

1. Запустите проверки конфигурации и подключения на устройствах:

   ```bash
   sudo iotedge check
   ```

На **устройстве верхнего слоя** вы должны увидеть выходные данные с несколькими пройденными оценками и по меньшей мере одним предупреждением. Проверка для `latest security daemon` выдаст предупреждение о том, что последней стабильной версией является другая версия IoT Edge, так как IoT Edge версии 1.2 находится на этапе общедоступной предварительной версии. Вы можете увидеть дополнительные предупреждения о политиках журналов и, в зависимости от вашей сети, политиках DNS.

На **устройстве нижнего слоя** вы должны увидеть результат, аналогичный устройству верхнего слоя, но с дополнительным предупреждением, указывающим, что модуль EdgeAgent не может быть извлечен из восходящего потока. Это приемлемо, так как модуль прокси-сервера API IoT Edge и модуль Реестра контейнеров Docker, через который устройства нижнего слоя будут извлекать образы, еще не развернуты на **устройстве верхнего слоя**.

Пример выходных данных команды `iotedge check` показан ниже:

[Пример результатов для конфигурации и подключения](./media/tutorial-nested-iot-edge/configuration-and-connectivity-check-results.png)

Когда конфигурации на каждом устройстве будут настроены должным образом, вы сможете продолжить работу.

## <a name="deploy-modules-to-the-top-layer-device"></a>Развертывание модулей на устройстве верхнего слоя

Модули служат для дополнения развертывания и среды выполнения IoT Edge на устройствах, а также для более детального определения структуры иерархии. Модуль прокси-сервера API IoT Edge осуществляет безопасную маршрутизацию трафика HTTP через один порт с устройств нижнего слоя. Модуль реестра Docker позволяет использовать репозиторий образов Docker, к которым устройства нижнего слоя могут получить доступ посредством маршрутизации операций извлечения образа на устройство верхнего слоя.

Для развертывания модулей на устройстве верхнего слоя можно использовать портал Azure или Azure CLI.

>[!NOTE]
>Оставшиеся действия для завершения настройки среды выполнения IoT Edge и развертывания рабочих нагрузок выполняются не на устройствах IoT Edge.

# <a name="portal"></a>[Портал](#tab/azure-portal)

[На портале Azure](https://ms.portal.azure.com/)

1. Перейдите в Центр Интернета вещей.

1. В меню слева в разделе **Автоматическое управление устройствами** выберите **IoT Edge**.

1. В списке устройств щелкните идентификатор устройства **верхнего слоя**.

1. На верхней панели выберите **Задание модулей**.

1. Щелкните пункт **Параметры среды выполнения** рядом со значком шестеренки.

1. В разделе **Центр Edge** в поле образа введите `mcr.microsoft.com/azureiotedge-hub:1.2.0-rc4`.

   ![Изменение образа центра Edge](./media/tutorial-nested-iot-edge/edge-hub-image.png)

1. Добавьте следующие переменные среды в модуль центра Edge:

    | Имя | Значение |
    | - | - |
    | `experimentalFeatures__enabled` | `true` |
    | `experimentalFeatures__nestedEdgeEnabled` | `true` |

   ![Изменение переменных среды центра Edge](./media/tutorial-nested-iot-edge/edge-hub-environment-variables.png)

1. В разделе **Агент Edge** в поле образа введите `mcr.microsoft.com/azureiotedge-agent:1.2.0-rc4`. Нажмите кнопку **Сохранить**.

1. Добавьте модуль реестра DOCKER на устройство верхнего слоя. Щелкните **Добавить** и выберите **Модуль IoT Edge** в раскрывающемся списке. Укажите имя `registry` для модуля реестра DOCKER и введите `registry:latest` в качестве URI образа. Затем добавьте переменные среды и создайте параметры для указания локального модуля реестра в реестре контейнеров Майкрософт, чтобы скачать образы контейнеров и обслужить эти образы в реестре 5000.

1. На вкладке переменных среды введите следующую пару имя-значение переменной среды:

    | Имя | Значение |
    | - | - |
    | `REGISTRY_PROXY_REMOTEURL` | `https://mcr.microsoft.com` |

1. На вкладке параметров создания контейнера введите следующий код JSON:

   ```json
   {
    "HostConfig": {
        "PortBindings": {
            "5000/tcp": [
                {
                    "HostPort": "5000"
                }
            ]
         }
      }
   }
   ```

1. Затем добавьте модуль прокси-сервера API на устройство верхнего слоя. Щелкните **Добавить** и выберите **Модуль в Marketplace** в раскрывающемся списке. Найдите `IoT Edge API Proxy` и выберите модуль. Прокси-сервер API IoT Edge использует порт 8000 и по умолчанию настроен для использования модуля реестра с именем `registry` для порта 5000.

1. Выберите **Просмотр и создание**, а затем нажмите кнопку **Создать** для завершения развертывания. Среда выполнения IoT Edge устройства верхнего слоя, которая имеет доступ к Интернету, выберет и запустит **общедоступную предварительную версию** конфигураций центра IoT Edge и агента IoT Edge.

   ![Полное развертывание с центром Edge, агентом Edge, модулем реестра и модулем прокси-сервера API](./media/tutorial-nested-iot-edge/complete-top-layer-deployment.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. В [Azure Cloud Shell](https://shell.azure.com/) введите следующую команду, чтобы создать файл deployment.json:

   ```azurecli-interactive
   code deploymentTopLayer.json
   ```

1. Скопируйте содержимое приведенного ниже файла JSON в deployment.json, сохраните его и закройте.

   ```json
   {
       "modulesContent": {
           "$edgeAgent": {
               "properties.desired": {
                   "modules": {
                       "registry": {
                           "settings": {
                               "image": "registry:latest",
                               "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5000/tcp\":[{\"HostPort\":\"5000\"}]}}}"
                           },
                           "type": "docker",
                           "version": "1.0",
                           "env": {
                               "REGISTRY_PROXY_REMOTEURL": {
                                   "value": "https://mcr.microsoft.com"
                               } 
                           },
                           "status": "running",
                           "restartPolicy": "always"
                       },
                       "IoTEdgeAPIProxy": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-api-proxy",
                               "createOptions": "{\"HostConfig\": {\"PortBindings\": {\"8000/tcp\": [{\"HostPort\":\"8000\"}]}}}"
                           },
                           "type": "docker",
                           "env": {
                               "NGINX_DEFAULT_PORT": {
                                   "value": "8000"
                               },
                               "DOCKER_REQUEST_ROUTE_ADDRESS": {
                                   "value": "registry:5000"
                               },
                               "BLOB_UPLOAD_ROUTE_ADDRESS": {
                                   "value": "AzureBlobStorageonIoTEdge:11002"
                               }
                           },
                           "status": "running",
                           "restartPolicy": "always",
                           "version": "1.0"
                       }
                   },
                   "runtime": {
                       "settings": {
                           "minDockerVersion": "v1.25"
                       },
                       "type": "docker"
                   },
                   "schemaVersion": "1.1",
                   "systemModules": {
                       "edgeAgent": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-agent:1.2.0-rc4",
                               "createOptions": ""
                           },
                           "type": "docker"
                       },
                       "edgeHub": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-hub:1.2.0-rc4",
                               "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
                           },
                           "type": "docker",
                           "env": {
                               "experimentalFeatures__enabled": {
                                   "value": "true"
                               },
                               "experimentalFeatures__nestedEdgeEnabled": {
                                   "value": "true"
                               }
                           },
                           "status": "running",
                           "restartPolicy": "always"
                       }
                   }
               }
           },
           "$edgeHub": {
               "properties.desired": {
                   "routes": {
                       "route": "FROM /messages/* INTO $upstream"
                   },
                   "schemaVersion": "1.1",
                   "storeAndForwardConfiguration": {
                       "timeToLiveSecs": 7200
                   }
               }
           }
       }
   }
   ```

1. Введите следующую команду, чтобы создать развертывание на устройстве Edge верхнего слоя:

   ```azurecli-interactive
   az iot edge set-modules --device-id <top_layer_device_id> --hub-name <iot_hub_name> --content ./deploymentTopLayer.json
   ```

---

Если вы выполнили указанные выше действия правильно, **устройство верхнего слоя** должно сообщить о четырех модулях: модуль прокси-сервера API IoT Edge, модуль Реестра контейнеров Docker и системные модули, такие как **Указано в развертывании**. Устройству может потребоваться несколько минут на то, чтобы получить новое развертывание и запустить модули. Обновляйте страницу, пока не увидите модуль датчика температуры, указанный как **Данные, полученные от устройства**. После того как устройство сообщает о модулях, вы можете продолжить.

## <a name="deploy-modules-to-the-lower-layer-device"></a>Развертывание модулей на устройстве нижнего слоя

Модули также служат рабочими нагрузками для устройств нижнего слоя. Модуль имитированного датчика температуры создает пример данных телеметрии, чтобы обеспечить функциональный поток данных через иерархию устройств.

Для развертывания модулей на устройствах нижнего слоя можно использовать портал Azure или Azure CLI.

# <a name="portal"></a>[Портал](#tab/azure-portal)

[На портале Azure](https://ms.portal.azure.com/)

1. Перейдите в Центр Интернета вещей.

1. В меню слева в разделе **Автоматическое управление устройствами** выберите **IoT Edge**.

1. Щелкните идентификатор устройства нижнего слоя в списке устройств IoT Edge.

1. На верхней панели выберите **Задание модулей**.

1. Щелкните пункт **Параметры среды выполнения** рядом со значком шестеренки.

1. В разделе **Центр Edge** в поле образа введите `$upstream:8000/azureiotedge-hub:1.2.0-rc4`.

1. Добавьте следующие переменные среды в модуль центра Edge:

    | Имя | Значение |
    | - | - |
    | `experimentalFeatures__enabled` | `true` |
    | `experimentalFeatures__nestedEdgeEnabled` | `true` |

1. В разделе **Агент Edge** в поле образа введите `$upstream:8000/azureiotedge-agent:1.2.0-rc4`. Нажмите кнопку **Сохранить**.

1. Добавьте модуль датчика температуры. Щелкните **Добавить** и выберите **Модуль в Marketplace** в раскрывающемся списке. Найдите `Simulated Temperature Sensor` и выберите модуль.

1. В разделе **Модули IoT Edge** выберите только что добавленный модуль `Simulated Temperature Sensor` и обновите его URI, чтобы он указывал на `$upstream:8000/azureiotedge-simulated-temperature-sensor:1.0`.

1. Щелкните **Сохранить**, выберите **Просмотр и создание**, а затем нажмите кнопку **Создать** для завершения развертывания.

   ![Полное развертывание с центром Edge, агентом Edge и смоделированным датчиком температуры](./media/tutorial-nested-iot-edge/complete-lower-layer-deployment.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. В [Azure Cloud Shell](https://shell.azure.com/) введите следующую команду, чтобы создать файл deployment.json:

   ```azurecli-interactive
   code deploymentLowerLayer.json
   ```

1. Скопируйте содержимое приведенного ниже файла JSON в deployment.json, сохраните его и закройте.

   ```json
   {
       "modulesContent": {
           "$edgeAgent": {
               "properties.desired": {
                   "modules": {
                       "simulatedTemperatureSensor": {
                           "settings": {
                               "image": "$upstream:8000/azureiotedge-simulated-temperature-sensor:1.0",
                               "createOptions": ""
                           },
                           "type": "docker",
                           "status": "running",
                           "restartPolicy": "always",
                           "version": "1.0"
                       }
                   },
                   "runtime": {
                       "settings": {
                           "minDockerVersion": "v1.25"
                       },
                       "type": "docker"
                   },
                   "schemaVersion": "1.1",
                   "systemModules": {
                       "edgeAgent": {
                           "settings": {
                               "image": "$upstream:8000/azureiotedge-agent:1.2.0-rc4",
                               "createOptions": ""
                           },
                           "type": "docker"
                       },
                       "edgeHub": {
                           "settings": {
                               "image": "$upstream:8000/azureiotedge-hub:1.2.0-rc4",
                               "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
                           },
                           "type": "docker",
                           "env": {
                               "experimentalFeatures__enabled": {
                                   "value": "true"
                               },
                               "experimentalFeatures__nestedEdgeEnabled": {
                                   "value": "true"
                               }
                           },
                           "status": "running",
                           "restartPolicy": "always"
                       }
                   }
               }
           },
           "$edgeHub": {
               "properties.desired": {
                   "routes": {
                       "route": "FROM /messages/* INTO $upstream"
                   },
                   "schemaVersion": "1.1",
                   "storeAndForwardConfiguration": {
                       "timeToLiveSecs": 7200
                   }
               }
           }
       }
   }
   ```

1. Введите следующую команду, чтобы создать развертывание набора модулей на устройстве Edge нижнего слоя:

   ```azurecli-interactive
   az iot edge set-modules --device-id <lower_layer_device_id> --hub-name <iot_hub_name> --content ./deploymentLowerLayer.json

---

Notice that the image URI that we used for the simulated temperature sensor module pointed to `$upstream:8000` instead of to a container registry. We configured this device to not have direct connections to the cloud, because it's in a lower layer. To pull container images, this device requests the image from its parent device instead. At the top layer, the API proxy module routes this container request to the registry module, which handles the image pull.

If you completed the above steps correctly, your **lower layer device** should report three modules: the temperature sensor module and the system modules, as **Specified in Deployment**. It may take a few minutes for the device to receive its new deployment, request the container image, and start the module. Refresh the page until you see the temperature sensor module listed as **Reported by Device**. Once the modules are reported by the device, you are ready to continue.

## Troubleshooting

Run the `iotedge check` command to verify the configuration and to troubleshoot errors.

You can run `iotedge check` in a nested hierarchy, even if the child machines don't have direct internet access.

When you run `iotedge check` from the lower layer, the program tries to pull the image from the parent through port 443.

In this tutorial, we use port 8000, so we need to specify it:

```bash
sudo iotedge check --diagnostics-image-name <parent_device_fqdn_or_ip>:8000/azureiotedge-diagnostics:1.2.0-rc4
```

Значение `azureiotedge-diagnostics` извлекается из реестра контейнеров, связанного с модулем реестра. В этом учебнике по умолчанию задано значение https://mcr.microsoft.com:.

| Имя | Значение |
| - | - |
| `REGISTRY_PROXY_REMOTEURL` | `https://mcr.microsoft.com` |

Если вы используете частный реестр контейнеров, убедитесь, что все образы (например, IoTEdgeAPIProxy, edgeAgent, edgeHub и diagnostics) находятся в реестре контейнеров.

## <a name="view-generated-data"></a>Просмотр сформированных данных

Модуль **смоделированного датчика температуры**, который вы развернули, генерирует выборку данных о состоянии окружающей среды. Он отправляет сообщения, которые содержат данные о температуре и влажности окружающей среды, температуре и давлении оборудования, а также метки времени.

Вы можете просматривать сообщения, которые поступают в Центр Интернета вещей, используя [расширение Центра Интернета вещей Azure для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit).

Это также можно делать с помощью [Azure Cloud Shell](https://shell.azure.com/):

   ```azurecli-interactive
   az iot hub monitor-events -n <iothub_name> -d <lower-layer-device-name>
   ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы избежать расходов, можно удалить локальные конфигурации и ресурсы Azure, созданные при работе с этой статьей.

Удаление ресурсов:

1. Войдите в портал [Azure](https://portal.azure.com) и выберите **Группы ресурсов**.

2. Выберите группу ресурсов, содержащую тестовые ресурсы IoT Edge. 

3. Просмотрите список ресурсов, содержащихся в группе ресурсов. Если вы хотите удалить их все, щелкните **Удалить группу ресурсов**. Если вы хотите удалить только некоторые из них, щелкните нужные ресурсы отдельно. 

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим учебником вы настроили два устройства IoT Edge в качестве шлюзов и настроили одно из них для выполнения роли родительского устройства для другого. Затем вы выполнили извлечение образа контейнера в дочернее устройство через шлюз.

Вы можете перейти к изучению других руководств, чтобы узнать, как Azure IoT Edge может создавать решения для вашего бизнеса.

> [!div class="nextstepaction"]
> [Руководство: развертывание службы "Машинное обучение Azure" в качестве модуля IoT Edge (предварительная версия)](tutorial-deploy-machine-learning.md)
