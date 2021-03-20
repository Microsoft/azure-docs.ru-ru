---
title: Справочник по YAML для группы контейнеров
description: Справочник по файлу YAML, поддерживаемому экземплярами контейнеров Azure для настройки группы контейнеров
ms.topic: article
ms.date: 07/06/2020
ms.openlocfilehash: d0ec8d13eebba1c60f5a52f8c43bdd8b90eeb913
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87084766"
---
# <a name="yaml-reference-azure-container-instances"></a>Справочник по YAML: экземпляры контейнеров Azure

В этой статье описывается синтаксис и свойства файла YAML, поддерживаемого экземплярами контейнеров Azure, для настройки [группы контейнеров](container-instances-container-groups.md). Используйте файл YAML для ввода конфигурации группы в команду [AZ Container Create][az-container-create] в Azure CLI. 

Файл YAML — это удобный способ настройки группы контейнеров для воспроизводимых развертываний. Это краткая альтернатива использованию [шаблона диспетчер ресурсов](/azure/templates/Microsoft.ContainerInstance/2019-12-01/containerGroups) или пакетов SDK для экземпляров контейнеров Azure для создания или обновления группы контейнеров.

> [!NOTE]
> Эта ссылка относится к файлам YAML для экземпляров контейнеров Azure REST API версии `2019-12-01` .

## <a name="schema"></a>схема 

