---
title: Настройка средства создания моментальных снимков для приложений Azure для Azure NetApp Files | Документация Майкрософт
description: Содержит инструкции по запуску команды configure средства создания моментального снимка с единообразием приложений Azure, которое можно использовать с Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 12/14/2020
ms.author: phjensen
ms.openlocfilehash: 0875aae8bb9049fc96377c1c98efa7391211d08f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97632975"
---
# <a name="configure-azure-application-consistent-snapshot-tool-preview"></a>Настройка средства создания моментальных снимков для приложений Azure (Предварительная версия)

В этой статье представлено руководство по запуску команды configure средства создания моментальных снимков с единообразием приложений Azure, которое можно использовать с Azure NetApp Files.

## <a name="introduction"></a>Введение

Файл конфигурации можно создать или изменить с помощью `azacsnap -c configure` команды.

## <a name="command-options"></a>Параметры команды

`-c configure`Команда имеет следующие параметры.

- `--configuration new` для создания нового файла конфигурации.

- `--configuration edit` для изменения существующего файла конфигурации.

- `[--configfile <config filename>]` — Необязательный параметр, который позволяет использовать пользовательские имена файлов конфигурации.

## <a name="configuration-file-for-snapshot-tools"></a>Файл конфигурации для средств создания моментальных снимков

Файл конфигурации можно создать, выполнив `azacsnap -c configure --configuration new` .  По умолчанию имя файла конфигурации — `azacsnap.json` .  С параметром можно использовать имя пользовательского файла `--configfile=` (например, `--configfile=<customname>.json` ) следующий пример предназначен для конфигурации крупных экземпляров Azure:

```bash
azacsnap -c configure --configuration new
```

```output
Building new config file
Add comment to config file (blank entry to exit adding comments):This is a new config file for `azacsnap`
Add comment to config file (blank entry to exit adding comments):
Add database to config? (y/n) [n]: y
HANA SID (for example, H80): H80
HANA Instance Number (for example, 00): 00
HANA HDB User Store Key (for example, `hdbuserstore List`): AZACSNAP
HANA Server's Address (hostname or IP address): testing01
Add ANF Storage to database section? (y/n) [n]:
Add HLI Storage to database section? (y/n) [n]: y
Add DATA Volume to HLI Storage section of Database section? (y/n) [n]: y
Storage User Name (for example, clbackup25): clt1h80backup
Storage IP Address (for example, 192.168.1.30): 172.18.18.11
Storage Volume Name (for example, hana_data_h80_testing01_mnt00001_t250_vol): hana_data_h80_testing01_mnt00001_t020_vol
Add DATA Volume to HLI Storage section of Database section? (y/n) [n]:
Add OTHER Volume to HLI Storage section of Database section? (y/n) [n]:
Add HLI Storage to database section? (y/n) [n]:
Add database to config? (y/n) [n]:
Editing configuration complete, writing output to 'azacsnap.json'
```

## <a name="details-of-required-values"></a>Сведения о необходимых значениях

В следующих разделах содержатся подробные инструкции по различным значениям, необходимым для файла конфигурации.

### <a name="sap-hana-values"></a>Значения SAP HANA

При добавлении *базы данных* в конфигурацию требуются следующие значения.

- **Адрес сервера Hana** — имя узла или IP-адрес SAP HANA сервера.
- **Hana SID** = идентификатор системы SAP HANA.
- **Экземпляр Hana** — номер экземпляра SAP HANA.
- **Hana HDB User Store Key** — пользователь SAP Hana, настроенный с разрешениями на выполнение резервного копирования базы данных.

- Один узел: IP-адрес и имя узла
- HSR с STONITH: IP и имя узла
- Горизонтальное масштабирование (N + N, N + M): текущий IP-адрес главного узла и имя узла
- HSR без STONITH: IP и имя узла
- Multi SID на одном узле: hostname и IP узла, на котором размещены эти идентификаторы безопасности

### <a name="azure-large-instance-hli-storage-values"></a>Значения хранилища больших экземпляров Azure (ХЛИ)

