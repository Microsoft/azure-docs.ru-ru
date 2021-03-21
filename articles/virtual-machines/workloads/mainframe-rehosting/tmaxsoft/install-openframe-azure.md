---
title: Установка Тмакссофт Опенфраме на виртуальных машинах Azure
description: Узнайте, как настроить среду Опенфраме в Azure, подходящую для разработки, демонстрации, тестирования или рабочих нагрузок.
services: virtual-machines
documentationcenter: ''
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: virtual-machines
ms.subservice: workloads
ms.openlocfilehash: 01a053745ad0b2b280e214af8284113a02458a27
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102549181"
---
# <a name="install-tmaxsoft-openframe-on-azure"></a>Установка Тмакссофт Опенфраме в Azure

Узнайте, как настроить среду Опенфраме в Azure, подходящую для разработки, демонстрации, тестирования или рабочих нагрузок. В этом руководстве описывается каждый шаг.

Опенфраме содержит несколько компонентов, которые создают среду эмуляции мэйнфреймов в Azure. Например, Опенфраме веб-службы заменить по промежуточного слоя мэйнфрейма, например систему управления информацией о клиенте IBM (CICS) и Опенфраме пакет, с его компонентом ТЖЕС, заменяет подсистему записи заданий (ЖЕС) IBM мэйнфрейма.

Опенфраме работает с любой реляционной базой данных, включая Oracle Database, Microsoft SQL Server, IBM DB2 и MySQL. В этой установке Опенфраме используется реляционная база данных Тмакссофт Тиберо. Как Опенфраме, так и Тиберо работают в операционной системе Linux. В этом руководстве мы устанавливаем CentOS 7,3, хотя вы можете использовать другие Поддерживаемые дистрибутивы Linux. Сервер приложений Опенфраме и база данных Тиберо установлены на одной виртуальной машине.

В руководстве описано, как установить компоненты Опенфраме Suite. Некоторые из них необходимо устанавливать отдельно.

Основные компоненты Опенфраме:

- Обязательные пакеты установки.
- База данных тиберо.
- Открытый доступ к базам данных (ODBC) используется приложениями в Опенфраме для взаимодействия с базой данных Тиберо.
- Опенфраме Base, по промежуточного слоя, которое управляет всей системой.
- Опенфраме пакет — решение, заменяющее системы пакетной обработки больших ЭВМ.
- ТАКФ — модуль службы, который управляет доступом пользователей к системам и ресурсам.
- Sort — средство сортировки для пакетных транзакций.
- ОФКОБОЛ, компилятор, который интерпретирует COBOL-программы мэйнфрейма.
- ОФАСМ — компилятор, который интерпретирует программы ассемблера мэйнфреймов.
- Опенфраме Server Type C (OSC) — решение, заменяющее по промежуточного слоя мэйнфрейма и IBM CICS.
- Решение Java Enterprise (ЖЕУС) — сервер веб-приложений, сертифицированный для Java Enterprise Edition 6.
- ОФГВ, компонент шлюза Опенфраме, предоставляющий прослушиватель 3270.
- Офманажер — решение, предоставляющее операции Опенфраме и функции управления в веб-среде.

Другие необходимые компоненты Опенфраме:

- OSI — решение, заменяющее по промежуточного слоя мэйнфрейма и КОНТРОЛЛЕРы МГНОВЕНных сообщений.
- ТЖЕС — решение, предоставляющее ЖЕС среду мэйнфрейма.
- ОФТСАМ — решение, которое позволяет использовать (V) файлы SAM в открытой системе.
- Офхидб — решение, заменяющее базу данных МГНОВЕНных сообщений мэйнфрейма.
- ОФПЛИ, компилятор, который интерпретирует программы PL/I мэйнфрейма.
- ПРОТРИЕВЕ — решение, которое выполняет Еаситриеве на языке мэйнфреймов.
- Офминер — решение, которое анализирует ресурсы мэйнфреймов и затем переносит их в Azure.

## <a name="architecture"></a>Архитектура

На следующем рисунке представлен обзор архитектурных компонентов Опенфраме 7,0, установленных в этом руководстве.

![Компоненты Опенфраме](media/openframe-02.png)

## <a name="azure-system-requirements"></a>Требования к системе для Azure

В следующей таблице перечислены требования для установки в Azure.
<!-- markdownlint-disable MD033 -->

<table>
<thead>
    <tr><th>Требование</th><th>Описание</th></tr>
</thead>
<tbody>
<tr><td>Поддерживаемые дистрибутивы Linux в Azure
</td>
<td>
Linux x86 2,6 (32-разрядная версия, 64-bit)<br/>
Red Hat 7. x<br/>
CentOS 7. x<br/>
</td>
</tr>
<tr><td>Оборудование
</td>
<td>Ядра: 2 (минимум)<br/>
Память: 4 ГБ (минимум)<br/>
Область подкачки: 1 ГБ (минимум)<br/>
Жесткий диск: 100 ГБ (минимум)<br/>
</td>
</tr>
<tr><td>Дополнительное программное обеспечение для пользователей Windows
</td>
<td>Выводимое значение: используется в этом руководство для настройки функций виртуальной машины.<br/>
WinSCP: популярный клиент SFTP и FTP-клиент, который можно использовать<br/>
Eclipse для Windows: платформа разработки, поддерживаемая Тмакссофт<br/>
(В настоящее время Microsoft Visual Studio не поддерживается)
</td>
</tr>
</tbody>
</table>

<!-- markdownlint-enable MD033 -->

## <a name="prerequisites"></a>Предварительные условия

Составьте план на несколько дней, чтобы собрать все необходимое программное обеспечение и выполнить все ручные процессы.

Прежде чем приступить к работе, выполните следующие действия.

