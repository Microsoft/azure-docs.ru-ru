---
title: Установка инструмента для создания моментальных снимков с единым приложением Azure для Azure NetApp Files | Документация Майкрософт
description: Содержит инструкции по установке инструмента для создания моментальных снимков, совместимых с приложениями Azure, которые можно использовать с Azure NetApp Files.
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
ms.openlocfilehash: 00aaa5bdc0d48adb735679fc4a71b3431970ef09
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98737173"
---
# <a name="install-azure-application-consistent-snapshot-tool-preview"></a>Установить средство создания моментальных снимков для приложений Azure (Предварительная версия)

В этой статье представлено руководство по установке средства создания моментальных снимков с единым приложением Azure, которое можно использовать с Azure NetApp Files.

## <a name="introduction"></a>Введение

Загружаемый самоустановщик разработан для упрощения настройки и запуска средств создания моментальных снимков с непривилегированными привилегиями пользователей (например, азакснап). Установщик настроит пользователя и поместит средства создания моментальных снимков в подкаталог Users `$HOME/bin` (Default = `/home/azacsnap/bin` ).
Самостоятельный установщик пытается определить правильные параметры и пути для всех файлов в зависимости от конфигурации пользователя, выполняющего установку (например, root). Если предварительные действия (Включение связи с хранилищем и SAP HANA) выполнялись от имени root, то при установке будет скопирован закрытый ключ и `hdbuserstore` в расположение пользователя резервной копии. Тем не менее можно выполнить действия, обеспечивающие взаимодействие с серверной части хранилища и SAP HANA вручную администратором после установки.

## <a name="prerequisites-for-installation"></a>Необходимые условия для установки

Следуйте указаниям по настройке и выполнению команд моментальных снимков и аварийного восстановления. Перед установкой и использованием средств создания моментальных снимков рекомендуется выполнить следующие действия.

