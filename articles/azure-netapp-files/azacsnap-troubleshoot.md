---
title: Устранение неполадок инструмента для создания моментальных снимков с помощью приложений Azure для Azure NetApp Files | Документация Майкрософт
description: Содержит сведения об устранении неполадок с использованием инструмента для создания моментальных снимков, совместимых с приложениями Azure, которые можно использовать с Azure NetApp Files.
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
ms.topic: troubleshooting
ms.date: 12/14/2020
ms.author: phjensen
ms.openlocfilehash: 903cb3323b9441ec8bb382054f065760875e3e89
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97632831"
---
# <a name="troubleshoot-azure-application-consistent-snapshot-tool-preview"></a>Устранение неполадок инструмента моментальных снимков для приложений Azure (Предварительная версия)

В этой статье содержатся сведения об устранении неполадок с использованием инструмента для создания моментальных снимков, совместимых с приложениями Azure, которые можно использовать с Azure NetApp Files.

Ниже приведены распространенные проблемы, которые могут возникнуть при выполнении команд. Выполните инструкции по устранению проблемы, описанные в этой статье. Если по-прежнему возникают проблемы, откройте запрос на обслуживание из портал Azure и назначьте запрос в очередь SAP HANA крупные экземпляры, чтобы служба поддержки Майкрософт ответить.

## <a name="log-files"></a>Файлы журнала

Одним из лучших источников информации для отладки любых ошибок с Азакснап являются файлы журналов.  

### <a name="log-file-location"></a>Расположение файла журнала

Файлы журнала хранятся в каталоге, настроенном согласно `logPath` параметру в файле конфигурации азакснап.  Имя файла конфигурации по умолчанию — `azacsnap.json` и значение по умолчанию для `logPath` — это означает, что `"./logs"` файлы журнала записываются в `./logs` каталог относительно места `azacsnap` выполнения команды.  Создание `logPath` абсолютного расположения (например, `/home/azacsnap/logs` ) обеспечит `azacsnap` вывод журналов в `/home/azacsnap/logs` независимо от того, где `azacsnap` была выполнена команда.

### <a name="log-file-naming"></a>Именование файлов журнала

Имя файла журнала основано на имени приложения (например, `azacsnap` ), используемом параметре команды () (например,,, и `-c` `backup` `test` `details` т. д.) и имени файла конфигурации (например, Default = `azacsnap.json` ).  Поэтому при использовании `-c backup` команды имя файла журнала по умолчанию будет `azacsnap-backup-azacsnap.log` и записывается в каталог, настроенный `logPath` .  

Это соглашение об именовании было установлено для предоставления нескольких файлов конфигурации, по одному на одну базу данных и для облегчения поиска связанных журналов.  Таким образом, если имя файла конфигурации — `SID.json` , то при использовании `azacsnap -c backup --configfile SID.json` параметров будет использоваться имя файла результата `azacsnap-backup-SID.log` .

### <a name="result-file-and-syslog"></a>Файл результатов и системный журнал