- Получите установочный носитель Опенфраме из Тмакссофт. Если вы являетесь существующим клиентом Тмакссофт, обратитесь к своему представителяу Тмакссофт для получения лицензионной копии. В противном случае запросите пробную версию из [тмакссофт](https://www.tmaxsoft.com/contact/).

- Запросите документацию по Опенфраме, отправив сообщение электронной почты по адресу <support@tmaxsoft.com> .

- Получите подписку Azure, если у вас ее еще нет. Можно также создать [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) перед началом.

- Необязательный параметр. Настройте туннель VPN типа "сеть — сеть" или Jumpbox, которые ограничивают доступ к виртуальной машине Azure разрешенными пользователями в вашей организации. Этот шаг не является обязательным, но рекомендуется.

## <a name="set-up-a-vm-on-azure-for-openframe-and-tibero"></a>Настройка виртуальной машины в Azure для Опенфраме и Тиберо

Среду Опенфраме можно настроить с помощью различных шаблонов развертывания, но в следующей процедуре показано, как развернуть сервер приложений Опенфраме и базу данных Тиберо на одной виртуальной машине. В более крупных средах и для рабочих нагрузок немалым рекомендуется развертывать базу данных отдельно на собственной виртуальной машине для повышения производительности.

**Создание виртуальной машины**

1. Перейдите на портал Azure по адресу <https://portal.azure.com> и войдите в свою учетную запись.

2. Нажмите **Виртуальные машины**.

    ![Список ресурсов в портал Azure](media/vm-01.png)

3. Нажмите кнопку **Добавить**.

    ![Добавление параметра в портал Azure](media/vm-02.png)

4. Справа от **операционных систем** щелкните **Дополнительно**.

     ![Дополнительные возможности в портал Azure](media/vm-03.png)

5. Щелкните **CentOS на основе 7,3** , чтобы выполнить этот проход в точности, или выберите другой поддерживаемый дистрибутив Linux.

     ![Параметры операционной системы в портал Azure](media/vm-04.png)

6. В области **основные** параметры введите **имя**, **имя пользователя**, **тип проверки подлинности**, **Подписка** (с оплатой по мере использования — это AWS стиль оплаты) и **группу ресурсов** (используйте существующую или создайте группу тмакссофт).

7. По завершении (включая пару открытого и закрытого ключей для **типа проверки подлинности**) нажмите кнопку **Отправить**.

> [!NOTE]
> Если для **типа проверки подлинности** используется открытый ключ SSH, см. действия, описанные в следующем разделе, чтобы создать пару открытого и закрытого ключей, а затем возобновите шаги здесь.

### <a name="generate-a-publicprivate-key-pair"></a>Создание пары открытого и закрытого ключей

Если используется операционная система Windows, необходимо PuTTYgen создать пару из открытого и закрытого ключей.

Открытый ключ может быть свободно предоставлен, но закрытый ключ должен храниться полностью в секрете и никогда не должен использоваться совместно с другой стороной. После создания ключей необходимо вставить **открытый ключ SSH** в конфигурацию — по сути, загружая его на виртуальную машину Linux. Он хранится внутри разрешенных \_ ключей в \~ каталоге/.SSH. домашнего каталога учетной записи пользователя. Затем виртуальная машина Linux сможет распознать и проверить подключение после предоставления соответствующего **закрытого ключа SSH** в SSH-клиенте (в нашем примере это выявление).

При предоставлении новым пользователям доступа к виртуальной машине: 

- Каждый новый пользователь создает собственные открытые и закрытые ключи с помощью PuTTYgen.
- Частные пользователи сохраняют собственные закрытые ключи и отправляют сведения об открытом ключе администратору виртуальной машины.
- Администратор вставит содержимое открытого ключа в \~ \_ файл ключей/.СШ/аусоризед.
- Новая отдельная подключается через выводимые.

**Создание пары открытого и закрытого ключей**

1.  Скачайте PuTTYgen из <https://www.putty.org/> и установите его, используя все параметры по умолчанию.

2.  Чтобы открыть PuTTYgen, перейдите в каталог установки с выводимыми файлами в C: \\ Program Files \\ .

    ![Интерфейс выводимых](media/puttygen-01.png)

3.  Щелкните **Создать**.

    ![Снимок экрана с диалоговым окном генератора ключа выводимых ключей и выделением кнопки Создать.](media/puttygen-02.png)

4.  После создания сохраните как открытый ключ, так и закрытый ключ. Вставьте содержимое открытого ключа в раздел **открытый ключ SSH** в области **\> Основные сведения о создании виртуальной машины** (см. шаги 6 и 7 в предыдущем разделе).

    ![Диалоговое окно «Генератор ключей»](media/puttygen-03.png)

### <a name="configure-vm-features"></a>Настройка функций виртуальной машины

1. В портал Azure в колонке **Выбор размера** выберите нужные параметры оборудования Linux. *Минимальные* требования для установки обеих Тиберо и опенфраме: 2 ЦП и 4 ГБ ОЗУ, как показано в этом примере установки.

    ![Создание виртуальной машины — основы](media/create-vm-01.png)

2. Щелкните **3 параметра** и используйте параметры по умолчанию для настройки дополнительных компонентов.
3. Проверьте сведения об оплате.

    ![Создание виртуальной машины — Покупка](media/create-vm-02.png)

4. Отправьте свои настройки. Azure начинает развертывание виртуальной машины. Этот процесс займет несколько минут.

5. При развертывании виртуальной машины отображается ее панель мониторинга, в которой отображаются все параметры, выбранные во время настройки. Запишите **общедоступный IP-адрес**.

    ![тмакс на панели мониторинга Azure](media/create-vm-03.png)

6. Откройте PuTTY.

7. В поле **имя узла** введите имя пользователя и общедоступный IP-адрес, который вы скопировали. Например, **username \@ publicip**.

    ![Снимок экрана с диалоговым окном настройки выводимых параметров и выделяет поле имени узла (или IP-адреса).](media/putty-01.png)

8. В поле **Категория** выберите **подключение \> SSH \> AUTH**. Укажите путь к файлу **закрытого ключа** .

    ![Диалоговое окно «Конфигурация выводимых»](media/putty-02.png)

9. Нажмите кнопку **Открыть** , чтобы открыть окно вывода. В случае успеха вы подключитесь к новой виртуальной машине CentOS, работающей в Azure.

10. Чтобы войти в систему от имени привилегированного пользователя, введите **sudo Bash**.

    ![Вход пользователя root в окне командной строки](media/putty-03.png)

## <a name="set-up-the-environment-and-packages"></a>Настройка среды и пакетов

После создания виртуальной машины и входа в систему необходимо выполнить несколько шагов установки и установить необходимые пакеты предустановки.

1. Сопоставьте имя **офдемо** с ЛОКАЛЬНЫМ IP-адресом с помощью VI, чтобы изменить файл hosts ( `vi /etc/hosts` ). Предполагая, что наш IP-адрес — это 192.168.96.148 офдемо, это до изменения:

    ```vi
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
    ::1              localhost localhost.localdomain localhost6 localhost6.localdomain 
    <IP Address>    <your hostname>
    ```

     Это происходит после изменения:

    ```vi
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
    ::1              localhost localhost.localdomain localhost6 localhost6.localdomain 
    192.168.96.148   ofdemo
    ```

2. Создание групп и пользователей:

    ```vi
    [root@ofdemo ~]# adduser -d /home/oframe7 oframe7 
    [root@ofdemo ~]# passwd oframe7
    ```

3. Измените пароль для пользователя oframe7:

    ```vi
    New password: 
    Retype new password: 
    passwd: all authentication tokens updated successfully.
    ```

4. Обновите параметры ядра в/ЕТК/сисктл.конф:

    ```vi
    [root@ofdemo ~]# vi /etc/sysctl.conf
    kernel.shmall = 7294967296 
    kernel.sem = 10000 32000 10000 10000
    ```

5. Динамически обновлять параметры ядра без перезагрузки:

    ```vi
    [root@ofdemo ~]# /sbin/sysctl -p
    ```

6. Получите необходимые пакеты: Убедитесь, что сервер подключен к Интернету, скачайте следующие пакеты, а затем установите их:

     - dos2unix
     - glibc
     - glibc. i686 glibc. x86 \_ 64
     - либаио
     - нкурсес

          > [!NOTE]
          > После установки пакета нкурсес создайте следующие символьные ссылки:
         ```
         ln -s /usr/lib64/libncurses.so.5.9 /usr/lib/libtermcap.so
         ln -s /usr/lib64/libncurses.so.5.9 /usr/lib/libtermcap.so.2
         ```

     - GCC;
     - GCC-c + +
     - либаио-которые. x86 \_ 64
     - страце
     - лтраце
     - gdb

7. В случае установки в Java RPM выполните следующие действия.

```
root@ofdemo ~]# rpm -ivh jdk-7u79-linux-x64.rpm
[root@ofdemo ~]# vi .bash_profile

# JAVA ENV
export JAVA_HOME=/usr/java/jdk1.7.0_79/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar

[root@ofdemo ~]# source /etc/profile
[root@ofdemo ~]# java –version

java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

[root@ofdemo ~]# echo $JAVA_HOME /usr/java/jdk1.7.0_79/
```

## <a name="install-the-tibero-database"></a>Установка базы данных Тиберо

Тиберо предоставляет несколько ключевых функций в среде Опенфраме в Azure:

- Тиберо используется в качестве внутреннего хранилища данных Опенфраме для различных системных функций.
- ВСАМ файлы, включая КСДС, РРДС и ЕСДС, используют для хранения данных внутреннюю базу данных Тиберо.
- Репозиторий данных ТАКФ хранится в Тиберо.
- Сведения о каталоге Опенфраме хранятся в Тиберо.
- Базу данных Тиберо можно использовать для замены IBM DB2 для хранения данных приложения.

**Установка Тиберо**

1. Убедитесь, что файл двоичного установщика Тиберо имеется, и проверьте номер версии.
2. Скопируйте программное обеспечение Тиберо в учетную запись пользователя Тиберо (офраме). Пример:

    ```
    [oframe7@ofdemo ~]$ tar -xzvf tibero6-bin-6_rel_FS04-linux64-121793-opt-tested.tar.gz 
    [oframe7@ofdemo ~]$ mv license.xml /opt/tmaxdb/tibero6/license/
    ```

3. Откройте профиль Bash \_ в VI ( `vi .bash_profile` ) и вставьте в него следующий текст:

    ```
    # Tibero6 ENV
    export TB_HOME=/opt/tmaxdb/tibero6 
    export TB_SID=TVSAM export TB_PROF_DIR=$TB_HOME/bin/prof 
    export LD_LIBRARY_PATH=$TB_HOME/lib:$TB_HOME/client/lib:$LD_LIBRARY_PATH 
    export PATH=$TB_HOME/bin:$TB_HOME/client/bin:$PATH
    ```

4. Чтобы выполнить профиль bash, в командной строке введите:

    ```
    source .bash_profile
    ```

5. Создайте файл TIP (файл конфигурации для Тиберо), а затем откройте его в VI. Пример:

    ```
    [oframe7@ofdemo ~]$ sh $TB_HOME/config/gen_tip.sh
    [oframe7@ofdemo ~]$ vi $TB_HOME/config/$TB_SID.tip
    ```

6. Измените \$ ТБ \_ Home/Client/config/тбдсн. TBR и введите 127.0.0.1 вместо офлокалхост, как показано ниже:

    ```
    TVSAM=( 
    (INSTANCE=(HOST=127.0.0.1)
        (PT=8629)
    (DB_NAME=TVSAM)
          )
     )
    ```

7. Создайте базу данных. Отображаются следующие результаты:

    ```
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof.
    Listener port = 8629
    Tibero 6
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NOMOUNT mode).
     /--------------------- newmount sql ------------------------/
    create database character set MSWIN949 national character set UTF16;
    /-----------------------------------------------------------/
    Database created.
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof.
    Listener port = 8629
    Tibero 6
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NORMAL mode).
    /opt/tmaxdb/tibero6/bin/tbsvr
    ………………………..
    Creating agent table...
    Done.
    For details, check /opt/tmaxdb/tibero6/instance/TVSAM/log/system_init.log.
    ************************************************** 
    * Tibero Database TVSAM is created successfully on Fri Aug 12 19:10:43 UTC 2016.
    *     Tibero home directory ($TB_HOME) =
    *         /opt/tmaxdb/tibero6
    *     Tibero service ID ($TB_SID) = TVSAM
    *     Tibero binary path =
    *         /opt/tmaxdb/tibero6/bin:/opt/tmaxdb/tibero6/client/bin
    *     Initialization parameter file =
    *         /opt/tmaxdb/tibero6/config/TVSAM.tip
    * 
    * Make sure that you always set up environment variables $TB_HOME and
    * $TB_SID properly before you run Tibero.
     ******************************************************************************
    ```

8. Чтобы перезапустить Тиберо, сначала выключите его с помощью `tbdown` команды. Пример:

    ```
    [oframe7@ofdemo ~]$$ tbdown 
    Tibero instance terminated (NORMAL mode).
    ```

9. Теперь загрузите Тиберо с помощью `tbboot` . Пример:

    ```
    [oframe7@ofdemo ~]$ tbboot
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof. Listener port = 8629

    Tibero 6  
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NORMAL mode).
    ```

10. Чтобы создать табличное пространство, откройте базу данных с помощью SYS User (sys/тмакс), а затем создайте необходимое табличное пространство для тома по умолчанию и ТАКФ:

    ```
    [oframe7@ofdemo ~]$ tbsql tibero/tmax
    tbSQL 6  
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Connected to Tibero.
    ```

11. Теперь введите следующие команды SQL:

    ```
    SQL> create tablespace "DEFVOL" datafile 'DEFVOL.dbf' size 500M autoextend on; create tablespace "TACF00" datafile 'TACF00.dbf' size 500M autoextend on; create tablespace "OFM_REPOSITORY" datafile 'ofm_repository.dbf' size 300M autoextend on;
    SQL> Tablespace 'DEFVOL' created.
    SQL> Tablespace 'TACF00' created.
    SQL> Tablespace ' OFM_REPOSITORY ' created.
    SQL> SQL> Disconnected.
    ```

12. Загрузите Тиберо и убедитесь, что процессы Тиберо запущены:

    ```
    [oframe7@ofdemo ~]$ tbboot 
    ps -ef | egrep tbsvr
    ```

Выходные данные:

![Выходные данные тиберо](media/tibero-01.png)

## <a name="install-odbc"></a>Установка ODBC

Приложения в Опенфраме взаимодействуют с базой данных Тиберо с помощью API ODBC, предоставляемого проектом unixODBC с открытым кодом.

Чтобы установить ODBC, выполните следующие действия.

1. Убедитесь, что файл установщика файл unixodbc-2.3.4. tar. gz имеется, или используйте `wget unixODBC-2.3.4.tar.gz` команду. Пример:

     ```
     [oframe7@ofdemo ~]$ wget ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.4.tar.gz
     ```

2. Распакуйте двоичный файл. Пример:

     ```
     [oframe7@ofdemo ~]$ tar -zxvf unixODBC-2.3.4.tar.gz
     ```

3. Перейдите в каталог unixODBC-2.3.4 и создайте файл makefile с помощью проверки сведений о компьютере. Пример:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ ./configure --prefix=/opt/tmaxapp/unixODBC/ --sysconfdir=/opt/tmaxapp/unixODBC/etc
     ```

     По умолчанию unixODBC устанавливается в/usr/local, поэтому `--prefix` для изменения расположения передается значение. Аналогичным образом файлы конфигурации устанавливаются в/etc по умолчанию, поэтому `--sysconfdir` передает значение нужного расположения.

4. Выполнить файл Makefile: `[oframe7@ofdemo unixODBC-2.3.4]$ make`

5. Скопируйте исполняемый файл в каталог программы после компиляции. Пример:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ make install
     ```

6. С помощью VI измените профиль bash ( `vi ~/.bash_profile` ) и добавьте следующее:

     ```
     # UNIX ODBC ENV 
     export ODBC_HOME=$HOME/unixODBC 
     export PATH=$ODBC_HOME/bin:$PATH 
     export LD_LIBRARY_PATH=$ODBC_HOME/lib:$LD_LIBRARY_PATH 
     export ODBCINI=$HOME/unixODBC/etc/odbc.ini 
     export ODBCSYSINI=$HOME
     ```

7. Примените ODBC. Внесите соответствующие изменения в следующие файлы. Пример:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ source ~/.bash_profile

     [oframe7@ofdemo ~]$ cd

     [oframe7@ofdemo ~]$ odbcinst -j unixODBC 2.3.4
     DRIVERS............: /home/oframe7/odbcinst.ini
     SYSTEM DATA SOURCES: /home/oframe7/odbc.ini
     FILE DATA SOURCES..: /home/oframe7/ODBCDataSources
     USER DATA SOURCES..: /home/oframe7/unixODBC/etc/odbc.ini
     SQLULEN Size.......: 8
     SQLLEN Size........: 8
     SQLSETPOSIROW Size.: 8

     [oframe7@ofdemo ~]$ vi odbcinst.ini

     [Tibero]
     Description = Tibero ODBC driver for Tibero6
     Driver = /opt/tmaxdb/tibero6/client/lib/libtbodbc.so
     Setup = 
     FileUsage = 
     CPTimeout = 
     CPReuse = 
     Driver Logging = 7

     [ODBC]
     Trace = NO 
     TraceFile = /home/oframe7/odbc.log 
     ForceTrace = Yes 
     Pooling = No 
     DEBUG = 1

     [oframe7@ofdemo ~]$ vi odbc.ini

     [TVSAM]
     Description = Tibero ODBC driver for Tibero6 
     Driver = Tibero 
     DSN = TVSAM 
     SID = TVSAM 
     User = tibero 
     password = tmax
     ```

8. Создайте символьную ссылку и проверьте подключение к базе данных Тиберо:

     ```
     [oframe7@ofdemo ~]$ ln $ODBC_HOME/lib/libodbc.so $ODBC_HOME/lib/libodbc.so.1 [oframe7@ofdemo ~]$ ln $ODBC_HOME/lib/libodbcinst.so 
     $ODBC_HOME/lib/libodbcinst.so.1

     [oframe7@ofdemo lib]$ isql TVSAM tibero tmax
     ```

Выводится следующий результат.

![Выходные данные ODBC, отображающие соединение с SQL](media/odbc-01.png)

## <a name="install-openframe-base"></a>Базовая установка Опенфраме

Базовый сервер приложений устанавливается перед отдельными службами, которые Опенфраме использует для управления системой в Azure, включая процессы обработки транзакций сервера.

**Установка базового Опенфраме**

1. Убедитесь, что установка Тиберо прошла удачно, а затем убедитесь, что указан следующий файл \_ установщика опенфраме Base7 \_ 0 \_ Linux \_ x86 \_ 64. bin и базовый файл конфигурации. Properties.

2. Обновите профиль bash, используя следующие сведения, относящиеся к Тиберо:

     ```bash
     alias ofhome='cd $OPENFRAME_HOME'
     alias ulog='cd $OPENFRAME_HOME/log/tmax/ulog'
     alias sysjcl='cd $OPENFRAME_HOME/volume_default/SYS1.JCLLIB'
     alias sysload='cd $OPENFRAME_HOME/volume_default/SYS1.LOADLIB'
     alias sysproc='cd $OPENFRAME_HOME/volume_default/SYS1.PROCLIB'
     alias oscsrc='cd $OPENFRAME_HOME/osc/oivp'
     alias osisrc='cd $OPENFRAME_HOME/osi/oivp'
     alias defvol='cd $OPENFRAME_HOME/volume_default'
     ```

3. Выполните профиль Bash:`[oframe7@ofdemo ~]$ . .bash_profile`
4. Убедитесь, что процессы Тиберо запущены. Пример:

     ```linux
     [oframe7@ofdemo ~]$ ps -ef|grep tbsvr
     ```

    ![Основной](media/base-01.png)

     > [!IMPORTANT]
     > Убедитесь, что вы начали Тиберо перед установкой.

5. Создайте лицензию на сайте [TechNet.tmaxsoft.com](https://technet.tmaxsoft.com/en/front/main/main.do) и вставьте в соответствующую папку опенфраме Base, Batch, ТАКФ, OSC.

     ```
     [oframe7@ofdemo ~]$ cp license.dat /opt/tmaxapp/OpenFrame/core/license/
     [oframe7@ofdemo ~]$ cp lictjes.dat lictacf.dat licosc.dat $OPENFRAME_HOME/license/
     ```

6. Скачайте файлы базового двоичного файла Опенфраме и базового. Properties:

     ```
     [oframe7@ofdemo ~]$ vi base.properties
     OPENFRAME_HOME= <appropriate location for installation> ex. /opt/tmaxapp/OpenFrame TP_HOST_NAME=<your IP Hostname> ex. ofdemo 
     TP_HOST_IP=<your IP Address> ex. 192.168.96.148 
     TP_SHMKEY=63481 
     TP_TPORTNO=6623 
     TP_UNBLOCK_PORT=6291 
     TP_NODE_NAME=NODE1 
     TP_NODE_LIST=NODE1 
     MASCAT_NAME=SYS1.MASTER.ICFCAT 
     MASCAT_CREATE=YES 
     DEFAULT_VOLSER=DEFVOL 
     VOLADD_DEFINE=YES TSAM_USERNAME=tibero 
     TSAM_PASSWORD=tmax 
     TSAM_DATABASE=oframe 
     DATASET_SHMKEY=63211 
     DSLOCK_DATA=SYS1.DSLOCK.DATA 
     DSLOCK_LOG=SYS1.DSLOCK.LOG 
     DSLOCK_SEQ=dslock_seq.dat 
     DSLOCK_CREATE=YES 
     OPENFRAME_LICENSE_PATH=/opt/tmaxapp/license/OPENFRAME TMAX_LICENSE_PATH=/opt/tmaxapp/license/TMAX
     ```

7. Запустите установщик с помощью файла Base. Properties. Пример:

    ```
    [oframe7@ofdemo ~]$ chmod a+x OpenFrame_Base7_0_Linux_x86_64.bin 
    [oframe7@ofdemo ~]$ ./OpenFrame_Base7_0_Linux_x86_64.bin -f base.properties
    ```

    По завершении сообщение об окончании установки будет воспроизводиться.

8. Проверьте структуру каталогов базового каталога Опенфраме с помощью `ls -ltr` команды. Пример:

     ```
     [oframe7@ofdemo OpenFrame]$ ls -ltr
     total 44

     drwxrwxr-x. 4 oframe7 oframe7 61 Nov 30 16:57 UninstallerData 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 bin 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 cpm drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 data 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 include 
     drwxrwxr-x. 2 oframe7 oframe7 8192 Nov 30 16:57 lib 
     drwxrwxr-x. 6 oframe7 oframe7 48 Nov 30 16:57 log 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 profile 
     drwxrwxr-x. 7 oframe7 oframe7 62 Nov 30 16:57 sample 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 schema 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 temp 
     drwxrwxr-x. 3 oframe7 oframe7 16 Nov 30 16:57 shared 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 license 
     drwxrwxr-x. 23 oframe7 oframe7 4096 Nov 30 16:58 core 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 config 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 scripts 
     drwxrwxr-x. 2 oframe7 oframe7 25 Nov 30 16:58 volume_default
     ```

9. Запустить Опенфраме Base:

     ```
     [oframe7@ofdemo ~]$ cp /usr/lib/libtermcap.so.2 $TMAXDIR/lib
     Startup Tmax Server
     [oframe7@ofdemo ~]$ tmboot
     ```

     ![выходные данные команды тмбут](media/base-02.png)

10. Убедитесь, что состояние процесса Готово, с помощью команды тмадмин в Si. РДИ отображается в столбце **состояние** для каждого из процессов:

     ![выходные данные команды тмадмин](media/base-03.png)

11. Завершение работы Опенфраме Base:

     ```
     [oframe7@ofdemo ~]$ tmdown 
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(vtammgr:43) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofruisvr:41) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrsmlog:42) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: CLH downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: CLL downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TLM downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TMM downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-openframe-batch"></a>Установка пакета Опенфраме