1. **Обновление ОС**: см. статью Установка ИСПРАВЛЕНИЙ и SMT в [статье как установить и настроить SAP HANA (крупные экземпляры) в Azure](../virtual-machines/workloads/sap/hana-installation.md#operating-system).
1. **Синхронизация времени настроена**. Клиент должен будет предоставить NTP-совместимый сервер времени и соответствующим образом настроить ОС.
1. **Hana установлен** . см. инструкции по установке Hana в [установке SAP NetWeaver в базе данных Hana](/archive/blogs/saponsqlserver/sap-netweaver-installation-on-hana-database).
1. **[Включение связи с хранилищем](#enable-communication-with-storage)** (Дополнительные сведения см. в отдельном разделе): Клиент должен настроить SSH с парой закрытых и открытых ключей и предоставить открытый ключ для каждого узла, на котором запланировано выполнение средств создания моментальных снимков в Microsoft Operations для установки в серверной части хранилища.
   1. **Для Azure NetApp Files (Дополнительные сведения см. в отдельном разделе)**: Клиент должен создать файл проверки подлинности субъекта-службы.
   1. **Для крупных экземпляров Azure (Дополнительные сведения см. в отдельном разделе)**: Клиент должен настроить SSH с парой закрытых и открытых ключей и предоставить открытый ключ для каждого узла, на котором запланировано выполнение средств создания моментальных снимков в Microsoft Operations для установки в серверной части хранилища.

      Проверьте это с помощью SSH для подключения к одному из узлов (например, `ssh -l <Storage UserName> <Storage IP Address>` ).
      Введите `exit` , чтобы выйти из командной строки хранилища.

      Microsoft Operations предоставит пользователю хранилища и IP-адрес хранилища во время подготовки.
  
1. **[Включить связь с SAP HANA](#enable-communication-with-sap-hana)** (Дополнительные сведения см. в отдельном разделе): Клиент должен настроить соответствующего пользователя SAP HANA с необходимыми привилегиями для выполнения моментального снимка.
   1. Этот параметр можно проверить из командной строки, как показано ниже, используя текст в `grey`
      1. HANAv1

            `hdbsql -n <HANA IP address> -i <HANA instance> -U <HANA user> "\s"`

      1. HANAv2

            `hdbsql -n <HANA IP address> -i <HANA instance> -d SYSTEMDB -U <HANA user> "\s"`

      - Приведенные выше примеры предназначены для обмена данными не по протоколу SSL с SAP HANA.

## <a name="enable-communication-with-storage"></a>Включение взаимодействия с хранилищем

В этом разделе объясняется, как включить связь с хранилищем.

### <a name="azure-netapp-files"></a>Azure NetApp Files

Создание субъекта-службы RBAC

1. В Azure Cloud Shell сеансе убедитесь, что вы вошли в подписку, в которой вы хотите связать с субъектом-службой по умолчанию:

    ```azurecli-interactive
    az account show
    ```

1. Если подписка неверна, используйте

    ```azurecli-interactive
    az account set -s <subscription name or id>
    ```

1. Создайте субъект-службу, используя Azure CLI в следующем примере.

    ```azurecli-interactive
    az ad sp create-for-rbac --sdk-auth
    ```

    1. При этом необходимо создать выходные данные, как в следующем примере:

        ```output
        {
          "clientId": "00aa000a-aaaa-0000-00a0-00aa000aaa0a",
          "clientSecret": "00aa000a-aaaa-0000-00a0-00aa000aaa0a",
          "subscriptionId": "00aa000a-aaaa-0000-00a0-00aa000aaa0a",
          "tenantId": "00aa000a-aaaa-0000-00a0-00aa000aaa0a",
          "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
          "resourceManagerEndpointUrl": "https://management.azure.com/",
          "activeDirectoryGraphResourceId": "https://graph.windows.net/",
          "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
          "galleryEndpointUrl": "https://gallery.azure.com/",
          "managementEndpointUrl": "https://management.core.windows.net/"
        }
        ```

    > Эта команда автоматически назначает роль участника RBAC субъекту-службе на уровне подписки. Вы можете ограничить область до определенной группы ресурсов, в которой тесты будут создавать ресурсы.

1. Вырежьте и вставьте выходное содержимое в файл `azureauth.json` с именем, сохраненный в той же системе, что `azacsnap` и команда, и обеспечьте защиту файла с соответствующими разрешениями системы.

### <a name="azure-large-instance"></a>Крупные экземпляры Azure

Связь с серверной части хранилища выполняется через зашифрованный канал SSH. В следующих примерах приведены инструкции по настройке SSH для этого обмена данными.

1. Изменение `/etc/ssh/ssh_config` файла

    См. следующие выходные данные, в которых `MACs hmac-sha1` добавлена строка:

    ```output
    # RhostsRSAAuthentication no
    # RSAAuthentication yes
    # PasswordAuthentication yes
    # HostbasedAuthentication no
    # GSSAPIAuthentication no
    # GSSAPIDelegateCredentials no
    # GSSAPIKeyExchange no
    # GSSAPITrustDNS no
    # BatchMode no
    # CheckHostIP yes
    # AddressFamily any
    # ConnectTimeout 0
    # StrictHostKeyChecking ask
    # IdentityFile ~/.ssh/identity
    # IdentityFile ~/.ssh/id_rsa
    # IdentityFile ~/.ssh/id_dsa
    # Port 22
    Protocol 2
    # Cipher 3des
    # Ciphers aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-cbc,3des-
    cbc
    # MACs hmac-md5,hmac-sha1,umac-64@openssh.com,hmac-ripemd
    MACs hmac-sha
    # EscapeChar ~
    # Tunnel no
    # TunnelDevice any:any
    # PermitLocalCommand no
    # VisualHostKey no
    # ProxyCommand ssh -q -W %h:%p gateway.example.com
    ```

1. Создание пары закрытых и открытых ключей

    Используя следующий пример команды для создания пары ключей, не вводите пароль при создании ключа.

    ```bash
    ssh-keygen -t rsa –b 5120 -C ""
    ```

1. Отправка открытого ключа в Microsoft Operations

    Отправьте выходные данные `cat /root/.ssh/id_rsa.pub` команды (пример ниже) в Microsoft Operations, чтобы обеспечить взаимодействие средств моментальных снимков с подсистемой хранения.

    ```bash
    cat /root/.ssh/id_rsa.pub
    ```

    ```output
    ssh-rsa
    AAAAB3NzaC1yc2EAAAADAQABAAABAQDoaRCgwn1Ll31NyDZy0UsOCKcc9nu2qdAPHdCzleiTWISvPW
    FzIFxz8iOaxpeTshH7GRonGs9HNtRkkz6mpK7pCGNJdxS4wJC9MZdXNt+JhuT23NajrTEnt1jXiVFH
    bh3jD7LjJGMb4GNvqeiBExyBDA2pXdlednOaE4dtiZ1N03Bc/J4TNuNhhQbdsIWZsqKt9OPUuTfD
    j0XvwUTLQbR4peGNfN1/cefcLxDlAgI+TmKdfgnLXIsSfbacXoTbqyBRwCi7p+bJnJD07zSc9YCZJa
    wKGAIilSg7s6Bq/2lAPDN1TqwIF8wQhAg2C7yeZHyE/ckaw/eQYuJtN+RNBD
    ```

## <a name="enable-communication-with-sap-hana"></a>Включение взаимодействия с SAP HANA

Средства моментальных снимков взаимодействуют с SAP HANA и требуют пользователя с соответствующими разрешениями для инициации и освобождения точки сохранения базы данных. В следующем примере показано, как настроить пользователя SAP HANA v2 и `hdbuserstore` для взаимодействия с SAP HANA базой данных.

В приведенных ниже примерах команд настраивается пользователь (АЗАКСНАП) в СИСТЕМДБ на SAP HANA 2.
база данных, измените IP-адрес, имена пользователей и пароли, как нужно:

1. Подключение к СИСТЕМДБ для создания пользователя

    ```bash
    hdbsql -n <IP_address_of_host>:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD>
    ```

    ```output
    Welcome to the SAP HANA Database interactive terminal.

    Type: \h for help with commands
    \q to quit

    hdbsql SYSTEMDB=>
    ```

1. Создание пользователя

    В этом примере создается пользователь АЗАКСНАП в СИСТЕМДБ.

    ```sql
    hdbsql SYSTEMDB=> CREATE USER AZACSNAP PASSWORD <AZACSNAP_PASSWORD_CHANGE_ME> NO FORCE_FIRST_PASSWORD_CHANGE;
    ```

1. Предоставление разрешений пользователю

    В этом примере задается разрешение для пользователя АЗАКСНАП, разрешающее выполнение моментального снимка хранилища, целостного в базе данных.

    ```sql
    hdbsql SYSTEMDB=> GRANT BACKUP ADMIN, CATALOG READ, MONITORING TO AZACSNAP;
    ```

1. *Необязательно* — запретить истечение срока действия пароля пользователя

    > [!NOTE]
    > Перед внесением этого изменения проверьте корпоративную политику.

   В этом примере отключается истечение срока действия пароля для пользователя АЗАКСНАП без этого изменения срок действия пароля пользователя истечет, что приведет к правильному выполнению моментальных снимков.  

   ```sql
   hdbsql SYSTEMDB=> ALTER USER AZACSNAP DISABLE PASSWORD LIFETIME;
   ```

1. Настройка SAP HANA безопасного хранилища пользователей (изменение пароля). в этом примере `hdbuserstore` команда из оболочки Linux используется для настройки SAP HANA безопасного хранилища пользователей.

    ```bash
    hdbuserstore Set AZACSNAP <IP_address_of_host>:30013 AZACSNAP <AZACSNAP_PASSWORD_CHANGE_ME>
    ```

1. Проверьте SAP HANA безопасное хранилище пользователей, чтобы проверить, правильно ли настроено безопасное хранилище пользователей. Используйте команду, `hdbuserstore` чтобы вывести результат, аналогичный приведенному ниже примеру. Дополнительные сведения об использовании доступны `hdbuserstore` на веб-сайте SAP.

    ```bash
    hdbuserstore List
    ```

    ```output
    DATA FILE : /home/azacsnap/.hdb/sapprdhdb80/SSFS_HDB.DAT
    KEY FILE : /home/azacsnap/.hdb/sapprdhdb80/SSFS_HDB.KEY

    KEY AZACSNAP
    ENV : <IP_address_of_host>:
    USER: AZACSNAP
    ```

### <a name="additional-instructions-for-using-the-log-trimmer-sap-hana-20-and-later"></a>Дополнительные инструкции по использованию средства усечения журнала (SAP HANA 2,0 и более поздних версий)

При использовании средства усечения журнала, приведенный ниже пример команды настроили пользователя (АЗАКСНАП) в базах данных клиента в системе базы данных SAP HANA 2,0. Не забудьте изменить IP-адрес, имена пользователей и пароли соответствующим образом:

1. Подключитесь к базе данных клиента, чтобы создать пользователя, сведения о конкретном клиенте: `<IP_address_of_host>` и `<SYSTEM_USER_PASSWORD>` .  Кроме того, обратите внимание на порт ( `30015` ), необходимый для взаимодействия с базой данных клиента.

    ```bash
    hdbsql -n <IP_address_of_host>:30015 - i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD>
    ```

    ```output  
    Welcome to the SAP HANA Database interactive terminal.

    Type: \h for help with commands
    \q to quit

    hdbsql TENANTDB=>
    ```

1. Создание пользователя

    В этом примере создается пользователь АЗАКСНАП в СИСТЕМДБ.

    ```sql
    hdbsql TENANTDB=> CREATE USER AZACSNAP PASSWORD <AZACSNAP_PASSWORD_CHANGE_ME> NO FORCE_FIRST_PASSWORD_CHANGE;
    ```

1. Предоставление разрешений пользователю

    В этом примере задается разрешение для пользователя АЗАКСНАП, разрешающее выполнение моментального снимка хранилища, целостного в базе данных.

    ```sql
    hdbsql TENANTDB=> GRANT BACKUP ADMIN, CATALOG READ, MONITORING TO AZACSNAP;
    ```

1. *Необязательно* — запретить истечение срока действия пароля пользователя

    > [!NOTE]
    > Перед внесением этого изменения проверьте корпоративную политику.

   В этом примере отключается истечение срока действия пароля для пользователя АЗАКСНАП без этого изменения срок действия пароля пользователя истечет, что приведет к правильному выполнению моментальных снимков.  

   ```sql
   hdbsql TENANTDB=> ALTER USER AZACSNAP DISABLE PASSWORD LIFETIME;
   ```

> [!NOTE]  
> Повторите эти действия для всех баз данных клиента. Сведения о подключении для всех клиентов можно получить с помощью следующего SQL запроса к СИСТЕМДБ.

```sql
SELECT HOST, SQL_PORT, DATABASE_NAME FROM SYS_DATABASES.M_SERVICES WHERE SQL_PORT LIKE '3%'
```

См. Следующий пример запроса и выходных данных.

```bash
hdbsql -jaxC -n 10.90.0.31:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD> " SELECT HOST,SQL_PORT, DATABASE_NAME FROM SYS_DATABASES.M_SERVICES WHERE SQL_PORT LIKE '3%' "
```

```output
sapprdhdb80,30013,SYSTEMDB
sapprdhdb80,30015,H81
sapprdhdb80,30041,H82
```

### <a name="using-ssl-for-communication-with-sap-hana"></a>Использование SSL для связи с SAP HANA

`azacsnap`Средство использует `hdbsql` команду SAP HANA для взаимодействия с SAP HANA. Сюда входит использование параметров SSL при шифровании связи с SAP HANA. `azacsnap` использует `hdbsql` Параметры SSL команды, как показано ниже.

При использовании параметра всегда используются следующие `azacsnap --ssl` Параметры:

- `-e` — Включает TLS Енкриптионтлс/SSL-шифрование. Сервер выбирает самый высокий доступный.
- `-ssltrustcert` — Указывает, следует ли проверять сертификат сервера.
- `-sslhostnameincert "*"` — Указывает имя узла, используемое для проверки удостоверения сервера. Указывая в `"*"` качестве имени узла, имя узла сервера не проверяется.

Для связи SSL также требуются файлы хранилища ключей и хранилища доверия.  Хотя эти файлы можно хранить в расположениях по умолчанию в установке Linux, чтобы обеспечить использование правильного материала ключа для различных SAP HANA систем (т. е. в случаях, когда для каждой системы SAP HANA используются разные файлы хранилища ключей и доверия), `azacsnap` ожидает, что файлы хранилища ключей и хранилища доверия хранятся в `securityPath` расположении, как указано в `azacsnap` файле конфигурации.

#### <a name="key-store-files"></a>Файлы хранилища ключей

- При использовании нескольких идентификаторов безопасности с одним и тем же материалом ключа проще создать ссылки на Секуритипас расположение, как определено в `azacsnap` файле конфигурации.  Убедитесь, что эти значения существуют для каждого идентификатора безопасности с помощью SSL.
  - Для OpenSSL:
    - `ln $HOME/.ssl/key.pem <securityPath>/<SID>_keystore`
  - Для коммонкрипто:
    - `ln $SECUDIR/sapcli.pse <securityPath>/<SID>_keystore`
- При использовании нескольких идентификаторов безопасности с различными материалами ключа на ИД безопасности скопируйте (или переместите и переименуйте) файлы в расположение Секуритипас, как определено в `azacsnap` файле конфигурации SID.
  - Для OpenSSL:
    - `mv key.pem <securityPath>/<SID>_keystore`
  - Для коммонкрипто:
    - `mv sapcli.pse <securityPath>/<SID>_keystore`

При `azacsnap` вызовах `hdbsql` он добавляется `-sslkeystore=<securityPath>/<SID>_keystore` в командную строку.

#### <a name="trust-store-files"></a>Доверять файлам хранилища

- При использовании нескольких идентификаторов SID с одинаковым материалом ключа создайте жесткие связи в расположении Секуритипас, как определено в `azacsnap` файле конфигурации.  Убедитесь, что эти значения существуют для каждого идентификатора безопасности с помощью SSL.
  - Для OpenSSL:
    - `ln $HOME/.ssl/trust.pem <securityPath>/<SID>_truststore`
  - Для коммонкрипто:
    - `ln $SECUDIR/sapcli.pse <securityPath>/<SID>_truststore`
- При использовании нескольких идентификаторов безопасности с различными материалами ключа на ИД безопасности скопируйте (или переместите и переименуйте) файлы в расположение Секуритипас, как определено в `azacsnap` файле конфигурации SID.
  - Для OpenSSL:
    - `mv trust.pem <securityPath>/<SID>_truststore`
  - Для коммонкрипто:
    - `mv sapcli.pse <securityPath>/<SID>_truststore`

> [!NOTE]
> `<SID>`Компонент имен файлов должен быть SAP HANA идентификатором системы в верхнем регистре (например,, и `H80` `PR1` т. д.).

При `azacsnap` вызовах `hdbsql` он добавляется `-ssltruststore=<securityPath>/<SID>_truststore` в командную строку.

Таким образом, `azacsnap -c test --test hana --ssl openssl` Если приложение `SID` находится `H80` в файле конфигурации, оно будет выполнять `hdbsql` подключения следующим образом:

```bash
hdbsql \
    -e \
    -ssltrustcert \
    -sslhostnameincert "*" \
    -sslprovider openssl \
    -sslkeystore ./security/H80_keystore \
    -ssltruststore ./security/H80_truststore
    "sql statement"
```

> [!NOTE]
> `\`Символ — это Командная строка, позволяющая улучшить ясность нескольких параметров, передаваемых в командной строке.

## <a name="installing-the-snapshot-tools"></a>Установка средств создания моментальных снимков

Загружаемый самоустановщик разработан для упрощения настройки и запуска средств создания моментальных снимков с непривилегированными привилегиями пользователей (например, азакснап). Установщик настроит пользователя и поместит средства создания моментальных снимков в подкаталог Users `$HOME/bin` (Default = `/home/azacsnap/bin` ).

Самостоятельный установщик пытается определить правильные параметры и пути для всех файлов в зависимости от конфигурации пользователя, выполняющего установку (например, root). Если предыдущие шаги настройки (Включение связи с хранилищем и SAP HANA) выполнялись от имени root, то при установке будет скопирован закрытый ключ и в `hdbuserstore` расположение пользователя резервной копии. Тем не менее можно выполнить шаги, которые обеспечивают взаимодействие с серверной части хранилища и SAP HANA вручную администратором после установки.

> [!NOTE]
> Для более ранних SAP HANA при установке крупных экземпляров Azure каталог предварительно установленных средств создания моментальных снимков был `/hana/shared/<SID>/exe/linuxx86_64/hdb` .

После выполнения [необходимых действий](#prerequisites-for-installation) можно установить средства создания моментальных снимков с помощью самоустановки следующим образом.

1. Скопируйте скачанный самоустановщик в целевую систему.
1. Выполните самоустановщик от имени `root` пользователя, как показано в следующем примере. При необходимости сделайте файл исполняемым с помощью `chmod +x *.run` команды.

При выполнении команды самоустановки без аргументов будет отображаться Справка по использованию установщика для установки средств создания моментальных снимков следующим образом.

```bash
chmod +x azacsnap_installer_v5.0.run
./azacsnap_installer_v5.0.run
```

```output
Usage: ./azacsnap_installer_v5.0.run [-v] -I [-u <HLI Snapshot Command user>]
./azacsnap_installer_v5.0.run [-v] -X [-d <directory>]
./azacsnap_installer_v5.0.run [-h]

Switches enclosed in [] are optional for each command line.
- h prints out this usage.
- v turns on verbose output.
- I starts the installation.
- u is the Linux user to install the scripts into, by default this is
'azacsnap'.
- X will only extract the commands.
- d is the target directory to extract into, by default this is
'./snapshot_cmds'.
Examples of a target directory are ./tmp or /usr/local/bin
```

> [!NOTE]
> Самостоятельный установщик имеет возможность извлечь (-X) средства создания моментальных снимков из пакета без выполнения каких-либо пользовательских операций по созданию и настройке. Это позволяет опытному администратору вручную выполнить действия по установке или скопировать команды для обновления существующей установки.

### <a name="easy-installation-of-snapshot-tools-default"></a>Простая установка средств создания моментальных снимков (по умолчанию)

Установщик разработан для быстрой установки средств создания моментальных снимков для SAP HANA в Azure. По умолчанию, если установщик запускается только с параметром-I, он выполняет следующие действия:

1. Создание пользователя моментального снимка "азакснап", домашнего каталога и задание членства в группе.
1. Настройте имя входа пользователя азакснап `~/.profile` .
1. Поиск в файловой системе для каталогов, добавляемых в азакснап `$PATH` , обычно это пути к средствам SAP Hana, таким как `hdbsql` и `hdbuserstore` .
1. Поиск в файловой системе для добавления каталогов в азакснап `$LD_LIBRARY_PATH` . Многим командам требуется указать путь к библиотеке для правильной работы, настроив его для установленного пользователя.
1. Скопируйте ключи SSH для серверного хранилища для азакснап из корневого пользователя (пользователя, запустившего установку). Предполагается, что пользователь root уже настроил подключение к хранилищу.
    - см. раздел "[Включение связи с хранилищем](#enable-communication-with-storage)".
1. Скопируйте SAP HANA безопасное хранилище пользователей подключения для целевого пользователя азакснап. Предполагается, что корневой пользователь уже настроил безопасное хранилище пользователей — см. раздел "Включение связи с SAP HANA".
1. Средства создания моментальных снимков извлекаются в `/home/azacsnap/bin/` .
1. Команды в `/home/azacsnap/bin/` имеют свои разрешения (владение и двоичный файл и т. д.).

В следующем примере показаны правильные выходные данные установщика при запуске с параметром установки по умолчанию.

```bash
./azacsnap_installer_v5.0.run -I
```

```output
+-----------------------------------------------------------+
| Azure Application Consistent Snapshot tool Installer      |
+-----------------------------------------------------------+
|-> Installer version '5.0'
|-> Create Snapshot user 'azacsnap', home directory, and set group membership.
|-> Configure azacsnap .profile
|-> Search filesystem for directories to add to azacsnap's $PATH
|-> Search filesystem for directories to add to azacsnap's $LD_LIBRARY_PATH
|-> Copying SSH keys for back-end storage for azacsnap.
|-> Copying HANA connection keystore for azacsnap.
|-> Extracting commands into /home/azacsnap/bin/.
|-> Making commands in /home/azacsnap/bin/ executable.
|-> Creating symlink for hdbsql command in /home/azacsnap/bin/.
+-----------------------------------------------------------+
| Install complete! Follow the steps below to configure.    |
+-----------------------------------------------------------+
+-----------------------------------------------------------+
|  Install complete!  Follow the steps below to configure.  |
+-----------------------------------------------------------+

1. Change into the snapshot user account.....
     su - azacsnap
2. Set up the HANA Secure User Store..... (command format below)
     hdbuserstore Set <ADMIN_USER> <HOSTNAME>:<PORT> <admin_user> <password>
3. Change to location of commands.....
     cd /home/azacsnap/bin/
4. Configure the customer details file.....
     azacsnap -c configure --configuration new
5. Test the connection to storage.....
     azacsnap -c test --test storage
6. Test the connection to HANA.....
   a. without SSL
     azacsnap -c test --test hana
   b. with SSL,  you will need to choose the correct SSL option
     azacsnap -c test --test hana --ssl=<commoncrypto|openssl>
7. Run your first snapshot backup..... (example below)
     azacsnap -c backup --volume=data --prefix=hana_test --frequency=15min --retention=1
```

### <a name="uninstall-the-snapshot-tools"></a>Удаление средств создания моментальных снимков

Если средства создания моментальных снимков установлены с использованием параметров по умолчанию, для удаления необходимо только удалить пользователя, для которого были установлены команды (по умолчанию — азакснап).

```bash
userdel -f -r azacsnap
```

### <a name="manual-installation-of-the-snapshot-tools"></a>Ручная установка средств создания моментальных снимков

В некоторых случаях необходимо установить средства вручную, но рекомендуется использовать установщик по умолчанию, чтобы упростить этот процесс.

В каждой строке, начинающейся с `#` символа, показаны примеры команд, следующих за символом, которые выполняются привилегированным пользователем. В `\` конце строки находится Стандартный символ продолжения строки для команды Shell.

Как привилегированный суперпользователь, ручная установка может быть достигнута следующим образом.

1. Получение идентификатора группы "sapsys", в данном случае идентификатор группы = 1010

    ```bash
    grep sapsys /etc/group
    ```

    ```output
    sapsys:x:1010:
    ```

1. Создайте пользователя моментального снимка "азакснап", домашнего каталога и задайте членство в группе, используя идентификатор группы из шага 1.

    ```bash
    useradd -m -g 1010 -c "Azure SAP HANA Snapshots User" azacsnap
    ```

1. Убедитесь, что имя входа пользователя азакснап `.profile` существует.

    ```bash
    echo "" >> /home/azacsnap/.profile
    ```

1. Поиск в файловой системе для каталогов, добавляемых в $PATH азакснап, обычно это пути к SAP HANAным инструментам, таким как `hdbsql` и `hdbuserstore` .

    ```bash
    HDBSQL_PATH=`find -L /hana/shared/[A-z0-9][A-z0-9][A-z0-9]/HDB*/exe /usr/sap/hdbclient -name hdbsql -exec dirname {} + 2> /dev/null | sort | uniq | tr '\n' ':'`
    ```

1. Добавление обновленного пути к профилю пользователя

    ```bash
    echo "export PATH=\"\$PATH:$HDBSQL_PATH\"" >> /home/azacsnap/.profile
    ```

1. Поиск в файловой системе для добавления каталогов в _LIBRARY_PATH $LD азакснап.

    ```bash
    NEW_LIB_PATH=`find -L /hana/shared/[A-z0-9][A-z0-9][A-z0-9]/HDB*/exe /usr/sap/hdbclient -name "*.so" -exec dirname {} + 2> /dev/null | sort | uniq | tr '\n' ':'`
    ```

1. Добавление обновленного пути к библиотеке в профиль пользователя

    ```bash
    echo "export LD_LIBRARY_PATH=\"\$LD_LIBRARY_PATH:$NEW_LIB_PATH\"" >> /home/azacsnap/.profile
    ```

1. В крупных экземплярах Azure
    1. Скопируйте ключи SSH для серверного хранилища для азакснап из корневого пользователя (пользователя, запустившего установку). Предполагается, что пользователь root уже настроил подключение к хранилищу.
       > см. раздел "[Включение связи с хранилищем](#enable-communication-with-storage)".

        ```bash
        cp -pr ~/.ssh /home/azacsnap/.
        ```

    1. Правильно задайте разрешения пользователя для SSH-файлов

        ```bash
        chown -R azacsnap.sapsys /home/azacsnap/.ssh
        ```

1. На Azure NetApp Files
    1. Настройте `DOTNET_BUNDLE_EXTRACT_BASE_DIR` путь пользователя в руководстве по извлечению одиночных файлов .NET Core.
        1. SUSE Linux

            ```bash
            echo "export DOTNET_BUNDLE_EXTRACT_BASE_DIR=\$HOME/.net" >> /home/azacsnap/.profile
            echo "[ -d $DOTNET_BUNDLE_EXTRACT_BASE_DIR] && chmod 700 $DOTNET_BUNDLE_EXTRACT_BASE_DIR" >> /home/azacsnap/.profile
            ```

        1. RHEL

            ```bash
            echo "export DOTNET_BUNDLE_EXTRACT_BASE_DIR=\$HOME/.net" >> /home/azacsnap/.bash_profile
            echo "[ -d $DOTNET_BUNDLE_EXTRACT_BASE_DIR] && chmod 700 $DOTNET_BUNDLE_EXTRACT_BASE_DIR" >> /home/azacsnap/.bash_profile
            ```

1. Скопируйте SAP HANA безопасное хранилище пользователей подключения для целевого пользователя азакснап. Предполагается, что пользователь root уже настроил безопасное хранилище пользователей.
    > см. раздел "[Включение связи с SAP HANA](#enable-communication-with-sap-hana)".

    ```bash
    cp -pr ~/.hdb /home/azacsnap/.
    ```

1. Правильно задайте разрешения пользователя для `hdbuserstore` файлов

    ```bash
    chown -R azacsnap.sapsys /home/azacsnap/.hdb
    ```

1. Извлечение средств создания моментальных снимков в/Хоме/азакснап/бин/.

    ```bash
    ./azacsnap_installer_v5.0.run -X -d /home/azacsnap/bin
    ```

1. Сделать команды исполняемыми

    ```bash
    chmod 700 /home/azacsnap/bin/*
    ```

1. Убедитесь, что для домашнего каталога пользователя заданы правильные разрешения владения.

    ```bash
    chown -R azacsnap.sapsys /home/azacsnap/*
    ```

### <a name="complete-the-setup-of-snapshot-tools"></a>Завершение установки средств создания моментальных снимков

Установщик предоставляет инструкции по завершении установки средств создания моментальных снимков.
Выполните следующие действия, чтобы настроить и проверить средства создания моментальных снимков.  После успешного тестирования выполните предварительный моментальный снимок хранилища с единообразной базой данных.

В следующих выходных данных показаны действия, которые необходимо выполнить после запуска установщика с параметрами установки по умолчанию.

1. Изменение учетной записи пользователя моментального снимка
    1. `su - azacsnap`
1. Настройка хранилища безопасного пользователя HANA
   1. `hdbuserstore Set <ADMIN_USER> <HOSTNAME>:<PORT> <admin_user> <password>`
1. Изменение расположения команд
   1. `cd /home/azacsnap/bin/`
1. Настройка файла сведений о клиенте
   1. `azacsnap -c configure –-configuration new`
1. Проверьте подключение к хранилищу....
   1. `azacsnap -c test –-test storage`
1. Проверьте подключение к HANA....
    1. без SSL
       1. `azacsnap -c test –-test hana`
    1. При использовании SSL необходимо выбрать правильный параметр SSL.
       1. `azacsnap -c test –-test hana --ssl=<commoncrypto|openssl>`
1. Запуск первой резервной копии моментальных снимков
    1. `azacsnap -c backup –-volume data--prefix=hana_test --retention=1`

Шаг 2 потребуется, если параметр "[включить связь с SAP HANA](#enable-communication-with-sap-hana)" не был выполнен до установки.

> [!NOTE]
> Команды тестирования должны выполняться правильно. В противном случае команды могут завершиться ошибкой.

## <a name="configuring-the-database"></a>Настройка базы данных

В этом разделе объясняется, как настроить базу данных.

### <a name="sap-hana-configuration"></a>Конфигурация SAP HANA

Существуют некоторые рекомендованные изменения, которые необходимо применить к SAP HANA для обеспечения защиты резервных копий и каталога журналов. По умолчанию `basepath_logbackup` и `basepath_catalogbackup` будет выводить файлы в `$(DIR_INSTANCE)/backup/log` каталог, и маловероятно, что этот путь находится на томе, `azacsnap` настроенном для моментального снимка. Эти файлы не будут защищены с помощью моментальных снимков хранилища.

Приведенные ниже `hdbsql` примеры команд предназначены для демонстрации настройки путей к журналам и каталогам в расположениях, которые находятся на томах хранилища, которые могут быть моментальными снимками `azacsnap` . Обязательно проверьте, соответствуют ли значения в командной строке локальной конфигурации SAP HANA.

### <a name="configure-log-backup-location"></a>Настройка расположения резервной копии журнала

В этом примере в параметр вносятся изменения `basepath_logbackup` .

```bash
hdbsql -jaxC -n <HANA_ip_address>:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD> "ALTER SYSTEM ALTER CONFIGURATION ('global.ini', 'SYSTEM') SET ('persistence', 'basepath_logbackup') = '/hana/logbackups/H80' WITH RECONFIGURE"
```

### <a name="configure-catalog-backup-location"></a>Настройка расположения резервной копии каталога

В этом примере в параметр вносятся изменения `basepath_catalogbackup` . Сначала убедитесь, что `basepath_catalogbackup` путь существует в файловой системе, если не создать путь с тем же владельцем, что и каталог.

```bash
ls -ld /hana/logbackups/H80/catalog
```

```output
drwxr-x--- 4 h80adm sapsys 4096 Jan 17 06:55 /hana/logbackups/H80/catalog
```

Если необходимо создать путь, в следующем примере создается путь и устанавливаются правильные права владения и разрешения. Эти команды нужно будет выполнять от имени привилегированного пользователя.

```bash
mkdir /hana/logbackups/H80/catalog
chown --reference=/hana/shared/H80/HDB00 /hana/logbackups/H80/catalog
chmod --reference=/hana/shared/H80/HDB00 /hana/logbackups/H80/catalog
ls -ld /hana/logbackups/H80/catalog
```

```output
drwxr-x--- 4 h80adm sapsys 4096 Jan 17 06:55 /hana/logbackups/H80/catalog
```

В следующем примере изменится параметр SAP HANA.

```bash
hdbsql -jaxC -n <HANA_ip_address>:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD> "ALTER SYSTEM ALTER CONFIGURATION ('global.ini', 'SYSTEM') SET ('persistence', 'basepath_catalogbackup') = '/hana/logbackups/H80/catalog' WITH RECONFIGURE"
```

### <a name="check-log-and-catalog-backup-locations"></a>Проверка расположений резервных копий журналов и каталогов

После внесения изменений проверьте правильность параметров с помощью следующей команды.
В этом примере параметры, которые были установлены после приведенного выше правила, будут отображаться как системные параметры.

> Этот запрос также возвращает параметры по УМОЛЧАНИю для сравнения.

```bash
hdbsql -jaxC -n <HANA_ip_address> - i 00 -U AZACSNAP "select * from sys.m_inifile_contents where (key = 'basepath_databackup' or key ='basepath_datavolumes' or key = 'basepath_logbackup' or key = 'basepath_logvolumes' or key = 'basepath_catalogbackup')"
```

```output
global.ini,DEFAULT,,,persistence,basepath_catalogbackup,$(DIR_INSTANCE)/backup/log
global.ini,DEFAULT,,,persistence,basepath_databackup,$(DIR_INSTANCE)/backup/data
global.ini,DEFAULT,,,persistence,basepath_datavolumes,$(DIR_GLOBAL)/hdb/data
global.ini,DEFAULT,,,persistence,basepath_logbackup,$(DIR_INSTANCE)/backup/log
global.ini,DEFAULT,,,persistence,basepath_logvolumes,$(DIR_GLOBAL)/hdb/log
global.ini,SYSTEM,,,persistence,basepath_catalogbackup,/hana/logbackups/H80/catalog
global.ini,SYSTEM,,,persistence,basepath_datavolumes,/hana/data/H80
global.ini,SYSTEM,,,persistence,basepath_logbackup,/hana/logbackups/H80
global.ini,SYSTEM,,,persistence,basepath_logvolumes,/hana/log/H80
```

### <a name="configure-log-backup-timeout"></a>Настройка времени ожидания резервного копирования журнала

Значение по умолчанию для SAP HANA для резервного копирования журнала составляет 900 секунд (15 минут). Рекомендуется сократить это значение до 300 секунд (то есть 5 минут).  Затем можно выполнить регулярное резервное копирование (например, каждые 10 минут), добавив log_backups том в раздел другой том файла конфигурации.

```bash
hdbsql -jaxC -n <HANA_ip_address>:30013 -i 00 -u SYSTEM -p <SYSTEM_USER_PASSWORD> "ALTER SYSTEM ALTER CONFIGURATION ('global.ini', 'SYSTEM') SET ('persistence', 'log_backup_timeout_s') = '300' WITH RECONFIGURE"
```

#### <a name="check-log-backup-timeout"></a>Проверить время ожидания резервного копирования журнала

После внесения изменений в время ожидания резервного копирования журнала убедитесь, что оно задано следующим образом.
В этом примере настроенные параметры будут отображаться как системные параметры, но этот запрос также возвращает параметры по УМОЛЧАНИю для сравнения.

```bash
hdbsql -jaxC -n <HANA_ip_address> - i 00 -U AZACSNAP "select * from sys.m_inifile_contents where key like '%log_backup_timeout%' "
```

```output
global.ini,DEFAULT,,,persistence,log_backup_timeout_s,900
global.ini,SYSTEM,,,persistence,log_backup_timeout_s,300
```

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка средства создания моментальных снимков с единообразным применением приложений Azure](azacsnap-cmd-ref-configure.md)