Для `-c backup` параметра команды азакснап записывает данные в `*.result` файл и системный журнал ( `/var/log/messages` ) с помощью `logger` команды.  Имя файла совпадает с `*.result` базовым именем [файла журнала](#log-file-naming) и размещается в том же [расположении, что и файл журнала](#log-file-location).  Это простой однострочный выходной файл для следующих примеров.

Пример выходных данных из `*.result` файла.
```output
Database # 1 (PR1) : completed ok
```

Пример выходных данных из `/var/log/messages` файла.
```output
Dec 17 09:01:13 azacsnap-rhel azacsnap: Database # 1 (PR1) : completed ok
```

## <a name="failed-communication-with-sap-hana"></a>Сбой связи с SAP HANA

При проверке связи с SAP HANA путем запуска теста с `azacsnap -c test --test hana` и выдается следующая ошибка:

```output
> azacsnap -c test --test hana
BEGIN : Test process started for 'hana'
BEGIN : SAP HANA tests
CRITICAL: Command 'test' failed with error:
Cannot get SAP HANA version, exiting with error: 127
```

**Решение:**

1. Проверьте файл конфигурации (например, `azacsnap.json` ) для каждого экземпляра Hana, чтобы убедиться в правильности значений SAP HANA базы данных.
1. Попробуйте выполнить приведенную ниже команду, чтобы убедиться, что `hdbsql` команда находится в пути и может подключиться к серверу SAP HANA. В следующем примере показана правильная работа команды и ее выходные данные.

    ```bash
    hdbsql -n 172.18.18.50 - i 00 -d SYSTEMDB -U AZACSNAP "\s"
    ```

    ```output
    host          : 172.18.18.50
    sid           : H80
    dbname        : SYSTEMDB
    user          : AZACSNAP
    kernel version: 2.00.040.00.1553674765
    SQLDBC version:        libSQLDBCHDB 2.04.126.1551801496
    autocommit    : ON
    locale        : en_US.UTF-8
    input encoding: UTF8
    sql port      : saphana1:30013
    ```

    В этом примере `hdbsql` команда отсутствует в списке пользователей `$PATH` .

    ```bash
    hdbsql -n 172.18.18.50 - i 00 -U SCADMIN "select version from sys.m_database"
    ```

    ```output
    If 'hdbsql' is not a typo you can use command-not-found to lookup the package that contains it, like this:
    cnf hdbsql
    ```

    В этом примере `hdbsql` команда временно добавляется к пользователю `$PATH` , но при запуске показывается, что ключ подключения не был правильно настроен с помощью `hdbuserstore Set` команды (Дополнительные сведения см. в руководстве по начало работы).

    ```bash
    export PATH=$PATH:/hana/shared/H80/exe/linuxx86_64/hdb/
    ```

    ```bash
    hdbsql -n 172.18.18.50 -i 00 -U SCADMIN "select version from sys.m_database"
    ```

    ```output
    * -10104: Invalid value for KEY (SCADMIN)
    ```

    > [!NOTE]
    > Для окончательного добавления в пользователь `$PATH` обновите файл пользователя. `$HOME/.profile`

## <a name="the-hdbuserstore-location"></a>`hdbuserstore`Расположение

При настройке связи с SAP HANA `hdbuserstore` программа используется для создания параметров безопасного подключения.  `hdbuserstore`Обычно эта программа находится в папке `/usr/sap/<SID>/SYS/exe/hdb/` или `/usr/sap/hdbclient` .  Обычно установщик добавляет правильное расположение в папку `azacsnap` пользователя `$PATH` .

## <a name="failed-test-with-storage"></a>Сбой теста с хранилищем

Команда `azacsnap -c test --test storage` не выполнена.

### <a name="azure-large-instance"></a>Крупные экземпляры Azure

Следующий пример демонстрирует выполнение `azacsnap` на SAP HANA в крупном экземпляре Azure:

```bash
azacsnap -c test --test storage
```

```output
The authenticity of host '172.18.18.11 (172.18.18.11)' can't be established.
ECDSA key fingerprint is SHA256:QxamHRn3ZKbJAKnEimQpVVCknDSO9uB4c9Qd8komDec.
Are you sure you want to continue connecting (yes/no)?
```

**Решение:** Описанная выше ошибка обычно появляется, когда пользователь хранилища крупных экземпляров Azure не имеет доступа к базовому хранилищу. Чтобы проверить доступ к хранилищу с помощью указанного пользователя хранилища, выполните `ssh` команду, чтобы проверить связь с платформой хранения.

```bash
ssh <StorageBackupname>@<Storage IP address> "volume show -fields volume"
```

Пример с ожидаемыми выходными данными:

```bash
ssh clt1h80backup@10.8.0.16 "volume show -fields volume"
```

```output
vserver volume
--------------------------------- ------------------------------
osa33-hana-c01v250-client25-nprod hana_data_h80_mnt00001_t020_vol
osa33-hana-c01v250-client25-nprod hana_data_h80_mnt00002_t020_vol
```

#### <a name="the-authenticity-of-host-172181811-172181811-cant-be-established"></a>Не удается установить подлинность узла "172.18.18.11 (172.18.18.11)"

```bash
azacsnap -c test --test storage
```

```output
BEGIN : Test process started for 'storage'
BEGIN : Storage test snapshots on 'data' volumes
BEGIN : 1 task(s) to Test Snapshots for Storage Volume Type 'data'
The authenticity of host '10.3.0.18 (10.3.0.18)' can't be established.
ECDSA key fingerprint is SHA256:cONAr0lpafb7gY4l31AdWTzM3s9LnKDtpMdPA+cxT7Y.
Are you sure you want to continue connecting (yes/no)?
```

**Решение:** Не выбирайте Да. Убедитесь, что ваш IP-адрес хранилища указан правильно. Если по-прежнему возникают проблемы, проверьте IP-адрес хранилища с помощью Microsoft Operations Team.

### <a name="azure-netapp-files"></a>Azure NetApp Files

Следующий пример демонстрирует выполнение `azacsnap` на виртуальной машине с помощью Azure NetApp Files:

```bash
azacsnap --configfile azacsnap.json.NOT-WORKING -c test --test storage
```

```output
BEGIN : Test process started for 'storage'
BEGIN : Storage test snapshots on 'data' volumes
BEGIN : 1 task(s) to Test Snapshots for Storage Volume Type 'data'
ERROR: Could not create StorageANF object [authFile = 'azureauth.json']
```

**Решение:**

1. Проверьте наличие файла субъекта-службы, `azureauth.json` как указано в `azacsnap.json` файле конфигурации.
1. Проверьте файл журнала (например,), `logs/azacsnap-test-azacsnap.log` чтобы узнать, имеет ли субъект-служба ( `azureauth.json` ) правильное содержимое.  Пример из журнала следующим образом:

      ```output
      [19/Nov/2020:18:39:49 +13:00] DEBUG: [PID:0020080:StorageANF:659] [1] Innerexception: Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException AADSTS7000215: Invalid client secret is provided.
      ```

1. Проверьте файл журнала (например,), `logs/azacsnap-test-azacsnap.log` чтобы узнать, истек ли срок действия субъекта-службы ( `azureauth.json` ). Пример из журнала следующим образом:

      ```output
      [19/Nov/2020:18:41:10 +13:00] DEBUG: [PID:0020257:StorageANF:659] [1] Innerexception: Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException AADSTS7000222: The provided client secret keys are expired. Visit the Azure Portal to create new keys for your app, or consider using certificate credentials for added security: https://docs.microsoft.com/azure/active-directory/develop/active-directory-certificate-credentials
      ```

## <a name="next-steps"></a>Дальнейшие действия

- [Советы](azacsnap-tips.md)