Пакет Опенфраме состоит из нескольких компонентов, имитирующих среды пакетной обработки больших ЭВМ и используемых для выполнения пакетных заданий в Azure.

**Установка пакета**

1. Убедитесь, что базовая установка прошла удачно, а затем убедитесь, что \_ \_ файл конфигурации опенфраме Batch7 0 \_ Fix2 \_ MVS \_ Linux \_ x86 \_ 64. bin и пакет. Properties установлены:

2. В командной строке введите, `vi batch.properties` чтобы изменить файл пакетной службы. Properties с помощью VI.

3. Измените параметры следующим образом:

     ```
     OPENFRAME_HOME = /opt/tmaxapp/OpenFrame
     DEFAULT_VOLSER=DEFVOL 
     TP_NODE_NAME=NODE1 
     TP_NODE_LIST=NODE1 
     RESOURCE_SHMKEY=66991 
     #JOBQ_DATASET_CREATE=YES 
     #OUTPUTQ_DATASET_CREATE=YES 
     DEFAULT_JCLLIB_CREATE=YES 
     DEFAULT_PROCLIB_CREATE=YES 
     DEFAULT_USERLIB_CREATE=YES 
     TJES_USERNAME=tibero 
     TJES_PASSWORD=tmax 
     TJES_DATABASE=oframe 
     BATCH_TABLE_CREATE=YES
     ```

