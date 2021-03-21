---
title: Выполнение издателя OPC (Azure) | Документация Майкрософт
description: В этой статье описано, как запускать и отлаживать OPC Publisher. Также здесь представлены рекомендации по производительности и управлению памятью.
author: dominicbetts
ms.author: dobett
ms.date: 06/10/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: f7d6581a1892ebd74a1adba5c09c0af9d3cf9d43
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92079009"
---
# <a name="run-opc-publisher"></a>Выполнение издателя OPC

> [!IMPORTANT]
> Актуальную информацию по этой теме см. в статье [Промышленный Интернет вещей в Azure](https://azure.github.io/Industrial-IoT/).

В этой статье описано, как запускать и отлаживать издателя OPC. Также здесь представлены рекомендации по производительности и управлению памятью.

## <a name="command-line-options"></a>Параметры командной строки

Использование приложений демонстрируется с параметром командной строки `--help`, как указано далее:

```sh/cmd
Current directory is: /appdata
Log file is: <hostname>-publisher.log
Log level is: info

OPC Publisher V2.3.0
Informational version: V2.3.0+Branch.develop_hans_methodlog.Sha.0985e54f01a0b0d7f143b1248936022ea5d749f9

Usage: opcpublisher.exe <applicationname> [<IoT Hubconnectionstring>] [<options>]

OPC Edge Publisher to subscribe to configured OPC UA servers and send telemetry to Azure IoT Hub.
To exit the application, just press CTRL-C while it is running.

applicationname: the OPC UA application name to use, required
                  The application name is also used to register the publisher under this name in the
                  IoT Hub device registry.

IoT Hubconnectionstring: the IoT Hub owner connectionstring, optional

There are a couple of environment variables which can be used to control the application:
_HUB_CS: sets the IoT Hub owner connectionstring
_GW_LOGP: sets the filename of the log file to use
_TPC_SP: sets the path to store certificates of trusted stations
_GW_PNFP: sets the filename of the publishing configuration file

Command line arguments overrule environment variable settings.

Options:
      --pf, --publishfile=VALUE
                              the filename to configure the nodes to publish.
                                Default: '/appdata/publishednodes.json'
      --tc, --telemetryconfigfile=VALUE
                              the filename to configure the ingested telemetry
                                Default: ''
  -s, --site=VALUE           the site OPC Publisher is working in. if specified
                                this domain is appended (delimited by a ':' to
                                the 'ApplicationURI' property when telemetry is
                                sent to IoT Hub.
                                The value must follow the syntactical rules of a
                                DNS hostname.
                                Default: not set
      --ic, --iotcentral     publisher will send OPC UA data in IoTCentral
                                compatible format (DisplayName of a node is used
                                as key, this key is the Field name in IoTCentral)
                                . you need to ensure that all DisplayName's are
                                unique. (Auto enables fetch display name)
                                Default: False
      --sw, --sessionconnectwait=VALUE
                              specify the wait time in seconds publisher is
                                trying to connect to disconnected endpoints and
                                starts monitoring unmonitored items
                                Min: 10
                                Default: 10
      --mq, --monitoreditemqueuecapacity=VALUE
                              specify how many notifications of monitored items
                                can be stored in the internal queue, if the data
                                can not be sent quick enough to IoT Hub
                                Min: 1024
                                Default: 8192
      --di, --diagnosticsinterval=VALUE
                              shows publisher diagnostic info at the specified
                                interval in seconds (need log level info).
                                -1 disables remote diagnostic log and diagnostic
                                output
                                0 disables diagnostic output
                                Default: 0
      --ns, --noshutdown=VALUE
                              same as runforever.
                                Default: False
      --rf, --runforever     publisher can not be stopped by pressing a key on
                                the console, but will run forever.
                                Default: False
      --lf, --logfile=VALUE  the filename of the logfile to use.
                                Default: './<hostname>-publisher.log'
      --lt, --logflushtimespan=VALUE
                              the timespan in seconds when the logfile should be
                                flushed.
                                Default: 00:00:30 sec
      --ll, --loglevel=VALUE the loglevel to use (allowed: fatal, error, warn,
                                info, debug, verbose).
                                Default: info
        --ih, --IoT Hubprotocol=VALUE
                              the protocol to use for communication with IoT Hub (
                                allowed values: Amqp, Http1, Amqp_WebSocket_Only,
                                  Amqp_Tcp_Only, Mqtt, Mqtt_WebSocket_Only, Mqtt_
                                Tcp_Only) or IoT EdgeHub (allowed values: Mqtt_
                                Tcp_Only, Amqp_Tcp_Only).
                                Default for IoT Hub: Mqtt_WebSocket_Only
                                Default for IoT EdgeHub: Amqp_Tcp_Only
      --ms, --IoT Hubmessagesize=VALUE
                              the max size of a message which can be send to
                                IoT Hub. when telemetry of this size is available
                                it will be sent.
                                0 will enforce immediate send when telemetry is
                                available
                                Min: 0
                                Max: 262144
                                Default: 262144
      --si, --IoT Hubsendinterval=VALUE
                              the interval in seconds when telemetry should be
                                send to IoT Hub. If 0, then only the
                                IoT Hubmessagesize parameter controls when
                                telemetry is sent.
                                Default: '10'
      --dc, --deviceconnectionstring=VALUE
                              if publisher is not able to register itself with
                                IoT Hub, you can create a device with name <
                                applicationname> manually and pass in the
                                connectionstring of this device.
                                Default: none
  -c, --connectionstring=VALUE
                              the IoT Hub owner connectionstring.
                                Default: none
      --hb, --heartbeatinterval=VALUE
                              the publisher is using this as default value in
                                seconds for the heartbeat interval setting of
                                nodes without
                                a heartbeat interval setting.
                                Default: 0
      --sf, --skipfirstevent=VALUE
                              the publisher is using this as default value for
                                the skip first event setting of nodes without
                                a skip first event setting.
                                Default: False
      --pn, --portnum=VALUE  the server port of the publisher OPC server
                                endpoint.
                                Default: 62222
      --pa, --path=VALUE     the enpoint URL path part of the publisher OPC
                                server endpoint.
                                Default: '/UA/Publisher'
      --lr, --ldsreginterval=VALUE
                              the LDS(-ME) registration interval in ms. If 0,
                                then the registration is disabled.
                                Default: 0
      --ol, --opcmaxstringlen=VALUE
                              the max length of a string opc can transmit/
                                receive.
                                Default: 131072
      --ot, --operationtimeout=VALUE
                              the operation timeout of the publisher OPC UA
                                client in ms.
                                Default: 120000
      --oi, --opcsamplinginterval=VALUE
                              the publisher is using this as default value in
                                milliseconds to request the servers to sample
                                the nodes with this interval
                                this value might be revised by the OPC UA
                                servers to a supported sampling interval.
                                please check the OPC UA specification for
                                details how this is handled by the OPC UA stack.
                                a negative value will set the sampling interval
                                to the publishing interval of the subscription
                                this node is on.
                                0 will configure the OPC UA server to sample in
                                the highest possible resolution and should be
                                taken with care.
                                Default: 1000
      --op, --opcpublishinginterval=VALUE
                              the publisher is using this as default value in
                                milliseconds for the publishing interval setting
                                of the subscriptions established to the OPC UA
                                servers.
                                please check the OPC UA specification for
                                details how this is handled by the OPC UA stack.
                                a value less than or equal zero will let the
                                server revise the publishing interval.
                                Default: 0
      --ct, --createsessiontimeout=VALUE
                              specify the timeout in seconds used when creating
                                a session to an endpoint. On unsuccessful
                                connection attemps a backoff up to 5 times the
                                specified timeout value is used.
                                Min: 1
                                Default: 10
      --ki, --keepaliveinterval=VALUE
                              specify the interval in seconds the publisher is
                                sending keep alive messages to the OPC servers
                                on the endpoints it is connected to.
                                Min: 2
                                Default: 2
      --kt, --keepalivethreshold=VALUE
                              specify the number of keep alive packets a server
                                can miss, before the session is disconneced
                                Min: 1
                                Default: 5
      --aa, --autoaccept     the publisher trusts all servers it is
                                establishing a connection to.
                                Default: False
      --tm, --trustmyself=VALUE
                              same as trustowncert.
                                Default: False
      --to, --trustowncert   the publisher certificate is put into the trusted
                                certificate store automatically.
                                Default: False
      --fd, --fetchdisplayname=VALUE
                              same as fetchname.
                                Default: False
      --fn, --fetchname      enable to read the display name of a published
                                node from the server. this will increase the
                                runtime.
                                Default: False
      --ss, --suppressedopcstatuscodes=VALUE
                              specifies the OPC UA status codes for which no
                                events should be generated.
                                Default: BadNoCommunication,
                                BadWaitingForInitialData
      --at, --appcertstoretype=VALUE
                              the own application cert store type.
                                (allowed values: Directory, X509Store)
                                Default: 'Directory'
      --ap, --appcertstorepath=VALUE
                              the path where the own application cert should be
                                stored
                                Default (depends on store type):
                                X509Store: 'CurrentUser\UA_MachineDefault'
                                Directory: 'pki/own'
      --tp, --trustedcertstorepath=VALUE
                              the path of the trusted cert store
                                Default: 'pki/trusted'
      --rp, --rejectedcertstorepath=VALUE
                              the path of the rejected cert store
                                Default 'pki/rejected'
      --ip, --issuercertstorepath=VALUE
                              the path of the trusted issuer cert store
                                Default 'pki/issuer'
      --csr                  show data to create a certificate signing request
                                Default 'False'
      --ab, --applicationcertbase64=VALUE
                              update/set this applications certificate with the
                                certificate passed in as bas64 string
      --af, --applicationcertfile=VALUE
                              update/set this applications certificate with the
                                certificate file specified
      --pb, --privatekeybase64=VALUE
                              initial provisioning of the application
                                certificate (with a PEM or PFX fomat) requires a
                                private key passed in as base64 string
      --pk, --privatekeyfile=VALUE
                              initial provisioning of the application
                                certificate (with a PEM or PFX fomat) requires a
                                private key passed in as file
      --cp, --certpassword=VALUE
                              the optional password for the PEM or PFX or the
                                installed application certificate
      --tb, --addtrustedcertbase64=VALUE
                              adds the certificate to the applications trusted
                                cert store passed in as base64 string (multiple
                                strings supported)
      --tf, --addtrustedcertfile=VALUE
                              adds the certificate file(s) to the applications
                                trusted cert store passed in as base64 string (
                                multiple filenames supported)
      --ib, --addissuercertbase64=VALUE
                              adds the specified issuer certificate to the
                                applications trusted issuer cert store passed in
                                as base64 string (multiple strings supported)
      --if, --addissuercertfile=VALUE
                              adds the specified issuer certificate file(s) to
                                the applications trusted issuer cert store (
                                multiple filenames supported)
      --rb, --updatecrlbase64=VALUE
                              update the CRL passed in as base64 string to the
                                corresponding cert store (trusted or trusted
                                issuer)
      --uc, --updatecrlfile=VALUE
                              update the CRL passed in as file to the
                                corresponding cert store (trusted or trusted
                                issuer)
      --rc, --removecert=VALUE
                              remove cert(s) with the given thumbprint(s) (
                                multiple thumbprints supported)
      --dt, --devicecertstoretype=VALUE
                              the IoT Hub device cert store type.
                                (allowed values: Directory, X509Store)
                                Default: X509Store
      --dp, --devicecertstorepath=VALUE
                              the path of the iot device cert store
                                Default Default (depends on store type):
                                X509Store: 'My'
                                Directory: 'CertificateStores/IoT Hub'
  -i, --install              register OPC Publisher with IoT Hub and then exits.
                                Default:  False
  -h, --help                 show this message and exit
      --st, --opcstacktracemask=VALUE
                              ignored, only supported for backward comaptibility.
      --sd, --shopfloordomain=VALUE
                              same as site option, only there for backward
                                compatibility
                                The value must follow the syntactical rules of a
                                DNS hostname.
                                Default: not set
      --vc, --verboseconsole=VALUE
                              ignored, only supported for backward comaptibility.
      --as, --autotrustservercerts=VALUE
                              same as autoaccept, only supported for backward
                                cmpatibility.
                                Default: False
      --tt, --trustedcertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the trusted cert store will always reside in a
                                directory.
      --rt, --rejectedcertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the rejected cert store will always reside in a
                                directory.
      --it, --issuercertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the trusted issuer cert store will always
                                reside in a directory.
```

Как правило, вы можете указать строку подключения владельца Центра Интернета вещей только при первом выполнении приложения. Она хранится в хранилище сертификатов платформы в зашифрованном виде. При следующих выполнениях приложение считывает строку подключения из хранилища сертификатов. Если вы будете указывать строку подключения при каждом выполнении, созданное в реестре устройств Центра Интернета вещей устройство будет удаляться и создаваться заново.

## <a name="run-natively-on-windows"></a>Встроенная поддержка выполнения в Windows

Откройте проект **opcpublisher.sln** в Visual Studio, скомпилируйте и опубликуйте решение. Приложение можно запустить в **целевом каталоге**, в котором вы его опубликовали, с помощью следующей команды:

```cmd
dotnet opcpublisher.dll <applicationname> [<IoT Hubconnectionstring>] [options]
```

## <a name="use-a-self-built-container"></a>Использование автоматически созданного контейнера

Создайте собственный контейнер и запустите его:

```sh/cmd
docker run <your-container-name> <applicationname> [<IoT Hubconnectionstring>] [options]
```

## <a name="use-a-container-from-microsoft-container-registry"></a>Использование контейнера из Реестра контейнеров Майкрософт

В Реестре контейнеров Майкрософт доступен готовый контейнер. Запустите его:

```sh/cmd
docker run mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

Проверьте в [Docker Hub](https://hub.docker.com/_/microsoft-iotedge-opc-publisher) список поддерживаемых операционных систем и архитектур процессоров. Если используемые ОС и архитектура ЦП поддерживаются, Docker автоматически выберет правильный контейнер.

## <a name="run-as-an-azure-iot-edge-module"></a>Выполнение в виде модуля Azure IoT Edge

Издатель OPC может использоваться в виде модуля [Azure IoT Edge](../iot-edge/index.yml). При использовании издателя OPC в виде модуля IoT Edge поддерживаются только транспортные протоколы **Amqp_Tcp_Only** и **Mqtt_Tcp_Only**.

Чтобы добавить издателя OPC в виде модуля в развертывание IoT Edge, перейдите к параметрам Центра Интернета вещей на портале Azure и выполните следующие действия:

1. Откройте **IoT Edge**, создайте или выберите устройство IoT Edge.
1. Выберите **задать модули**.
1. В разделе **Модули развертывания** выберите **Добавить**, а затем **Модуль IoT Edge**.
1. В поле **Имя** введите **publisher** (издатель).
1. В поле **URI образа** введите `mcr.microsoft.com/iotedge/opc-publisher:<tag>`.
1. Список доступных тегов можно найти в [Docker Hub](https://hub.docker.com/_/microsoft-iotedge-opc-publisher).
1. Вставьте следующий код JSON в поле **Параметры создания контейнера**:

    ```json
    {
        "Hostname": "publisher",
        "Cmd": [
            "--aa"
        ]
    }
    ```

    Эта конфигурация настраивает IoT Edge так, чтобы он запускал контейнер с именем **publisher** из образа издателя OPC. Имя узла для системы контейнера получает значение **publisher**. Издатель OPC вызывается со следующим аргументом командной строки: `--aa`. При этом издатель OPC доверяет сертификатам серверов OPC UA, к которым он подключается. Вы можете использовать любые параметры командной строки, поддерживаемые издателем OPC. Единственное ограничение — поддерживаемый для IoT Edge размер **параметров создания контейнера**.

1. Оставьте без изменений другие параметры и выберите **Сохранить**.
1. Если вы хотите локально обрабатывать выходные данные издателя OPC с помощью другого модуля IoT Edge, вернитесь на страницу **Настройка модулей**. Затем перейдите на вкладку **Указание маршрутов** и добавьте новый маршрут с таким кодом JSON:

    ```json
    {
      "routes": {
        "processingModuleToIoT Hub": "FROM /messages/modules/processingModule/outputs/* INTO $upstream",
        "opcPublisherToProcessingModule": "FROM /messages/modules/publisher INTO BrokeredEndpoint(\"/modules/processingModule/inputs/input1\")"
      }
    }
    ```

1. Вернитесь на страницу **Настройка модулей** и нажимайте кнопку **Далее**, пока не достигнете последней страницы конфигурации.
1. Нажмите **Отправить**, чтобы передать конфигурацию в IoT Edge.
1. Когда вы запустите IoT Edge на граничном устройстве и контейнер docker **publisher** будет успешно работать, выходные данные журнала для издателя OPC можно получить с помощью команды `docker logs -f publisher` или файла журнала. В предыдущем примере файл журнала располагается выше `d:\iiotegde\publisher-publisher.log`. Вы также можете использовать [средство iot-edge-opc-publisher-diagnostics](https://github.com/Azure-Samples/iot-edge-opc-publisher-diagnostics).

### <a name="make-the-configuration-files-accessible-on-the-host"></a>Предоставление файлов конфигурации на узле

Чтобы сделать файлы конфигурации модуля IoT Edge доступными в файловой системе узла, воспользуйтесь следующим вариантом **параметров создания контейнера**. Пример ниже демонстрирует развертывание с использованием контейнеров Linux для Windows:

```json
{
    "Hostname": "publisher",
    "Cmd": [
        "--pf=./pn.json",
        "--aa"
    ],
    "HostConfig": {
        "Binds": [
            "d:/iiotedge:/appdata"
        ]
    }
}
```

С этими параметрами издатель OPC считывает узлы, которые ему нужно опубликовать, из файла `./pn.json`, а при запуске назначает `/appdata` рабочим каталогом для контейнера. С этими параметрами издатель OPC считывает из контейнера файл `/appdata/pn.json`, чтобы получить его конфигурацию. Без параметра `--pf` издатель OPC пытается прочитать файл конфигурации по умолчанию (`./publishednodes.json`).

Файл журнала со стандартным именем `publisher-publisher.log` сохраняется в `/appdata`, где также создается каталог `CertificateStores`.

Чтобы все эти файлы были доступны в файловой системе узла, следует добавить в конфигурацию контейнера подключение тома с привязкой. Привязка `d://iiotedge:/appdata` сопоставляет каталог `/appdata`, который при запуске контейнера назначается текущим рабочим каталогом, с каталогом `d://iiotedge` на узле. Без этого параметра данные файлов не сохраняются при следующем запуске контейнера.

Если вы используете контейнеры Windows, синтаксис параметра `Binds` будет другим. При запуске контейнера рабочим каталогом назначается `c:\appdata`. Чтобы поместить файл конфигурации в каталог `d:\iiotedge` на узле, укажите следующее сопоставление в разделе `HostConfig`:

```json
"HostConfig": {
    "Binds": [
        "d:/iiotedge:c:/appdata"
    ]
}
```

Если вы используете контейнеры Linux в ОС Linux, синтаксис параметра `Binds` также будет другим. При запуске контейнера рабочим каталогом назначается `/appdata`. Чтобы поместить файл конфигурации в каталог `/iiotedge` на узле, укажите следующее сопоставление в разделе `HostConfig`:

```json
"HostConfig": {
    "Binds": [
        "/iiotedge:/appdata"
    ]
}
```

## <a name="considerations-when-using-a-container"></a>Рекомендации при использовании контейнера

В следующих разделах перечислены некоторые особенности, о которых следует помнить при использовании контейнера.

### <a name="access-to-the-opc-publisher-opc-ua-server"></a>Доступ к серверу OPC UA издателя OPC

Сервер OPC UA издателя OPC по умолчанию прослушивает порт 62222. Чтобы настроить для контейнера другой порт, используйте следующую команду:

```sh/cmd
docker run -p 62222:62222 mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="enable-intercontainer-name-resolution"></a>Включение разрешения имен между контейнерами

Чтобы включить разрешение имен из контейнера для других контейнеров, создайте определяемый пользователем сетевой мост docker и подключите контейнер к этой сети с помощью параметра `--network`. Также присвойте контейнеру имя с помощью параметра `--name`, как показано ниже:

```sh/cmd
docker network create -d bridge iot_edge
docker run --network iot_edge --name publisher mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

Теперь другие контейнеры в этой сети смогут обратиться к этому контейнеру по имени `publisher`.

### <a name="access-other-systems-from-within-the-container"></a>Доступ к другим системам из контейнера

Обращаться к другим контейнерам можно с помощью параметров, которые описаны в предыдущем разделе. Если операционная система, в которой размещается Docker, поддерживает DNS, будет работать доступ ко всем системам, которые известны в DNS.

В тех сетях, где используется разрешение имен NetBIOS, для доступа к другим системам следует запускать контейнер с параметром `--add-host`. Этот параметр фактически добавляет в файл узла контейнера следующую запись:

```cmd/sh
docker run --add-host mydevbox:192.168.178.23  mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="assign-a-hostname"></a>Присвоение имени узла

Издатель OPC использует имя узла компьютера, на котором он выполняется, для создания сертификата и конечной точки. Docker выбирает случайное имя узла, если вы не укажете имя в параметре `-h`. Следующий пример демонстрирует, как задать для контейнера внутреннее имя узла `publisher`:

```sh/cmd
docker run -h publisher mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="use-bind-mounts-shared-filesystem"></a>Использование подключения с привязкой (совместно используемая файловая система)

Вместо файловой системы контейнера вы можете хранить сведения о конфигурации и файлы журнала в файловой системе узла. Чтобы настроить этот вариант, используйте параметр `-v` команды `docker run` в режиме подключения с привязкой.

## <a name="opc-ua-x509-certificates"></a>Сертификаты X.509 для OPC UA

OPC UA использует сертификаты X.509 для аутентификации клиента и сервера OPC UA, когда они устанавливают соединение, и для шифрования обмена данными между ними. Издатель OPC использует хранилища сертификатов, поддерживаемые в стеке OPC UA, для управления всеми сертификатами. При запуске издатель OPC проверяет, существует ли для него сертификат. Если сертификат отсутствует в хранилище сертификатов и не передан в командной строке, издатель OPC создает самозаверяющий сертификат. Подробные сведения о методе **InitApplicationSecurityAsync** приводятся здесь: `OpcApplicationConfigurationSecurity.cs`.

Самозаверяющие сертификаты не обеспечивает безопасность, так как не имеют подписи доверенного центра сертификации.

Издатель OPC поддерживает параметры командной строки, позволяющие выполнять следующие действия:

- получение сведений о CSR для текущего сертификата приложения, используемого издателем OPC;
- предоставление издателю OPC сертификата с подписью центра сертификации;
- предоставление издателю OPC новой пары ключей и соответствующего сертификата с подписью центра сертификации;
- добавление сертификатов в доверенный одноранговый узел или в доверенное хранилище сертификатов издателей.
- добавление списка отзыва сертификатов;
- удаление сертификата из доверенного однорангового узла или доверенного хранилища сертификатов издателей.

Все эти варианты позволяют передавать параметры через файлы или строки в кодировке base64.

Для всех хранилищ сертификатов в качестве хранилища по умолчанию используется файловая система, но вы можете переопределить ее через параметры командной строки. Так как контейнер не поддерживает постоянное хранилище в файловой системе, необходимо выбрать другой тип хранилища. Используйте параметр Docker `-v`, чтобы сохранить сертификат в файловой системе узла или на томе Docker. Если вы используете том Docker, сертификаты можно передавать в виде строк в кодировке base64.

Среда выполнения влияет на механизм сохранения сертификатов. Избегайте создания новых хранилищ сертификатов при каждом запуске приложения:

- В собственной среде Windows вы не можете использовать для приложения хранилище сертификатов с типом `Directory`, так как доступ к закрытому ключу не поддерживается. В этом случае используйте вариант `--at X509Store`.
- При запуске в виде контейнера docker для Linux вы можете сопоставить хранилища сертификатов с базовой файловой системой, используя параметр запуска docker `-v <hostdirectory>:/appdata`. Этот параметр обеспечивает сохранение сертификата между запусками приложения.
- При запуске в виде контейнера docker для Linux и при необходимости использовать для сертификата приложения хранилище X509 укажите параметр запуска docker `-v x509certstores:/root/.dotnet/corefx/cryptography/x509stores` и параметр приложения `--at X509Store`.

## <a name="performance-and-memory-considerations"></a>Рекомендации по производительности и управлению памятью

В этом разделе мы обсудим доступные возможности для управления памятью и производительностью.

### <a name="command-line-parameters-to-control-performance-and-memory"></a>Параметры командной строки для управления производительностью и памятью

При запуске издателя OPC вам важно понимать требования к производительности и ресурсам памяти, доступным на узле.

Память и производительность зависят друг от друга и вместе зависят от настроенного числа узлов для публикации. Убедитесь, что следующие параметры соответствуют вашим требованиям:

- Интервал отправки данных в Центр Интернета вещей: `--si`
- Размер сообщения для Центра Интернета вещей (по умолчанию `1`): `--ms`
- Емкость очереди отслеживаемых объектов: `--mq`

Параметр `--mq` определяет верхнюю границу объема для внутренней очереди, которая помещает в буфер все уведомления об изменении значений для узла OPC. Если издатель OPC не может достаточно быстро отправлять сообщения в Центр Интернета вещей, эта очередь накапливает уведомления. Параметр задает количество уведомлений, которые можно сохранить в буфере. Если вы заметите, что число элементов в этой очереди увеличивается во время тестовых запусков, во избежание потери сообщений следует принять следующие меры:

- уменьшить интервал отправки данных в Центра Интернета вещей;
- увеличить размер сообщения для Центра Интернета вещей.

Параметр `--si` вынуждает издателя OPC отправлять сообщения в Центр Интернета вещей с указанным интервалом. Издатель OPC отправляет сообщение, как только его размер достигает предела, указанного в параметре `--ms`, или истекает интервал, указанный в параметре `--si`. Чтобы отключить отправку по размеру сообщения, используйте `--ms 0`. В этом случае издатель OPC комплектует пакеты данных вплоть до максимально допустимой длины сообщений Центра Интернета вещей (256 КБ).

Параметр `--ms` позволяет собирать в пакеты сообщения, отправляемые в Центр Интернета вещей. Используемый протокол определяет, будут ли высокими издержки на отправку сообщения в Центр Интернета по сравнению с фактическим временем отправки полезных данных. Если ваш сценарий допускает некоторую задержку в поступлении данных в Центр Интернета вещей, настройте для издателя OPC максимальный размер сообщения 256 КБ.

Прежде чем использовать издателя OPC в рабочих сценариях, проверьте его производительность и использование памяти в условиях, близких к реальным. С помощью параметра `--di` вы можете указать интервал в секундах, за который издатель OPC собирает диагностические сведения.

### <a name="test-measurements"></a>Тестовые измерения

Следующий пример диагностики демонстрирует измерения с разными значениями параметров `--si` и `--ms` для публикации 500 узлов с интервалом публикации OPC длительностью 1 секунда.  Этот тест запускал отладочную версию издателя OPC, скомпилированную в собственном коде Windows 10, на период в 120 секунд. Для Центра Интернета вещей использовался протокол по умолчанию (MQTT).

#### <a name="default-configuration---si-10---ms-262144"></a>Конфигурация по умолчанию (--si 10 --ms 262144)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:33:05 (started @ 26.10.2017 15:31:09)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54363
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54363
---------------------------------
messages sent to IoT Hub: 109
last successful msg sent @: 26.10.2017 15:33:04
bytes sent to IoT Hub: 12709429
avg msg size: 116600
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 10
--ms setting: 262144
--ih setting: Mqtt
==========================================================================
```

Конфигурация по умолчанию отправляет данные в Центр Интернета вещей каждые 10 секунд или при сборе 256 КБ данных, доступных для отправки в Центр Интернета вещей. Эта конфигурация добавляет среднюю задержку около 10 секунд, но обеспечивает самую низкую вероятность потери данных из-за большого размера сообщения. Диагностические данные показывают, что обновления узла OPC не теряются: `monitored item notifications enqueue failure: 0`.

#### <a name="constant-send-interval---si-1---ms-0"></a>Постоянный интервал отправки (--si 1 --ms 0)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:35:59 (started @ 26.10.2017 15:34:03)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54243
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54243
---------------------------------
messages sent to IoT Hub: 109
last successful msg sent @: 26.10.2017 15:35:59
bytes sent to IoT Hub: 12683836
avg msg size: 116365
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 1
--ms setting: 0
--ih setting: Mqtt
==========================================================================
```

Если размер сообщения имеет значение 0, издатель OPC внутренне собирает пакеты вплоть до максимально допустимого размера сообщений для Центра Интернета вещей (256 КБ). Диагностические данные показывают, что средний размер сообщения составляет 115 019 байт. В такой конфигурации издатель OPC не теряет обновления значений узла OPC и демонстрирует более низкую задержку по сравнению с режимом по умолчанию.

### <a name="send-each-opc-node-value-update---si-0---ms-0"></a>Отправка каждого обновления значений для узла OPC (--si 0 --ms 0)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:39:33 (started @ 26.10.2017 15:37:37)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 8184
monitored item notifications enqueued: 54232
monitored item notifications enqueue failure: 44624
monitored item notifications dequeued: 1424
---------------------------------
messages sent to IoT Hub: 1423
last successful msg sent @: 26.10.2017 15:39:33
bytes sent to IoT Hub: 333046
avg msg size: 234
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 96
--si setting: 0
--ms setting: 0
--ih setting: Mqtt
==========================================================================
```

В такой конфигурации каждое обновление значений для узла OPC отправляется отдельным сообщением в Центр Интернета вещей. Диагностические данные сообщают, что средний размер сообщения составляет 234 байт, что очень мало. Преимуществом такой конфигурации будет отсутствие задержек в работе издателя OPC. Количество потерянных обновлений значений для узла OPC (`monitored item notifications enqueue failure: 44624`) будет высоким, что делает такую конфигурацию непригодной для сценариев с большими объемами публикуемой телеметрии.

### <a name="maximum-batching---si-0---ms-262144"></a>Максимальные объемы пакетов (--si 0 --ms 262144)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:42:55 (started @ 26.10.2017 15:41:00)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54137
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54137
---------------------------------
messages sent to IoT Hub: 48
last successful msg sent @: 26.10.2017 15:42:55
bytes sent to IoT Hub: 12565544
avg msg size: 261782
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 0
--ms setting: 262144
--ih setting: Mqtt
==========================================================================
```

Эта конфигурация собирает в пакеты максимально возможное число обновлений значений для узла OPC. Здесь используется максимальный размер сообщения для Центра Интернета вещей (256 КБ). Интервал отправки не используется, то есть задержка определяется только объемом данных, который может обработать Центр Интернета вещей. Такая конфигурация обеспечивает минимальную вероятность потери информации о значениях для узла OPC и хорошо подходит для публикации большого числа узлов. При использовании этой конфигурации важно следить, чтобы не возникало ситуаций чрезмерно большой задержки из-за медленного достижения размера 256 КБ для сообщений.

## <a name="debug-the-application"></a>Отладка приложения

Чтобы выполнить отладку приложения, откройте файл решения **opcpublisher.sln** в Visual Studio и воспользуйтесь средствами отладки Visual Studio.

Если вам требуется доступ к серверу OPC UA из издателя OPC, убедитесь, что брандмауэр разрешает доступ к порту, который прослушивает сервер. Порт по умолчанию: 62222.

## <a name="control-the-application-remotely"></a>Удаленное управление приложением

Настройку узлов для публикации можно выполнить с помощью прямых методов Центра Интернета вещей.

Издатель OPC реализует несколько дополнительных вызовов прямых методов Центра Интернета вещей для чтения данных:

- Общие сведения.
- Диагностические сведения о сеансах OPC, подписках и отслеживаемых элементах.
- Диагностические сведения о сообщениях и событиях Центра Интернета вещей.
- Журнал запуска.
- Последние 100 строк журнала.
- Завершение работы приложения.

В следующих репозиториях GitHub содержатся средства для [настройки узлов для публикации](https://github.com/Azure-Samples/iot-edge-opc-publisher-nodeconfiguration) и [чтения диагностических сведений](https://github.com/Azure-Samples/iot-edge-opc-publisher-diagnostics). Оба средства также доступны в виде контейнеров в Docker Hub.

## <a name="use-a-sample-opc-ua-server"></a>Использование примера сервера OPC UA

Если у вас нет физических серверов OPC UA, вы можете использовать для работы [пример OPC UA PLC](https://github.com/Azure-Samples/iot-edge-opc-plc). Этот пример PLC также можно найти в Docker Hub.

Он реализует несколько тегов, которые генерируют случайные данные и теги с аномальными результатами. Этот пример можно расширить, если потребуется имитировать значения других тегов.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как запускать издателя OPC, мы рекомендуем перейти к изучению [двойника OPC](overview-opc-twin.md) и [хранилища OPC](overview-opc-vault.md).