Ниже приведена схема для файла YAML, включая комментарии для выделения ключевых свойств. Описание свойств этой схемы см. в разделе [значения свойств](#property-values) .

```yml
name: string  # Name of the container group
apiVersion: '2019-12-01'
location: string
tags: {}
identity: 
  type: string
  userAssignedIdentities: {}
properties: # Properties of container group
  containers: # Array of container instances in the group
  - name: string # Name of an instance
    properties: # Properties of an instance
      image: string # Container image used to create the instance
      command:
      - string
      ports: # External-facing ports exposed on the instance, must also be set in group ipAddress property 
      - protocol: string
        port: integer
      environmentVariables:
      - name: string
        value: string
        secureValue: string
      resources: # Resource requirements of the instance
        requests:
          memoryInGB: number
          cpu: number
          gpu:
            count: integer
            sku: string
        limits:
          memoryInGB: number
          cpu: number
          gpu:
            count: integer
            sku: string
      volumeMounts: # Array of volume mounts for the instance
      - name: string
        mountPath: string
        readOnly: boolean
      livenessProbe:
        exec:
          command:
          - string
        httpGet:
          path: string
          port: integer
          scheme: string
        initialDelaySeconds: integer
        periodSeconds: integer
        failureThreshold: integer
        successThreshold: integer
        timeoutSeconds: integer
      readinessProbe:
        exec:
          command:
          - string
        httpGet:
          path: string
          port: integer
          scheme: string
        initialDelaySeconds: integer
        periodSeconds: integer
        failureThreshold: integer
        successThreshold: integer
        timeoutSeconds: integer
  imageRegistryCredentials: # Credentials to pull a private image
  - server: string
    username: string
    password: string
  restartPolicy: string
  ipAddress: # IP address configuration of container group
    ports:
    - protocol: string
      port: integer
    type: string
    ip: string
    dnsNameLabel: string
  osType: string
  volumes: # Array of volumes available to the instances
  - name: string
    azureFile:
      shareName: string
      readOnly: boolean
      storageAccountName: string
      storageAccountKey: string
    emptyDir: {}
    secret: {}
    gitRepo:
      directory: string
      repository: string
      revision: string
  diagnostics:
    logAnalytics:
      workspaceId: string
      workspaceKey: string
      logType: string
      metadata: {}
  networkProfile: # Virtual network profile for container group
    id: string
  dnsConfig: # DNS configuration for container group
    nameServers:
    - string
    searchDomains: string
    options: string
  sku: string # SKU for the container group
  encryptionProperties:
    vaultBaseUrl: string
    keyName: string
    keyVersion: string
  initContainers: # Array of init containers in the group
  - name: string
    properties:
      image: string
      command:
      - string
      environmentVariables:
      - name: string
        value: string
        secureValue: string
      volumeMounts:
      - name: string
        mountPath: string
        readOnly: boolean
```

## <a name="property-values"></a>Значения свойств

В следующих таблицах описаны значения, которые следует указать в этой схеме.



### <a name="microsoftcontainerinstancecontainergroups-object"></a>Объект Microsoft. Контаинеринстанце/Контаинерграупс

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  name | строка | Да | Имя группы контейнеров. |
|  версия_API | enum | Да | 2018-10-01 |
|  location | строка | Нет | Местоположение ресурса. |
|  tags | object | Нет | Теги ресурсов. |
|  удостоверение | object | Нет | Удостоверение группы контейнеров, если оно настроено. - [Объект Контаинерграупидентити](#containergroupidentity-object) |
|  properties | объект | Да | [Объект Контаинерграуппропертиес](#containergroupproperties-object) |




### <a name="containergroupidentity-object"></a>Объект Контаинерграупидентити

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  тип | enum | Нет | Тип удостоверения, используемого для группы контейнеров. Тип "SystemAssigned, Усерассигнед" включает как неявно созданное удостоверение, так и набор удостоверений, назначенных пользователем. Тип "нет" удалит все удостоверения из группы контейнеров. -SystemAssigned, Усерассигнед, SystemAssigned, Усерассигнед, нет |
|  userAssignedIdentities | object | Нет | Список удостоверений пользователей, связанных с группой контейнеров. Ссылки на ключ словаря удостоверений пользователя будут Azure Resource Manager идентификаторы ресурсов в формате "/Субскриптионс/{субскриптионид}/ресаурцеграупс/{ресаурцеграупнаме}/провидерс/Микрософт.манажедидентити/усерассигнедидентитиес/{идентитинаме}". |




### <a name="containergroupproperties-object"></a>Объект Контаинерграуппропертиес

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  containers | array | Да | Контейнеры в группе контейнеров. - [Объект контейнера](#container-object) |
|  imageRegistryCredentials | array | Нет | Учетные данные реестра образов, по которым создается группа контейнеров. - [Объект Имажерегистрикредентиал](#imageregistrycredential-object) |
|  restartPolicy | enum | Нет | Перезапустите политику для всех контейнеров в группе контейнеров. - `Always` Всегда перезагружать- `OnFailure` перезапустить при сбое — `Never` никогда не перезапускать. Всегда, onFailure, никогда |
|  ipAddress | object | Нет | Тип IP-адреса группы контейнеров. - [IpAddress, объект](#ipaddress-object) |
|  osType | enum | Да | Тип операционной системы, необходимый контейнерам в группе контейнеров. — Windows или Linux |
|  volumes. | array | Нет | Список томов, которые могут быть подключены контейнерами в этой группе контейнеров. - [Объект Volume](#volume-object) |
|  диагностика | object | Нет | Диагностические сведения для группы контейнеров. - [Объект Контаинерграупдиагностикс](#containergroupdiagnostics-object) |
|  networkProfile | object | Нет | Сведения о сетевом профиле для группы контейнеров. - [Объект Контаинерграупнетворкпрофиле](#containergroupnetworkprofile-object) |
|  dnsConfig | object | Нет | Сведения о конфигурации DNS для группы контейнеров. - [Объект Днсконфигуратион](#dnsconfiguration-object) |
| sku | enum | Нет | SKU для группы контейнеров — стандартный или выделенный |
| енкриптионпропертиес | object | Нет | Свойства шифрования для группы контейнеров. - [Объект Енкриптионпропертиес](#encryptionproperties-object) | 
| инитконтаинерс | array | Нет | Контейнеры init для группы контейнеров. - [Объект Инитконтаинердефинитион](#initcontainerdefinition-object) |




### <a name="container-object"></a>Объект контейнера

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  name | строка | Да | Предоставленное пользователем имя экземпляра контейнера. |
|  properties | объект | Да | Свойства экземпляра контейнера. - [Объект Контаинерпропертиес](#containerproperties-object) |




### <a name="imageregistrycredential-object"></a>Объект Имажерегистрикредентиал

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  server | строка | Да | Сервер реестра образа DOCKER без протокола, например "http" и "HTTPS". |
|  username | строка | Да | Имя пользователя для частного реестра. |
|  password | Строка | Нет | Пароль для частного реестра. |




### <a name="ipaddress-object"></a>IpAddress, объект

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  ports; | array | Да | Список портов, предоставляемых в группе контейнеров. - [Объект Port](#port-object) |
|  тип | enum | Да | Указывает, является ли IP-адрес общедоступным или частной ВИРТУАЛЬНОЙ сетью. — Открытый или частный |
|  см | Строка | Нет | IP-адрес, предоставляемый общедоступному Интернету. |
|  днснамелабел | Строка | Нет | Метка имени DNS для IP-адреса. |




### <a name="volume-object"></a>Объект Volume

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  name | строка | Да | Имя тома. |
|  азурефиле | object | Нет | Том файлов Azure. - [Объект Азурефилеволуме](#azurefilevolume-object) |
|  emptyDir | object | Нет | Пустой том каталога. |
|  secret | object | Нет | Секретный том. |
|  gitRepo | object | Нет | Том репозитория Git. - [Объект Гитреповолуме](#gitrepovolume-object) |




### <a name="containergroupdiagnostics-object"></a>Объект Контаинерграупдиагностикс

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  logAnalytics | object | Нет | Сведения о log Analytics для группы контейнеров. - [Объект LogAnalytics](#loganalytics-object) |




### <a name="containergroupnetworkprofile-object"></a>Объект Контаинерграупнетворкпрофиле

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  идентификатор | строка | Да | Идентификатор сетевого профиля. |




### <a name="dnsconfiguration-object"></a>Объект Днсконфигуратион

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  Серверах имен | array | Да | DNS-серверы для группы контейнеров. -String |
|  сеарчдомаинс | Строка | Нет | Домены поиска DNS для поиска имени узла в группе контейнеров. |
|  options | Строка | Нет | Параметры DNS для группы контейнеров. |


### <a name="encryptionproperties-object"></a>Объект Енкриптионпропертиес

| Имя  | Type  | Обязательно  | Значение |
|  ---- | ---- | ---- | ---- |
| ваултбасеурл  | строка    | Да   | Базовый URL-адрес keyvault. |
| keyName   | строка    | Да   | Имя ключа шифрования. |
| кэйверсион    | строка    | Да   | Версия ключа шифрования. |

### <a name="initcontainerdefinition-object"></a>Объект Инитконтаинердефинитион

| Имя  | Type  | Обязательно  | Значение |
|  ---- | ---- | ---- | ---- |
| name  | строка |  Да | Имя для контейнера init. |
| properties    | объект    | Да   | Свойства для контейнера init. - [Объект Инитконтаинерпропертиесдефинитион](#initcontainerpropertiesdefinition-object)


### <a name="containerproperties-object"></a>Объект Контаинерпропертиес

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  Изображение | строка | Да | Имя образа, используемого для создания экземпляра контейнера. |
|  . | array | Нет | Команды для выполнения в экземпляре контейнера в Exec Form. -String |
|  ports; | array | Нет | Доступные порты в экземпляре контейнера. - [Объект Контаинерпорт](#containerport-object) |
|  environmentVariables | array | Нет | Переменные среды, которые необходимо задать в экземпляре контейнера. - [Объект EnvironmentVariable](#environmentvariable-object) |
|  ресурсов | object | Да | Требования к ресурсам для экземпляра контейнера. - [Объект Ресаурцерекуирементс](#resourcerequirements-object) |
|  волумемаунтс | array | Нет | Тома, доступные для экземпляра контейнера. - [Объект Волумемаунт](#volumemount-object) |
|  ливенесспробе | object | Нет | Проверка актуальности. - [Объект Контаинерпробе](#containerprobe-object) |
|  реадинесспробе | object | Нет | Проба готовности. - [Объект Контаинерпробе](#containerprobe-object) |




### <a name="port-object"></a>Объект Port

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  protocol | enum | Нет | Протокол, связанный с портом. -TCP или UDP |
|  порт | Целое число | Да | номер порта. |




### <a name="azurefilevolume-object"></a>Объект Азурефилеволуме

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  Сетев | строка | Да | Имя файлового ресурса Azure, который будет подключен как том. |
|  readOnly | Логическое | Нет | Флаг, указывающий, является ли общий файл Azure, подключенный как том, только для чтения. |
|  storageAccountName | строка | Да | Имя учетной записи хранения, содержащей файловый ресурс Azure. |
|  storageAccountKey | Строка | Нет | Ключ доступа к учетной записи хранения, используемый для доступа к файловому ресурсу Azure. |




### <a name="gitrepovolume-object"></a>Объект Гитреповолуме

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  directory. | Строка | Нет | Имя целевого каталога. Не должен содержать или начинаться с "..".  Если указан параметр ".", каталог томов будет репозиторием Git.  В противном случае, если он указан, том будет содержать репозиторий Git в подкаталоге с заданным именем. |
|  repository | строка | Да | URL-адрес репозитория |
|  revision | Строка | Нет | Зафиксировать хэш для указанной редакции. |



### <a name="loganalytics-object"></a>Объект LogAnalytics

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  workspaceId | строка | Да | Идентификатор рабочей области для log Analytics |
|  workspaceKey | строка | Да | Ключ рабочей области для log Analytics |
|  логтипе | enum | Нет | Используемый тип журнала. -Контаинеринсигхтс или Контаинеринстанцелогс |
|  метаданные | object | Нет | Метаданные для log Analytics. |


### <a name="initcontainerpropertiesdefinition-object"></a>Объект Инитконтаинерпропертиесдефинитион

| Имя  | Type  | Обязательно  | Значение |
|  ---- | ---- | ---- | ---- |
| Изображение | строка    | Нет    | Изображение контейнера init. |
| .   | array | Нет    | Команда, которую необходимо выполнить в контейнере init в Exec Form. -String |
| environmentVariables | array  | Нет |Переменные среды, которые необходимо задать в контейнере init. - [Объект EnvironmentVariable](#environmentvariable-object)
| волумемаунтс |array   | Нет    | Подключения тома, доступные для контейнера init. - [Объект Волумемаунт](#volumemount-object)

### <a name="containerport-object"></a>Объект Контаинерпорт

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  protocol | enum | Нет | Протокол, связанный с портом. -TCP или UDP |
|  порт | Целое число | Да | Номер порта, предоставленный в группе контейнеров. |




### <a name="environmentvariable-object"></a>Объект EnvironmentVariable

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  name | строка | Да | Имя переменной среды. |
|  value | строка | Нет | Значение переменной среды. |
|  secureValue | Строка | Нет | Значение переменной защищенной среды. |




### <a name="resourcerequirements-object"></a>Объект Ресаурцерекуирементс

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  requests | object | Да | Запросы ресурса этого экземпляра контейнера. - [Объект Ресаурцерекуестс](#resourcerequests-object) |
|  ограничения | object | Нет | Ограничения ресурсов этого экземпляра контейнера. - [Объект Ресаурцелимитс](#resourcelimits-object) |




### <a name="volumemount-object"></a>Объект Волумемаунт

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  name | строка | Да | Имя подключения тома. |
|  mountPath | строка | Да | Путь в контейнере, куда должен быть подключен том. Не должно содержать двоеточие (:). |
|  readOnly | Логическое | Нет | Флаг, указывающий, что подключение тома доступно только для чтения. |




### <a name="containerprobe-object"></a>Объект Контаинерпробе

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  exec | object | Нет | Команда выполнения для проверки [объекта контаинерексек](#containerexec-object) |
|  httpGet | object | Нет | Объект HTTP Get Settings to зонд- [контаинерхттпжет](#containerhttpget-object) |
|  инитиалделайсекондс | Целое число | Нет | Начальная задержка в секундах. |
|  периодсекондс | Целое число | Нет | Период в секундах. |
|  фаилуресрешолд | Целое число | Нет | Пороговое значение сбоя. |
|  сукцесссрешолд | Целое число | Нет | Пороговое значение успеха. |
|  timeoutSeconds | Целое число | Нет | Время ожидания (в секундах). |




### <a name="resourcerequests-object"></a>Объект Ресаурцерекуестс

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  меморингб | Число | Да | Запрос памяти в ГБ этого экземпляра контейнера. |
|  cpu | Число | Да | Запрос ЦП этого экземпляра контейнера. |
|  совмещен | object | Нет | Запрос GPU этого экземпляра контейнера. - [Объект Гпуресаурце](#gpuresource-object) |




### <a name="resourcelimits-object"></a>Объект Ресаурцелимитс

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  меморингб | Число | Нет | Предельный объем памяти (в ГБ) данного экземпляра контейнера. |
|  cpu | Число | Нет | Ограничение ЦП данного экземпляра контейнера. |
|  совмещен | object | Нет | Предельный размер графического процессора для этого экземпляра контейнера. - [Объект Гпуресаурце](#gpuresource-object) |




### <a name="containerexec-object"></a>Объект Контаинерексек

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  . | array | Нет | Команды для выполнения в контейнере. -String |




### <a name="containerhttpget-object"></a>Объект Контаинерхттпжет

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  path | строка | Нет | Путь для проверки. |
|  порт | Целое число | Да | Номер порта для проверки. |
|  схема | enum | Нет | Схема. -HTTP или HTTPS |




### <a name="gpuresource-object"></a>Объект Гпуресаурце

|  Имя | Type | Обязательно | Значение |
|  ---- | ---- | ---- | ---- |
|  count | Целое число | Да | Число ресурсов GPU. |
|  sku | enum | Да | Номер SKU ресурса GPU. -K80, P100, V100 |


## <a name="next-steps"></a>Дальнейшие действия

См. Руководство по [развертыванию группы с несколькими контейнерами с помощью файла YAML](container-instances-multi-container-yaml.md).

См. примеры использования файла YAML для развертывания групп контейнеров в [виртуальной сети](container-instances-vnet.md) или [подключения внешнего тома](container-instances-volume-azure-files.md).

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create