4. Для выполнения установщика пакетной службы в командной строке введите:

     ```
     ./OpenFrame_Batch7_0_Fix2_MVS_Linux_x86_64.bin -f batch.properties
     ```

5. После завершения установки запустите установленные пакеты Опенфраме, введя `tmboot` в командной строке.

    ![выходные данные тмбут](media/tmboot-01.png)

6. Введите `tmadmin` в командной строке, чтобы проверить процесс опенфраме.

    ![Экран администратора тмакс](media/tmadmin-01.png)

7. Выполните следующие команды:

     ```
     $$2 NODE1 (tmadm): quit 
     ADM quit for node (NODE1)
     ```

8. `tmdown`Для запуска и завершения работы пакетной службы используйте команду:

     ```
     [oframe7@ofdemo ~]$tmdown
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:44) downed: Wed Sep  7 16:01:46 2016
     TMDOWN: SERVER(vtammgr: 43) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:45) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:46) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:47) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjschd:54) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjinit:55) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:48) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjspbk:57) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:49) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:50) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:51) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:52) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:53) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjhist:56) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofruisvr:41) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmtsmgr:59) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrpmsvr:58) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrsmlog:42) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: CLL downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TLM downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: CLH downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TMM downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-tacf"></a>Установка ТАКФ

ТАКФ Manager — это модуль службы Опенфраме, который управляет доступом пользователей к системам и ресурсам через RACF Security.

**Установка ТАКФ**

1. Убедитесь, что файл \_ конфигурации опенфраме Tacf7 \_ 0 \_ Fix2 \_ Linux \_ x86 \_ 64. bin установлен в настоящее время.
2. Убедитесь, что установка пакета прошла, а затем с помощью VI откройте файл такф. Properties ( `vi tacf.properties` ).
3. Измените параметры ТАКФ:

     ```
     OPENFRAME_HOME=/opt/tmaxapp/OpenFrame 
     USE_OS_AUTH=NO 
     TACF_USERNAME=tibero 
     TACF_PASSWORD=tmax 
     TACF_DATABASE=oframe 
     TACF_TABLESPACE=TACF00 
     TACF_TABLE_CREATE=YES
     ```

4. После завершения установки ТАКФ примените переменные среды ТАКФ. В командной строке введите следующее:

     ```
     source \~/.bash\_profile
     ```

5. Выполните установщик ТАКФ. В командной строке введите следующее:

     ```
     ./OpenFrame_Tacf7_0_Fix2_Linux_x86_64.bin -f tacf.properties
     ```

     Выходные данные выглядят следующим образом:

     ```
     Wed Dec 07 17:36:42 EDT 2016
     Free Memory: 18703 kB 
     Total Memory: 28800 kB

     4 Command Line Args: 
     0:  -f 1:  tacf.properties 
     2:  -m 
     3:  SILENT 
     java.class.path: 
     /tmp/install.dir.41422/InstallerData 
     /tmp/install.dir.41422/InstallerData/installer.zip 
     ZGUtil.CLASS_PATH: 
     /tmp/install.dir.41422/InstallerData 
     tmp/install.dir.41422/InstallerData/installer.zip 
     sun.boot.class.path: 
     /tmp/install.dir.41422/Linux/resource/jre/lib/resources.jar /tmp/install.dir.41422/Linux/resource/jre/lib/rt.jar /tmp/install.dir.41422/Linux/resource/jre/lib/sunrsasign.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jsse.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jce.jar /tmp/install.dir.41422/Linux/resource/jre/lib/charsets.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jfr.jar /tmp/install.dir.41422/Linux/resource/jre/classes
     ```

6. В командной строке введите команду, `tmboot` чтобы перезапустить опенфраме. Выходные данные выглядят следующим образом:

     ```
     TMBOOT for node(NODE1) is starting: 
     Welcome to Tmax demo system: it will expire 2016/11/4 
     Today: 2016/9/7 
     TMBOOT: TMM is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: CLL is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: CLH is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: TLM(tlm) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrsasvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrlhsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrdmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrdsedt) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrcmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofruisvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrsmlog) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(vtammgr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjschd) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjinit) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjhist) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjspbk) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrpmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmtsmgr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(tmsvr) is starting: Wed Sep  7 17:48:53 2016
     ```

7. Убедитесь, что состояние процесса готово с помощью `tmadmin` в `si` команде. Пример:

     ```
     [oframe7\@ofdemo \~]\$ tmadmin
     ```

     В столбце **Status (состояние** ) отображается РДИ:

    ![РДИ в столбце Status (состояние)](media/tmboot-02.png)

8. Выполните следующие команды:

     ```
     $$2 NODE1 (tmadm): quit 
     DM quit for node (NODE1)

     [oframe7@ofdemo ~]$ tacfmgr 
     Input USERNAME  : ROOT 
     Input PASSWORD  : SYS1

     TACFMGR: TACF MANAGER START!!!
     QUIT TACFMGR: TACF MANAGER END!!!

     [oframe7@ofdemo ~]$ tmdow
     ```

9. Завершите работу сервера с помощью `tmdown` команды. Выходные данные выглядят следующим образом:

     ```
     [oframe7@ofdemo ~]$ tmdown 
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjschd:54) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:47) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:48) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:50) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjhist:56) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjspbk:57) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(tmsvr:60) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrpmsvr:58) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmtsmgr:59) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: CLL downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: CLH downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TLM downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TMM downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-prosort"></a>Установить сортировку