При добавлении *хранилища Хли* в раздел базы данных требуются следующие значения:

- **Имя пользователя хранилища** — это имя пользователя, используемое для УСТАНОВЛЕНия SSH-подключения к хранилищу.
- **IP-адрес хранилища** — адрес системы хранения.
- **Имя тома хранилища** = имя тома для моментального снимка.  Это значение можно определить несколькими способами. возможно, проще всего попробовать следующую команду оболочки:

    ```bash
    grep nfs /etc/fstab | cut -f2 -d"/" | sort | uniq
    ```

    ```output
    hana_data_p40_soldub41_mnt00001_t020_vol
    hana_log_backups_p40_soldub41_t020_vol
    hana_log_p40_soldub41_mnt00001_t020_vol
    hana_shared_p40_soldub41_t020_vol
    ```

### <a name="azure-netapp-files-anf-storage-values"></a>Значения хранилища Azure NetApp Files (использовании)

При добавлении *хранилища использовании* в раздел базы данных требуются следующие значения:

- **Имя файла проверки подлинности субъекта-службы** — это `authfile.json` файл, созданный в Cloud Shell при настройке взаимодействия с хранилищем Azure NetApp Files.
- **Полный идентификатор ресурса тома хранилища использовании** = полный идентификатор ресурса для моментального снимка тома.  Это можно получить из: портал Azure – > использовании – > том — > параметры/свойства — > идентификатор ресурса.

## <a name="configuration-file-overview-azacsnapjson"></a>Общие сведения о файле конфигурации ( `azacsnap.json` )

В следующем примере для задан `azacsnap.json` один идентификатор безопасности.

Значения параметров должны быть заданы для конкретной среды SAP HANA клиента.
Для системы **крупных экземпляров Azure** эта информация предоставляется службой управления службами Майкрософт во время вызова процедуры адаптации или передачи вам образа и становится доступной в файле Excel, который предоставляется во время передачи вам образа. Если вам потребуется снова предоставить эти сведения, откройте запрос на обслуживание.

Ниже приведен только пример, обновите все значения соответствующим образом.

```bash
cat azacsnap.json
```

```output
{
  "version": "5.0 Preview",
  "logPath": "./logs",
  "securityPath": "./security",
  "comments": [],
  "database": [
    {
      "hana": {
        "serverAddress": "sapprdhdb80",
        "sid": "H80",
        "instanceNumber": "00",
        "hdbUserStoreName": "SCADMIN",
        "savePointAbortWaitSeconds": 600,
        "hliStorage": [
          {
            "dataVolume": [
              {
                "backupName": "clt1h80backup",
                "ipAddress": "172.18.18.11",
                "volume": "hana_data_h80_azsollabbl20a31_mnt00001_t210_vol"
              },
              {
                "backupName": "clt1h80backup",
                "ipAddress": "172.18.18.11",
                "volume": "hana_shared_h80_azsollabbl20a31_t210_vol"
              }
            ],
            "otherVolume": [
              {
                "backupName": "clt1h80backup",
                "ipAddress": "172.18.18.11",
                "volume": "hana_log_backups_h80_azsollabbl20a31_t210_vol"
              }
            ]
          }
        ],
        "anfStorage": []
      }
    }
  ]
}
```

> [!NOTE]
> В случае аварийного восстановления, когда резервное копирование выполняется на сайте аварийного восстановления, имя сервера HANA, настроенное в файле конфигурации АВАРИЙного восстановления (например, `DR.json` ) на сайте аварийного восстановления, должно совпадать с именем рабочего сервера.

> [!NOTE]
> Для крупных экземпляров Azure IP-адрес хранилища должен находиться в той же подсети, что и пул серверов. Например, в этом случае подсеть пула серверов — 172. 18. 18.0/24 и наш назначенный IP-адрес хранилища — 172.18.18.11.

## <a name="next-steps"></a>Дальнейшие действия

- [Тестирование средства создания моментального снимка с единообразием в приложении Azure](azacsnap-cmd-ref-test.md)
