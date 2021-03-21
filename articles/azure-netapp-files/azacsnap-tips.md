---
title: Советы и рекомендации по использованию средства создания моментальных снимков для приложений Azure для Azure NetApp Files | Документация Майкрософт
description: Содержит советы и рекомендации по использованию средства создания моментальных снимков с единообразием приложений Azure, которые можно использовать с Azure NetApp Files.
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
ms.topic: how-to
ms.date: 12/14/2020
ms.author: phjensen
ms.openlocfilehash: 08edd86fd19e7698a791e411f42a2a89084a91f7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98737139"
---
# <a name="tips-and-tricks-for-using-azure-application-consistent-snapshot-tool-preview"></a>Советы и рекомендации по использованию средства создания моментальных снимков для приложений Azure (Предварительная версия)

В этой статье приводятся советы и рекомендации, которые могут быть полезны при использовании Азакснап.

## <a name="limit-service-principal-permissions"></a>Ограничение разрешений субъекта-службы

Может потребоваться ограничить область действия субъекта-службы Азакснап.  Дополнительные сведения о детальном управлении доступом к ресурсам Azure см. в [документации по Azure RBAC](../role-based-access-control/index.yml) .  

Ниже приведен пример определения роли с минимальными необходимыми действиями, необходимыми для функционирования Азакснап.

```bash
az role definition create --role-definition '{ \
  "Name": "Azure Application Consistent Snapshot tool", \
  "IsCustom": "true", \
  "Description": "Perform snapshots on ANF volumes.", \
  "Actions": [ \
    "Microsoft.NetApp/*/read", \
    "Microsoft.NetApp/netAppAccounts/capacityPools/volumes/snapshots/write", \
    "Microsoft.NetApp/netAppAccounts/capacityPools/volumes/snapshots/delete" \
  ], \
  "NotActions": [], \
  "DataActions": [], \
  "NotDataActions": [], \
  "AssignableScopes": ["/subscriptions/<insert your subscription id>"] \
}'
```

## <a name="take-snapshots-manually"></a>Создание моментальных снимков вручную

Перед выполнением каких-либо команд резервного копирования ( `azacsnap -c backup` ) Проверьте конфигурацию, выполнив команды тестирования и убедившись, что они успешно выполнены.  Правильное выполнение этих тестов `azacsnap` может взаимодействовать с установленной базой данных SAP HANA и базовой системой хранения SAP HANA в **крупном экземпляре Azure** или в **Azure NetApp Files** системе.

- `azacsnap -c test --test hana`
- `azacsnap -c test --test storage`

Затем, чтобы создать резервную копию моментального снимка базы данных вручную, выполните следующую команду:

```bash
azacsnap -c backup --volume data --prefix hana_TEST --retention=1
```

## <a name="setup-automatic-snapshot-backup"></a>Настройка автоматического резервного копирования моментальных снимков

Обычно в системах UNIX и Linux используется `cron` для автоматизации выполнения команд в системе. Стандартной практикой для средств создания моментальных снимков является настройка пользователя `crontab` .

Ниже приведен пример, `crontab` `azacsnap` позволяющий пользователю автоматизировать создание моментальных снимков.

```output
MAILTO=""
# =============== TEST snapshot schedule ===============
# Data Volume Snapshots - taken every hour.
@hourly (. /home/azacsnap/.profile ; cd /home/azacsnap/bin ; azacsnap -c backup --volume data --prefix hana_TEST --retention=9)
# Other Volume Snapshots - taken every 5 minutes, excluding the top of the hour when hana snapshots taken
5,10,15,20,25,30,35,40,45,50,55 * * * * (. /home/azacsnap/.profile ; cd /home/azacsnap/bin ; azacsnap -c backup --volume other --prefix logs_TEST --retention=9)
# Other Volume Snapshots - using an alternate config file to snapshot the boot volume daily.
@daily (. /home/azacsnap/.profile ; cd /home/azacsnap/bin ; azacsnap -c backup --volume other --prefix DailyBootVol --retention=7 --configfile boot-vol.json)
```