Sort — это служебная программа, используемая в пакетных транзакциях для сортировки данных.

**Установка сортировки**

1. Убедитесь, что Пакетная установка выполнена успешно, а затем убедитесь, что файл установщика **Sorting-bin-Sort \_ 2sp3-Linux64-2123-opt. tar. gz** имеется.

2. Запустите установщик с помощью файла свойств. В командной строке введите следующее:

     ```
     tar -zxvf prosort-bin-prosort\_2sp3-linux64-2123-opt.tar.gz
     ```

3. Переместите каталог сортировки в домашнюю папку. В командной строке введите следующее:

     ```
     mv prosort /opt/tmaxapp/prosort
     ```

4. Создайте подкаталог лицензии и скопируйте в него файл лицензии. Пример:

     ```
     cd /opt/tmaxapp/prosort 
     mkdir license 
     cp /opt/tmaxsw/oflicense/prosort/license.xml /opt/tmaxapp/prosort/license
     ```

5. Откройте bash. Profile в VI ( `vi .bash_profile` ) и обновите его следующим образом:

     ```bash
     #       PROSORT

     PROSORT_HOME=/opt/tmaxapp/prosort 
     PROSORT_SID=gbg 
     PATH=$PATH:$PROSORT_HOME/bin LD_LIBRARY_PATH=$PROSORT_HOME/lib:$LD_LIBRARY_PATH LIBPATH$PROSORT_HOME/lib:$LIBPATH 
     export PROSORT_HOME PROSORT_SID 
     PATH LD_LIBRARY_PATH LIBPATH 
     PATH=$PATH:$OPENFRAME_HOME/shbin 
     export PATH
     ```

