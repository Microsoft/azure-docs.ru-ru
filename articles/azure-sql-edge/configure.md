---
title: Настройка SQL Azure для пограничных вычислений
description: Узнайте, как настроить Azure SQL ребро.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 0c49f5ab9f10456c32f7f8516cba0e851fa80e74
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93392339"
---
# <a name="configure-azure-sql-edge"></a>Настройка SQL Azure для пограничных вычислений

SQL Azure для пограничных вычислений поддерживает настройку одним из следующих двух способов:

- Переменные среды
- Файл MSSQL. conf, помещенный в папку/Вар/ОПТ/мсскл

> [!NOTE]
> Задание переменных среды переопределяет параметры, указанные в файле MSSQL. conf.

## <a name="configure-by-using-environment-variables"></a>Настройка с помощью переменных среды

SQL Azure для пограничных вычислений предоставляет несколько различных переменных среды, которые можно использовать для настройки контейнера SQL Azure для пограничных вычислений. Эти переменные среды являются подмножеством доступных для SQL Server на Linux. Дополнительные сведения о переменных среды SQL Server на Linux см. в разделе [переменные среды](/sql/linux/sql-server-linux-configure-environment-variables/).

Следующие новые переменные среды были добавлены в Azure SQL. 

| Переменная среды | Описание | Значения |     
|-----|-----| ---------- | 
| **PlanId** | Указывает номер SKU Azure SQL, который будет использоваться во время инициализации. Эта переменная среды требуется только при развертывании Azure SQL с помощью Azure IoT Edge. | **Асде-Developer-on-IOT-ребр** или **Асде-Premium-on-IOT-ребро** | 
| **MSSQL_TELEMETRY_ENABLED** | Включение и отключение сбора данных об использовании и диагностике. | TRUE или FALSE |  
| **MSSQL_TELEMETRY_DIR** | Задает целевой каталог для файлов аудита сбора данных об использовании и диагностике. | Расположение папки в контейнере границ SQL. Эту папку можно сопоставить с Томом узла с помощью точек подключения или томов данных. | 
| **MSSQL_PACKAGE** | Указывает расположение развертываемого пакета DACPAC или BACPAC. | Папка, файл или URL-адрес SAS, содержащий пакеты DACPAC или BACPAC. Дополнительные сведения см. [в статье Развертывание пакетов DACPAC и пакета BACPAC базы данных SQL в SQL Server](deploy-dacpac.md). |


Следующая переменная среды SQL Server на Linux не поддерживается для Azure SQL. Если этот флаг определен, эта переменная среды будет пропущена во время инициализации контейнера.

| Переменная среды | Описание |
|-----|-----|
| **MSSQL_ENABLE_HADR** | Включите группу доступности. Например, **1** включен, а **0** — отключено. |

> [!IMPORTANT]
> Переменная среды **MSSQL_PID** для SQL Azure для пограничных вычислений допускает в качестве допустимых значений только **Премиум** и **Разработка**. Azure SQL Server не поддерживает инициализацию с помощью ключа продукта.

### <a name="specify-the-environment-variables"></a>Укажите переменные среды

Укажите переменные среды для SQL Server при развертывании службы с помощью [портал Azure](deploy-portal.md). Их можно добавить либо в разделе **переменные среды** в развертывании модуля, либо в составе **параметров создания контейнера**.

Добавьте значения в **переменные среды**.

![Задать с помощью списка переменных среды](media/configure/set-environment-variables.png)

Добавьте значения в **Параметры создания контейнера**.

![Задание с помощью параметров создания контейнера](media/configure/set-environment-variables-using-create-options.png)

> [!NOTE]
> В режиме отключенного развертывания переменные среды можно указать с помощью `-e` `--env` команды или или `--env-file` параметра `docker run` .

## <a name="configure-by-using-an-mssqlconf-file"></a>Настройка с помощью файла MSSQL. conf

Azure SQL Server не включает в себя [программу настройки MSSQL-CONF](/sql/linux/sql-server-linux-configure-mssql-conf/) , например SQL Server на Linux. Необходимо вручную настроить файл MSSQL. conf и поместить его в постоянный диск хранилища, сопоставленный с папкой/Вар/ОПТ/мсскл/в модуле SQL. При развертывании SQL Server из Azure Marketplace это сопоставление указывается в качестве параметра **Mount** в параметрах **создания контейнера**.

```json
    {
        "Mounts": [
          {
            "Type": "volume",
            "Source": "sqlvolume",
            "Target": "/var/opt/mssql"
          }
        ]
      }
    }
```

Следующие новые параметры MSSQL. conf были добавлены для Azure SQL. 