Объяснение приведенной выше копии crontab.

- `MAILTO=""`: если у вас есть пустое значение, cron автоматически не пытается отправить пользователю сообщение электронной почты при выполнении записи crontab, так как он, скорее всего, будет находиться в файле электронной почты локального пользователя.
- Сокращенные версии времени для записей crontab говорят сами за себя:
  - `@monthly` = Запускать один раз в месяц, то есть "0 0 1 * *".
  - `@weekly`  = Запускать один раз в неделю, то есть "0 0 * * 0".
  - `@daily`   = Запускать один раз в день, то есть "0 0 * *".
  - `@hourly`  = Выполнить один раз в час, то есть "0 * * * *".
- Первые пять столбцов используются для обозначения времени, см. примеры столбцов ниже:
  - `0,15,30,45`: Каждые 15 минут
  - `0-23`: Каждый час
  - `*` : Каждый день
  - `*` : Каждый месяц
  - `*` : Каждый день недели
- Командная строка для выполнения, заключенная в квадратные скобки "()"
  - `. /home/azacsnap/.profile` = извлечь в пользовательском профиле пользователя, чтобы настроить среду, включая $PATH и т. д.
  - `cd /home/azacsnap/bin` = Измените каталог выполнения на расположение "/Хоме/азакснап/бин", где находятся файлы конфигурации.
  - `azacsnap -c .....` — полная команда азакснап для выполнения, включая все параметры.

Дополнительные сведения о cron и формате файла crontab см. здесь: <https://en.wikipedia.org/wiki/Cron>

> [!NOTE]
> Пользователи несут ответственность за мониторинг заданий cron, чтобы гарантировать успешное создание моментальных снимков.

## <a name="monitor-the-snapshots"></a>Мониторинг моментальных снимков

Для обеспечения работоспособности системы следует отслеживать следующие условия.

1. Доступное место на диске. Моментальные снимки медленно потребляют место на диске, так как хранение старых блоков дисков в моментальном снимке будет сохраняться.
    1. Чтобы автоматизировать Управление дисковым пространством, используйте `--retention` `--trim` Параметры и для автоматической очистки старых моментальных снимков и файлов журналов базы данных.
1. Успешное выполнение средств создания моментальных снимков
    1. Проверьте `*.result` файл на предмет успешности или неудачи последнего запуска `azacsnap` .
    1. Проверьте `/var/log/messages` выходные данные `azacsnap` команды.
1. Согласованность моментальных снимков, периодически восстанавливая их в другую систему.

> [!NOTE]
> Чтобы получить список сведений о снимке, выполните команду `azacsnap -c details` .

## <a name="delete-a-snapshot"></a>Удаление моментального снимка

Чтобы удалить моментальный снимок, выполните команду `azacsnap -c delete` . Невозможно удалить моментальные снимки с уровня операционной системы. `azacsnap -c delete`Для удаления моментальных снимков хранилища необходимо использовать правильную команду ().

> [!IMPORTANT]
> Будьте бдительным при удалении моментального снимка. После удаления **невозможно** восстановить удаленные моментальные снимки.

## <a name="restore-a-snapshot"></a>Восстановление моментального снимка

Моментальный снимок тома хранилища можно восстановить на новый том ( `-c restore --restore snaptovol` ).  Для крупных экземпляров Azure том можно восстановить в виде моментального снимка ( `-c restore --restore revertvolume` ).

> [!NOTE]
> Не указана команда **восстановления базы данных** .

Моментальный снимок можно скопировать обратно в область данных SAP HANA, но SAP HANA не должен выполняться при создании копии ( `cp /hana/data/H80/mnt00001/.snapshot/hana_hourly.2020-06-17T113043.1586971Z/*` ).