6. Чтобы выполнить профиль bash, в командной строке введите: `. .bash_profile`

7. Создайте файл конфигурации. Пример:

     ```
     oframe@oframe7: cd /opt/tmaxapp/prosort/config 
     oframe@oframe7: ./gen_tip.sh 
     Using PROSORT_SID "gbg"
      /home/oframe7/prosort/config/gbg.tip generated
     ```

8. Создайте символьную ссылку. Пример:

     ```
     oframe@oframe7: cd /opt/tmaxapp/OpenFrame/util/ 
     oframe@oframe7home/oframe7/OpenFrame/util :  ln -s DFSORT SORT
     ```

9. Проверьте установку сортировки, выполнив `prosort -h` команду. Пример:

     ```
     oframe@oframe7: prosort -h

     Usage: prosort [options] [sort script files]
     options ------
     -h             Display this information 
     -v             Display version information 
     -s             Display state information 
     -j             Display profile information 
     -x             Use SyncSort compatible mode
     ```

## <a name="install-ofcobol"></a>Установка ОФКОБОЛ

ОФКОБОЛ — это компилятор Опенфраме, который интерпретирует COBOL-программы мэйнфрейма. 

**Установка ОФКОБОЛ**

1. Убедитесь, что установка пакета/в сети прошла удачно, а затем убедитесь в \_ \_ \_ наличии файла установщика опенфраме COBOL3 0 40 \_ Linux \_ x86 \_ 64. bin.

2. Чтобы выполнить установщик ОФКОБОЛ, в командной строке введите:

     ```
      ./OpenFrame\_COBOL3\_0\_40\_Linux\_x86\_64.bin
     ```

3. Прочтите лицензионное соглашение и нажмите клавишу ВВОД, чтобы продолжить.

4. Примите лицензионное соглашение. После завершения установки появится следующее:

     ```
     Choose Install Folder 
     --------------------
     Where would you like to install?
     Default Install Folder: /home/oframe7/OFCOBOL

     ENTER AN ABSOLUTE PATH, OR PRESS <ENTER> TO ACCEPT THE DEFAULT : /opt/tmaxapp/OFCOBOL

     INSTALL FOLDER IS: /opt/tmaxapp/OFCOBOL 
     IS THIS CORRECT? (Y/N): Y[oframe7@ofdemo ~]$ vi .bash_profile

     ============================================================================ Installing... 
     ------------
     [==================|==================|==================|==================] 
     [------------------|------------------|------------------|------------------]

     =============================================================================== Installation Complete 
     --------------------
     Congratulations. OpenFrame_COBOL has been successfully installed
     PRESS <ENTER> TO EXIT THE INSTALLER
     ```

5. Откройте профиль Bash в VI ( `vi .bash_profile` ) и убедитесь, что он обновлен с помощью ПЕРЕМЕННЫХ офкобол.
6. Выполните профиль bash. В командной строке введите следующее:

     ```
      source ~/.bash_profile
     ```

7. Скопируйте лицензию ОФКОБОЛ в установленную папку. Пример:
     ```
     mv licofcob.dat $OFCOB_HOME/license
     ```
8. Перейдите к файлу конфигурации Опенфраме тжклрун. conf и откройте его в VI. Пример:
     ```
     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/config 
     [oframe7@ofdemo ~]$ vi tjclrun.conf
     ```

   Ниже приведен раздел СИСЛИБ перед изменением.
     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bin LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${COBDIR}/lib:/ usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib
     ```
   Ниже приведен раздел СИСЛИБ после изменения:
     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bin LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${COBDIR}/lib:/ usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib :${ODBC_HOME}/lib 
     :${OFCOB_HOME}/lib
     ```
9. Проверьте файл Опенфраме \_ COBOL \_ InstallLog. log в VI и убедитесь в отсутствии ошибок. Пример:
     ```
     [oframe7@ofdemo ~]$ vi $OFCOB_HOME/UninstallerData/log/OpenFrame_COBOL_InstallLog.log 
     …….. 
     Summary 
     ------
     Installation: Successful. 
     131 Successes 
     0 Warnings 
     0 NonFatalErrors 
     0 FatalError
     ```
10. Используйте `ofcob --version` команду и проверьте номер версии, чтобы проверить установку. Пример:

     ```
     [oframe7@ofdemo ~]$ ofcob --version 
     OpenFrame COBOL Compiler 3.0.54 
     CommitTag:: 645f3f6bf7fbe1c366a6557c55b96c48454f4bf
     ```

11. Перезагрузите Опенфраме с помощью `tmdown/tmboot` команды.

## <a name="install-ofasm"></a>Установка ОФАСМ

ОФАСМ — это компилятор Опенфраме, который интерпретирует программы ассемблера мэйнфреймов.

**Установка ОФАСМ**

1. Убедитесь, что установка пакета/в сети прошла удачно, а затем убедитесь в \_ \_ наличии файла установщика опенфраме ASM3 0 для \_ Linux \_ x86 \_ 64. bin.

2. Запустите установщик. Пример:

     ```
     [oframe7@ofdemo ~]$ ./OpenFrame_ASM3_0_Linux_x86_64.bin
     ```

3. Прочтите лицензионное соглашение и нажмите клавишу ВВОД, чтобы продолжить.
4. Примите лицензионное соглашение.
5. Убедитесь, что профиль Bash обновлен с помощью переменных ОФАСМ. Пример:

     ```
     [oframe7@ofdemo ~]$ source .bash_profile
     [oframe7@ofdemo ~]$ ofasm --version 
     # TmaxSoft OpenFrameAssembler v3 r328 
     (3ff35168d34f6e2046b96415bbe374160fcb3a34)

     [oframe7@ofdemo OFASM]$ vi .bash_profile

     # OFASM ENV 
     export OFASM_HOME=/opt/tmaxapp/OFASM 
     export OFASM_MACLIB=$OFASM_HOME/maclib/free_macro 
     export PATH="${PATH}:$OFASM_HOME/bin:" 
     export LD_LIBRARY_PATH="./:$OFASM_HOME/lib:$LD_LIBRARY_PATH"
     ```