|Параметр|Описание|
|:---|:---|
|**customerfeedback** | Выберите, если SQL Server отправляет отзыв в корпорацию Майкрософт. Дополнительные сведения см. в разделе [Отключение сбора данных об использовании и диагностике](usage-and-diagnostics-data-configuration.md#disable-usage-and-diagnostic-data-collection) .|      
|**userrequestedlocalauditdirectory** | Задает целевой каталог для файлов аудита сбора данных об использовании и диагностике. Дополнительные сведения см. в разделе [локальный аудит использования и сбора диагностических данных](usage-and-diagnostics-data-configuration.md#local-audit-of-usage-and-diagnostic-data-collection) . |        

Следующие параметры MSSQL. conf не применимы к SQL Server.

|Параметр|Описание|
|:---|:---|
|**Отзывы пользователей** | Выберите, если SQL Server отправляет отзыв в корпорацию Майкрософт. |
|**Профиль компонента Database Mail** | Настройка профиля Database Mail по умолчанию для SQL Server на Linux. |
|**Высокая доступность** | Включение групп доступности. |
|**Координатор распределенных транзакций Майкрософт** | Настройка координатора распределенных транзакций Майкрософт на платформе Linux. Дополнительные параметры конфигурации, связанные с распределенными транзакциями, не поддерживаются для SQL Server. Дополнительные сведения об этих дополнительных параметрах конфигурации см. в разделе [Настройка MSDTC](/sql/linux/sql-server-linux-configure-mssql-conf#msdtc). |
|**Лицензионные соглашения служб ML** | Примите лицензионные соглашения R и Python для пакетов Машинное обучение Azure. Применимо только к SQL Server 2019.|
|**outboundnetworkaccess** |Включение исходящего сетевого доступа для расширений R, Python и Java [Службы машинного обучения](/sql/linux/sql-server-linux-setup-machine-learning/).|

Следующий пример файла MSSQL. conf работает для SQL Server. Дополнительные сведения о формате файла MSSQL. conf см. в разделе [Формат MSSQL. conf](/sql/linux/sql-server-linux-configure-mssql-conf#mssql-conf-format).

```ini
[EULA]
accepteula = Y

[coredump]
captureminiandfull = true
coredumptype = full

[filelocation]
defaultbackupdir = /var/opt/mssql/backup/
defaultdatadir = /var/opt/mssql/data/
defaultdumpdir = /var/opt/mssql/data/
defaultlogdir = /var/opt/mssql/log/

[language]
lcid = 1033

[memory]
memorylimitmb = 6144

[sqlagent]
errorlogfile = /var/opt/mssql/log/sqlagentlog.log
errorlogginglevel = 7

[traceflag]
traceflag0 = 3604
traceflag1 = 3605
traceflag2 = 1204
```

## <a name="run-azure-sql-edge-as-non-root-user"></a>Запуск Azure SQL ребр от имени пользователя, не являющегося корневым

По умолчанию контейнеры Azure SQL работают с некорневым пользователем или группой. При развертывании с помощью Azure Marketplace (или с помощью DOCKER Run), если не указан другой пользователь или группа, контейнеры SQL будут запускаться как пользователь MSSQL (не корневой). Чтобы указать другого пользователя, не являющегося корневым, во время развертывания добавьте `*"User": "<name|uid>[:<group|gid>]"*` пару "ключ-значение" в разделе Параметры создания контейнера. В приведенном ниже примере SQL ребр настроено для запуска от имени пользователя `*IoTAdmin*` .

```json
{
    ..
    ..
    ..
    "User": "IoTAdmin",
    "Env": [
        "MSSQL_AGENT_ENABLED=TRUE",
        "ClientTransportType=AMQP_TCP_Only",
        "MSSQL_PID=Premium"
    ]
}
```

Чтобы разрешить пользователю, не являющемуся корневым, доступ к файлам базы данных на подключенных томах, убедитесь, что пользователь или группа, в которой выполняется контейнер, имеет разрешения на чтение & записи в постоянное хранилище файлов. В приведенном ниже примере для пользователя, не являющегося корневым, в качестве владельца файлов устанавливается user_id 10001. 

```bash
chown -R 10001:0 <database file dir>
```

### <a name="upgrading-from-earlier-ctp-releases"></a>Обновление предыдущих выпусков CTP

Ранее CTP Azure SQL ребр настроены для запуска от имени привилегированных пользователей. При обновлении с предыдущей версии CTP доступны следующие параметры.

- Продолжайте использовать привилегированного пользователя. чтобы продолжить использование привилегированного пользователя, добавьте `*"User": "0:0"*` пару "ключ-значение" в разделе Параметры создания контейнера.
- Использование пользователя MSSQL по умолчанию для использования пользователя MSSQL по умолчанию выполните следующие действия.
  - Добавьте пользователя с именем MSSQL на узле DOCKER. В приведенном ниже примере мы добавим пользователя MSSQL с ИДЕНТИФИКАТОРом 10001. Этот пользователь также добавляется в корневую группу.
    ```bash
    sudo useradd -M -s /bin/bash -u 10001 -g 0 mssql
    ```
  - Изменение разрешения на том каталога или подключения, где находится файл базы данных 
    ```bash
    sudo chgrp -R 0 /var/lib/docker/volumes/kafka_sqldata/
    sudo chmod -R g=u /var/lib/docker/volumes/kafka_sqldata/
    ```
- Использовать другую учетную запись пользователя, не являющейся корневой, для использования другой некорневой учетной записи пользователя
  - Обновите параметры создания контейнера, чтобы указать `*"User": "user_name | user_id*` пару "ключ-значение" в разделе Параметры создания контейнера. Замените user_name или user_id на фактическое user_name или user_id от узла DOCKER. 
  - Измените разрешения для тома каталога или подключения.

## <a name="persist-your-data"></a> Сохранение данных

Изменения конфигурации SQL Azure и файлы базы данных сохраняются в контейнере, даже если контейнер перезапускается с помощью `docker stop` и `docker start` . Однако если удалить контейнер с помощью `docker rm` , все в контейнере будет удалено, включая Azure SQL Server и базы данных. В следующем разделе описывается, как можно использовать **тома данных** для сохранения файлов базы данных даже в случае удаления связанных контейнеров.

> [!IMPORTANT]
> Для Azure SQL ребро важно понимать, как сохранять данные в DOCKER. Помимо этого раздела, мы также рекомендуем вам ознакомиться с информацией об [управлении данными в контейнерах Docker](https://docs.docker.com/engine/tutorials/dockervolumes/) в документации по Docker.

### <a name="mount-a-host-directory-as-data-volume"></a>Подключение каталога узла в качестве тома данных

Первый способ состоит в подключении каталога на вашем узле в качестве тома данных для контейнера. Для этого используйте команду `docker run` с флагом `-v <host directory>:/var/opt/mssql`. Такой подход позволяет восстанавливать данные в перерывах между выполнениями контейнера.

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -v <host directory>/data:/var/opt/mssql/data -v <host directory>/log:/var/opt/mssql/log -v <host directory>/secrets:/var/opt/mssql/secrets -d mcr.microsoft.com/azure-sql-edge
```

```PowerShell
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>" -p 1433:1433 -v <host directory>/data:/var/opt/mssql/data -v <host directory>/log:/var/opt/mssql/log -v <host directory>/secrets:/var/opt/mssql/secrets -d mcr.microsoft.com/azure-sql-edge
```

Кроме того, этот способ позволяет предоставлять общий доступ к файлам на узле и просматривать их за пределами Docker.

> [!IMPORTANT]
> Сопоставление томов узла для **Docker в Windows** в настоящее время не поддерживает сопоставление полного каталога `/var/opt/mssql`. Однако можно сопоставить подкаталог, например `/var/opt/mssql/data`, с хост-компьютером.

> [!IMPORTANT]
> Сопоставление томов узла для **DOCKER на Mac** с помощью образа Azure SQL не поддерживается. Вместо этого следует использовать контейнеры томов данных. Это ограничение относится только к каталогу `/var/opt/mssql`. Операции чтения из подключенного каталога осуществляются в нормальном режиме. Например, вы можете подключить каталог узла с помощью команды -v на Mac и восстановить резервную копию из файла с расширением BAK, который находится на узле.

### <a name="use-data-volume-containers"></a>Использование контейнеров томов данных

Второй способ подразумевает использование контейнеров томов данных. Чтобы создать контейнер тома данных, укажите имя тома вместо каталога узла с параметром `-v`. В следующем примере создается общий том данных с именем **sqlvolume**.

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -v sqlvolume:/var/opt/mssql -d mcr.microsoft.com/azure-sql-edge
```

```PowerShell
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>" -p 1433:1433 -v sqlvolume:/var/opt/mssql -d mcr.microsoft.com/azure-sql-edge
```

> [!NOTE]
> Этот способ неявного создания тома данных в рамках команды выполнения не работает в старых версиях Docker. В таком случае следует явно выполнить действия, которые описываются в разделе [Создание и подключение контейнера тома данных](https://docs.docker.com/engine/tutorials/dockervolumes/#creating-and-mounting-a-data-volume-container) документации по Docker.

Даже если вы остановите и удалите этот контейнер, том данных будет сохранен. Вы сможете просмотреть его с помощью команды `docker volume ls`.

```bash
docker volume ls
```

Если затем создать другой контейнер с тем же именем тома, новый контейнер будет использовать те же данные Azure SQL, которые содержатся в томе.

Чтобы удалить контейнер тома данных, воспользуйтесь командой `docker volume rm`.

> [!WARNING]
> Если удалить контейнер томов данных, все данные Azure SQL в контейнере будут удалены *без возможности восстановления* .


## <a name="next-steps"></a>Дальнейшие действия

- [Подключение к службе SQL Azure для пограничных вычислений](connect.md)
- [Создание комплексного решения для Интернета вещей с помощью SQL для пограничных вычислений](tutorial-deploy-azure-resources.md)