Для крупных экземпляров Azure можно обратиться в службу Microsoft Operations, открыв запрос на обслуживание, чтобы восстановить нужный моментальный снимок из существующих доступных моментальных снимков. Запрос на обслуживание можно открыть из портал Azure: <https://portal.azure.com>

Если решено выполнить аварийное восстановление после сбоя, `azacsnap -c restore --restore revertvolume` команда на сайте аварийного восстановления автоматически сделает доступными последние ( `/hana/data` и `/hana/logbackups` ) моментальные снимки томов, чтобы разрешить восстановление SAP HANA. Используйте эту команду с осторожностью, так как она прерывает репликацию между рабочими сайтами и узлами аварийного восстановления.

## <a name="set-up-snapshots-for-boot-volumes-only"></a>Настройка моментальных снимков только для загрузочных томов

> [!IMPORTANT]
> Эта операция применяется только к крупному экземпляру Azure.

В некоторых случаях у клиентов уже есть средства для защиты SAP HANA и требуется настроить только моментальные снимки томов загрузки.  В этом случае задача упрощена, и необходимо выполнить следующие действия.

1. Выполните шаги 1-4 предварительных требований для установки.
1. Разрешить обмен данными с хранилищем.
1. Скачайте запустите установщик, чтобы установить средства создания моментальных снимков.
1. Завершите установку средств создания моментальных снимков.
1. Создайте новый файл конфигурации, как показано ниже. Сведения о загрузочном томе должны быть в Осерволуме станза (записи пользователей <span style="color:red">красным цветом</span>):
    ```output
    > <span style="color:red">azacsnap -c configure --configuration new --configfile BootVolume.json</span>
    Building new config file
    Add comment to config file (blank entry to exit adding comments):<span style="color:red">Boot only config file.</span>
    Add comment to config file (blank entry to exit adding comments):
    Add database to config? (y/n) [n]: <span style="color:red">y</span>
    HANA SID (for example, H80): <span style="color:red">X</span>
    HANA Instance Number (for example, 00): <span style="color:red">X</span>
    HANA HDB User Store Key (for example, `hdbuserstore List`): <span style="color:red">X</span>
    HANA Server's Address (hostname or IP address): <span style="color:red">X</span>
    Add ANF Storage to database section? (y/n) [n]:
    Add HLI Storage to database section? (y/n) [n]: <span style="color:red">y</span>
    Add DATA Volume to HLI Storage section of Database section? (y/n) [n]:
    Add OTHER Volume to HLI Storage section of Database section? (y/n) [n]: <span style="color:red">y</span>
    Storage User Name (for example, clbackup25): <span style="color:red">shoasnap</span>
    Storage IP Address (for example, 192.168.1.30): <span style="color:red">10.1.1.10</span>
    Storage Volume Name (for example, hana_data_soldub41_t250_vol): <span style="color:red">t210_sles_boot_azsollabbl20a31_vol</span>
    Add OTHER Volume to HLI Storage section of Database section? (y/n) [n]:
    Add HLI Storage to database section? (y/n) [n]:
    Add database to config? (y/n) [n]:

    Editing configuration complete, writing output to 'BootVolume.json'.
    ```