6. Откройте файл конфигурации Опенфраме тжклрун. conf в VI и измените его следующим образом:

     ```
     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/config 
     [oframe7@ofdemo ~]$ vi tjclrun.conf
     ```

     Вот раздел [СИСЛИБ] *перед* изменением:

     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bi n:${OPENFRAME_HOME}/volume_default/SYS1.LOADLIB LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${CO BDIR}/lib:/usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib:${OFCOB_HOM E}/lib:${ODBC_HOME}/lib:${OFPLI_HOME}/lib
     ```

     Ниже приведен раздел [СИСЛИБ] *после* изменения:

     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bi n:${OPENFRAME_HOME}/volume_default/SYS1.LOADLIB LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${CO BDIR}/lib:/usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib:${OFCOB_HOM E}/lib:${ODBC_HOME}/lib:${OFPLI_HOME}/lib:${OFASM_HOME}/lib
     ```

7. Откройте файл Опенфраме \_ ASM \_ InstallLog. log в VI и убедитесь в отсутствии ошибок. Пример:

     ```
     [oframe7@ofdemo ~]$ vi 
     $OFASM_HOME/UninstallerData/log/OpenFrame_ASM_InstallLog.log 
     …….. 
     Summary 
     ------

     Installation: Successful.

     55 Successes 
     0 Warnings 
     0 NonFatalErrors 
     0 FatalErrors
     ```

8. Перезагрузите Опенфраме, выполнив одну из следующих команд:

     ```
     tmdown / tmboot
     ```

     —или—

     ```
     oscdown / oscboot
     ```

## <a name="install-osc"></a>Установка OSC

OSC — это среда Опенфраме, аналогичная IBM CICS, которая поддерживает высокоскоростные транзакции OLTP и другие функции управления.

**Установка OSC**

1. Убедитесь, что базовая установка прошла удачно, а затем убедитесь, что \_ \_ файл конфигурации опенфраме OSC7 0 \_ Fix2 \_ Linux \_ x86 \_ 64. bin установлен в настоящее время.
2. Измените следующие параметры в файле OSC. Properties:
     ```
     OPENFRAME_HOME=/opt/tmaxapp/OpenFrame OSC_SYS_OSC_NCS_PATH=/opt/tmaxapp/OpenFrame/temp/OSC_NCS OSC_APP_OSC_TC_PATH=/opt/tmaxapp/OpenFrame/temp/OSC_TC
     ```

3. Запустите установщик, используя файл свойств, как показано ниже.

     ```
     [oframe7@ofdemo ~]$ chmod a+x OpenFrame_OSC7_0_Fix2_Linux_x86_64.bin [oframe7@ofdemo ~]$ ./OpenFrame_OSC7_0_Fix2_Linux_x86_64.bin -f osc.properties
     ```

     По завершении появится сообщение "Установка завершена".

4. Убедитесь, что профиль Bash обновлен с помощью переменных OSC.
5. Проверьте файл Опенфраме \_ OSC7 \_ 0 \_ Fix2 \_ InstallLog. log. Должно отобразиться примерно следующее:

     ```
     Summary 
     ------ 
     Installation: Successful.

     233 Successes
     0 Warnings
     0 NonFatalErrors
     0 FatalError
     ```

6. Откройте файл конфигурации офсис. seq с помощью VI. Пример:

     ```
     vi $OPENFRAME_HOME/config/ofsys.seq
     ```

7. В \# разделах базовый и \# пакет измените параметры, как показано ниже.

     ```
     Before changes
     #BASE
     ofrsasvr
     ofrlhsvr
     ofrdmsvr
     ofrdsedt
     ofrcmsvr
     ofruisvr
     ofrsmlog
     vtammgr
     TPFMAGENT

     #BATCH 
     #BATCH#obmtsmgr
     #BATCH#ofrpmsvr
     #BATCH#obmjmsvr
     #BATCH#obmjschd
     #BATCH#obmjinit
     #BATCH#obmjhist
     #BATCH#obmjspbk
     #TACF #TACF#tmsvr

     After changes  #BATCH
     #BASE          obmtsmgr 
     ofrsasvr       ofrpmsvr
     ofrlhsvr       obmjmsvr
     ofrdmsvr       obmjschd
     ofrdsedt       obmjinit
     ofrcmsvr       obmjhist
     ofruisvr       obmjspbk
     ofrsmlog
     vtammgr        #TACF
     TPFMAGENT      tmsvr
    ```

8. Скопируйте файл лицензии. Пример:

     ```
     [oframe7@ofdemo ~]$ cp /home/oframe7/oflicense/ofonline/licosc.dat 

     $OPENFRAME_HOME/license

     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/license 
     oframe@oframe7/OpenFrame/license / ls -l 
     -rwxr-xr-x. 1 oframe mqm 80 Sep 12 01:37 licosc.dat 
     -rwxr-xr-x. 1 oframe mqm 80 Sep  8 09:40 lictacf.dat 
     -rwxrwxr-x. 1 oframe mqm 80 Sep  3 11:54 lictjes.da
     ```

9. Чтобы запустить и завершить работу OSC, инициализируйте общую память CICS Region, введя `osctdlinit OSCOIVP1` в командной строке.

10. Выполните команду `oscboot` , чтобы загрузить OSC. Выходные данные выглядят следующим образом:

     ```
     OSCBOOT : pre-processing       [ OK ]

     TMBOOT for node(NODE1) is starting: 
     Welcome to Tmax demo system: it will expire 2016/11/4 
     Today: 2016/9/12 
          TMBOOT: TMM is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: CLL is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: CLH is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: TLM(tlm) is starting: Mon Sep 12 01:40:25 2016 
     ```

11. Чтобы убедиться, что состояние процесса — "Готово", используйте `tmadmin` команду в Si. Все процессы должны отображать РДИ в столбце **Status (состояние** ).

    ![Процессы, отображающие РДИ](media/tmadmin-02.png)

12. Завершите работу OSC с помощью `oscdown` команды.

## <a name="install-jeus"></a>Установка ЖЕУС

ЖЕУС (решение для пользователей Java Enterprise) предоставляет уровень представления для сервера веб-приложений Опенфраме.

Перед установкой ЖЕУС установите пакет Apache Ant, который предоставляет библиотеки и программы командной строки, необходимые для установки ЖЕУС.

**Установка Apache Ant**

1. Скачайте двоичный файл Ant с помощью `wget` команды. Пример:

     ```
     wget http://apache.mirror.cdnetworks.com/ant/binaries/apacheant-1.9.7-bin.tar.gz
     ```

2. С помощью `tar` программы извлеките двоичный файл и переместите его в соответствующее расположение. Пример:

     ```
     tar -xvzf apache-ant-1.9.7-bin.tar.gz
     ```

3. Для повышения эффективности создайте символьную ссылку:

     ```
     ln -s apache-ant-1.9.7 ant
     ```

4. Откройте профиль Bash в VI ( `vi .bash_profile` ) и обновите его, используя следующие переменные:

     ```
     # Ant ENV
     export ANT_HOME=$HOME/ant 
     export PATH=$HOME/ant/bin:$PATH
     ```

