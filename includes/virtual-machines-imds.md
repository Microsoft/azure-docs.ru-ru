---
author: LittleBoxOfSunshine
manager: KumariSupriya
ms.service: virtual-machines
ms.subservice: monitoring
ms.topic: include
ms.date: 01/04/2021
ms.author: chhenk
ms.reviewer: azmetadatadev
ms.custom: references_regions
ms.openlocfilehash: b720e98fc83fd12744c289cb99814748b469b15d
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102123689"
---
# <a name="azure-instance-metadata-service"></a>Служба метаданных экземпляров Azure

Служба метаданных экземпляров Azure (IMDS) предоставляет сведения о текущих экземплярах виртуальных машин. Его можно использовать для управления виртуальными машинами и их настройки.
Эти сведения включают SKU, хранилище, конфигурации сети и будущие события обслуживания. Полный список доступных данных см. в разделе [Сводка по категориям конечных точек](#endpoint-categories).

IMDS доступен для запуска экземпляров виртуальных машин и экземпляров масштабируемых наборов виртуальных машин. Все конечные точки поддерживают виртуальные машины, созданные и управляемые с помощью [Azure Resource Manager](/rest/api/resources/). Только подтвержденная Категория и сетевая часть категории экземпляров поддерживают виртуальные машины, созданные с помощью классической модели развертывания. Аттестация конечной точки выполняется только в ограниченном экстенте.

IMDS — это REST API, который доступен на хорошо известном IP-адресе, не поддерживающем маршрутизацию ( `169.254.169.254` ). Доступ к нему можно получить только с виртуальной машины. Обмен данными между виртуальной машиной и IMDS никогда не покидает узел.
Попросите клиентов HTTP пропускать веб-прокси в виртуальной машине при запросе IMDS и обрабатывать `169.254.169.254` их так же, как [`168.63.129.16`](../articles/virtual-network/what-is-ip-address-168-63-129-16.md) .

## <a name="usage"></a>Использование

### <a name="access-azure-instance-metadata-service"></a>Доступ к службе метаданных экземпляров Azure

Чтобы получить доступ к IMDS, создайте виртуальную машину на основе [Azure Resource Manager](/rest/api/resources/) или [портал Azure](https://portal.azure.com)и используйте следующие примеры.
Дополнительные примеры см. в статье [примеры метаданных экземпляров Azure](https://github.com/microsoft/azureimds).

Ниже приведен пример кода для получения всех метаданных для экземпляра. Сведения о доступе к определенному источнику данных см. в разделе [категории конечных точек](#endpoint-categories) обзор всех доступных функций.

**Запрос**

> [!IMPORTANT]
> Этот пример обходит учетные записи-посредники. При запросе IMDS **необходимо** обходить прокси-серверы. Дополнительные сведения см. в разделе [учетные записи-посредники](#proxies) .

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance?api-version=2020-09-01" | ConvertTo-Json -Depth 64
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance?api-version=2020-09-01" | jq
```

---

**Ответ**

> [!NOTE]
> Ответ представляет собой строку JSON. Следующие примеры ответов представлены в удобном для чтения формате.

[!INCLUDE [virtual-machines-imds-full-instance-response](./virtual-machines-imds-full-instance-response.md)]

## <a name="security-and-authentication"></a>Безопасность и проверка подлинности

Служба метаданных экземпляра доступна только в работающем экземпляре виртуальной машины на IP-адресе, не поддерживающем маршрутизацию. Виртуальные машины ограничены взаимодействием с метаданными и функциями, которые относятся к самим. API является только HTTP и никогда не оставляет узел.

Чтобы обеспечить непосредственное назначение запросов для IMDS и предотвратить непреднамеренное или нежелательное перенаправление запросов, запросы:
- **Должен** содержать заголовок `Metadata: true`
- **Не** должен содержать `X-Forwarded-For` заголовок

Любой запрос, который не соответствует **обоим** этим требованиям, будет отклонен службой.

> [!IMPORTANT]
> IMDS **не** является каналом для конфиденциальных данных. Этот API не прошел проверку подлинности и открывается для всех процессов на виртуальной машине. Сведения, предоставляемые посредством этой службы, следует рассматривать как общие сведения для всех приложений, запущенных на виртуальной машине.

## <a name="proxies"></a>прокси-серверы;

IMDS **не** предназначен для использования за прокси-сервером, и это не поддерживается. Большинство клиентов HTTP предоставляют возможность отключить прокси-серверы в запросах, и эту функцию следует использовать при взаимодействии с IMDS. Дополнительные сведения см. в документации клиента.

> [!IMPORTANT]
> Даже если вы не узнаете о конфигурации прокси-сервера в вашей среде, **все равно необходимо переопределить параметры прокси-сервера клиента по умолчанию**. Конфигурации прокси-сервера могут быть автоматически обнаружены, и при этом не удается обойти такие конфигурации, что позволяет избежать риска, если конфигурация компьютера будет изменена в будущем.

## <a name="rate-limiting"></a>Ограничение частоты

Как правило, запросы к IMDS ограничены 5 запросами в секунду. Запросы, превышающие это пороговое значение, будут отклонены с ответами 429. Запросы к [управляемой категории идентификации](#managed-identity) ограничены 20 запросами в секунду и 5 одновременными запросами.

## <a name="http-verbs"></a>Команды HTTP

В настоящее время поддерживаются следующие глаголы HTTP:

| Команда | Описание |
|------|-------------|
| `GET` | Получение запрошенного ресурса

## <a name="parameters"></a>Параметры

Конечные точки могут поддерживать обязательные и (или) необязательные параметры. Дополнительные сведения см. в разделе [схема](#schema) и документация по конкретной конечной точке.

### <a name="query-parameters"></a>Параметры запроса

Конечные точки IMDS поддерживают параметры строки HTTP-запроса. Пример: 

```
http://169.254.169.254/metadata/instance/compute?api-version=2019-06-04&format=json
```

Задает параметры:

| Имя | Значение |
|------|-------|
| `api-version` | `2019-06-04`
| `format` | `json`

Запросы с повторяющимися именами параметров запроса будут отклонены.

### <a name="route-parameters"></a>Параметры маршрута

Для некоторых конечных точек, возвращающих большие двоичные объекты JSON, мы поддерживаем Добавление параметров маршрута к конечной точке запроса для фильтрации по подмножеству ответа: 

```
http://169.254.169.254/metadata/<endpoint>/[<filter parameter>/...]?<query parameters>
```
Параметры соответствуют индексам или ключам, которые будут использоваться для прохода по объекту JSON при взаимодействии с проанализированным представлением.

Например, `/metatadata/instance` возвращает объект JSON:
```json
{
    "compute": { ... },
    "network": {
        "interface": [
            {
                "ipv4": {
                   "ipAddress": [{
                        "privateIpAddress": "10.144.133.132",
                        "publicIpAddress": ""
                    }],
                    "subnet": [{
                        "address": "10.144.133.128",
                        "prefix": "26"
                    }]
                },
                "ipv6": {
                    "ipAddress": [
                     ]
                },
                "macAddress": "0011AAFFBB22"
            },
            ...
        ]
    }
}
```

Если бы нам нужно отфильтровать ответ только по свойству COMPUTE, мы отправили запрос: 
```
http://169.254.169.254/metadata/instance/compute?api-version=<version>
```

Аналогично, если нужно отфильтровать вложенное свойство или конкретный элемент массива, мы будем добавлять ключи: 
```
http://169.254.169.254/metadata/instance/network/interface/0?api-version=<version>
```
будет выполнять фильтрацию по первому элементу из `Network.interface` Свойства и возвращать:

```json
{
    "ipv4": {
       "ipAddress": [{
            "privateIpAddress": "10.144.133.132",
            "publicIpAddress": ""
        }],
        "subnet": [{
            "address": "10.144.133.128",
            "prefix": "26"
        }]
    },
    "ipv6": {
        "ipAddress": [
         ]
    },
    "macAddress": "0011AAFFBB22"
}
```

> [!NOTE]
> При фильтрации на конечный узел `format=json` не работает. Для этих запросов необходимо `format=text` явно указать, так как форматом по умолчанию является JSON.

## <a name="schema"></a>схема

### <a name="data-format"></a>Формат данных

По умолчанию IMDS возвращает данные в формате JSON ( `Content-Type: application/json` ). Однако конечные точки, поддерживающие фильтрацию ответов (см. раздел [Параметры маршрута](#route-parameters)), также поддерживают формат `text` .

Для доступа к нестандартному формату ответа укажите запрошенный формат в качестве параметра строки запроса. Пример:

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance?api-version=2017-08-01&format=text"
```

---

В ответах JSON все примитивы будут иметь тип `string` , а отсутствующие или неприменимые значения всегда включаются, но для них будет задана пустая строка.

### <a name="versioning"></a>Управление версиями

IMDS имеет версию, и указание версии API в HTTP-запросе является обязательным. Единственным исключением из этого требования является конечная точка [версий](#versions) , которую можно использовать для динамического извлечения доступных версий API.

Так как мы добавляем новые версии, вы все еще можете получить доступ к предыдущим версиям для обеспечения совместимости, если используемый скрипт зависит от конкретных форматов данных.

Если вы не укажете версию, появится сообщение об ошибке со списком последних поддерживаемых версий:
```json
{
    "error": "Bad request. api-version was not specified in the request. For more information refer to aka.ms/azureimds",
    "newest-versions": [
        "2020-10-01",
        "2020-09-01",
        "2020-07-15"
    ]
}
```

#### <a name="supported-api-versions"></a>Поддерживаемые версии API
- 2017-03-01
- 2017-04-02
- 2017-08-01 
- 2017-10-01
- 2017-12-01 
- 2018-02-01
- 2018-04-02
- 2018-10-01
- 2019-02-01
- 2019-03-11
- 2019-04-30
- 2019-06-01
- 2019-06-04
- 01.08.2019
- 2019-08-15
- 2019-11-01
- 2020-06-01
- 2020-07-15
- 2020-09-01
- 2020-10-01
- 2020-12-01

### <a name="swagger"></a>Swagger

Полное определение Swagger для IMDS можно найти по адресу: https://github.com/Azure/azure-rest-api-specs/blob/master/specification/imds/data-plane/readme.md

## <a name="regional-availability"></a>Доступность по регионам

Служба **общедоступна** во всех облаках Azure.

## <a name="root-endpoint"></a>Корневая конечная точка

Корневая конечная точка — `http://169.254.169.254/metadata` .

## <a name="endpoint-categories"></a>Категории конечных точек

API IMDS содержит несколько категорий конечных точек, представляющих различные источники данных, каждый из которых содержит одну или несколько конечных точек. Дополнительные сведения см. в каждой категории.

| Корень категории | Описание | Представленная версия |
|---------------|-------------|--------------------|
| `/metadata/attested` | См. раздел [Аттестованные данные](#attested-data). | 2018-10-01
| `/metadata/identity` | См. раздел [управляемое удостоверение через IMDS](#managed-identity) . | 2018-02-01
| `/metadata/instance` | См. [метаданные экземпляра](#instance-metadata) | 2017-04-02
| `/metadata/loadbalancer` | См. статью [Получение метаданных Load Balancer через IMDS](#load-balancer-metadata) . | 2020-10-01
| `/metadata/scheduledevents` | См. [запланированные события через IMDS](#scheduled-events) | 2017-08-01
| `/metadata/versions` | См. [версии](#versions) | Н/Д

## <a name="versions"></a>Версии

> [!NOTE]
> Эта функция была выпущена вместе с версией 2020-10-01, которая в настоящее время находится в процессе развертывания и может быть еще не доступна в каждом регионе.

### <a name="list-api-versions"></a>Вывод списка версий API

Возвращает набор поддерживаемых версий API.

```
GET /metadata/versions
```

#### <a name="parameters"></a>Параметры

Нет (Эта конечная точка имеет неустановленную версию).

#### <a name="response"></a>Ответ

```json
{
  "apiVersions": [
    "2017-03-01",
    "2017-04-02",
    ...
  ]
}
```

## <a name="instance-metadata"></a>Метаданные экземпляра

### <a name="get-vm-metadata"></a>Получение метаданных виртуальной машины

Предоставляет важные метаданные для экземпляра виртуальной машины, включая вычислительные ресурсы, сети и хранилище. 

```
GET /metadata/instance
```

#### <a name="parameters"></a>Параметры

| Имя | Обязательный/необязательный | Описание |
|------|-------------------|-------------|
| `api-version` | Обязательно | Версия, используемая для обслуживания запроса.
| `format` | Используемых | Формат ответа ( `json` или `text` ). * Примечание. может потребоваться при использовании параметров запроса

Эта конечная точка поддерживает фильтрацию ответов с помощью [параметров маршрута](#route-parameters).

#### <a name="response"></a>Ответ

[!INCLUDE [virtual-machines-imds-full-instance-response](./virtual-machines-imds-full-instance-response.md)]

Разбиение схемы:

**Среда выполнения приложений**

| Данные | Описание | Представленная версия |
|------|-------------|--------------------|
| `azEnvironment` | Среда Azure, где запущена виртуальная машина | 2018-10-01
| `customData` | Эта функция сейчас отключена. Мы будем обновлять эту документацию, когда она станет доступной | 2019-02-01
| `evictionPolicy` | Указывает, как будет удалена [плашечная виртуальная машина](../articles/virtual-machines/spot-vms.md) . | 2020-12-01
| `isHostCompatibilityLayerVm` | Определяет, выполняется ли виртуальная машина на уровне совместимости узла | 2020-06-01
| `licenseType` | Тип лицензии для [преимущество гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-benefit). Эта возможность имеется только для виртуальных машин с поддержкой АХБ | 2020-09-01
| `location` | Регион Azure, в котором запущена виртуальная машина | 2017-04-02
| `name` | Имя виртуальной машины. | 2017-04-02
| `offer` | Предоставление сведений для образа виртуальной машины (эти значения доступны только для образов, развернутых из коллекции образов Azure) | 2017-04-02
| `osProfile.adminUsername` | Указывает имя учетной записи администратора. | 2020-07-15
| `osProfile.computerName` | Указывает имя компьютера | 2020-07-15
| `osProfile.disablePasswordAuthentication` | Указывает, отключена ли проверка пароля. Эта проблема имеется только для виртуальных машин Linux | 2020-10-01
| `osType` | Windows или Linux | 2017-04-02
| `placementGroupId` | [Группа размещения](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md) масштабируемого набора виртуальных машин | 2017-08-01
| `plan` | [План](/rest/api/compute/virtualmachines/createorupdate#plan), указывающий наименование, продукт и издателя виртуальной машины (если используется образ Azure Marketplace) | 2018-04-02
| `platformUpdateDomain` |  [Домен обновления](../articles/virtual-machines/manage-availability.md), на котором запущена виртуальная машина | 2017-04-02
| `platformFaultDomain` | [Домен сбоя](../articles/virtual-machines/manage-availability.md), на котором запущена виртуальная машина | 2017-04-02
| `priority` | Приоритет виртуальной машины. Дополнительные сведения см. в разделе [точки виртуальных машин](../articles/virtual-machines/spot-vms.md) . | 2020-12-01
| `provider` | Поставщик виртуальной машины | 2018-10-01
| `publicKeys` | [Коллекция открытых ключей](/rest/api/compute/virtualmachines/createorupdate#sshpublickey), назначенная виртуальной машине и путям | 2018-04-02
| `publisher` | Издатель образа виртуальной машины | 2017-04-02
| `resourceGroupName` | [Группа ресурсов](../articles/azure-resource-manager/management/overview.md) для виртуальной машины | 2017-08-01
| `resourceId` | [Полный](/rest/api/resources/resources/getbyid) идентификатор ресурса | 2019-03-11
| `sku` | Определенный номер SKU для образа виртуальной машины | 2017-04-02
| `securityProfile.secureBootEnabled` | Определяет, включена ли безопасная Загрузка UEFI на виртуальной машине | 2020-06-01
| `securityProfile.virtualTpmEnabled` | Определяет, включен ли на виртуальной машине виртуальный доверенный платформенный модуль (TPM) (TPM) | 2020-06-01
| `storageProfile` | См. профиль хранилища ниже | 2019-06-01
| `subscriptionId` | Подписка Azure для виртуальной машины | 2017-08-01
| `tags` | [Теги](../articles/azure-resource-manager/management/tag-resources.md) для виртуальной машины  | 2017-08-01
| `tagsList` | Теги форматируются в виде массива JSON для упрощения программного анализа  | 2019-06-04
| `version` | Версия образа виртуальной машины | 2017-04-02
| `vmId` | [Уникальный идентификатор](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) для виртуальной машины | 2017-04-02
| `vmScaleSetName` | [Имя масштабируемого набора виртуальных машин](../articles/virtual-machine-scale-sets/overview.md) для масштабируемого набора виртуальных машин | 2017-12-01
| `vmSize` | [Размер виртуальной машины](../articles/virtual-machines/sizes.md) | 2017-04-02
| `zone` | [Зона доступности](../articles/availability-zones/az-overview.md) виртуальной машины | 2017-12-01

**Профиль хранилища**

Профиль хранилища виртуальной машины делится на три категории: ссылка на образ, диск ОС и диски данных.

Объект ссылки на изображение содержит следующие сведения об образе ОС.

| Данные | Описание |
|------|-------------|
| `id` | Идентификатор ресурса
| `offer` | Предоставление образа платформы или Marketplace
| `publisher` | Издатель образа.
| `sku` | Номер SKU образа
| `version` | Версия образа платформы или Marketplace

Объект диска ОС содержит следующие сведения о диске ОС, который используется виртуальной машиной.

| Данные | Описание |
|------|-------------|
| `caching` | Требования к кэшированию
| `createOption` | Сведения о способе создания виртуальной машины
| `diffDiskSettings` | Параметры временного диска
| `diskSizeGB` | Размер диска (ГБ)
| `image`   | Виртуальный жесткий диск исходного образа пользователя
| `lun`     | Номер логического устройства диска (LUN)
| `managedDisk` | Параметры управляемого диска
| `name`    | Имя диска
| `vhd`     | Виртуальный жесткий диск
| `writeAcceleratorEnabled` | Включен ли параметр writeAccelerator на диске

Массив дисков данных содержит список дисков данных, подключенных к виртуальной машине. Каждый объект диска данных содержит следующие сведения.

Данные | Описание |
-----|-------------|
| `caching` | Требования к кэшированию
| `createOption` | Сведения о способе создания виртуальной машины
| `diffDiskSettings` | Параметры временного диска
| `diskSizeGB` | Размер диска (ГБ)
| `encryptionSettings` | Параметры шифрования для диска
| `image` | Виртуальный жесткий диск исходного образа пользователя
| `managedDisk` | Параметры управляемого диска
| `name` | Имя диска
| `osType` | Тип ОС, добавленной на диск
| `vhd` | Виртуальный жесткий диск
| `writeAcceleratorEnabled` | Включен ли параметр writeAccelerator на диске

**Network**

| Данные | Описание | Представленная версия |
|------|-------------|--------------------|
| `ipv4.privateIpAddress` | Локальный IPv4-адрес виртуальной машины | 2017-04-02
| `ipv4.publicIpAddress` | Общедоступный IPv4-адрес виртуальной машины | 2017-04-02
| `subnet.address` | Адрес подсети виртуальной машины | 2017-04-02
| `subnet.prefix` | Префикс подсети, пример 24 | 2017-04-02
| `ipv6.ipAddress` | Локальный IPv6-адрес виртуальной машины | 2017-04-02
| `macAddress` | Mac-адрес виртуальной машины | 2017-04-02


#### <a name="sample-1-tracking-vm-running-on-azure"></a>Пример 1 Отслеживание виртуальной машины в Azure

Поставщику услуг может потребоваться отслеживать количество виртуальных машин, использующих ваше программное обеспечение, или установить агенты, отслеживающие уникальность виртуальной машины. Чтобы получить уникальный идентификатор для виртуальной машины, используйте поле `vmId` службы метаданных экземпляров.

**Запрос**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-08-01&format=text"
```

---

**Ответ**

```
5c08b38e-4d57-4c23-ac45-aca61037f084
```

#### <a name="sample-2-placement-of-different-data-replicas"></a>Пример 2. размещение различных реплик данных

В некоторых сценариях размещение разных реплик имеет первостепенное значение. Например, при [размещении реплики HDFS](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Replica_Placement:_The_First_Baby_Steps) или размещении контейнера с помощью [Orchestrator](https://kubernetes.io/docs/user-guide/node-selection/) может потребоваться информация о том, `platformFaultDomain` `platformUpdateDomain` на котором запущена виртуальная машина.
Кроме того, для принятия этих решений можно применить [Зоны доступности](../articles/availability-zones/az-overview.md) для экземпляров.
Эти данные можно запросить непосредственно через IMDS.

**Запрос**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

---

**Ответ**

```
0
```

#### <a name="sample-3-get-vm-tags"></a>Пример 3. получение тегов ВМ

Теги виртуальных машин включены в API экземпляра в конечной точке экземпляра, вычислений или тегов.
К виртуальной машине Azure можно применить теги, чтобы логически организовать их в таксономию. Теги, назначенные виртуальной машине, можно получить с помощью запроса ниже.

**Запрос**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/tags?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

---

**Ответ**

```
Department:IT;ReferenceNumber:123456;TestStatus:Pending
```

Поле `tags` представляет собой строку с тегами, разделенными точкой с запятой. Эти выходные данные могут быть проблемой, если в самих тегах используются точки с запятой. Если средство синтаксического анализа написано для программного извлечения тегов, следует полагаться на `tagsList` поле. `tagsList`Поле является массивом JSON без разделителей и, следовательно, проще в синтаксическом анализе. Тагслист, назначенный виртуальной машине, можно получить с помощью приведенного ниже запроса.

**Запрос**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/tagsList?api-version=2019-06-04" | ConvertTo-Json -Depth 64
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/tagsList?api-version=2019-06-04" | jq
```

---

**Ответ**

#### <a name="windows"></a>[Windows](#tab/windows/)

```json
{
    "value":  [
                  {
                      "name":  "Department",
                      "value":  "IT"
                  },
                  {
                      "name":  "ReferenceNumber",
                      "value":  "123456"
                  },
                  {
                      "name":  "TestStatus",
                      "value":  "Pending"
                  }
              ],
    "Count":  3
}
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```json
[
  {
    "name": "Department",
    "value": "IT"
  },
  {
    "name": "ReferenceNumber",
    "value": "123456"
  },
  {
    "name": "TestStatus",
    "value": "Pending"
  }
]
```

---


#### <a name="sample-4-get-more-information-about-the-vm-during-support-case"></a>Пример 4. получение дополнительных сведений о виртуальной машине во время обращения в службу поддержки

Чтобы получить дополнительные сведения о виртуальной машине, поставщик услуг может позвонить в службу поддержки. Если клиент предоставит сведения о метаданных вычислений, специалист службы поддержки получит основные сведения о виртуальной машине в Azure.

**Запрос**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute?api-version=2020-09-01" | ConvertTo-Json -Depth 64
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute?api-version=2020-09-01"
```

---

**Ответ**

> [!NOTE]
> Ответ представляет собой строку JSON. Следующие примеры ответов представлены в удобном для чтения формате.

#### <a name="windows"></a>[Windows](#tab/windows/)
```json
{
    "azEnvironment": "AZUREPUBLICCLOUD",
    "isHostCompatibilityLayerVm": "true",
    "licenseType":  "Windows_Client",
    "location": "westus",
    "name": "examplevmname",
    "offer": "WindowsServer",
    "osProfile": {
        "adminUsername": "admin",
        "computerName": "examplevmname",
        "disablePasswordAuthentication": "true"
    },
    "osType": "Windows",
    "placementGroupId": "f67c14ab-e92c-408c-ae2d-da15866ec79a",
    "plan": {
        "name": "planName",
        "product": "planProduct",
        "publisher": "planPublisher"
    },
    "platformFaultDomain": "36",
    "platformUpdateDomain": "42",
    "publicKeys": [{
            "keyData": "ssh-rsa 0",
            "path": "/home/user/.ssh/authorized_keys0"
        },
        {
            "keyData": "ssh-rsa 1",
            "path": "/home/user/.ssh/authorized_keys1"
        }
    ],
    "publisher": "RDFE-Test-Microsoft-Windows-Server-Group",
    "resourceGroupName": "macikgo-test-may-23",
    "resourceId": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/virtualMachines/examplevmname",
    "securityProfile": {
        "secureBootEnabled": "true",
        "virtualTpmEnabled": "false"
    },
    "sku": "2019-Datacenter",
    "storageProfile": {
        "dataDisks": [{
            "caching": "None",
            "createOption": "Empty",
            "diskSizeGB": "1024",
            "image": {
                "uri": ""
            },
            "lun": "0",
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampledatadiskname",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }],
        "imageReference": {
            "id": "",
            "offer": "WindowsServer",
            "publisher": "MicrosoftWindowsServer",
            "sku": "2019-Datacenter",
            "version": "latest"
        },
        "osDisk": {
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "diskSizeGB": "30",
            "diffDiskSettings": {
                "option": "Local"
            },
            "encryptionSettings": {
                "enabled": "false"
            },
            "image": {
                "uri": ""
            },
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampleosdiskname",
            "osType": "Windows",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }
    },
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "baz:bash;foo:bar",
    "version": "15.05.22",
    "vmId": "02aab8a4-74ef-476e-8182-f6d2ba4166a6",
    "vmScaleSetName": "crpteste9vflji9",
    "vmSize": "Standard_A3",
    "zone": ""
}
```

#### <a name="linux"></a>[Linux](#tab/linux/)
```json
{
    "azEnvironment": "AZUREPUBLICCLOUD",
    "isHostCompatibilityLayerVm": "true",
    "licenseType":  "Windows_Client",
    "location": "westus",
    "name": "examplevmname",
    "offer": "UbuntuServer",
    "osProfile": {
        "adminUsername": "admin",
        "computerName": "examplevmname",
        "disablePasswordAuthentication": "true"
    },
    "osType": "Linux",
    "placementGroupId": "f67c14ab-e92c-408c-ae2d-da15866ec79a",
    "plan": {
        "name": "planName",
        "product": "planProduct",
        "publisher": "planPublisher"
    },
    "platformFaultDomain": "36",
    "platformUpdateDomain": "42",
    "publicKeys": [{
            "keyData": "ssh-rsa 0",
            "path": "/home/user/.ssh/authorized_keys0"
        },
        {
            "keyData": "ssh-rsa 1",
            "path": "/home/user/.ssh/authorized_keys1"
        }
    ],
    "publisher": "Canonical",
    "resourceGroupName": "macikgo-test-may-23",
    "resourceId": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/virtualMachines/examplevmname",
    "securityProfile": {
        "secureBootEnabled": "true",
        "virtualTpmEnabled": "false"
    },
    "sku": "18.04-LTS",
    "storageProfile": {
        "dataDisks": [{
            "caching": "None",
            "createOption": "Empty",
            "diskSizeGB": "1024",
            "image": {
                "uri": ""
            },
            "lun": "0",
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampledatadiskname",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }],
        "imageReference": {
            "id": "",
            "offer": "UbuntuServer",
            "publisher": "Canonical",
            "sku": "16.04.0-LTS",
            "version": "latest"
        },
        "osDisk": {
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "diskSizeGB": "30",
            "diffDiskSettings": {
                "option": "Local"
            },
            "encryptionSettings": {
                "enabled": "false"
            },
            "image": {
                "uri": ""
            },
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampleosdiskname",
            "osType": "linux",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }
    },
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "baz:bash;foo:bar",
    "version": "15.05.22",
    "vmId": "02aab8a4-74ef-476e-8182-f6d2ba4166a6",
    "vmScaleSetName": "crpteste9vflji9",
    "vmSize": "Standard_A3",
    "zone": ""
}
```

---

#### <a name="sample-5-get-the-azure-environment-where-the-vm-is-running"></a>Пример 5. получение среды Azure, в которой запущена виртуальная машина

Azure имеет различные национальные облака, например [Azure для государственных организаций](https://azure.microsoft.com/overview/clouds/government/). Иногда требуется, чтобы решения касательно среды выполнения принимала среда Azure. В следующем примере показано, как добиться такого поведения.

**Запрос**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2018-10-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2018-10-01&format=text"
```

---

**Ответ**

```
AzurePublicCloud
```

Облако и значения среды Azure перечислены здесь.

| Cloud | Среда Azure |
|-------|-------------------|
| [Все общедоступные глобальные регионы Azure](https://azure.microsoft.com/regions/) | AzurePublicCloud
| [Azure для государственных организаций](https://azure.microsoft.com/overview/clouds/government/) | AzureUSGovernmentCloud
| [Azure China 21Vianet](https://azure.microsoft.com/global-infrastructure/china/) | AzureChinaCloud;
| [Azure для Германии](https://azure.microsoft.com/overview/clouds/germany/) | AzureGermanCloud


#### <a name="sample-6-retrieve-network-information"></a>Пример 6. Получение сведений о сети

**Запрос**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/network?api-version=2017-08-01" | ConvertTo-Json  -Depth 64
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/network?api-version=2017-08-01"
```

---

**Ответ**

```json
{
  "interface": [
    {
      "ipv4": {
        "ipAddress": [
          {
            "privateIpAddress": "10.1.0.4",
            "publicIpAddress": "X.X.X.X"
          }
        ],
        "subnet": [
          {
            "address": "10.1.0.0",
            "prefix": "24"
          }
        ]
      },
      "ipv6": {
        "ipAddress": []
      },
      "macAddress": "000D3AF806EC"
    }
  ]
}
```

#### <a name="sample-7-retrieve-public-ip-address"></a>Пример 7. получение общедоступного IP-адреса

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri "http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text"
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text"
```

---

## <a name="attested-data"></a>Аттестация данных

### <a name="get-attested-data"></a>Получение Аттестацических данных

IMDS помогает предоставлять гарантию того, что предоставленные данные поступают из Azure. Корпорация Майкрософт подписывает часть этих сведений, чтобы вы могли убедиться, что образ в Azure Marketplace — это тот, который вы используете в Azure.

```
GET /metadata/attested/document
```

#### <a name="parameters"></a>Параметры

| Имя | Обязательный/необязательный | Описание |
|------|-------------------|-------------|
| `api-version` | Обязательно | Версия, используемая для обслуживания запроса.
| `nonce` | Необязательно | Строка из 10 цифр, которая служит в качестве криптографического nonce. Если значение не указано, IMDS использует текущую метку времени в формате UTC.

#### <a name="response"></a>Ответ

```json
{
    "encoding":"pkcs7",
    "signature":"MIIEEgYJKoZIhvcNAQcCoIIEAzCCA/8CAQExDzANBgkqhkiG9w0BAQsFADCBugYJKoZIhvcNAQcBoIGsBIGpeyJub25jZSI6IjEyMzQ1NjY3NjYiLCJwbGFuIjp7Im5hbWUiOiIiLCJwcm9kdWN0IjoiIiwicHVibGlzaGVyIjoiIn0sInRpbWVTdGFtcCI6eyJjcmVhdGVkT24iOiIxMS8yMC8xOCAyMjowNzozOSAtMDAwMCIsImV4cGlyZXNPbiI6IjExLzIwLzE4IDIyOjA4OjI0IC0wMDAwIn0sInZtSWQiOiIifaCCAj8wggI7MIIBpKADAgECAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBBAUAMCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tMB4XDTE4MTEyMDIxNTc1N1oXDTE4MTIyMDIxNTc1NlowKzEpMCcGA1UEAxMgdGVzdHN1YmRvbWFpbi5tZXRhZGF0YS5henVyZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAML/tBo86ENWPzmXZ0kPkX5dY5QZ150mA8lommszE71x2sCLonzv4/UWk4H+jMMWRRwIea2CuQ5RhdWAHvKq6if4okKNt66fxm+YTVz9z0CTfCLmLT+nsdfOAsG1xZppEapC0Cd9vD6NCKyE8aYI1pliaeOnFjG0WvMY04uWz2MdAgMBAAGjYDBeMFwGA1UdAQRVMFOAENnYkHLa04Ut4Mpt7TkJFfyhLTArMSkwJwYDVQQDEyB0ZXN0c3ViZG9tYWluLm1ldGFkYXRhLmF6dXJlLmNvbYIQZ8VuSofHbJRAQNBNpiASdDANBgkqhkiG9w0BAQQFAAOBgQCLSM6aX5Bs1KHCJp4VQtxZPzXF71rVKCocHy3N9PTJQ9Fpnd+bYw2vSpQHg/AiG82WuDFpPReJvr7Pa938mZqW9HUOGjQKK2FYDTg6fXD8pkPdyghlX5boGWAMMrf7bFkup+lsT+n2tRw2wbNknO1tQ0wICtqy2VqzWwLi45RBwTGB6DCB5QIBATA/MCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBCwUAMA0GCSqGSIb3DQEBAQUABIGAld1BM/yYIqqv8SDE4kjQo3Ul/IKAVR8ETKcve5BAdGSNkTUooUGVniTXeuvDj5NkmazOaKZp9fEtByqqPOyw/nlXaZgOO44HDGiPUJ90xVYmfeK6p9RpJBu6kiKhnnYTelUk5u75phe5ZbMZfBhuPhXmYAdjc7Nmw97nx8NnprQ="
}
```

> [!NOTE]
> Из-за механизма кэширования IMDS может возвращаться ранее кэшированное значение nonce.

BLOB-объект сигнатуры — это версия документа, подписанная [PKCS7](https://aka.ms/pkcs7). Он содержит сертификат, используемый для подписывания, а также определенные сведения, относящиеся к виртуальной машине.

Для виртуальных машин, созданных с помощью Azure Resource Manager, документ включает,,, `vmId` `sku` `nonce` `subscriptionId` , `timeStamp` для создания и окончания срока действия документа, а также сведения о плане изображения. Сведения о плане заполняются только для образов из Azure Marketplace.

Для виртуальных машин, созданных с помощью классической модели развертывания, `vmId` гарантируется только заполнение. Вы можете извлечь сертификат из ответа и использовать его для подтверждения того, что ответ действителен и поступает из Azure.

Декодированный документ содержит следующие поля:

| Данные | Описание | Представленная версия |
|------|-------------|--------------------|
| `licenseType` | Тип лицензии для [преимущество гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-benefit). Это относится только к виртуальным машинам с поддержкой АХБ. | 2020-09-01
| `nonce` | Строка, которую можно дополнительно указать с помощью запроса. Если `nonce` значение не указано, используется текущая отметка времени координированного скоординированного периода. | 2018-10-01
| `plan` | [План образа Azure Marketplace](/rest/api/compute/virtualmachines/createorupdate#plan). Содержит идентификатор плана (имя), образ продукта или предложение (продукт) и идентификатор издателя (издатель). | 2018-10-01
| `timestamp.createdOn` | Метка времени в формате UTC для момента создания подписанного документа | 2018-20-01
| `timestamp.expiresOn` | Метка времени в формате UTC для момента истечения срока действия подписанного документа | 2018-10-01
| `vmId` | [Уникальный идентификатор](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) для виртуальной машины | 2018-10-01
| `subscriptionId` | Подписка Azure для виртуальной машины | 2019-04-30
| `sku` | Определенный номер SKU для образа виртуальной машины | 2019-11-01

> [!NOTE]
> Для классических (не Azure Resource Manager) виртуальных машин гарантируется заполнение только vmId.

Пример документа:
```json
{
   "nonce":"20201130-211924",
   "plan":{
      "name":"planName",
      "product":"planProduct",
      "publisher":"planPublisher"
   },
   "sku":"Windows-Server-2012-R2-Datacenter",
   "subscriptionId":"8d10da13-8125-4ba9-a717-bf7490507b3d",
   "timeStamp":{
      "createdOn":"11/30/20 21:19:19 -0000",
      "expiresOn":"11/30/20 21:19:24 -0000"
   },
   "vmId":"02aab8a4-74ef-476e-8182-f6d2ba4166a6"
}
```

#### <a name="sample-1-validate-that-the-vm-is-running-in-azure"></a>Пример 1. Проверка того, что виртуальная машина запущена в Azure

Поставщики в Azure Marketplace хотят убедиться, что их программное обеспечение лицензировано для запуска только в Azure. Если кто-то копирует виртуальный жесткий диск в локальную среду, поставщик должен иметь возможность обнаружить это. С помощью IMDS эти поставщики могут получать подписанные данные, гарантирующие ответ только из Azure.

> [!NOTE]
> Для работы с этим образцом требуется установка служебной программы JQ.

**Проверка**

#### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
# Get the signature
$attestedDoc = Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Proxy $Null -Uri http://169.254.169.254/metadata/attested/document?api-version=2020-09-01
# Decode the signature
$signature = [System.Convert]::FromBase64String($attestedDoc.signature)
```

Убедитесь, что подпись относится к Microsoft Azure и проверьте цепочку сертификатов на наличие ошибок.

```powershell
# Get certificate chain
$cert = [System.Security.Cryptography.X509Certificates.X509Certificate2]($signature)
$chain = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Chain
$chain.Build($cert)
# Print the Subject of each certificate in the chain
foreach($element in $chain.ChainElements)
{
    Write-Host $element.Certificate.Subject
}

# Get the content of the signed document
Add-Type -AssemblyName System.Security
$signedCms = New-Object -TypeName System.Security.Cryptography.Pkcs.SignedCms
$signedCms.Decode($signature);
$content = [System.Text.Encoding]::UTF8.GetString($signedCms.ContentInfo.Content)
Write-Host "Attested data: " $content
$json = $content | ConvertFrom-Json
# Do additional validation here
```

#### <a name="linux"></a>[Linux](#tab/linux/)

```bash
# Get the signature
curl --silent -H Metadata:True --noproxy "*" "http://169.254.169.254/metadata/attested/document?api-version=2020-09-01" | jq -r '.["signature"]' > signature
# Decode the signature
base64 -d signature > decodedsignature
# Get PKCS7 format
openssl pkcs7 -in decodedsignature -inform DER -out sign.pk7
# Get Public key out of pkc7
openssl pkcs7 -in decodedsignature -inform DER  -print_certs -out signer.pem
# Get the intermediate certificate
curl -s -o intermediate.cer "$(openssl x509 -in signer.pem -text -noout | grep " CA Issuers -" | awk -FURI: '{print $2}')"
openssl x509 -inform der -in intermediate.cer -out intermediate.pem
# Verify the contents
openssl smime -verify -in sign.pk7 -inform pem -noverify
```

**Результаты**

```json
Verification successful
{
  "nonce": "20181128-001617",
  "plan":
    {
      "name": "",
      "product": "",
      "publisher": ""
    },
  "timeStamp":
    {
      "createdOn": "11/28/18 00:16:17 -0000",
      "expiresOn": "11/28/18 06:16:17 -0000"
    },
  "vmId": "d3e0e374-fda6-4649-bbc9-7f20dc379f34",
  "licenseType": "Windows_Client",  
  "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
  "sku": "RS3-Pro"
}
```

Убедитесь, что подпись относится к Microsoft Azure, и проверьте цепочку сертификатов на наличие ошибок.

```bash
# Verify the subject name for the main certificate
openssl x509 -noout -subject -in signer.pem
# Verify the issuer for the main certificate
openssl x509 -noout -issuer -in signer.pem
#Validate the subject name for intermediate certificate
openssl x509 -noout -subject -in intermediate.pem
# Verify the issuer for the intermediate certificate
openssl x509 -noout -issuer -in intermediate.pem
# Verify the certificate chain, for Azure China 21Vianet the intermediate certificate will be from DigiCert Global Root CA
openssl verify -verbose -CAfile /etc/ssl/certs/Baltimore_CyberTrust_Root.pem -untrusted intermediate.pem signer.pem
```

---

> [!NOTE]
> Из-за механизма кэширования IMDS может возвращаться ранее кэшированное `nonce` значение.

Объект `nonce` в подписанном документе можно сравнить, если `nonce` в первоначальном запросе был указан параметр.

> [!NOTE]
> Сертификат для общедоступного облака и каждого независимых облака будут отличаться.

| Cloud | Сертификат |
|-------|-------------|
| [Все общедоступные глобальные регионы Azure](https://azure.microsoft.com/regions/) | *.metadata.azure.com
| [Azure для государственных организаций](https://azure.microsoft.com/overview/clouds/government/) | *.metadata.azure.us
| [Azure China 21Vianet](https://azure.microsoft.com/global-infrastructure/china/) | *.metadata.azure.cn
| [Azure для Германии](https://azure.microsoft.com/overview/clouds/germany/) | *.metadata.microsoftazure.de

> [!NOTE]
> Сертификаты могут не иметь точного соответствия `metadata.azure.com` для общедоступного облака. По этой причине проверка сертификата должна допускать общее имя из любого `.metadata.azure.com` поддомена.

В случаях, когда промежуточный сертификат не может быть загружен из-за сетевых ограничений во время проверки, можно закрепить промежуточный сертификат. Azure выполняет перебор сертификатов, что является стандартной практикой PKI. Если происходит смена, необходимо обновить закрепленные сертификаты. При каждом запланированном изменении промежуточного сертификата блог Azure обновляется, и клиенты Azure получают уведомления. 

Промежуточные сертификаты можно найти в [репозитории PKI](https://www.microsoft.com/pki/mscorp/cps/default.htm). Для разных регионов могут использоваться различные промежуточные сертификаты.

> [!NOTE]
> Промежуточный сертификат для Azure китайского центра для Китая будет находиться в глобальном корневом ЦС DigiCert вместо Baltimore.
Если вы закрепляете промежуточные сертификаты для Azure для Китая в рамках изменения центра корневых цепочек, необходимо обновить промежуточные сертификаты.


## <a name="managed-identity"></a>Управляемое удостоверение

Управляемое удостоверение, назначенное системой, может быть включено на виртуальной машине. Можно также назначить виртуальной машине одно или несколько управляемых удостоверений, назначенных пользователем.
Затем можно запросить маркеры для управляемых удостоверений из IMDS. Используйте эти токены для проверки подлинности в других службах Azure, например Azure Key Vault.

Подробные инструкции по включению этой функции см. в разделе [Получение маркера доступа](../articles/active-directory/managed-identities-azure-resources/how-to-use-vm-token.md).

## <a name="load-balancer-metadata"></a>Метаданные Load Balancer
При помещении экземпляров виртуальных машин или наборов виртуальных машин за Load Balancer (цен. категория "Стандартный") Azure можно использовать IMDS для получения метаданных, связанных с подсистемой балансировки нагрузки и экземплярами. Дополнительные сведения см. в разделе [Получение сведений о подсистеме балансировки нагрузки](../articles/load-balancer/instance-metadata-service-load-balancer.md).

## <a name="scheduled-events"></a>Запланированные события
Состояние запланированных событий можно получить с помощью IMDS. Затем пользователь может указать набор действий, выполняемых при выполнении этих событий. Дополнительные сведения см. в статье [запланированные события для Linux](../articles/virtual-machines/linux/scheduled-events.md) или [запланированных событий для Windows](../articles/virtual-machines/windows/scheduled-events.md).


## <a name="sample-code-in-different-languages"></a>Пример кода на разных языках

В следующей таблице перечислены примеры вызова IMDS с использованием разных языков в виртуальной машине.

| Язык | Пример |
|----------|---------|
| Bash | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.sh
| C# | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.cs
| Go | https://github.com/Microsoft/azureimds/blob/master/imdssample.go
| Java | https://github.com/Microsoft/azureimds/blob/master/imdssample.java
| Node.js | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.js
| Perl | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.pl
| PowerShell | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.ps1
| Puppet | https://github.com/keirans/azuremetadata
| Python | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.py
| Ruby | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.rb

## <a name="errors-and-debugging"></a>Ошибки и отладка

Если элемент данных не найден или запрос неправильно сформирован, службы метаданных экземпляров возвращают стандартные ошибки HTTP. Пример:

| Код состояния HTTP | Причина |
|------------------|--------|
| `200 OK` | Запрос выполнен успешно.
| `400 Bad Request` | Отсутствует `Metadata: true` заголовок или отсутствующий параметр `format=json` при запросе конечного узла
| `404 Not Found` | Запрашиваемый элемент не существует
| `405 Method Not Allowed` | Метод HTTP (verb) не поддерживается в конечной точке.
| `410 Gone` | Подождите немного и повторите попытку в течение 70 секунд
| `429 Too Many Requests` | Превышены [пределы частоты](#rate-limiting) API
| `500 Service Error` | Повторите попытку через некоторое время

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

**Я получаю сообщение об ошибке `400 Bad Request, Required metadata header not specified` . Что это означает?**

IMDS требует передачи заголовка `Metadata: true` в запрос. Передача этого заголовка в вызове RESTFUL обеспечивает доступ к IMDS.

**Почему я не получаю сведения о вычислениях для виртуальной машины?**

В настоящее время IMDS поддерживает только экземпляры, созданные с помощью Azure Resource Manager.

**Моя виртуальная машина создана с помощью Azure Resource Manager некоторое время назад. Почему я не вижу сведений о метаданных вычислений?**

Если вы создали виртуальную машину после сентября 2016, добавьте [тег](../articles/azure-resource-manager/management/tag-resources.md) , чтобы начать просмотр метаданных вычислений. Если вы создали виртуальную машину до сентября 2016, добавьте или удалите расширения или диски данных для экземпляра виртуальной машины, чтобы обновить метаданные.

**Почему я не вижу все данные, заполненные для новой версии?**

Если вы создали виртуальную машину после сентября 2016, добавьте [тег](../articles/azure-resource-manager/management/tag-resources.md) , чтобы начать просмотр метаданных вычислений. Если вы создали виртуальную машину до сентября 2016, добавьте или удалите расширения или диски данных для экземпляра виртуальной машины, чтобы обновить метаданные.

**Почему я получаю сообщение об ошибке `500 Internal Server Error` или `410 Resource Gone` ?**

Повторите запрос. Дополнительные сведения см. в разделе [обработка временных сбоев](/azure/architecture/best-practices/transient-faults). Если проблема будет повторяться, создайте ошибку поддержки в портал Azure виртуальной машины.

**Будет ли это работать для экземпляров масштабируемого набора виртуальных машин?**

Да, IMDS доступен для экземпляров масштабируемых наборов виртуальных машин.

**Я обновил теги в масштабируемых наборах виртуальных машин, но они не отображаются в экземплярах (в отличие от виртуальных машин с одним экземпляром). Я делаю что-то не так?**

В настоящее время теги для масштабируемых наборов виртуальных машин отображаются только для виртуальной машины при перезагрузке, повторном образе или изменении диска на экземпляр.

**Почему время ожидания запроса истекло для моего вызова службы?**

Вызовы метаданных должны выполняться с основного IP-адреса, назначенного основной сетевой карте виртуальной машины. Кроме того, если вы изменили маршруты, в локальной таблице маршрутизации виртуальной машины должен быть маршрут для адреса 169.254.169.254/32.

#### <a name="windows"></a>[Windows](#tab/windows/)

1. Выведите дамп локальной таблицы маршрутизации и найдите запись IMDS. Пример:
    ```console
    > route print
    IPv4 Route Table
    ===========================================================================
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
              0.0.0.0          0.0.0.0      172.16.69.1      172.16.69.7     10
            127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
            127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
      127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
        168.63.129.16  255.255.255.255      172.16.69.1      172.16.69.7     11
      169.254.169.254  255.255.255.255      172.16.69.1      172.16.69.7     11
    ... (continues) ...
    ```
1. Убедитесь, что маршрут существует для `169.254.169.254` , и обратите внимание на соответствующий сетевой интерфейс (например, `172.16.69.7` ).
1. Выведите дамп конфигурации интерфейса и найдите интерфейс, соответствующий тому, на который ссылается таблица маршрутизации, указывая MAC-адрес (физический).
    ```console
    > ipconfig /all
    ... (continues) ...
    Ethernet adapter Ethernet:

       Connection-specific DNS Suffix  . : xic3mnxjiefupcwr1mcs1rjiqa.cx.internal.cloudapp.net
       Description . . . . . . . . . . . : Microsoft Hyper-V Network Adapter
       Physical Address. . . . . . . . . : 00-0D-3A-E5-1C-C0
       DHCP Enabled. . . . . . . . . . . : Yes
       Autoconfiguration Enabled . . . . : Yes
       Link-local IPv6 Address . . . . . : fe80::3166:ce5a:2bd5:a6d1%3(Preferred)
       IPv4 Address. . . . . . . . . . . : 172.16.69.7(Preferred)
       Subnet Mask . . . . . . . . . . . : 255.255.255.0
    ... (continues) ...
    ```
1. Убедитесь, что интерфейс соответствует основному сетевому адаптеру виртуальной машины и основному IP-адресу. Основной сетевой адаптер и IP-адрес можно найти, просмотрев конфигурацию сети в портал Azure или выполнив поиск в Azure CLI. Обратите внимание на частные IP-адреса (и MAC-адрес, если вы используете интерфейс командной строки). Вот пример PowerShell CLI:
    ```powershell
    $ResourceGroup = '<Resource_Group>'
    $VmName = '<VM_Name>'
    $NicNames = az vm nic list --resource-group $ResourceGroup --vm-name $VmName | ConvertFrom-Json | Foreach-Object { $_.id.Split('/')[-1] }
    foreach($NicName in $NicNames)
    {
        $Nic = az vm nic show --resource-group $ResourceGroup --vm-name $VmName --nic $NicName | ConvertFrom-Json
        Write-Host $NicName, $Nic.primary, $Nic.macAddress
    }
    # Output: wintest767 True 00-0D-3A-E5-1C-C0
    ```
1. Если они не совпадают, обновите таблицу маршрутизации, чтобы обеспечить целевую основную сетевую карту и IP-адрес.

#### <a name="linux"></a>[Linux](#tab/linux/)

 1. Выведите дамп локальной таблицы маршрутизации с помощью команды, например, `netstat -r` и найдите запись IMDS (например,):
    ```console
    ~$ netstat -r
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    default         _gateway        0.0.0.0         UG        0 0          0 eth0
    168.63.129.16   _gateway        255.255.255.255 UGH       0 0          0 eth0
    169.254.169.254 _gateway        255.255.255.255 UGH       0 0          0 eth0
    172.16.69.0     0.0.0.0         255.255.255.0   U         0 0          0 eth0
    ```
1. Убедитесь, что маршрут существует для `169.254.169.254` , и обратите внимание на соответствующий сетевой интерфейс (например, `eth0` ).
1. Выведите дамп конфигурации интерфейса для соответствующего интерфейса в таблице маршрутизации (Обратите внимание, что может отличаться точное имя файла конфигурации).
    ```console
    ~$ cat /etc/netplan/50-cloud-init.yaml
    network:
    ethernets:
        eth0:
            dhcp4: true
            dhcp4-overrides:
                route-metric: 100
            dhcp6: false
            match:
                macaddress: 00:0d:3a:e4:c7:2e
            set-name: eth0
    version: 2
    ```
1. При использовании динамического IP-адреса запишите MAC-адрес. Если вы используете статический IP-адрес, вы можете заметить указанные IP-адреса (s) и/или MAC.
1. Убедитесь, что интерфейс соответствует основному сетевому адаптеру виртуальной машины и основному IP-адресу. Основной сетевой адаптер и IP-адрес можно найти, просмотрев конфигурацию сети в портал Azure или выполнив поиск в Azure CLI. Обратите внимание на частные IP-адреса (и MAC-адрес, если вы используете интерфейс командной строки). Вот пример PowerShell CLI:
    ```powershell
    $ResourceGroup = '<Resource_Group>'
    $VmName = '<VM_Name>'
    $NicNames = az vm nic list --resource-group $ResourceGroup --vm-name $VmName | ConvertFrom-Json | Foreach-Object { $_.id.Split('/')[-1] }
    foreach($NicName in $NicNames)
    {
        $Nic = az vm nic show --resource-group $ResourceGroup --vm-name $VmName --nic $NicName | ConvertFrom-Json
        Write-Host $NicName, $Nic.primary, $Nic.macAddress
    }
    # Output: ipexample606 True 00-0D-3A-E4-C7-2E
    ```
1. Если они не совпадают, обновите таблицу маршрутизации таким путем, чтобы основной сетевой адаптер или IP-адрес был целевым.

---

**Отказоустойчивая кластеризация в Windows Server**

При запросе IMDS с помощью отказоустойчивой кластеризации иногда необходимо добавить маршрут в таблицу маршрутизации. Это делается так.

1. Откройте окно командной строки с правами администратора.

1. Выполните следующую команду и запишите адрес интерфейса для назначения сети ( `0.0.0.0` ) в таблице маршрутов IPv4.

```bat
route print
```

> [!NOTE]
> Ниже приведен пример выходных данных из виртуальной машины Windows Server с включенным отказоустойчивым кластером. Для простоты выходные данные содержат только таблицу маршрутов IPv4.

```
IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0         10.0.1.1        10.0.1.10    266
         10.0.1.0  255.255.255.192         On-link         10.0.1.10    266
        10.0.1.10  255.255.255.255         On-link         10.0.1.10    266
        10.0.1.15  255.255.255.255         On-link         10.0.1.10    266
        10.0.1.63  255.255.255.255         On-link         10.0.1.10    266
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
      169.254.0.0      255.255.0.0         On-link     169.254.1.156    271
    169.254.1.156  255.255.255.255         On-link     169.254.1.156    271
  169.254.255.255  255.255.255.255         On-link     169.254.1.156    271
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link     169.254.1.156    271
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link     169.254.1.156    271
  255.255.255.255  255.255.255.255         On-link         10.0.1.10    266
```

Выполните следующую команду и используйте адрес интерфейса для назначения сети ( `0.0.0.0` ), `10.0.1.10` в этом примере — ().

```bat
route add 169.254.169.254/32 10.0.1.10 metric 1 -p
```

## <a name="support"></a>Поддержка

Если вы не сможете получить ответ на метаданные после нескольких попыток, можно создать ошибку поддержки в портал Azure.

## <a name="product-feedback"></a>Отзыв о продукте

Отзывы и идеи по продуктам можно предоставить в канале отзывов пользователей в разделе Виртуальные машины > служба метаданных экземпляра. https://feedback.azure.com/forums/216843-virtual-machines?category_id=394627

## <a name="next-steps"></a>Дальнейшие действия

[Получение маркера доступа для виртуальной машины](../articles/active-directory/managed-identities-azure-resources/how-to-use-vm-token.md)

[Запланированные события для Linux](../articles/virtual-machines/linux/scheduled-events.md)

[Запланированные события для Windows](../articles/virtual-machines/windows/scheduled-events.md)
