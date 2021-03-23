---
title: Работа с Azure Defender для Интернета вещей с помощью API
description: Используйте внешний REST API для доступа к данным, обнаруженным датчиками и консолями управления, и выполнения действий с этими данными.
ms.date: 12/14/2020
ms.topic: reference
ms.openlocfilehash: d509f2674a61af1d0ab03892186526b1cb109eee
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104778837"
---
# <a name="defender-for-iot-sensor-and-management-console-apis"></a>API-интерфейсы для датчиков IoT и консоли управления

Используйте внешний REST API для доступа к данным, обнаруженным датчиками и консолями управления, и выполнения действий с этими данными.

Подключения защищаются по протоколу SSL.

## <a name="getting-started"></a>Начало работы

Как правило, если вы используете внешний API в защитнике Azure для датчика IoT или локальной консоли управления, необходимо создать маркер доступа. Маркеры не требуются для API проверки подлинности, которые используются на датчике и локальной консоли управления.

Создание токена:

1. В окне **системные параметры** выберите **токены доступа**.
  
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/access-tokens.png" alt-text="Снимок экрана: окна параметров системы, в которых выделяется кнопка &quot;маркеры доступа&quot;.":::

2. Выберите **Создать новый маркер**.
   
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/new-token.png" alt-text="Нажмите кнопку, чтобы создать новый маркер.":::

3. Опишите назначение нового маркера и нажмите кнопку **Далее**.
   
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/token-name.png" alt-text="Создайте новый токен и введите имя связанного с ним интеграции.":::

4. Появится маркер доступа. Скопируйте его, так как он больше не будет отображаться.
   
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/token-code.png" alt-text="Скопируйте маркер доступа для интеграции.":::

5. Нажмите кнопку **Готово**. Создаваемые токены отображаются в диалоговом окне **маркеры доступа** .
   
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/access-token-window.png" alt-text="Снимок экрана: диалоговое окно &quot;маркеры устройства&quot; с заполненными маркерами":::

   **Используется** указывает время последнего получения внешнего вызова с этим маркером.

   Если в поле **использовано** для этого маркера отображается значение **N/A** , соединение между датчиком и подключенным сервером не работает.

6. Добавьте в запрос заголовок HTTP с именем **authorization** и присвойте ему значение созданного токена.

## <a name="sensor-api-specifications"></a>Спецификации API датчика

В этом разделе описаны следующие API-интерфейсы датчика:

- [Получение сведений об устройстве —/API/V1/Devices](#retrieve-device-information---apiv1devices)

- [Получение сведений о подключении устройства —/API/V1/Devices/Connections](#retrieve-device-connection-information---apiv1devicesconnections)

- [Получение сведений о CVE-/API/V1/Devices/cves](#retrieve-information-on-cves---apiv1devicescves)

- [Получение сведений о предупреждении —/API/V1/Alerts](#retrieve-alert-information---apiv1alerts)

- [Получение событий временной шкалы —/API/V1/Events](#retrieve-timeline-events---apiv1events)

- [Получение сведений об уязвимости —/API/V1/Reports/Vulnerabilities/Devices](#retrieve-vulnerability-information---apiv1reportsvulnerabilitiesdevices)

- [Получение уязвимостей безопасности —/API/V1/Reports/Vulnerabilities/Security](#retrieve-security-vulnerabilities---apiv1reportsvulnerabilitiessecurity)

- [Получение операционных уязвимостей —/API/V1/Reports/Vulnerabilities/Operational](#retrieve-operational-vulnerabilities---apiv1reportsvulnerabilitiesoperational)

- [Проверка учетных данных пользователя —/АПИ/екстернал/аусентикатион/валидатион](#validate-user-credentials---apiexternalauthenticationvalidation)

- [Изменение пароля —/екстернал/аусентикатион/set_password](#change-password---externalauthenticationset_password)

- [Обновление пароля пользователя по системному администратору —/екстернал/аусентикатион/set_password_by_admin](#user-password-update-by-system-admin---externalauthenticationset_password_by_admin)

### <a name="retrieve-device-information---apiv1devices"></a>Получение сведений об устройстве —/API/V1/Devices

Используйте этот API для запроса списка всех устройств, обнаруженных защитником для центра Интернета вещей.

#### <a name="method"></a>Метод

**GET**

Запрашивает список всех устройств, обнаруженных защитником для центра Интернета вещей.

#### <a name="query-parameters"></a>Параметры запроса

- **авторизовано**: Фильтрация только авторизованных и неавторизованных устройств.

  **Примеры**:

  `/api/v1/devices?authorized=true`

  `/api/v1/devices?authorized=false`

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, представляющих устройства.

#### <a name="device-fields"></a>Поля устройства

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **id** | Числовой | Нет | - |
| **ipAddresses** | Массив JSON | Да | IP-адреса (могут быть более одного адреса в случае адресов Интернета или устройства с двумя сетевыми картами) |
| **name** | Строка | Нет | - |
| **type** | Строка | Нет | Неизвестный, Инженерная станция, ХМИ, historian, контроллер домена, сервер базы данных, точка беспроводного доступа, маршрутизатор, коммутатор, сервер, Рабочая станция, IP-камера, принтер, брандмауэр, станция терминала, шлюз VPN, Интернет или многоадресная рассылка и широковещательная рассылка |
| **макаддрессес** | Массив JSON | Да | MAC-адреса (могут быть более одного адреса в случае устройства с двумя сетевыми картами) |
| **operatingSystem** | Строка | Да | - |
| **енгинирингстатион** | Логическое | Нет | Значение true или false |
| **вирусов** | Логическое | Нет | Значение true или false |
| **авторизовано** | Логическое | Нет | Значение true или false |
| **разработчика** | Строка | Да | - |
| **CHAP** | Массив JSON | Да | Объект протокола |
| **встроенного по** | Массив JSON | Да | Объект встроенного по |

#### <a name="protocol-fields"></a>Поля протокола

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **имя**; | Строка | Нет |  |
| **Адреса** | Массив JSON | Да | Основные или числовые значения |

#### <a name="firmware-fields"></a>Поля встроенного по

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **подключение** | Строка | Нет | Н/д или фактическое значение |
| **model** | Строка | Нет | Н/д или фактическое значение |
| **FirmwareVersion** | Double | Нет | Н/д или фактическое значение |
| **аддитионалдата** | Строка | Нет | Н/д или фактическое значение |
| **модулеаддресс** | Строка | Нет | Н/д или фактическое значение |
| **оптимизирован** | Строка | Нет | Н/д или фактическое значение |
| **резервирован** | Строка | Нет | Н/д или фактическое значение |
| **address** | Строка | Нет | Н/д или фактическое значение |

#### <a name="response-example"></a>Пример ответа

```rest
[

    {
    
    "vendor": null,
    
    "name": "10.4.14.102",
    
    "firmware": [
    
        {
        
            "slot": "N/A",
            
            "additionalData": "N/A",
            
            "moduleAddress": "Network: Local network (0), Node: 0, Unit: CPU (0x0)",
            
            "rack": "N/A",
            
            "address": "10.4.14.102",
            
            "model": "AAAAAAAAAA",
            
            "serial": "N/A",
            
            "firmwareVersion": "20.55"
        
        },
    
        {
        
            "slot": "N/A",
            
            "additionalData": "N/A",
            
            "moduleAddress": "Network: Local network (0), Node: 0, Unit: Unknown (0x3)",
            
            "rack": "N/A",
            
            "address": "10.4.14.102",
            
            "model": "AAAAAAAAAAAAAAAAAAAA",
            
            "serial": "N/A",
            
            "firmwareVersion": "20.55"
        
        },
    
        {
        
            "slot": "N/A",
            
            "additionalData": "N/A",
            
            "moduleAddress": "Network: Local network (0), Node: 3, Unit: CPU (0x0)",
            
            "rack": "N/A",
            
            "address": "10.4.14.102",
            
            "model": "AAAAAAAAAAAAAAAAAAAA",
            
            "serial": "N/A",
            
            "firmwareVersion": "20.55"
        
        },
    
        {
        
            "slot": "N/A",
            
            "additionalData": "N/A",
            
            "moduleAddress": "Network: 3, Node: 0, Unit: CPU (0x0)",
            
            "rack": "N/A",
            
            "address": "10.4.14.102",
            
            "model": "AAAAAAAAAAAAAAAAAAAA",
            
            "serial": "N/A",
            
            "firmwareVersion": "20.55"
        
        }
    
    ],
    
    "id": 79,
    
    "macAddresses": null,
    
    "authorized": true,
    
    "ipAddresses": [
    
        "10.4.14.102"
    
    ],
    
    "engineeringStation": false,
    
    "type": "PLC",
    
    "operatingSystem": null,
    
    "protocols": [
    
        {
        
            "addresses": [],
            
            "id": 62,
            
            "name": "Omron FINS"
        
        }
    
    ],
    
    "scanner": false
    
}

]
```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| GET | фигурный-k-H "Авторизация: <AUTH_TOKEN>" https://<IP_ADDRESS>/API/V1/Devices | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https: <span> //127 <span> . 0.0.1/API/V1/Devices? authorizationed = true |

### <a name="retrieve-device-connection-information---apiv1devicesconnections"></a>Получение сведений о подключении устройства —/API/V1/Devices/Connections

Используйте этот API, чтобы запросить список всех подключений для каждого устройства.

#### <a name="method"></a>Метод

**GET**

#### <a name="query-parameters"></a>Параметры запроса

Если не задать параметры запроса, возвращаются все подключения к устройствам.

**Пример**:

`/api/v1/devices/connections`

- **deviceId**: Фильтрация по ОПРЕДЕЛЕННОму идентификатору устройства для просмотра его соединений.

  **Пример**:

  `/api/v1/devices/<deviceId>/connections`

- **ластактивеинминутес**: интервал времени с обратной стороны, по минуте, в течение которого подключения были активны.

  **Пример**:

  `/api/v1/devices/2/connections?lastActiveInMinutes=20`

- **дисковередбефоре**: Фильтрация только подключений, обнаруженных до определенного времени (в миллисекундах, в формате UTC).

  **Пример**:

  `/api/v1/devices/2/connections?discoveredBefore=<epoch>`

- **дисковередафтер**. Отфильтруйте только соединения, обнаруженные по истечении определенного времени (в миллисекундах, в формате UTC).

  **Пример**:

  `/api/v1/devices/2/connections?discoveredAfter=<epoch>`

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, которые представляют подключения устройств.

#### <a name="fields"></a>Поля

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **фирстдевицеид** | Числовой | Нет | - |
| **секонддевицеид** | Числовой | Нет | - |
| **ластсин** | Числовой | Нет | Эпоха (UTC) |
| **обнаружения** | Числовой | Нет | Эпоха (UTC) |
| **порты** | Массив чисел | Нет | - |
| **CHAP** | Массив JSON | Нет | Поле протокола |

#### <a name="protocol-field"></a>Поле протокола

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **name** | Строка | Нет | - |
| **меню** | Массив строк | Нет | - |

#### <a name="response-example"></a>Пример ответа

```rest
[

    {
    
        "firstDeviceId": 171,
        
        "secondDeviceId": 22,
        
        "lastSeen": 1511281457933,
        
        "discovered": 1511872830000,
        
        "ports": [
        
            502
        
        ],
    
        "protocols": [
        
        {
        
            name: "modbus",
            
            commands: [
            
                "Read Coils"
        
            ]
        
        },
    
        {
        
            name: "ams",
            
            commands: [
            
                "AMS Write"
        
            ]
        
        },
    
        {
        
            name: "http",
            
            commands: [
        
            ]
        
        }
    
    ]
    
    },
    
    {
    
        "firstDeviceId": 171,
        
        "secondDeviceId": 23,
        
        "lastSeen": 1511281457933,
        
        "discovered": 1511872830000,
        
        "ports": [
        
            502
        
        ],
        
        "protocols": [
        
            {
            
                name: "s7comm",
                
                commands: [
                
                    "Download block",
                    
                    "Upload"
            
                ]
            
            }
        
        ]
    
    }

]
```

#### <a name="curl-command"></a>Команда cURL

> [!div class="mx-tdBreakAll"]
> | Тип | Программные интерфейсы | Пример |
> |--|--|--|
> | GET | фигурный-k-H "Авторизация: <AUTH_TOKEN>" https://<IP_ADDRESS>/API/V1/Devices/Connections | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/API/V1/Devices/Connections |
> | GET | фигурная-k-H "Авторизация: <AUTH_TOKEN>" "https://<IP_ADDRESS>/API/V1/Devices/ <deviceId> /коннектионс? ластактивеинминутес =&дисковередбефоре =&дисковередафтер =" | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" "https:/ <span> /127.0.0.1/API/V1/Devices/2/Connections? ластактивеинминутес = 20&дисковередбефоре = 1594550986000&дисковередафтер = 1594550986000" |

### <a name="retrieve-information-on-cves---apiv1devicescves"></a>Получение сведений о CVE-/API/V1/Devices/cves

Этот API используется для запроса списка всех известных CVE, обнаруженных на устройствах в сети.

#### <a name="method"></a>Метод

**GET**

#### <a name="query-parameters"></a>Параметры запроса

По умолчанию этот API предоставляет список всех IP-адресов устройств с CVE, до 100.

**Пример**:

`/api/v1/devices/cves`

- **deviceId**: отфильтровать по ОПРЕДЕЛЕННОМУ IP-адресу устройства, чтобы получить до 100 CVE с высшим рейтингом, идентифицированных на этом конкретном устройстве.

  **Пример**:

  `/api/v1/devices/<ipAddress>/cves`

- **Top**: сколько лучших CVE, которые можно получить для каждого IP-адреса устройства.

  **Пример**:

  `/api/v1/devices/cves?top=50`

  `/api/v1/devices/<ipAddress>/cves?top=50`

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, представляющих CVE, определенных на IP-адресах.

#### <a name="fields"></a>Поля

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **квеид** | Строка | Нет | - |
| **IP** | Строка | Нет | IP-адрес |
| **понять** | Строка | Нет | 0,0-10,0 |
| **аттакквектор** | Строка | Нет | Сеть, смежная сеть, локальная или физическая |
| **description** | Строка | Нет | - |

#### <a name="response-example"></a>Пример ответа

```rest
[

    {
    
        "cveId": "CVE-2007-0099",
        
        "score": "9.3",
        
        "ipAddress": "10.35.1.51",
        
        "attackVector": "NETWORK",
        
        "description": "Race condition in the msxml3 module in Microsoft XML Core
        
        Services 3.0, as used in Internet Explorer 6 and other
        
        applications, allows remote attackers to execute arbitrary
        
        code or cause a denial of service (application crash) via many
        
        nested tags in an XML document in an IFRAME, when synchronous
        
        document rendering is frequently disrupted with asynchronous
        
        events, as demonstrated using a JavaScript timer, which can
        
        trigger NULL pointer dereferences or memory corruption, aka
        
        \"MSXML Memory Corruption Vulnerability.\""
    
    },
    
    {
    
        "cveId": "CVE-2009-1547",
        
        "score": "9.3",
        
        "ipAddress": "10.35.1.51",
        
        "attackVector": "NETWORK",
        
        "description": "Unspecified vulnerability in Microsoft Internet Explorer 5.01
        
        SP4, 6, 6 SP1, and 7 allows remote attackers to execute
        
        arbitrary code via a crafted data stream header that triggers
        
        memory corruption, aka \"Data Stream Header Corruption
        
        Vulnerability.\""
    
    }

]
```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| GET | фигурный-k-H "Авторизация: <AUTH_TOKEN>" https://<IP_ADDRESS>/API/V1/Devices/cves | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/API/V1/Devices/cves |
| GET | фигурная-k-H "Авторизация: <AUTH_TOKEN>" https://<IP_ADDRESS>/API/V1/Devices/ <deviceIpAddress> /квес? Top = | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/API/V1/Devices/10.10.10.15/cves? Top = 50 |

### <a name="retrieve-alert-information---apiv1alerts"></a>Получение сведений о предупреждении —/API/V1/Alerts

Используйте этот API для запроса списка всех предупреждений, обнаруженных защитником для центра Интернета вещей.

#### <a name="method"></a>Метод

**GET**

#### <a name="query-parameters"></a>Параметры запроса

- **State**: Фильтрация только обработанных или необработанных оповещений.

  **Пример**:

  `/api/v1/alerts?state=handled`

- **фромтиме**: Фильтрация предупреждений, созданных за определенное время (в миллисекундах, в формате UTC).

  **Пример**:

  `/api/v1/alerts?fromTime=<epoch>`

- **тотиме**: Фильтрация предупреждений, созданных только до определенного времени (в миллисекундах, в формате UTC).

  **Пример**:

  `/api/v1/alerts?toTime=<epoch>`

- **тип**: для фильтрации предупреждений по определенному типу. Существующие типы для фильтрации: непредвиденные новые устройства, отсоединения.

  **Пример**:

  `/api/v1/alerts?type=disconnections`

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, представляющих предупреждения.

#### <a name="alert-fields"></a>Поля предупреждений

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **Идентификатор** | Числовой | Нет | - |
| **time** | Числовой | Нет | Эпоха (UTC) |
| **title** | Строка | Нет | - |
| **message** | Строка | Нет | - |
| **severity** | Строка | Нет | Предупреждение, дополнительный, основной или критический |
| **ядре** | Строка | Нет | Нарушение протокола, нарушение политики, вредоносная программа, аномалия или работоспособность |
| **саурцедевице** | Числовой | Да | Идентификатор устройства |
| **дестинатиондевице** | Числовой | Да | Идентификатор устройства |
| **аддитионалинформатион** | Дополнительный информационный объект | Да | - |

#### <a name="additional-information-fields"></a>Поля дополнительных сведений

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **description** | Строка | Нет | - |
| **об** | Массив JSON | нет | Строка |

#### <a name="response-example"></a>Пример ответа

```rest
[

    {
    
        "engine": "Policy Violation",
        
        "severity": "Major",
        
        "title": "Internet Access Detected",
        
        "additionalinformation": {
        
            "information": [
            
                "170.60.50.201 over port BACnet (47808)"
            
            ],
            
            "description": "External Addresses"
        
        },
    
        "sourceDevice": null,
        
        "destinationDevice": null,
        
        "time": 1509881077000,
        
        "message": "Device 192.168.0.13 tried to access an external IP address which is an address in the Internet and is not allowed by policy. It is recommended to notify the security officer of the incident.",
        
        "id": 1
    
    },
    
    {
    
        "engine": "Protocol Violation",
        
        "severity": "Major",
        
        "title": "Illegal MODBUS Operation (Exception Raised by Master)",
        
        "sourceDevice": 3,
        
        "destinationDevice": 4,
        
        "time": 1505651605000,
        
        "message": "A MODBUS master 192.168.110.131 attempted to initiate an illegal operation.\nThe operation is considered to be illegal since it incorporated function code \#129 which should not be used by a master.\nIt is recommended to notify the security officer of the incident.",
        
        "id": 2,
        
        "additionalInformation": null,
    
    }

]

```

#### <a name="curl-command"></a>Команда cURL

> [!div class="mx-tdBreakAll"]
> | Тип | Программные интерфейсы | Пример |
> |--|--|--|
> | GET | фигурная-k-H "Авторизация: <AUTH_TOKEN>" "https://<IP_ADDRESS>/API/V1/Alerts? State =&Фромтиме =&Тотиме =&Type =" | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" "https:/ <span> /127.0.0.1/API/V1/Alerts? State = необработанные&фромтиме = 1594550986000&тотиме = 1594550986001&Type = отсоединений" |

### <a name="retrieve-timeline-events---apiv1events"></a>Получение событий временной шкалы —/API/V1/Events

Этот API используется для запроса списка событий, передаваемых на временную шкалу событий.

#### <a name="method"></a>Метод

**GET**

#### <a name="query-parameters"></a>Параметры запроса

- **минутестимефраме**: интервал времени с обратной стороны, по минуте, в котором были переданы события.

  **Пример**:

  `/api/v1/events?minutesTimeFrame=20`

- **тип**: для фильтрации списка событий по определенному типу.

  **Примеры**:

  `/api/v1/events?type=DEVICE_CONNECTION_CREATED`

  `/api/v1/events?type=REMOTE_ACCESS&minutesTimeFrame`

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, представляющих предупреждения.

#### <a name="event-fields"></a>Поля событий

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|--|
| **timestamp** | Числовой | Нет | Эпоха (UTC) |
| **title** | Строка | Нет | - |
| **severity** | Строка | Нет | СВЕДЕНИЯ, уведомление или ОПОВЕЩЕНие |
| **владельцев** | Строка | Да | Если событие было создано вручную, в это поле будет включено имя пользователя, создавшего событие. |
| **content** | Строка | Нет | - |

#### <a name="response-example"></a>Пример ответа

```rest
[

    {
    
        "severity": "INFO",
        
        "title": "Back to Normal",
        
        "timestamp": 1504097077000,
        
        "content": "Device 10.2.1.15 was found responsive, after being suspected as disconnected",
        
        "owner": null,
        
        "type": "BACK_TO_NORMAL"
    
    },
    
    {
    
        "severity": "ALERT",
        
        "title": "Alert Detected",
        
        "timestamp": 1504096909000,
        
        "content": "Device 10.2.1.15 is suspected to be disconnected (unresponsive).",
        
        "owner": null,
        
        "type": "ALERT_REPORTED"
    
    },
    
    {
    
        "severity": "ALERT",
        
        "title": "Alert Detected",
        
        "timestamp": 1504094446000,
        
        "content": "A DNP3 Master 10.2.1.14 attempted to initiate a request which is not allowed by policy.\nThe policy indicates the allowed function codes, address ranges, point indexes and time intervals.\nIt is recommended to notify the security officer of the incident.",
        
        "owner": null,
        
        "type": "ALERT_REPORTED"
    
    },
    
    {
    
        "severity": "NOTICE",
        
        "title": "PLC Program Update",
        
        "timestamp": 1504094344000,
        
        "content": "Program update detected, sent from 10.2.1.25 to 10.2.1.14",
        
        "owner": null,
        
        "type": "PROGRAM_DEVICE"
    
    }

]

```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| GET | фигурная-k-H "Авторизация: <AUTH_TOKEN>" "https://<IP_ADDRESS>/API/V1/Events? Минутестимефраме =&Type =" | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" "https:/ <span> /127.0.0.1/API/V1/Events? минутестимефраме = 20&Type = DEVICE_CONNECTION_CREATED" |

### <a name="retrieve-vulnerability-information---apiv1reportsvulnerabilitiesdevices"></a>Получение сведений об уязвимости —/API/V1/Reports/Vulnerabilities/Devices

Используйте этот API для запроса результатов оценки уязвимостей для каждого устройства.

#### <a name="method"></a>Метод

**GET**

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, которые представляют устройства с оценкой.

Объект Device содержит:

- Общие данные

- Оценка оценки

- Уязвимости

#### <a name="device-fields"></a>Поля устройства

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **name** | Строка | Нет | - |
| **ipAddresses** | Массив JSON | Нет | - |
| **секуритискоре** | Числовой | Нет | - |
| **разработчика** | Строка | Да |  |
| **FirmwareVersion** | Строка | Да | - |
| **model** | Строка | Да | - |
| **исвирелессакцесспоинт** | Логическое | Нет | Значение true или false |
| **operatingSystem** | Объект операционной системы | Да | - |
| **делают** | Объект уязвимостей | Да | - |

#### <a name="operating-system-fields"></a>Поля операционной системы

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **имя**; | Строка | Да | - |
| **Тип** | Строка | Да | - |
| **Версия** | Строка | Да | - |
| **latestVersion** | Строка | Да | - |

#### <a name="vulnerabilities-fields"></a>Поля уязвимостей
 
| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **антивирусные программы** | Массив JSON | Да | Имена антивирусных программ |
| **плаинтекстпассвордс** | Массив JSON | Да | Объекты пароля |
| **Доступа** | Массив JSON | Да | Объекты удаленного доступа |
| **исбаккупсервер** | Логическое | Нет | Значение true или false |
| **опенедпортс** | Массив JSON | Да | Открытые объекты порта |
| **исенгинирингстатион** | Логическое | Нет | Значение true или false |
| **искновнсканнер** | Логическое | Нет | Значение true или false |
| **CVE** | Массив JSON | Да | CVE объектов |
| **не санкционировано** | Логическое | Нет | Значение true или false |
| **малвареиндикатионсдетектед** | Логическое | Нет | Значение true или false |
| **веакаусентикатион** | Массив JSON | Да | Обнаруженные приложения, использующие слабую проверку подлинности |

#### <a name="password-fields"></a>Поля пароля

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **password** | Строка | Нет | - |
| **protocol** | Строка | Нет | - |
| **сложность** | Строка | Нет | Очень слабый, слабый, средний или строгий |

#### <a name="remote-access-fields"></a>Поля удаленного доступа

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **port** | Числовой | Нет | - |
| **перемещения** | Строка | Нет | TCP или UDP |
| **компьютера** | Строка | Нет | IP-адрес |
| **клиентсофтваре** | Строка | Нет | SSH, VNC, удаленный рабочий стол или средство просмотра команд |

#### <a name="open-port-fields"></a>Открыть поля порта

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **port** | Числовой | Нет | - |
| **перемещения** | Строка | Нет | TCP или UDP |
| **protocol** | Строка | Да | - |
| **исконфликтингвисфиревалл** | Логическое | Нет | Значение true или false |

#### <a name="cve-fields"></a>CVE Fields

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **Идентификатор** | Строка | Нет | - |
| **понять** | Числовой | Нет | Double |
| **description** | Строка | Нет | - |

#### <a name="response-example"></a>Пример ответа

```rest
[

    {
    
        "name": "IED \#10",
        
        "ipAddresses": ["10.2.1.10"],
        
        "securityScore": 100,
        
        "vendor": "ABB Switzerland Ltd, Power Systems",
        
        "firmwareVersion": null,
        
        "model": null,
        
        "operatingSystem": {
        
            "name": "ABB Switzerland Ltd, Power Systems",
            
            "type": "abb",
            
            "version": null,
            
            "latestVersion": null
        
        },
        
        "vulnerabilities": {
            
        "antiViruses": [
            
        "McAfee"
            
        ],
            
        "plainTextPasswords": [
            
            {
            
                "password": "123456",
                
                "protocol": "HTTP",
                
                "lastSeen": 1462726930471,
                
                "strength": "Very Weak"
            
            }
            
        ],
            
        "remoteAccess": [
            
            {
            
                "port": 5900,
                
                "transport": "TCP",
                
                "clientSoftware": "VNC",
                
                "client": "10.2.1.20"
            
            }
            
        ],
            
        "isBackupServer": true,
            
        "openedPorts": [
            
            {
            
                "port": 445,
                
                "transport": "TCP",
                
                "protocol": "SMP Over IP",
                
                "isConflictingWithFirewall": false
            
            },
            
            {
            
                "port": 80,
                
                "transport": "TCP",
                
                "protocol": "HTTP",
                
                "isConflictingWithFirewall": false
            
            }
            
        ],
            
        "isEngineeringStation": false,
            
        "isKnownScanner": false,
            
        "cves": [
            
            {
            
                "id": "CVE-2015-6490",
                
                "score": 10,
                
                "description": "Frosty URL - Stack-based buffer overflow on Allen-Bradley MicroLogix 1100 devices before B FRN 15.000 and 1400 devices through B FRN 15.003 allows remote attackers to execute arbitrary code via unspecified vectors"
            
            },
            
            {
            
                "id": "CVE-2012-6437",
                
                "score": 10,
                
                "description": "MicroLogix 1100 and 1400 do not properly perform authentication for Ethernet firmware updates, which allows remote attackers to execute arbitrary code via a Trojan horse update image"
            
            },
            
            {
            
                "id": "CVE-2012-6440",
                
                "score": 9.3,
                
                "description": "MicroLogix 1100 and 1400 allows man-in-the-middle attackers to conduct replay attacks via HTTP traffic."
        
            }
        
        ],
        
        "isUnauthorized": false,
        
        "malwareIndicationsDetected": true
        
        }
    
    }

]

```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| GET | фигурный-k-H "Авторизация: <AUTH_TOKEN>" https://<IP_ADDRESS>/API/V1/Reports/Vulnerabilities/Devices | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/API/V1/Reports/Vulnerabilities/Devices |

### <a name="retrieve-security-vulnerabilities---apiv1reportsvulnerabilitiessecurity"></a>Получение уязвимостей безопасности —/API/V1/Reports/Vulnerabilities/Security

Используйте этот API для запроса результатов общей оценки уязвимостей. Эта оценка позволяет понять уровень безопасности системы.

Эта оценка основана на общих сетевых и системных данных, а не на оценке конкретных устройств.

#### <a name="method"></a>Метод

**GET**

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Объект JSON, представляющий оцененные результаты. Каждый ключ может допускать значение null. В противном случае он будет содержать объект JSON с ключами, не допускающими значения NULL.

### <a name="result-fields"></a>Поля результатов

**Ключи**

**унаусоризеддевицес**

| Имя поля | Тип | Список значений |
| ---------- | ---- | -------------- |
| **address** | Строка | IP-адрес |
| **name** | Строка | - |
| **фирстдетектионтиме** | Числовой | Эпоха (UTC) |
| ластсин | Числовой | Эпоха (UTC) |

**иллегалтраффикбифиреваллрулес**

| Имя поля | Тип | Список значений |
| ---------- | ---- | -------------- |
| **server** | Строка | IP-адрес |
| **компьютера** | Строка | IP-адрес |
| **port** | Числовой | - |
| **перемещения** | Строка | TCP, UDP или ICMP |

**веакфиреваллрулес**

| Имя поля | Тип | Список значений |
| ---------- | ---- | -------------- |
| **sources** | Массив источников JSON. Каждый источник может иметь любой из четырех форматов. | "Any", "IP-адрес (узел)", "от IP-к IP (диапазон)", "IP-адрес, маска подсети (сеть)" |
| **Конечно** | Массив назначений JSON. Каждое назначение может иметь любой из четырех форматов. | "Any", "IP-адрес (узел)", "от IP-к IP (диапазон)", "IP-адрес, маска подсети (сеть)" |
| **порты** | Массив JSON портов в любом из трех форматов | "Any", "порт (протокол, если он обнаружен)", "от порта к порту (протокол, если обнаружено)" |

**акцесспоинтс**

| Имя поля | Тип | Список значений |
| ---------- | ---- | -------------- |
| **macAddress** | Строка | MAC-адрес. |
| **разработчика** | Строка | Имя поставщика |
| **IP** | Строка | IP-адрес или н/д |
| **name** | Строка | Имя устройства или н/д |
| **подключение** | Строка | Нет, не имеет подозрений или Да |

**коннектионсбетвинсубнетс**

| Имя поля | Тип | Список значений |
| ---------- | ---- | -------------- |
| **server** | Строка | IP-адрес |
| **компьютера** | Строка | IP-адрес |

**индустриалмалвареиндикаторс**

| Имя поля | Тип | Список значений |
| ---------- | ---- | -------------- |
| **детектионтиме** | Числовой | Эпоха (UTC) |
| **alertMessage** | Строка | - |
| **description** | Строка | - |
| **устройствах** | Массив JSON | имена устройств; | 

**интернетконнектионс**

| Имя поля | Тип | Список значений |
| ---------- | ---- | -------------- |
| **интерналаддресс** | Строка | IP-адрес |
| **авторизовано** | Логическое | "Да" или "Нет". | 
| **екстерналаддрессес** | Массив JSON | IP-адрес |

#### <a name="response-example"></a>Пример ответа

```rest
{

    "unauthorizedDevices": [
    
        {
        
            "address": "10.2.1.14",
            
            "name": "PLC \#14",
            
            "firstDetectionTime": 1462645483000,
            
            "lastSeen": 1462645495000,
        
        }
    
    ],
    
    "redundantFirewallRules": [
    
        {
        
            "sources": "170.39.3.0/255.255.255.0",
            
            "destinations": "Any",
            
            "ports": "102"
        
        }
    
    ],
    
    "connectionsBetweenSubnets": [
    
        {
        
            "server": "10.2.1.22",
            
            "client": "170.39.2.0"
        
        }
    
    ],
    
    "industrialMalwareIndications": [
    
        {
        
            "detectionTime": 1462645483000,
            
            "alertMessage": "Suspicion of Malicious Activity (Regin)",
            
            "description": "Suspicious network activity was detected. Such behavior might be attributed to the Regin malware.",
            
            "addresses": [
            
                "10.2.1.4",
                
                "10.2.1.5"
            
            ]
        
        }
    
    ],
    
    "illegalTrafficByFirewallRules": [
    
        {
        
            "server": "10.2.1.7",
            
            "port": "20000",
            
            "client": "10.2.1.4",
            
            "transport": "TCP"
        
        },
    
        {
        
            "server": "10.2.1.8",
            
            "port": "20000",
            
            "client": "10.2.1.4",
            
            "transport": "TCP"
        
        },
    
        {
        
            "server": "10.2.1.9",
            
            "port": "20000",
            
            "client": "10.2.1.4",
            
            "transport": "TCP"
        
        }
    
    ],
    
    "internetConnections": [
    
        {
        
            "internalAddress": "10.2.1.1",
            
            "authorized": "Yes",
            
            "externalAddresses": ["10.2.1.2",”10.2.1.3”]
        
        }
    
    ],
    
    "accessPoints": [
    
        {
        
            "macAddress": "ec:08:6b:0f:1e:22",
            
            "vendor": "TP-LINK TECHNOLOGIES",
            
            "ipAddress": "173.194.112.22",
            
            "name": "Enterprise AP",
            
            "wireless": "Yes"
        
        }
    
    ],
    
    "weakFirewallRules": [
    
        {
        
            "sources": "170.39.3.0/255.255.255.0",
            
            "destinations": "Any",
            
            "ports": "102"
        
        }
    
    ]

}

```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| GET | фигурный-k-H "Авторизация: <AUTH_TOKEN>" https://<IP_ADDRESS>/API/V1/Reports/Vulnerabilities/Security | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/API/V1/Reports/Vulnerabilities/Security |

### <a name="retrieve-operational-vulnerabilities---apiv1reportsvulnerabilitiesoperational"></a>Получение операционных уязвимостей —/API/V1/Reports/Vulnerabilities/Operational

Используйте этот API для запроса результатов общей оценки уязвимостей. Эта оценка позволяет понять операционное состояние сети. Он основан на общей сетевой и системной информации, а не на оценке конкретного устройства.

#### <a name="method"></a>Метод

**GET**

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Объект JSON, представляющий оцененные результаты. Каждый ключ содержит массив результатов JSON.

#### <a name="result-fields"></a>Поля результатов

**Ключи**

**баккупсервер**

| Имя поля | Тип | Список значений |
|--|--|--|
| **source** | Строка | IP-адрес |
| **назначение** | Строка | IP-адрес |
| **port** | Числовой | - |
| **перемещения** | Строка | TCP или UDP |
| **баккупмаксималинтервал** | Строка | - |
| **ластсинбаккуп** | Числовой | Эпоха (UTC) |

**ипнетворкс**

| Имя поля | Тип | Список значений |
|--|--|--|
| **адрес** s | Числовой | - |
| **сети** | Строка | IP-адрес |
| **виде** | Строка | Маска подсети |

**протоколпроблемс**

| Имя поля | Тип | Список значений |
|--|--|--|
| **protocol** | Строка | - |
| **Addresses** | Массив JSON | IP-адреса |
| **получаете** | Строка | - |
| **репорттиме** | Числовой | Эпоха (UTC) |

**протоколдатаволумес**

| Имя поля | Тип | Список значений |
|--|--|--|
| protocol | Строка | - |
| том | Строка | "номер тома МБ" |

**отключения**

| Имя поля | Тип | Список значений |
|--|--|--|
| **ассетаддресс** | Строка | IP-адрес |
| **assetName** | Строка | - |
| **ластдетектионтиме** | Числовой | Эпоха (UTC) |
| **бакктонормалтиме** | Числовой | Эпоха (UTC) |     

#### <a name="response-example"></a>Пример ответа

```rest
{

    "backupServer": [
    
        {
        
            "backupMaximalInterval": "1 Hour, 29 Minutes",
            
            "source": "10.2.1.22",
            
            "destination": "170.39.2.14",
            
            "port": 10000,
            
            "transport": "TCP",
            
            "lastSeenBackup": 1462645483000
        
        }
    
    ],

    "ipNetworks": [
    
        {
        
            "addresses": "21",
            
            "network": "10.2.1.0",
            
            "mask": "255.255.255.0"
        
        },
        
        {
        
            "addresses": "3",
            
            "network": "170.39.2.0",
            
            "mask": "255.255.255.0"
        
        }
    
    ],

    "protocolProblems": [
    
        {
        
            "protocol": "DNP3",
            
            "addresses": [
            
                "10.2.1.7",
                
                "10.2.1.8"
            
            ],
            
            "alert": "Illegal DNP3 Operation",
            
            "reportTime": 1462645483000
        
        },
        
        {
        
            "protocol": "DNP3",
            
            "addresses": [
            
                "10.2.1.15"
            
            ],
            
            "alert": "Master Requested an Application Layer Confirmation",
            
            "reportTime": 1462645483000
        
        }
        
    ],

    "protocolDataVolumes": [
    
        {
        
            "protocol": "MODBUS (502)",
            
            "volume": "21.07 MB"
        
        },
        
        {
        
            "protocol": "SSH (22)",
            
            "volumn": "0.001 MB"
        
        }
    
    ],
    
    "disconnections": [
    
        {
        
            "assetAddress": "10.2.1.3",
            
            "assetName": "PLC \#3",
            
            "lastDetectionTime": 1462645483000,
            
            "backToNormalTime": 1462645484000
        
        }
    
    ]

}

```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| GET | фигурный-k-H "Авторизация: <AUTH_TOKEN>" https://<IP_ADDRESS>/API/V1/Reports/Vulnerabilities/Operational | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/API/V1/Reports/Vulnerabilities/Operational |

### <a name="validate-user-credentials---apiexternalauthenticationvalidation"></a>Проверка учетных данных пользователя —/АПИ/екстернал/аусентикатион/валидатион

Используйте этот API для проверки имени пользователя и пароля IoT в защитнике. С API можно работать все роли пользователей в защитнике IoT.

Для использования этого API нет необходимости в защитнике для маркера доступа IoT.

#### <a name="method"></a>Метод

**POST**

#### <a name="request-type"></a>Тип запроса

**JSON**

#### <a name="query-parameters"></a>Параметры запроса

| **имя**; | **Тип** | **Допускает значения NULL** |
|--|--|--|
| **username** | Строка | Нет |
| **password** | Строка | Нет |

#### <a name="request-example"></a>Пример запроса

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!"

}

```

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Строка сообщения с подробными сведениями о состоянии операции:

- **Успешно — сообщение**: проверка подлинности выполнена успешно

- **Сбой: ошибка** при проверке учетных данных

#### <a name="response-example"></a>Пример ответа

```rest
response:

{

    "msg": "Authentication succeeded."

}

```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| GET | фигурный-k-H "Авторизация: <AUTH_TOKEN>" https://<IP_ADDRESS>/АПИ/екстернал/аусентикатион/валидатион | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/API/External/Authentication/Validation |

### <a name="change-password---externalauthenticationset_password"></a>Изменение пароля —/екстернал/аусентикатион/set_password

Используйте этот API, чтобы позволить пользователям изменять свои пароли. С API можно работать все роли пользователей в защитнике IoT. Для использования этого API нет необходимости в защитнике для маркера доступа IoT.

#### <a name="method"></a>Метод

**POST**

#### <a name="request-type"></a>Тип запроса

**JSON**

#### <a name="request-example"></a>Пример запроса

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!",
    
    "new_password": "Test54321\!"

}

```

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Строка сообщения с подробными сведениями о состоянии операции:

- **Успешно — сообщение**: пароль был заменен.

- **Сбой — ошибка**. Ошибка проверки подлинности пользователя

- **Сбой — ошибка**: пароль не соответствует политике безопасности.

#### <a name="response-example"></a>Пример ответа

```rest
response:

{

    "error": {
    
        "userDisplayErrorMessage": "User authentication failure"
    
    }

}

```

#### <a name="device-fields"></a>Поля устройства

| **имя**; | **Тип** | **Допускает значения NULL** |
|--|--|--|
| **username** | Строка | Нет |
| **password** | Строка | Нет |
| **new_password** | Строка | Нет |

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| POST | фигурная-k-d "{" имя_пользователя ":" <USER_NAME> "," пароль ":" <CURRENT_PASSWORD> "," new_password ":" <NEW_PASSWORD> "}"-H "Content-Type: Application/JSON" https://<IP_ADDRESS>/АПИ/екстернал/аусентикатион/set_password | фигурная-k-d "{" имя_пользователя ":" myUser "," Password ":" 1234@abcd "," new_password ":" abcd@1234 "}"-H "тип содержимого: Application/JSON" https:/ <span> /127.0.0.1/API/External/Authentication/set_password |

### <a name="user-password-update-by-system-admin---externalauthenticationset_password_by_admin"></a>Обновление пароля пользователя по системному администратору —/екстернал/аусентикатион/set_password_by_admin

Используйте этот API, чтобы позволить системным администраторам изменять пароли для указанных пользователей. С API могут работать защитник для ролей пользователей с правами администратора IoT. Для использования этого API нет необходимости в защитнике для маркера доступа IoT.

#### <a name="method"></a>Метод

**POST**

#### <a name="request-type"></a>Тип запроса

**JSON**

#### <a name="request-example"></a>Пример запроса

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!",
    
    "new_password": "Test54321\!"

}
```

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Строка сообщения с подробными сведениями о состоянии операции:

- **Успешно — сообщение**: пароль был заменен.

- **Сбой — ошибка**. Ошибка проверки подлинности пользователя

- **Сбой — ошибка**: пользователь не существует

- **Сбой — ошибка**. пароль не соответствует политике безопасности

- **Сбой — ошибка**: у пользователя нет разрешений на изменение пароля

#### <a name="response-example"></a>Пример ответа

```rest
response:

{

    "error": {
    
        "userDisplayErrorMessage": "The user 'test_user' doesn't exist",
        
        "internalSystemErrorMessage": "The user 'yoavfe' doesn't exist"
    
    }

}

```

#### <a name="device-fields"></a>Поля устройства

| **имя**; | **Тип** | **Допускает значения NULL** |
|--|--|--|
| **admin_username** | Строка | Нет |
| **admin_password** | Строка | Нет |
| **username** | Строка | Нет |
| **new_password** | Строка | Нет |

#### <a name="curl-command"></a>Команда cURL

> [!div class="mx-tdBreakAll"]
> | Тип | Программные интерфейсы | Пример |
> |--|--|--|
> | POST | фигурная скобка-d "{" admin_username ":" <ADMIN_USERNAME> "," admin_password ":" <ADMIN_PASSWORD> "," Username ":" <user_name> "," new_password ":" <NEW_PASSWORD> "}"-H "тип содержимого: Application/JSON" https://<IP_ADDRESS>/АПИ/екстернал/аусентикатион/set_password_by_admin | фигурная-k-d "{" admin_user ":" adminUser "," admin_password ":" 1234@abcd "," Username ":" myuser "," new_password ":" abcd@1234 "}"-H "тип содержимого: Application/JSON" https:/ <span> /127.0.0.1/API/External/Authentication/set_password_by_admin |

## <a name="on-premises-management-console-api-specifications"></a>Спецификации API локальной консоли управления

В этом разделе описаны следующие API-интерфейсы локальной консоли управления:

- **/external/v1/alerts/<UUID>**

- **Исключения оповещений (период обслуживания)**

:::image type="content" source="media/references-work-with-defender-for-iot-apis/alert-exclusion-window.png" alt-text="Окно исключения предупреждений, в котором отображаются активные правила.":::

Определите условия, при которых оповещения не будут отправляться. Например, определите и обновите время окончания и начала работы, устройства или подсети, которые должны быть исключены при активации оповещений, или защитник для механизмов Интернета вещей, которые следует исключить. Например, в течение периода обслуживания может потребоваться отключить доставку всех предупреждений, за исключением оповещений о вредоносных программах на критических устройствах.

Интерфейсы API, которые вы определяете здесь, отображаются в окне **исключения предупреждений** локальной консоли управления как правило исключения только для чтения.

#### <a name="externalv1maintenancewindow"></a>/external/v1/maintenanceWindow

- **/екстернал/аусентикатион/валидатион**

- **Пример ответа**

- **ответ**

```rest
{
    "msg": "Authentication succeeded."
}

```

#### <a name="change-password---externalauthenticationset_password"></a>Изменение пароля —/екстернал/аусентикатион/set_password

Используйте этот API, чтобы позволить пользователям изменять свои пароли. С API можно работать все роли пользователей в защитнике IoT. Для использования этого API нет необходимости в защитнике для маркера доступа IoT.

#### <a name="user-password-update-by-system-admin---externalauthenticationset_password_by_admin"></a>Обновление пароля пользователя по системному администратору —/екстернал/аусентикатион/set_password_by_admin

Используйте этот API, чтобы позволить системным администраторам изменять пароли для конкретных пользователей. С API может работать защитник для ролей пользователей с правами администратора IoT. Для использования этого API нет необходимости в защитнике для маркера доступа IoT.

### <a name="retrieve-device-information---externalv1devices"></a>Получение сведений об устройстве —/External/v1/Devices

Этот API запрашивает список всех устройств, обнаруженных защитником для датчиков Интернета вещей, подключенных к локальной консоли управления.

#### <a name="method"></a>Метод

**GET**

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, представляющих устройства.

#### <a name="query-parameters"></a>Параметры запроса

- **авторизовано**: Фильтрация только авторизованных и неавторизованных устройств.

- Идентификатор **сайта**: Фильтрация только устройств, связанных с конкретными сайтами.

- **КодЗоны**: Фильтрация только устройств, связанных с конкретными зонами. [1](#1)

- **сенсорид**: Фильтрация только устройств, обнаруженных конкретными датчиками. [1](#1)

###### <a name="you-might-not-have-the-site-and-zone-id-if-this-is-the-case-query-all-devices-to-retrieve-the-site-and-zone-id"></a><a id="1">1</a> *возможно, у вас нет сайта и идентификатора зоны. Если это так, запросите все устройства, чтобы получить идентификатор сайта и зоны.*

#### <a name="query-parameters-example"></a>Пример параметров запроса

`/external/v1/devices?authorized=true`

`/external/v1/devices?authorized=false`

`/external/v1/devices?siteId=1,2`

`/external/v1/devices?zoneId=5,6`

`/external/v1/devices?sensorId=8`

#### <a name="device-fields"></a>Поля устройства

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **sensorId** | Числовой | Нет | - |
| **КодЗоны** | Числовой | Да | - |
| **Идентификатор** | Числовой | Да | - |
| **ipAddresses** | Массив JSON | Да | IP-адреса (могут быть более одного адреса в случае адресов Интернета или устройства с двумя сетевыми картами) |
| **name** | Строка | Нет | - |
| **type** | Строка | Нет | Неизвестный, Инженерная станция, ХМИ, historian, контроллер домена, сервер базы данных, точка беспроводного доступа, маршрутизатор, коммутатор, сервер, Рабочая станция, IP-камера, принтер, брандмауэр, станция терминала, шлюз VPN, Интернет или многоадресная рассылка и широковещательная рассылка |
| **макаддрессес** | Массив JSON | Да | MAC-адреса (могут быть более одного адреса в случае устройства с двумя сетевыми картами) |
| **operatingSystem** | Строка | Да | - |
| **енгинирингстатион** | Логическое | Нет | Значение true или false |
| **вирусов** | Логическое | Нет | Значение true или false |
| **авторизовано** | Логическое | Нет | Значение true или false |
| **разработчика** | Строка | Да | - |
| **Протоколы** | Массив JSON | Да | Объект протокола |
| **встроенного по** | Массив JSON | Да | Объект встроенного по |

#### <a name="protocol-fields"></a>Поля протокола

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| name | Строка | Нет | - |
| Адреса | Массив JSON | Да | Основные или числовые значения |

#### <a name="firmware-fields"></a>Поля встроенного по

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **подключение** | Строка | Нет | Н/д или фактическое значение |
| **model** | Строка | Нет | Н/д или фактическое значение |
| **FirmwareVersion** | Double | Нет | Н/д или фактическое значение |
| **аддитионалдата** | Строка | Нет | Н/д или фактическое значение |
| **модулеаддресс** | Строка | Нет | Н/д или фактическое значение |
| **оптимизирован** | Строка | Нет | Н/д или фактическое значение |
| **резервирован** | Строка | Нет | Н/д или фактическое значение |
| **address** | Строка | Нет | Н/д или фактическое значение |

#### <a name="response-example"></a>Пример ответа

```rest
[

    {
    
    "sensorId": 7,
    
    "zoneId": 1,
    
    "siteId": 1,
    
    "vendor": null,
    
    "name": "10.4.14.102",
    
    "firmware": [
    
    {
    
        "slot": "N/A",
        
        "additionalData": "N/A",
        
        "moduleAddress": "Network: Local network (0), Node: 0, Unit: CPU (0x0)",
        
        "rack": "N/A",
        
        "address": "10.4.14.102",
        
        "model": "AAAAAAAAAA",
        
        "serial": "N/A",
        
        "firmwareVersion": "20.55"
    
    },
    
    {
    
        "slot": "N/A",
        
        "additionalData": "N/A",
        
        "moduleAddress": "Network: Local network (0), Node: 0, Unit: Unknown (0x3)",
        
        "rack": "N/A",
        
        "address": "10.4.14.102",
        
        "model": "AAAAAAAAAAAAAAAAAAAA",
        
        "serial": "N/A",
        
        "firmwareVersion": "20.55"
    
    },
    
    {
    
        "slot": "N/A",
        
        "additionalData": "N/A",
        
        "moduleAddress": "Network: Local network (0), Node: 3, Unit: CPU (0x0)",
        
        "rack": "N/A",
        
        "address": "10.4.14.102",
        
        "model": "AAAAAAAAAAAAAAAAAAAA",
        
        "serial": "N/A",
        
        "firmwareVersion": "20.55"
    
    },
    
    {
    
        "slot": "N/A",
        
        "additionalData": "N/A",
        
        "moduleAddress": "Network: 3, Node: 0, Unit: CPU (0x0)",
        
        "rack": "N/A",
        
        "address": "10.4.14.102",
        
        "model": "AAAAAAAAAAAAAAAAAAAA",
        
        "serial": "N/A",
        
        "firmwareVersion": "20.55"
    
    }

],

"id": 79,

"macAddresses": null,

"authorized": true,

"ipAddresses": [

    "10.4.14.102"

],

"engineeringStation": false,

"type": "PLC",

"operatingSystem": null,

"protocols": [

    {
    
        "addresses": [],
        
        "id": 62,
        
        "name": "Omron FINS"
    
    }

],

"scanner": false

}

]
```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| GET | фигурная-k-H "Авторизация: <AUTH_TOKEN>" "https://<>IP_ADDRESS>/External/v1/Devices? идентификатор сайта =&КодЗоны =&Сенсорид =&авторизованный =" | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" "https:/ <span> /127.0.0.1/External/v1/Devices? код сайта = 1&КодЗоны = 2&сенсорид = 5&авторизованный = true" |

### <a name="retrieve-alert-information---externalv1alerts"></a>Получение сведений о предупреждении —/External/v1/Alerts

Этот API используется для получения всех или отфильтрованных предупреждений из локальной консоли управления.

#### <a name="method"></a>Метод

**GET**

#### <a name="query-parameters"></a>Параметры запроса

- **State**: Фильтрация только обработанных и необработанных оповещений.

  **Пример**:

  `/api/v1/alerts?state=handled`

- **фромтиме**: Фильтрация предупреждений, созданных за определенное время (в миллисекундах, в формате UTC).

  **Пример**:

  `/api/v1/alerts?fromTime=<epoch>`

- **тотиме**: Фильтрация предупреждений, созданных только до определенного времени (в миллисекундах, в формате UTC).

  **Пример**:

  `/api/v1/alerts?toTime=<epoch>`

- Идентификатор **сайта. сайт**, на котором было обнаружено предупреждение. [2](#2)

- **КодЗоны**— зона, в которой было обнаружено предупреждение. [2](#2)

- **датчик**: датчик, на котором было обнаружено предупреждение.

##### <a name="you-might-not-have-the-site-and-zone-id-if-this-is-the-case-query-all-devices-to-retrieve-the-site-and-zone-id"></a><a id="2">2</a> *возможно, у вас нет сайта и идентификатора зоны. Если это так, запросите все устройства, чтобы получить идентификатор сайта и зоны.*

#### <a name="alert-fields"></a>Поля предупреждений

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **Идентификатор** | Числовой | Нет | - |
| **time** | Числовой | Нет | Эпоха (UTC) |
| **title** | Строка | Нет | - |
| **message** | Строка | Нет | - |
| **severity** | Строка | Нет | Предупреждение, дополнительный, основной или критический |
| **ядре** | Строка | Нет | Нарушение протокола, нарушение политики, вредоносная программа, аномалия или работоспособность |
| **саурцедевице** | Числовой | Да | Идентификатор устройства |
| **дестинатиондевице** | Числовой | Да | Идентификатор устройства |
| **аддитионалинформатион** | Дополнительный информационный объект | Да | - |

#### <a name="additional-information-fields"></a>Поля дополнительных сведений

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **description** | Строка | Нет | - |
| **об** | Массив JSON | нет | Строка |

#### <a name="response-example"></a>Пример ответа

```rest
[

    {
    
        "engine": "Operational",
        
        "handled": false,
        
        "title": "Traffic Detected on sensor Interface",
        
        "additionalInformation": null,
        
        "sourceDevice": 0,
        
        "zoneId": 1,
        
        "siteId": 1,
        
        "time": 1594808245000,
        
        "sensorId": 1,
        
        "message": "The sensor resumed detecting network traffic on ens224.",
        
        "destinationDevice": 0,
        
        "id": 1,
        
        "severity": "Warning"
    
    },
    
    {
    
        "engine": "Anomaly",
        
        "handled": false,
        
        "title": "Address Scan Detected",
        
        "additionalInformation": null,
        
        "sourceDevice": 4,
        
        "zoneId": 1,
        
        "siteId": 1,
        
        "time": 1594808260000,
        
        "sensorId": 1,
        
        "message": "Address scan detected.\nScanning address: 10.10.10.22\nScanned subnet: 10.11.0.0/16\nScanned addresses: 10.11.1.1, 10.11.20.1, 10.11.20.10, 10.11.20.100, 10.11.20.2, 10.11.20.3, 10.11.20.4, 10.11.20.5, 10.11.20.6, 10.11.20.7...\nIt is recommended to notify the security officer of the incident.",
        
        "destinationDevice": 0,
        
        "id": 2,
        
        "severity": "Critical"
    
    },
    
    {
    
        "engine": "Operational",
        
        "handled": false,
        
        "title": "Suspicion of Unresponsive MODBUS Device",
        
        "additionalInformation": null,
        
        "sourceDevice": 194,
        
        "zoneId": 1,
        
        "siteId": 1,
        
        "time": 1594808285000,
        
        "sensorId": 1,
        
        "message": "Outstation device 10.13.10.1 (Protocol Address 53) seems to be unresponsive to MODBUS requests.",
        
        "destinationDevice": 0,
        
        "id": 3,
        
        "severity": "Minor"
    
    }
  
]
```

#### <a name="curl-command"></a>Команда cURL

> [!div class="mx-tdBreakAll"]
> | Тип | Программные интерфейсы | Пример |
> |--|--|--|
> | GET | Парный-k-H "Авторизация: <AUTH_TOKEN>" "https://<>IP_ADDRESS>/External/v1/Alerts? State =&КодЗоны =&Фромтиме =&Тотиме =&идентификатор сайта =&датчика =" | изогнутый-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" "https:/ <span> /127.0.0.1/External/v1/Alerts? State = необработанные&КодЗоны = 1&фромтиме = 0&тотиме = 1594551777000&ИД сайта = 1&датчика = 1" |

### <a name="qradar-alerts"></a>QRadar оповещения

Интеграция QRadar с защитником для Интернета вещей помогает находить предупреждения, созданные защитником для Интернета вещей, и выполнять действия с этими предупреждениями. QRadar получает данные от защитника для Интернета вещей, а затем обращается к компоненту локальной консоли управления API.

Чтобы отправить данные, обнаруженные защитником IoT, в QRadar, определите правило пересылки в защитнике для системы IoT и выберите параметр **обработка оповещений удаленной поддержки** .

:::image type="content" source="media/references-work-with-defender-for-iot-apis/edit-forwarding-rules.png" alt-text="Измените правила пересылки в соответствии со своими потребностями.":::

При выборе этого параметра в процессе настройки правил переадресации в QRadar отображаются следующие дополнительные поля:

- **UUID**: уникальный идентификатор оповещения, например 1-1555245116250.

- **Сайт**— сайт, на котором было обнаружено предупреждение.

- **Zone**— зона, в которой было обнаружено предупреждение.

Пример полезных данных, отправляемых в QRadar:

```
<9>May 5 12:29:23 sensor_Agent LEEF:1.0|CyberX|CyberX platform|2.5.0|CyberX platform Alert|devTime=May 05 2019 15:28:54 devTimeFormat=MMM dd yyyy HH:mm:ss sev=2 cat=XSense Alerts title=Device is Suspected to be Disconnected (Unresponsive) score=81 reporter=192.168.219.50 rta=0 alertId=6 engine=Operational senderName=sensor Agent UUID=5-1557059334000 site=Site zone=Zone actions=handle dst=192.168.2.2 dstName=192.168.2.2 msg=Device 192.168.2.2 is suspected to be disconnected (unresponsive).
```

#### <a name="externalv1alertsltuuidgt"></a>&lt;UUID/External/v1/Alerts/&gt;

#### <a name="method"></a>Метод

**PUT**

#### <a name="request-type"></a>Тип запроса

**JSON**

#### <a name="request-content"></a>Содержимое запроса

Объект JSON, представляющий действие, выполняемое с предупреждением, содержащим UUID.

#### <a name="action-fields"></a>Поля действий

| Имя | Тип | Допускает значения NULL | Список значений |
|--|--|--|--|
| **action** | Строка | Нет | Handle или Хандлеандлеарн |

#### <a name="request-example"></a>Пример запроса

```rest
{
    "action": "handle"
}

```

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, представляющих устройства.

#### <a name="response-fields"></a>Поля ответа


| Имя | Тип | Допускает значения NULL | Описание |
|--|--|--|--|
| **содержимое/ошибка** | Строка | Нет | Если запрос выполнен успешно, отображается свойство Content. В противном случае отображается свойство Error. |

#### <a name="possible-content-values"></a>Возможные значения содержимого

| Код состояния | Значение содержимого | Описание |
|--|--|--|
| 200 | Запрос на обновление предупреждения успешно завершен. | Запрос на обновление успешно завершен. Нет комментариев. |
| 200 | Предупреждение уже обработано (**Handle**). | Предупреждение уже было обработано при получении запроса на обработку предупреждения.<br />Предупреждение остается **обработанным**. |
| 200 | Предупреждение уже обработано и было получено (**хандлеандлеарн**). | Предупреждение уже обработано и было получено при получении запроса на **хандлеандлеарн** .<br />Оповещение остается в состоянии **хандледандлеарн** . |
| 200 | Предупреждение уже обработано (**обработано**).<br />В предупреждении был выполнен обработчик и обучение (**хандлеандлеарн**). | Предупреждение уже было обработано при получении запроса на **хандлеандлеарн** .<br />Оповещение превращается в **хандлеандлеарн**. |
| 200 | Предупреждение уже обработано и было получено (**хандлеандлеарн**). Проигнорирован запрос на обработку. | Предупреждение уже **хандлеандлеарно** при получении запроса на обработку предупреждения. Предупреждение остается **хандлеандлеарн**. |
| 500 | Недопустимое действие. | Отправленное действие не является допустимым действием для выполнения в предупреждении. |
| 500 | Произошла непредвиденная ошибка. | Произошла непредвиденная ошибка. Чтобы устранить эту проблему, обратитесь в службу технической поддержки. |
| 500 | Не удалось выполнить запрос, так как для этого UUID не найдено предупреждение. | Указанный идентификатор UUID предупреждения не найден в системе. |

#### <a name="response-example"></a>Пример ответа

**Успешно**

```rest
{
    "content": "Alert update request finished successfully"
}
```

**Неудачно**

```rest
{
    "error": "Invalid action"
}
```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| PUT | фигурная-k-X размещение-d "{" Action ":" <ACTION> "} '-H" Авторизация: <AUTH_TOKEN> "https://<IP_ADDRESS>/External/v1/Alerts/<UUID> | фигурная-k-X размещение-d "{" Action ":" Handle "} '-H" Авторизация: 1234b734a9244d54ab8d40aedddcabcd "https:/ <span> /127.0.0.1/External/v1/Alerts/1-1594550943000 |

### <a name="alert-exclusions-maintenance-window---externalv1maintenancewindow"></a>Исключения оповещений (период обслуживания) —/external/v1/maintenanceWindow

Определите условия, при которых оповещения не будут отправляться. Например, определите и обновите время окончания и начала работы, устройства или подсети, которые должны быть исключены при активации оповещений, или защитник для механизмов Интернета вещей, которые следует исключить. Например, в течение периода обслуживания может потребоваться отключить доставку предупреждений для всех предупреждений, за исключением оповещений о вредоносных программах на критических устройствах.

Интерфейсы API, которые вы определяете здесь, отображаются в окне **исключения предупреждений** локальной консоли управления как правило исключения только для чтения.

:::image type="content" source="media/references-work-with-defender-for-iot-apis/alert-exclusion-window.png" alt-text="В окне исключения предупреждений отображается список всех правил исключения. ":::

#### <a name="method---post"></a>Метод — POST

#### <a name="query-parameters"></a>Параметры запроса

- **тиккетид**: определяет идентификатор билета обслуживания в системах пользователя.

- **TTL**: определяет срок жизни (TTL), который является длительностью периода обслуживания в минутах. По истечении периода времени, определяемого этим параметром, система автоматически начинает отправлять предупреждения.

- **обработчики**: определяет, из какого модуля безопасности следует подавлять предупреждения во время процесса обслуживания:

   - АНОМАЛИЙ

   - АНАЛИЗИРУЮТ

   - РАБОЧЕГО

   - POLICY_VIOLATION

   - PROTOCOL_VIOLATION

- **сенсоридс**: определяет, из какого защитника для датчика Интернета вещей подавляет оповещения во время процесса обслуживания. Это тот же идентификатор, который извлекается из/API/V1/Appliances (GET).

- **подсети**: определяет, из какой подсети следует подавлять оповещения во время процесса обслуживания. Подсеть отправляется в следующем формате: 192.168.0.0/16.

#### <a name="error-codes"></a>Коды ошибок

- **201 (создано)**: действие успешно завершено.

- **400 (недопустимый запрос)**: отображается в следующих случаях:

   - Параметр **TTL** не является числовым или положительным.

   - Параметр **подсетей** был определен в неправильном формате.

   - Отсутствует параметр **тиккетид** .

   - Параметр **Engine** не соответствует существующим ядрам безопасности.

- **404 (не найдено)**: один из датчиков не существует.

- **409 (конфликт)**: идентификатор билета связан с другим открытым окном обслуживания.

- **500 (внутренняя ошибка сервера)**: любая другая непредвиденная ошибка.

> [!NOTE]
> Убедитесь, что идентификатор билета не связан с существующим открытым окном. Создается следующее правило исключения: обслуживание-{имя токена}-{идентификатор билета}.

#### <a name="method---put"></a>Метод — размещение

Позволяет обновлять длительность периода обслуживания после запуска процесса обслуживания, изменяя параметр **TTL** . Новое определение длительности переопределяет предыдущее.

Этот метод полезен, если требуется задать более длительную длительность, чем текущая настроенная длительность.

#### <a name="query-parameters"></a>Параметры запроса

- **тиккетид**: определяет идентификатор билета обслуживания в системах пользователя.

- **TTL**: определяет длительность периода в минутах.

#### <a name="error-code"></a>Код ошибки

- **200 (ОК)**: действие успешно завершено.

- **400 (недопустимый запрос)**: отображается в следующих случаях:

   - Параметр **TTL** не является числовым или положительным.

   - Отсутствует параметр **тиккетид** .

   - Отсутствует параметр **TTL** .

- **404 (не найдено)**: идентификатор билета не связан с открытым окном обслуживания.

- **500 (внутренняя ошибка сервера)**: любая другая непредвиденная ошибка.

> [!NOTE]
> Убедитесь, что идентификатор билета связан с существующим открытым окном.

#### <a name="method---delete"></a>Метод — DELETE

Закрытие существующего периода обслуживания.

#### <a name="query-parameters"></a>Параметры запроса

- **тиккетид**: записывает в журнал идентификатор билета обслуживания в системах пользователя.

#### <a name="error-code"></a>Код ошибки

- **200 (ОК)**: действие успешно завершено.

- **400 (недопустимый запрос)**: отсутствует параметр **тиккетид** .

- **404 (не найдено)**: идентификатор билета не связан с открытым окном обслуживания.

- **500 (внутренняя ошибка сервера)**: любая другая непредвиденная ошибка.

> [!NOTE]
> Убедитесь, что идентификатор билета связан с существующим открытым окном.

#### <a name="method---get"></a>Метод — GET

Получение журнала всех действий по открытию, закрытию и обновлению, которые были выполнены в системе во время обслуживания. Журнал можно получить только для периодов обслуживания, которые были активны в прошлом и были закрыты.

#### <a name="query-parameters"></a>Параметры запроса

- **fromDate**: фильтрует журналы из предопределенной даты и более поздних версий. Формат 2019-12-30.

- **toDate**: фильтрует журналы до предопределенной даты. Формат 2019-12-30.

- **тиккетид**: фильтрует журналы, связанные с конкретным идентификатором билета.

- **tokenName**: фильтрует журналы, связанные с указанным именем токена.

#### <a name="error-code"></a>Код ошибки

- **200 (ОК)**: действие успешно завершено.

- **400 (недопустимый запрос)**: неправильный формат даты.

- **204 (нет содержимого)**: нет данных для отображения.

- **500 (внутренняя ошибка сервера)**: любая другая непредвиденная ошибка.

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Массив объектов JSON, которые представляют операции периода обслуживания.

#### <a name="response-structure"></a>Структура ответа

| Имя | Тип | Комментарий | Допускает значения NULL |
|--|--|--|--|
| **dateTime** | Строка | Пример: "2012-04-23T18:25:43.511 Z" | Нет |
| **тиккетид** | Строка | Пример: "9a5fe99c-D914-4bda-9332-307384fe40bf" | Нет |
| **tokenName** | Строка | - | Нет |
| **созданных** | Массив строк | - | да |
| **сенсоридс** | Массив строк | - | да |
| **подсети** | Массив строк | - | да |
| **величин** | Числовой | - | да |
| **operationType** | Строка | Значения: "OPEN", "UPDATE" и "CLOSE" | Нет |

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| POST | фигурная-k-X после-d "{" Тиккетид ":" <TICKET_ID> ", TTL": <TIME_TO_LIVE>, "Engines": [<ENGINE1, ENGINE2... Енгинен>], "Сенсоридс": [<SENSOR_ID1, SENSOR_ID2... SENSOR_IDn>], "подсети": [<SUBNET1, подсети 2... Субнетн>]} '-H ' авторизация: <AUTH_TOKEN> "https:/ <span> /127.0.0.1/External/v1/maintenanceWindow | фигурная-k-X после-d "{" Тиккетид ":" a5fe99c-D914-4bda-9332-307384fe40bf "," TTL ":" 20 "," Engines ": [" АНОМАЛИя "]," Сенсоридс ": [" 5 "," 3 "]," подсети ": [" 10.0.0.3 "]}"-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/External/v1/maintenanceWindow |
| PUT | фигурная-k-X размещение-d "{" Тиккетид ":" <TICKET_ID> ", TTL": "<TIME_TO_LIVE>"} "-H" Authorization: <AUTH_TOKEN> "https:/ <span> /127.0.0.1/External/v1/maintenanceWindow | -k-X размещение-d "{" Тиккетид ":" a5fe99c-D914-4bda-9332-307384fe40bf "," TTL ":" 20 "}"-H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/ <span> /127.0.0.1/External/v1/maintenanceWindow |
| DELETE | прилистывание-k-X удаление-d "{" Тиккетид ":" <TICKET_ID> "}"-H "Authorization: <AUTH_TOKEN>" https:/ <span> /127.0.0.1/External/v1/maintenanceWindow | фигурный-k-X удаление-d "{" Тиккетид ":" a5fe99c-D914-4bda-9332-307384fe40bf "} '-H" Authorization: 1234b734a9244d54ab8d40aedddcabcd "https:/ <span> /127.0.0.1/External/v1/maintenanceWindow |
| GET | фигурная-k-H "Авторизация: <AUTH_TOKEN>" "https://<IP_ADDRESS>/external/v1/maintenanceWindow? fromDate =&toDate =&Тиккетид =&tokenName =" | фигурная-k-H "Авторизация: 1234b734a9244d54ab8d40aedddcabcd" "https:/ <span> /127.0.0.1/External/v1/maintenanceWindow? fromDate = 2020-01-01&toDate = 2020-07-14&тиккетид = a5fe99c-D914-4bda-9332-307384fe40bf&tokenName = a" |

### <a name="authenticate-user-credentials---externalauthenticationvalidation"></a>Проверка подлинности учетных данных пользователя —/екстернал/аусентикатион/валидатион

Используйте этот API для проверки учетных данных пользователя. С API можно работать все роли пользователей в защитнике IoT. Для использования этого API нет необходимости в защитнике для маркера доступа IoT.

#### <a name="method"></a>Метод

**POST**

#### <a name="request-type"></a>Тип запроса

**JSON**

#### <a name="request-example"></a>Пример запроса

```rest
request:

{

    "username": "test",

    "password": "Test12345\!"

}
```

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Строка сообщения с подробными сведениями о состоянии операции:

- **Успешно — сообщение**: проверка подлинности выполнена успешно

- **Сбой — ошибка**: проверка учетных данных не пройдена

#### <a name="device-fields"></a>Поля устройства

| **имя**; | **Тип** | **Допускает значения NULL** |
|--|--|--|
| **username** | Строка | Нет |
| **password** | Строка | Нет |

#### <a name="response-example"></a>Пример ответа

```rest
response:

{

    "msg": "Authentication succeeded."

}
```

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| POST | фигурная-k-d "{" имя_пользователя ":" <USER_NAME> "," Password ":" пароль "}" ' https://<IP_ADDRESS>/екстернал/аусентикатион/валидатион " | фигурная-k-d "{" имя_пользователя ":" myUser "," Password ":" 1234@abcd "}" "https:/ <span> /127.0.0.1/External/Authentication/Validation" |

### <a name="change-password---externalauthenticationset_password"></a>Изменение пароля —/екстернал/аусентикатион/set_password

Используйте этот API, чтобы позволить пользователям изменять свои пароли. С API можно работать все роли пользователей в защитнике IoT. Для использования этого API нет необходимости в защитнике для маркера доступа IoT.

#### <a name="method"></a>Метод

**POST**

#### <a name="request-type"></a>Тип запроса

**JSON**

#### <a name="request-example"></a>Пример запроса

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!",
    
    "new_password": "Test54321\!"

}

```

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Строка сообщения с подробными сведениями о состоянии операции:

- **Успешно — сообщение**: пароль был заменен.

- **Сбой — ошибка**. Ошибка проверки подлинности пользователя

- **Сбой — ошибка**: пароль не соответствует политике безопасности.

#### <a name="response-example"></a>Пример ответа

```rest
response:

{

    "error": {
    
        "userDisplayErrorMessage": "User authentication failure"
    
    }

}

```

#### <a name="device-fields"></a>Поля устройства

| **имя**; | **Тип** | **Допускает значения NULL** |
|--|--|--|
| **username** | Строка | Нет |
| **password** | Строка | Нет |
| **new_password** | Строка | Нет |

#### <a name="curl-command"></a>Команда cURL

| Тип | Программные интерфейсы | Пример |
|--|--|--|
| POST | фигурная-k-d "{" имя_пользователя ":" <USER_NAME> "," пароль ":" <CURRENT_PASSWORD> "," new_password ":" <NEW_PASSWORD> "}"-H "Content-Type: Application/JSON" https://<IP_ADDRESS>/екстернал/аусентикатион/set_password | фигурная-k-d "{" имя_пользователя ":" myUser "," Password ":" 1234@abcd "," new_password ":" abcd@1234 "}"-H "тип содержимого: Application/JSON" https:/ <span> /127.0.0.1/External/Authentication/set_password |

### <a name="user-password-update-by-system-admin---externalauthenticationset_password_by_admin"></a>Обновление пароля пользователя по системному администратору —/екстернал/аусентикатион/set_password_by_admin

Используйте этот API, чтобы позволить системным администраторам изменять пароли для указанных пользователей. С API может работать защитник для ролей пользователей с правами администратора IoT. Для использования этого API нет необходимости в защитнике для маркера доступа IoT.

#### <a name="method"></a>Метод

**POST**

#### <a name="request-type"></a>Тип запроса

**JSON**

#### <a name="request-example"></a>Пример запроса

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!",
    
    "new_password": "Test54321\!"

}
```

#### <a name="response-type"></a>Тип ответа

**JSON**

#### <a name="response-content"></a>Содержимое ответа

Строка сообщения с подробными сведениями о состоянии операции:

- **Успешно — сообщение**: пароль был заменен.

- **Сбой — ошибка**. Ошибка проверки подлинности пользователя

- **Сбой — ошибка**: пользователь не существует

- **Сбой — ошибка**. пароль не соответствует политике безопасности

- **Сбой — ошибка**: у пользователя нет разрешений на изменение пароля

#### <a name="response-example"></a>Пример ответа

```rest
response:

{

    "error": {
    
        "userDisplayErrorMessage": "The user 'test_user' doesn't exist",
        
        "internalSystemErrorMessage": "The user 'yoavfe' doesn't exist"
    
    }

}

```

#### <a name="device-fields"></a>Поля устройства

| **имя**; | **Тип** | **Допускает значения NULL** |
|--|--|--|
| **admin_username** | Строка | Нет |
| **admin_password** | Строка | Нет |
| **username** | Строка | Нет |
| **new_password** | Строка | Нет |

#### <a name="curl-command"></a>Команда cURL

> [!div class="mx-tdBreakAll"]
> | Тип | Программные интерфейсы | Пример |
> |--|--|--|
> | POST | фигурная скобка-d "{" admin_username ":" <ADMIN_USERNAME> "," admin_password ":" <ADMIN_PASSWORD> "," Username ":" <user_name> "," new_password ":" <NEW_PASSWORD> "}"-H "тип содержимого: Application/JSON" https://<IP_ADDRESS>/екстернал/аусентикатион/set_password_by_admin | фигурная-k-d "{" admin_user ":" adminUser "," admin_password ":" 1234@abcd "," Username ":" myuser "," new_password ":" abcd@1234 "}"-H "тип содержимого: Application/JSON" https:/ <span> /127.0.0.1/External/Authentication/set_password_by_admin |

## <a name="next-steps"></a>Следующие шаги

- [Изучение обнаруженных датчиков в данных инвентаризации устройств](how-to-investigate-sensor-detections-in-a-device-inventory.md)

- [Изучение всех обнаруженных датчиков предприятия в данных инвентаризации устройств](how-to-investigate-all-enterprise-sensor-detections-in-a-device-inventory.md)