5.  Примените измененную переменную среды. Пример:

     ```
     [oframe7\@ofdemo \~]\$ source \~/.bash\_profile
     ```

**Установка ЖЕУС**

1. Разверните установщик с помощью `tar` программы. Пример:

     ```
     [oframe7@ofdemo ~]$ tar -zxvf jeus704.tar.gz
     ```

2. Создайте папку **жеус** ( `mkdir jeus7` ) и распакуйте двоичный файл.
3. Перейдите в каталог **установки** (или используйте параметр жеус для своей среды). Пример:

     ```
     [oframe7@ofdemo ~]$ cd jeus7/setup/
     ```

4. Выполните `ant clean-all` перед выполнением сборки. Выходные данные выглядят следующим образом:

     ```
     Buildfile: /home/oframe7jeus7/setup/build.xml

     clean-bin:
     delete-domain:
     [echo] Deleting a domain configuration: domain = jeus_domain
     delete-nodesxml:
     clean-config:
     clean-all:
     BUILD SUCCESSFUL
     Total time: 0 seconds
     ```

5.  Создайте резервную копию файла Domain-config-Template. Properties. Пример:

     ```
     [oframe7@ofdemo ~]$ cp domain-config-template.properties domain-configtemplate.properties.bkp
     ```

6. Откройте файл Domain-config-Template. Properties в VI:

     ```
     [oframe7\@ofdemo setup]\$ vi domain-config-template.properties
     ```

7. Измените `jeus.password=jeusadmin nodename=Tmaxsoft` на `jeus.password=tmax1234 nodename=ofdemo`.

8. Выполните `ant install` команду, чтобы создать жеус.
9.  Обновите \_ файл профиля bash с помощью ПЕРЕМЕННЫХ жеус, как показано ниже.

     ```
     # JEUS ENV 
     export JEUS_HOME=/opt/tmaxui/jeus7 PATH="/opt/tmaxui/jeus7/bin:/opt/tmaxui/jeus7/lib/system:/opt/tmaxui/jeus7/webserver/bin:$ {PATH}" 
     export PATH
     ```

10. Выполните профиль bash. Пример:

     ```
     [oframe7@ofdemo setup]$ . .bash_profile
     ```

11. *Необязательно*. Создайте псевдоним для упрощения работы и загрузки компонентов ЖЕУС:

     ```     
     # JEUS alias

     alias dsboot='startDomainAdminServer -domain jeus_domain -u administrator -p jeusadmin'
     alias msboot='startManagedServer -domain jeus_domain -server server1 -u administrator -p jeusadmin' 
     alias msdown=‘jeusadmin -u administrator -p tmax1234 "stop-server server1“’ 
     alias dsdown=‘jeusadmin -domain jeus_domain -u administrator -p tmax1234 "local-shutdown“’
     ```

12. Чтобы проверить установку, запустите сервер администратора домена, как показано ниже.

     ```
     [oframe7@ofdemo ~]$ startDomainAdminServer -domain jeus_domain -u administrator -p jeusadmin
     ```

13. Проверьте веб-вход с помощью синтаксиса:

     ```
     http://<IP>:<port>/webadmin/login
     ```

     Например, <http://192.168.92.133:9736/webadmin/login.> экран входа в систему выглядит следующим образом:
    
     ![Экран входа в ЖЕУС Вебадмин](media/jeus-01.png)

     > [!NOTE]
     > При возникновении проблем с безопасностью портов откройте порт 9736 или отключите брандмауэр ( `systemctl stop firewall` ).

14. Чтобы изменить имя узла для Server1, щелкните **блокировка & изменить**, а затем выберите **Server1**. В окне сервера измените имя узла следующим образом:

    1.  Измените **nodeName** на **офдемо**.
    2.  Нажмите кнопку **ОК** в правой части окна.
    3.  Щелкните **Применить изменения** в нижней левой части окна и в поле Описание введите *изменение имени узла*.

    ![Экран ЖЕУС Вебадмин](media/jeus-02.png)

15. Убедитесь, что конфигурация выполнена успешно на экране подтверждения.

    ![экран сервера jeus_domain](media/jeus-03.png)

16. Запустите процесс управляемого сервера "Server1" с помощью следующей команды:

     ```
     [oframe7@ofdemo ~]$ startManagedServer -domain jeus_domain -server server1 -u administrator -p jeusadmin
     ```

## <a name="install-ofgw"></a>Установка ОФГВ

ОФГВ — это шлюз Опенфраме, который поддерживает обмен данными между эмулятором терминала 3270 и базой OSI и управляет сеансами между эмулятором терминала и OSI.

**Установка ОФГВ**

1. Убедитесь, что ЖЕУС успешно установлен, а затем убедитесь в \_ \_ \_ наличии файла установщика OFGW7 0 1 Generic. bin.
2. Запустите установщик. Пример:

     ```
     [oframe7@ofdemo ~]$ ./OFGW7_0_1_Generic.bin
     ````

3. Для соответствующих запросов используйте следующие расположения:
     -   Корневой каталог ЖЕУС
     -   ЖЕУС доменное имя
     -   Имя сервера ЖЕУС
     -   Драйвер тиберо
     -   Идентификатор тмакс узла офдемо

4. Примите остальные значения по умолчанию, а затем нажмите клавишу ВВОД, чтобы выйти из установщика.

5. Убедитесь, что URL-адрес для ОФГВ работает должным образом:

     ```
     Type URL 
     http://192.168.92.133:8088/webterminal/ and press enter
      < IP >               :8088/webterminal/
     ```

     Появится следующий экран:

    ![Опенфраме](media/ofgw-01.png)

## <a name="install-ofmanager"></a>Установка Офманажер

Офманажер предоставляет функции операций и управления для Опенфраме в веб-среде.

**Установка Офманажер**

1. Убедитесь в \_ наличии файла установщика Generic. bin OFManager7.
2. Запустите установщик. Пример:

     ```
     OFManager7_Generic.bin
     ```

3.  Нажмите клавишу ВВОД, чтобы продолжить, а затем примите условия лицензионного соглашения.
4.  Выберите папку установки.
5.  Примите значения по умолчанию.
6.  Выберите Тиберо в качестве базы данных.
7.  Нажмите клавишу ВВОД, чтобы выйти из установщика.
8.  Убедитесь, что URL-адрес для Офманажер работает должным образом:

     ```
     Type URL http://192.168.92.133:8088/ofmanager and press enter <  IP >  : < PORT >  ofmanager Enter ID:   ROOT 
     Password: SYS1
     ```

Появится начальный экран:

![Экран входа в тмакс Опенфраме Manager](media/ofmanager-01.png)

Это завершает установку компонентов Опенфраме.

## <a name="next-steps"></a>Дальнейшие действия

Если вы планируете перенос мэйнфреймов, мы можем помочь вам. См. дополнительные сведение о критериях выбора решений от партнеров в статье [Platform Modernization Alliance](https://datamigration.microsoft.com/) (Альянс модернизации платформы).

-   [Начало работы с Azure](../../../../index.yml)
-   [Документация по Host Integration Server (HIS)](/host-integration-server/)
-   [Центр обработки виртуальных данных Azure — инструкции по смене и использованию](/archive/blogs/azurecat/new-whitepaper-azure-virtual-datacenter-lift-and-shift-guide)