1. Проверьте файл конфигурации, см. Следующий пример:

    Используйте `cat` команду, чтобы отобразить содержимое файла конфигурации:

    ```bash
    cat BootVolume.json
    ```

    ```output
    {
      "version": "5.0 Preview",
      "logPath": "./logs",
      "securityPath": "./security",
      "comments": [
        "Boot only config file."
      ],
      "database": [
        {
          "hana": {
            "serverAddress": "X",
            "sid": "X",
            "instanceNumber": "X",
            "hdbUserStoreName": "X",
            "savePointAbortWaitSeconds": 600,
            "hliStorage": [
              {
                "dataVolume": [],
                "otherVolume": [
                  {
                    "backupName": "shoasnap",
                    "ipAddress": "10.1.1.10",
                    "volume": "t210_sles_boot_azsollabbl20a31_vol"
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

1. Тестирование резервной копии загрузочного тома

    ```bash
    azacsnap -c backup --volume other --prefix TestBootVolume --retention 1 --configfile BootVolume.json
    ```

1. Установите флажок в списке, обратите внимание на добавление `--snapshotfilter` параметра для ограничения возвращаемого списка моментальных снимков.

    ```bash
    azacsnap -c details --snapshotfilter TestBootVolume --configfile BootVolume.json
    ```

    Выходные данные команды:
    ```output
    List snapshot details called with snapshotFilter 'TestBootVolume'
    #, Volume, Snapshot, Create Time, HANA Backup ID, Snapshot Size
    #1, t210_sles_boot_azsollabbl20a31_vol, TestBootVolume.2020-07-03T034651.7059085Z, "Fri Jul 03 03:48:24 2020", "otherVolume Backup|azacsnap version: 5.0 Preview (20200617.75879)", 200KB
    , t210_sles_boot_azsollabbl20a31_vol, , , Size used by Snapshots, 1.31GB
    ```

1. Настройка автоматического резервного копирования моментальных снимков.

> [!NOTE]
> Настройка связи с SAP HANA не является обязательной.

## <a name="restore-a-boot-snapshot"></a>Восстановление "Boot" моментального снимка

> [!IMPORTANT]
> Эта операция предназначена для можно больших экземпляров Azure.
> Сервер будет восстановлен до момента, когда был сделан моментальный снимок.

Моментальный снимок "Boot" можно восстановить следующим образом:

1. Клиенту необходимо будет завершить работу сервера.
1. После завершения работы сервера клиенту необходимо будет открыть запрос на обслуживание, содержащий идентификатор компьютера и моментальный снимок для восстановления.
    > Клиенты могут открыть запрос на обслуживание из портал Azure: <https://portal.azure.com>
1. Корпорация Майкрософт восстановит LUN операционной системы с помощью указанного идентификатора компьютера и моментального снимка, а затем загрузится сервер.
1. Клиент должен подтвердить загрузку и работоспособность сервера.

Дополнительные действия после восстановления не выполняются.

## <a name="key-facts-to-know-about-snapshots"></a>Ключевые факты, о которых следует помнить о моментальных снимках

Ключевые атрибуты моментальных снимков томов хранилища:

- **Расположение моментальных снимков**. моментальные снимки можно найти в виртуальном каталоге ( `.snapshot` ) в томе.  См. следующие примеры для **крупных экземпляров Azure**:
  - СУБД `/hana/data/<SID>/mnt00001/.snapshot`
  - Используемый `/hana/shared/<SID>/.snapshot`
  - Журналы `/hana/logbackups/<SID>/.snapshot`
  - Загрузка: моментальные снимки загрузки для ХЛИ **не видны** на уровне ОС, но могут быть перечислены с помощью `azacsnap -c details` .

  > [!NOTE]
  >  `.snapshot` — это скрытая *Виртуальная* папка только для чтения, предоставляющая доступ только для чтения к моментальным снимкам.

- **Максимальный размер моментального снимка:** Оборудование может поддерживать до 250 моментальных снимков на том. Команда моментального снимка сохранит максимальное количество моментальных снимков для префикса на основе набора хранения в командной строке и удалит самый старый моментальный снимок, если он выходит за пределы максимального числа.
- **Имя моментального снимка:** Имя моментального снимка включает метку префикса, предоставленную клиентом.
- **Размер моментального снимка:** Зависит от размера и изменений на уровне базы данных.
- **Расположение файла журнала:** Файлы журнала, созданные командами, выводятся в папки в соответствии с определением в файле конфигурации JSON, который по умолчанию является вложенной папкой, в которой выполняется команда (например, `./logs` ).

## <a name="next-steps"></a>Дальнейшие действия

- [Устранение проблем](azacsnap-troubleshoot.md)