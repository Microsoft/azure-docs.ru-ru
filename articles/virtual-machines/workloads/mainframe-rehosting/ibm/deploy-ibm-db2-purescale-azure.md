---
title: Развертывание IBM DB2 pureScale в Azure
description: Узнайте, как развернуть пример архитектуры, который недавно использовался для переноса данных предприятия из среды IBM DB2 на z/OS в IBM DB2 pureScale в Azure.
author: njray
ms.service: virtual-machines
ms.subservice: mainframe-rehosting
ms.topic: how-to
ms.date: 11/09/2018
ms.author: edprice
ms.openlocfilehash: 33ff6174d7e5107076dda177731c9daec7e57266
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104956151"
---
# <a name="deploy-ibm-db2-purescale-on-azure"></a>Развертывание IBM DB2 pureScale в Azure

В этой статье описано развертывание [примера архитектуры](ibm-db2-purescale-azure.md), который недавно использовался клиентом предприятия для переноса данных из среды IBM DB2 на z/OS в IBM DB2 pureScale в Azure.

Чтобы выполнить шаги, используемые для миграции, воспользуйтесь скриптами установки в репозитории [DB2onAzure](https://aka.ms/db2onazure) на сайте GitHub. Эти скрипты основаны на архитектуре для обычной рабочей нагрузки OLTP среднего размера.

## <a name="get-started"></a>Начало работы

Чтобы развернуть эту архитектуру, скачайте и запустите сценарий deploy.sh, который находится в репозитории [DB2onAzure](https://aka.ms/db2onazure) на сайте GitHub.

Репозиторий также содержит скрипты для настройки панели мониторинга Grafana. Вы можете использовать панель мониторинга для запроса Prometheus, системы мониторинга и оповещения с открытым кодом и DB2.

> [!NOTE]
> Скрипт deploy.sh в клиенте создает закрытые ключи SSH и передает их в шаблон развертывания через HTTPS. Для безопасного хранения секретов, ключей и паролей рекомендуем использовать [Azure Key Vault](../../../../key-vault/general/overview.md).

## <a name="how-the-deployment-script-works"></a>Как работает скрипт развертывания

Скрипт Deploy.sh создает и настраивает ресурсы Azure для архитектуры. Скрипт запрашивает подписку Azure и виртуальные машины, используемые в целевой среде, а затем выполняет следующие операции:

-   настройка группы ресурсов, виртуальной сети и подсети в Azure для установки;

-   Настраивает группы безопасности сети и SSH для среды.

-   Настраивает несколько сетевых карт как для общего хранилища, так и для виртуальных машин Пурескале DB2.

-   Создает виртуальные машины общего хранилища. Если вы используете Локальные дисковые пространства или другое решение для хранения данных, см. раздел [Общие сведения о Локальные дисковые пространства](/windows-server/storage/storage-spaces/storage-spaces-direct-overview).

-   создание виртуальной машины jumpbox;

-   Создает виртуальные машины DB2 Пурескале.

-   Создает виртуальную машину-свидетель, которую Пурескале с помощью DB2. Пропустите эту часть развертывания, если для вашей версии DB2 Пурескале не требуется следящий сервер.

-   Создает виртуальную машину Windows для использования при тестировании, но не устанавливает что-либо на нее.

После этого скрипты развертывания создают виртуальную сеть хранения данных iSCSI (vSAN) для подключения к общему хранилищу в Azure. В этом примере iSCSI подключается к общему кластеру хранилища. В исходном решении клиента было использовано Глустерфс. Однако IBM больше не поддерживает такой подход. Чтобы поддерживать поддержку от IBM, необходимо использовать поддерживаемую iSCSI-совместимую файловую систему. Корпорация Майкрософт предлагает Локальные дисковые пространства (S2D) в качестве варианта.

Это решение также дает возможность установить цели iSCSI как один узел Windows. Сеть iSCSI предоставляет интерфейс общего блочного хранилища по TCP/IP, что позволяет процедуре установки DB2 pureScale использовать интерфейс устройства для подключения к общему хранилищу.

Скрипты развертывания выполняют следующие общие действия:

1.  Настройка кластера общего хранилища в Azure. При этом используется по крайней мере два узла Linux.

2.  Настройте прямой интерфейс iSCSI на целевых серверах Linux для кластера общих хранилищ.

3.  Настройка инициатора iSCSI на виртуальных машинах Linux. Инициатор будет получать доступ к общему кластеру хранилища с помощью цели iSCSI. Сведения о настройке см. в статье [How To Configure An iSCSI Target And Initiator In Linux](https://www.rootusers.com/how-to-configure-an-iscsi-target-and-initiator-in-linux/) (Настройка цели и инициатора iSCSI в Linux) документации по RootUsers.

4.  Установите общий уровень хранилища для интерфейса iSCSI.

После создания устройства iSCSI с помощью скрипта остается установить DB2 pureScale. В рамках установки Пурескале DB2 [масштаб в IBM спектров](https://www.ibm.com/support/knowledgecenter/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/t0057167.html) (прежнее название — гпфс) компилируется и устанавливается в кластере глустерфс. Эта кластерная файловая система позволяет DB2 pureScale обеспечить совместное использование данных на виртуальных машинах с подсистемой DB2 pureScale. Дополнительные сведения см. в документации по [IBM Spectrum Scale](https://www.ibm.com/support/knowledgecenter/en/STXKQY_4.2.0/ibmspectrumscale42_welcome.html) на сайте IBM.

## <a name="db2-purescale-response-file"></a>Файл ответов DB2 pureScale

Репозиторий GitHub включает DB2server.rsp, файл ответов, который позволяет создать автоматизированный скрипт для установки DB2 pureScale. В приведенной ниже таблице указаны параметры DB2 pureScale, которые файл ответов использует для установки. Вы можете настроить файл ответов для своей среды.

> [!NOTE]
> Пример файла ответов DB2server. rsp включен в репозиторий [DB2onAzure](https://aka.ms/db2onazure) на сайте GitHub. Чтобы этот файл работал в вашей среде, его сначала необходимо отредактировать.

| Имя экрана               | Поле                                        | Значение                                                                                                 |
|---------------------------|----------------------------------------------|-------------------------------------------------------------------------------------------------------|
| Экран приветствия                   |                                              | "New Install" (Новая установка)                                                                                           |
| "Choose a Product" (Выбор продукта)          |                                              | "DB2 Version 11.1.3.3. Server Editions with Db2 pureScale" (Db2 версии 11.1.3.3, серверные выпуски с Db2 pureScale)                                              |
| Конфигурация             | Каталог                                    | /data1/opt/ibm/db2/V11.1                                                                              |
|                           | "Select the installation type" (Выбор типа установки)                 | "Typical" (Стандартный)                                                                                               |
|                           | "I agree to the IBM terms" (Я принимаю условия IBM)                     | Флажок установлен                                                                                               |
| "Instance Owner" (Владелец экземпляра)            | "Existing User For Instance, User name" (Существующий пользователь экземпляра, имя пользователя)        | DB2sdin1                                                                                              |
| "Fenced User" (Изолированный пользователь)               | "Existing User, User name" (Существующий пользователь, имя пользователя)                     | DB2sdfe1                                                                                              |
| "Cluster File System" (Файловая система кластера)       | "Shared disk partition device path" (Путь к устройству с общим разделом диска)            | /dev/dm-2                                                                                             |
|                           | "Mount point" (Точка подключения)                                  | /DB2sd\_1804a                                                                                         |
|                           | "Shared disk for data" (Общий диск для данных)                         | /dev/dm-1                                                                                             |
|                           | "Mount point (Data)" (Точка подключения, данные)                           | /DB2fs/datafs1                                                                                        |
|                           | "Shared disk for log" (Общий диск для журнала)                          | /dev/dm-0                                                                                             |
|                           | "Mount point (Log)" (Точка подключения, журнал)                            | /DB2fs/logfs1                                                                                         |
|                           | "DB2 Cluster Services Tiebreaker. Device path" (Схема разрешения конфликтов служб кластеров Db2, путь к устройству) | /dev/dm-3                                                                                             |
| "Host List" (Список узлов)                 | d1 [eth1], d2 [eth1], cf1 [eth1], cf2 [eth1] |                                                                                                       |
|                           | "Preferred primary CF" (Предпочтительный основной CF)                         | cf1                                                                                                   |
|                           | "Preferred secondary CF" (Предпочтительный дополнительный CF)                       | cf2                                                                                                   |
| "Response File and Summary" (Файл ответов и сводка) | Первый параметр                                 | "Install DB2 Server Edition with the IBM DB2 pureScale feature and save my settings in a response file" (Установить DB2 Server Edition с функцией IBM DB2 pureScale и сохранить параметры в файле ответов) |
|                           | "Response file name" (Имя файла ответов)                           | /root/DB2server.rsp                                                                                   |

### <a name="notes-about-this-deployment"></a>Заметки об этом развертывании

- Значения /dev-dm0, /dev-dm1, /dev-dm2 и /dev-dm3 могут измениться после перезагрузки на виртуальной машине, где выполняется установка (d0 в автоматизированном скрипте). Чтобы найти правильные значения, перед заполнением файла ответов выполните следующую команду на сервере, на котором будет выполняться установка:

   ```
   [root\@d0 rhel]\# ls -als /dev/mapper
   total 0
   0 drwxr-xr-x 2 root root 140 May 30 11:07 .
   0 drwxr-xr-x 19 root root 4060 May 30 11:31 ..
   0 crw------- 1 root root 10, 236 May 30 11:04 control
   0 lrwxrwxrwx 1 root root 7 May 30 11:07 db2data1 -\> ../dm-1
   0 lrwxrwxrwx 1 root root 7 May 30 11:07 db2log1 -\> ../dm-0
   0 lrwxrwxrwx 1 root root 7 May 30 11:26 db2shared -\> ../dm-2
   0 lrwxrwxrwx 1 root root 7 May 30 11:08 db2tieb -\> ../dm-3
   ```

- В скриптах установки для дисков iSCSI используются псевдонимы, чтобы можно было легко найти фактические имена.

- При запуске скрипта установки на d0 значения **/dev/dm-\*** могут отличаться на d1, cf0 и cf1. Разница между значениями не влияет на настройки pureScale DB2.

## <a name="troubleshooting-and-known-issues"></a>Устранение неполадок и известные проблемы

Репозиторий GitHub включает базу знаний, которую ведут авторы. В ней вы найдете потенциальные проблемы, которые могут возникнуть, и способы их решения. Например, известные проблемы могут возникнуть в таких случаях:

-   вы пытаетесь связаться с IP-адресом шлюза;

-   вы компилируете General Public License (GPL);

-   сбое подтверждения безопасности между узлами;

-   при обнаружении установщиком DB2 имеющейся файловой системы;

-   вы вручную устанавливаете IBM Spectrum Scale;

-   вы устанавливаете DB2 pureScale, когда IBM Spectrum Scale уже создан;

-   вы удаляете DB2 pureScale и IBM Spectrum Scale.

Дополнительные сведения об этих и других известных проблемах см. в файле kb.md в репозитории [DB2onAzure](https://aka.ms/DB2onAzure).

## <a name="next-steps"></a>Дальнейшие действия

-   [Создание необходимых пользователей для установки компонентов Пурескале DB2](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/t0055374.html?pos=2)

-   [DB2icrt — команда создания экземпляра](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.cmd.doc/doc/r0002057.html)

-   [Решение данных кластеров DB2 Пурескале](https://www.ibmbigdatahub.com/blog/db2-purescale-clustered-database-solution-part-1)

-   [IBM Data Studio](https://www.ibm.com/developerworks/downloads/im/data/index.html/)

-   [Виртуальный центр обработки данных Azure: руководство по миграции методом lift-and-shift](https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/)
