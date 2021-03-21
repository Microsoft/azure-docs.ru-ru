---
title: Настройка IBM DB2 HADR на виртуальных машинах Azure | Документация Майкрософт
description: Установите высокий уровень доступности IBM DB2 LUW на виртуальных машинах Azure.
author: msjuergent
ms.service: virtual-machines-sap
ms.topic: article
ms.date: 10/16/2020
ms.author: juergent
ms.reviewer: cynthn
ms.openlocfilehash: faafce32c3452a5c4ff08783ec2edd28f7f961e9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101671886"
---
# <a name="high-availability-of-ibm-db2-luw-on-azure-vms-on-suse-linux-enterprise-server-with-pacemaker"></a>Высокий уровень доступности IBM DB2 LUW на виртуальных машинах Azure на SUSE Linux Enterprise Server с Pacemaker

IBM DB2 для Linux, UNIX и Windows (LUW) в [конфигурации высокой доступности и аварийного восстановления (HADR)](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html) состоит из одного узла, на котором выполняется экземпляр базы данных-источника и по крайней мере одного узла, на котором выполняется экземпляр базы данных-получателя. Изменения в экземпляре базы данных-источника реплицируются в экземпляр базы данных получателя синхронно или асинхронно, в зависимости от конфигурации. 

> [!NOTE]
> Эта статья содержит ссылки на *основные* и *подчиненные* термины, которые корпорация Майкрософт больше не использует. При удалении этих терминов из программного обеспечения мы удалим их из этой статьи.
   
В этой статье описывается, как развернуть и настроить виртуальные машины Azure, установить платформу кластера и установить конфигурацию IBM DB2 LUW с HADR. 

В этой статье не рассматривается установка и настройка IBM DB2 LUW с установкой программного обеспечения HADR или SAP. Чтобы помочь вам в выполнении этих задач, мы предоставляем ссылки на руководства по установке SAP и IBM. В этой статье рассматриваются части, относящиеся к среде Azure. 

Поддерживаются версии IBM DB2 10,5 и более поздних версий, как описано в примечании к SAP [1928533].

Перед началом установки ознакомьтесь со следующими примечаниями и документацией по SAP:

| Примечание SAP | Описание |
| --- | --- |
| [1928533] | Приложения SAP в Azure: поддерживаемые продукты и типы виртуальных машин Azure |
| [2015553] | SAP в Azure: предварительные требования для поддержки |
| [2178632] | Ключевые метрики мониторинга для SAP в Azure |
| [2191498] | SAP на платформе Linux в Azure: расширенный мониторинг |
| [2243692] | Виртуальная машина Linux на платформе Azure (IaaS): проблемы с лицензией SAP |
| [1984787] | SUSE Linux Enterprise Server 12 Замечания по установке |
| [1999351] | Устранение неполадок, связанных с расширенным мониторингом Azure для SAP |
| [2233094] | DB6: приложения SAP в Azure, использующие IBM DB2 для Linux, UNIX и Windows — Дополнительные сведения |
| [1612105] | DB6: часто задаваемые вопросы об DB2 с HADR |


| Документация | 
| --- |
| [Вики-сайт сообщества SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes): содержит все необходимые примечания SAP для Linux. |
| Руководств по [планированию и внедрению виртуальных машин Azure для SAP в Linux][planning-guide] |
| [Развертывание виртуальных машин Azure для SAP в Linux][deployment-guide] (Эта статья) |
| Руководство по [развертыванию системы управления базами данных на виртуальных машинах Azure (СУБД) для SAP в Linux][dbms-guide] |
| [Контрольный список для планирования и развертывания рабочей нагрузки SAP в Azure][azr-sap-plancheck] |
| [Руководства по SUSE Linux Enterprise Server для приложений SAP 12 SP4 рекомендации][sles-for-sap-bp] |
| [Расширение 4 (SP4) для SUSE Linux Enterprise с высоким уровнем доступности][sles-ha-guide] |
| [Развертывание СУБД IBM DB2 на Виртуальных машинах Azure для рабочей нагрузки SAP][dbms-db2] |
| [IBM DB2 HADR 11,1][db2-hadr-11.1] |
| [IBM DB2 HADR R 10,5][db2-hadr-10.5] |

## <a name="overview"></a>Обзор
Для обеспечения высокой доступности IBM DB2 LUW с HADR устанавливается по крайней мере на двух виртуальных машинах Azure, которые развертываются в [группе доступности Azure](../../windows/tutorial-availability-sets.md) или в [зоны доступности Azure](./sap-ha-availability-zones.md). 

На следующих рисунках показано, как настроить две виртуальные машины Azure сервера базы данных. Обе виртуальные машины Azure сервера базы данных имеют собственное хранилище, подключенное и работающее. В HADR один экземпляр базы данных на одной из виртуальных машин Azure имеет роль основного экземпляра. Все клиенты подключены к этому основному экземпляру. Все изменения в транзакциях базы данных сохраняются локально в журнале транзакций DB2. По мере сохранения записей журнала транзакций записи передаются через протокол TCP/IP на экземпляр базы данных на втором сервере базы данных, резервном сервере или резервном экземпляре. Резервный экземпляр обновляет локальную базу данных, выполнив накат переданных записей журнала транзакций. Таким образом резервный сервер остается синхронизированным с сервером-источником.

HADR — это только функция репликации. Он не имеет возможности обнаружения сбоев и автоматического перенаправление или отработки отказа. Администратор базы данных должен вручную инициировать перенаправление или передавать на резервный сервер. Чтобы обеспечить автоматическое перенаправление и обнаружение сбоев, можно использовать функцию кластеризации Pacemaker в Linux. Pacemaker отслеживает два экземпляра сервера базы данных. При аварийном завершении экземпляра сервера базы данных Pacemaker инициирует *Автоматический* перенаправление HADR на резервном сервере. Pacemaker также гарантирует, что виртуальный IP-адрес будет назначен новому основному серверу.

![Обзор высокой доступности IBM DB2](./media/dbms-guide-ha-ibm/ha-db2-hadr-lb.png)

Чтобы серверы приложений SAP подключались к базе данных-источнику, требуется имя виртуального узла и виртуальный IP-адрес. В случае отработки отказа серверы приложений SAP будут подключаться к новому экземпляру базы данных-источника. В среде Azure [балансировщик нагрузки Azure](https://microsoft.sharepoint.com/teams/WAG/AzureNetworking/Wiki/Load%20Balancing.aspx) требуется для использования виртуального IP-адреса так, как это требуется для HADR IBM DB2. 

На следующем рисунке представлен обзор высокодоступной настройки системы SAP на основе базы данных IBM DB2, которая поможет вам полностью понять, как IBM DB2 LUW с HADR и Pacemaker соответствует требованиям к высокодоступной настройке системы SAP. В этой статье рассматривается только IBM DB2, но содержатся ссылки на другие статьи о настройке других компонентов системы SAP.

![Обзор полной среды высокой доступности IBM DB2](.//media/dbms-guide-ha-ibm/end-2-end-ha.png)


### <a name="high-level-overview-of-the-required-steps"></a>Высокоуровневый обзор необходимых действий
Чтобы развернуть конфигурацию IBM DB2, необходимо выполнить следующие действия.

  + Спланируйте среду.
  + Разверните виртуальные машины.
  + Обновление SUSE Linux и настройка файловых систем.
  + Установите и настройте Pacemaker.
  + Установите [NFS высокой доступности][nfs-ha].
  + Установите [ASCS/ERS в отдельном кластере][ascs-ha].
  + Установите базу данных IBM DB2 с параметром распределенной или высокой доступности (SWPM).
  + Установите и создайте узел базы данных-получателя и экземпляр, а затем настройте транспорт HADR.
  + Убедитесь, что транспорт HADR работает.
  + Примените конфигурацию Pacemaker для управления IBM DB2.
  + Настройте Azure Load Balancer.
  + Установка основных и диалоговых серверов приложений.
  + Проверьте и адаптируйте конфигурацию серверов приложений SAP.
  + Выполнение тестов отработки отказа и перенаправление.



## <a name="plan-azure-infrastructure-for-hosting-ibm-db2-luw-with-hadr"></a>Планирование инфраструктуры Azure для размещения IBM DB2 LUW с помощью HADR

Завершите процесс планирования перед выполнением развертывания. Планирование строит основу для развертывания конфигурации DB2 с помощью HADR в Azure. Ключевые элементы, которые должны быть частью планирования для IMB DB2 LUW (часть базы данных в среде SAP), перечислены в следующей таблице:

| Раздел | Краткое описание |
| --- | --- |
| Определение групп ресурсов Azure | Группы ресурсов, в которых развертывается виртуальная машина, виртуальная сеть, Azure Load Balancer и другие ресурсы. Может быть существующим или новым. |
| Определение виртуальной сети или подсети | Где развертываются виртуальные машины для IBM DB2 и Azure Load Balancer. Может быть существующим или вновь созданным. |
| Виртуальные машины, на которых размещены IBM DB2 LUW | Размер виртуальной машины, хранилище, сеть, IP-адрес. |
| Имя виртуального узла и виртуальный IP-адрес для базы данных IBM DB2| Виртуальный IP-адрес или имя узла, используемые для подключения серверов приложений SAP. **DB-virt-hostname**, **DB-virt-IP**. |
| Ограждение Azure | Ограждение Azure или ограждение SBD (настоятельно рекомендуется). , Чтобы избежать ситуаций с разделением. |
| ВИРТУАЛЬНАЯ МАШИНА SBD | SBD размер виртуальной машины, хранилище, сеть. |
| Azure Load Balancer | Использование уровня "базовый" или "Стандартный" (рекомендуется), порт пробы для базы данных DB2 (наш Совет 62500) **пробный порт**. |
| Разрешение имен| Как работает разрешение имен в среде. Настоятельно рекомендуется использовать службу DNS. Можно использовать локальный файл hosts. |
    
Дополнительные сведения о Pacemaker Linux в Azure см. в статье [Настройка Pacemaker на SUSE Linux Enterprise Server в Azure](./high-availability-guide-suse-pacemaker.md).

## <a name="deployment-on-suse-linux"></a>Развертывание в SUSE Linux

Агент ресурсов для IBM DB2 LUW включен в SUSE Linux Enterprise Server для приложений SAP. Для настройки, описанной в этом документе, необходимо использовать SUSE Linux Server для приложений SAP. Azure Marketplace содержит образ для SUSE Enterprise Server для приложений SAP 12, который можно использовать для развертывания новых виртуальных машин Azure. При выборе образа виртуальной машины в Azure Marketplace необходимо учитывать различные модели поддержки или служб, предлагаемые SUSE в Azure Marketplace. 

### <a name="hosts-dns-updates"></a>Узлы: обновления DNS
Создайте список всех имен узлов, включая имена виртуальных узлов, и обновите DNS-серверы, чтобы включить правильный IP-адрес для разрешения имен узлов. Если DNS-сервер не существует или вы не можете обновлять и создавать записи DNS, необходимо использовать файлы локального узла отдельных виртуальных машин, участвующих в этом сценарии. Если вы используете записи файлов узлов, убедитесь, что записи применяются ко всем виртуальным машинам в среде системы SAP. Однако рекомендуется использовать DNS, который в идеале расширяется в Azure


### <a name="manual-deployment"></a>Развертывание вручную

Убедитесь, что выбранная ОС поддерживается IBM DB2 LUW для IBM/SAP. Список поддерживаемых версий ОС для виртуальных машин Azure и выпусков DB2 доступен в примечании SAP [1928533]. Список выпусков ОС отдельных версий DB2 доступен в матрице доступности продуктов SAP. Мы настоятельно рекомендуем использовать как минимум SLES 12 SP4 из-за улучшений производительности, связанных с Azure, в этой или более поздних версиях SUSE Linux.

1. Создайте или выберите группу ресурсов.
1. Создайте или выберите виртуальную сеть и подсеть.
1. Создайте группу доступности Azure или разверните зону доступности.
    + Для группы доступности Установите максимальное число доменов обновления равным 2.
1. Создайте виртуальную машину 1.
    + Используйте SLES для образа SAP в Azure Marketplace.
    + Выберите группу доступности Azure, созданную на шаге 3, или выберите зону доступности.
1.  Создайте виртуальную машину 2.
    + Используйте SLES для образа SAP в Azure Marketplace.
    + Выберите группу доступности Azure, созданную на шаге 3, или выберите зону доступности (не та же зона, что и на шаге 3).
1. Добавьте диски данных к виртуальным машинам, а затем проверьте рекомендацию по установке файловой системы в статье [развертывание СУБД IBM DB2 Azure для рабочей нагрузки SAP][dbms-db2].

## <a name="create-the-pacemaker-cluster"></a>Создание кластера Pacemaker
    
Сведения о создании базового кластера Pacemaker для этого сервера IBM DB2 см. в разделе [Настройка Pacemaker на SUSE Linux Enterprise Server в Azure][sles-pacemaker]. 

## <a name="install-the-ibm-db2-luw-and-sap-environment"></a>Установка среды IBM DB2 LUW и SAP

Прежде чем начать установку среды SAP на основе IBM DB2 LUW, ознакомьтесь со следующей документацией:

+ Документация по Azure
+ Документация по SAP
+ Документация по IBM

Ссылки на эту документацию приведены в разделе вводная часть этой статьи.

Ознакомьтесь с руководствами по установке SAP по установке приложений на основе NetWeaver в IBM DB2 LUW.

Руководства можно найти на портале справки SAP с помощью средства [поиска руководства по установке SAP][sap-instfind].

Количество направляющих, отображаемых на портале, можно уменьшить, установив следующие фильтры:

- Я хочу: "установить новую систему"
- Моя база данных: "IBM DB2 для Linux, UNIX и Windows"
- Дополнительные фильтры для версий SAP NetWeaver, конфигурации стека или операционной системы

### <a name="installation-hints-for-setting-up-ibm-db2-luw-with-hadr"></a>Указания по установке IBM DB2 LUW с HADR

Чтобы настроить основной экземпляр базы данных IBM DB2 LUW, выполните следующие действия.

- Используйте параметр Высокая доступность или распределенная.
- Установите SAP ASCS/ERS и экземпляр базы данных.
- Создайте резервную копию только что установленной базы данных.


> [!IMPORTANT] 
> Запишите "порт связи базы данных", заданный во время установки. Он должен быть одним и тем же номером порта для обоих экземпляров базы данных

Чтобы настроить резервный сервер базы данных с помощью процедуры копирования однородных систем SAP, выполните следующие действия.

1. Выберите параметр **копирования системы** > **целевые системы**  >  **распределенный**  >  **экземпляр базы данных**.
1. В качестве метода копирования выберите **однородную систему** , чтобы можно было использовать резервное копирование для восстановления резервной копии на экземпляре резервного сервера.
1. Когда вы найдете шаг выхода, чтобы восстановить базу данных для однородной копии системы, выйдите из установщика. Восстановите базу данных из резервной копии основного узла. Все последующие этапы установки уже выполнены на сервере базы данных — источнике.
1. Настройте транспорт HADR для IBM DB2.

   > [!NOTE]
   > Для установки и настройки, характерной для Azure и Pacemaker: во время процедуры установки с помощью диспетчера подготовки программного обеспечения SAP существует явный вопрос о высокой доступности для IBM DB2 LUW.
   >+ Не выбирайте **IBM DB2 пурескале**.
   >+ Не устанавливайте флажок **установить IBM Tivoli System Automation для многоплатформенных платформ**.
   >+ Не выбирайте **создать файлы конфигурации кластера**.

   При использовании устройства SBD для Linux Pacemaker задайте следующие параметры HADR для DB2:
   + Длительность однорангового окна HADR (в секундах) (HADR_PEER_WINDOW) = 300  
   + Значение времени ожидания HADR (HADR_TIMEOUT) = 60

   При использовании агента разграничения Azure Pacemaker задайте следующие параметры:
   + Длительность однорангового окна HADR (в секундах) (HADR_PEER_WINDOW) = 900  
   + Значение времени ожидания HADR (HADR_TIMEOUT) = 60

Мы рекомендуем использовать описанные выше параметры на основе первоначальной отработки отказа или тестирования перенаправление. Обязательно протестируйте правильную функциональность отработки отказа и перенаправление с этими параметрами параметров. Поскольку отдельные конфигурации могут различаться, для параметров может потребоваться корректировка. 

> [!IMPORTANT]
> Относится к IBM DB2 с конфигурацией HADR с нормальным запуском. перед запуском экземпляра базы данных-источника необходимо, чтобы резервный экземпляр сервера-получателя или сервера, в котором он выполняется, был запущен.

В демонстрационных целях и процедурах, описанных в этой статье, идентификатор безопасности базы данных — **ptr**.

#### <a name="ibm-db2-hadr-check"></a>Проверка IBM DB2 HADR
После настройки HADR и его состояния, ПОДКЛЮЧЕНного к основному и резервному узлам, выполните следующую проверку.

<pre><code>
Execute command as db2&lt;sid&gt; db2pd -hadr -db &lt;SID&gt;

#Primary output:
# Database Member 0 -- Database PTR -- Active -- Up 1 days 01:51:38 -- Date 2019-02-06-15.35.28.505451
# 
#                             <b>HADR_ROLE = PRIMARY
#                           REPLAY_TYPE = PHYSICAL
#                         HADR_SYNCMODE = NEARSYNC
#                            STANDBY_ID = 1
#                         LOG_STREAM_ID = 0
#                            HADR_STATE = PEER
#                            HADR_FLAGS = TCP_PROTOCOL
#                   PRIMARY_MEMBER_HOST = azibmdb02
#                      PRIMARY_INSTANCE = db2ptr
#                        PRIMARY_MEMBER = 0
#                   STANDBY_MEMBER_HOST = azibmdb01
#                      STANDBY_INSTANCE = db2ptr
#                        STANDBY_MEMBER = 0
#                   HADR_CONNECT_STATUS = CONNECTED</b>
#              HADR_CONNECT_STATUS_TIME = 02/05/2019 13:51:47.170561 (1549374707)
#           HEARTBEAT_INTERVAL(seconds) = 15
#                      HEARTBEAT_MISSED = 0
#                    HEARTBEAT_EXPECTED = 6137
#                 HADR_TIMEOUT(seconds) = 60
#         TIME_SINCE_LAST_RECV(seconds) = 13
#              PEER_WAIT_LIMIT(seconds) = 0
#            LOG_HADR_WAIT_CUR(seconds) = 0.000
#     LOG_HADR_WAIT_RECENT_AVG(seconds) = 0.000025
#    LOG_HADR_WAIT_ACCUMULATED(seconds) = 434.595
#                   LOG_HADR_WAIT_COUNT = 223713
# SOCK_SEND_BUF_REQUESTED,ACTUAL(bytes) = 0, 46080
# SOCK_RECV_BUF_REQUESTED,ACTUAL(bytes) = 0, 374400
#             PRIMARY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#             STANDBY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#                   HADR_LOG_GAP(bytes) = 0
#      STANDBY_REPLAY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#        STANDBY_RECV_REPLAY_GAP(bytes) = 0
#                      PRIMARY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#                      STANDBY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#               STANDBY_REPLAY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#          STANDBY_RECV_BUF_SIZE(pages) = 2048
#              STANDBY_RECV_BUF_PERCENT = 0
#            STANDBY_SPOOL_LIMIT(pages) = 0
#                 STANDBY_SPOOL_PERCENT = NULL
#                    STANDBY_ERROR_TIME = NULL
#                  PEER_WINDOW(seconds) = 300
#                       PEER_WINDOW_END = 02/06/2019 15:40:25.000000 (1549467625)
#              READS_ON_STANDBY_ENABLED = N

#Secondary output:
# Database Member 0 -- Database PTR -- Standby -- Up 1 days 01:46:43 -- Date 2019-02-06-15.38.25.644168
# 
#                             <b>HADR_ROLE = STANDBY
#                           REPLAY_TYPE = PHYSICAL
#                         HADR_SYNCMODE = NEARSYNC
#                            STANDBY_ID = 0
#                         LOG_STREAM_ID = 0
#                            HADR_STATE = PEER
#                            HADR_FLAGS = TCP_PROTOCOL
#                   PRIMARY_MEMBER_HOST = azibmdb02
#                      PRIMARY_INSTANCE = db2ptr
#                        PRIMARY_MEMBER = 0
#                   STANDBY_MEMBER_HOST = azibmdb01
#                      STANDBY_INSTANCE = db2ptr
#                        STANDBY_MEMBER = 0
#                   HADR_CONNECT_STATUS = CONNECTED</b>
#              HADR_CONNECT_STATUS_TIME = 02/05/2019 13:51:47.205067 (1549374707)
#           HEARTBEAT_INTERVAL(seconds) = 15
#                      HEARTBEAT_MISSED = 0
#                    HEARTBEAT_EXPECTED = 6186
#                 HADR_TIMEOUT(seconds) = 60
#         TIME_SINCE_LAST_RECV(seconds) = 5
#              PEER_WAIT_LIMIT(seconds) = 0
#            LOG_HADR_WAIT_CUR(seconds) = 0.000
#     LOG_HADR_WAIT_RECENT_AVG(seconds) = 0.000023
#    LOG_HADR_WAIT_ACCUMULATED(seconds) = 434.595
#                   LOG_HADR_WAIT_COUNT = 223725
# SOCK_SEND_BUF_REQUESTED,ACTUAL(bytes) = 0, 46080
# SOCK_RECV_BUF_REQUESTED,ACTUAL(bytes) = 0, 372480
#             PRIMARY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#             STANDBY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#                   HADR_LOG_GAP(bytes) = 0
#      STANDBY_REPLAY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#        STANDBY_RECV_REPLAY_GAP(bytes) = 155
#                      PRIMARY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#                      STANDBY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#               STANDBY_REPLAY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#          STANDBY_RECV_BUF_SIZE(pages) = 2048
#              STANDBY_RECV_BUF_PERCENT = 0
#            STANDBY_SPOOL_LIMIT(pages) = 0
#                 STANDBY_SPOOL_PERCENT = NULL
#                    STANDBY_ERROR_TIME = NULL
#                  PEER_WINDOW(seconds) = 300
#                       PEER_WINDOW_END = 02/06/2019 15:43:19.000000 (1549467799)
#              READS_ON_STANDBY_ENABLED = N
</code></pre>



## <a name="db2-pacemaker-configuration"></a>Конфигурация Pacemaker DB2

При использовании Pacemaker для автоматической отработки отказа в случае сбоя узла необходимо настроить экземпляры DB2 и Pacemaker соответствующим образом. Этот тип конфигурации описан в этом разделе.

Следующие элементы имеют префикс:

- **[A]**: применимо ко всем узлам
- **[1]**: применимо только к узлу 1 
- **[2]**: применимо только к узлу 2

**[A]** необходимые условия для настройки Pacemaker:
1. Завершите работу обоих серверов баз данных с помощью пользователя DB2 \<sid> с db2stop.
1. Измените среду оболочки для пользователя DB2 \<sid> на */бин/КШ*. Рекомендуется использовать средство YaST. 


### <a name="pacemaker-configuration"></a>Конфигурация Pacemaker

> [!IMPORTANT]
> В ходе последних тестов были выявлены ситуации, когда netcat перестает отвечать на запросы из-за невыполненной работы и ограничения на обработку только одного подключения. Ресурс netcat прекращает прослушивать запросы Azure Load Balancer, и плавающий IP-адрес становится недоступным.  
> Для существующих кластеров Pacemaker ранее рекомендовалось заменять netcat на socat. Сейчас мы рекомендуем использовать агент ресурса azure-lb, который входит в состав пакета resource-agents со следующими требованиями к версии пакета:
> - Минимальная версия для SLES 12 с пактом обновления 4 или 5 (SP4/SP5) — resource-agents-4.3.018.a7fb5035-3.30.1.  
> - Минимальная версия для SLES 15/15 с пакетом обновления 1 (SP1) — resource-agents-4.3.0184.6ee15eb2-4.13.1.  
>
> Обратите внимание, что для этого изменения потребуется незначительное время простоя.  
> Если конфигурация существующих кластеров Pacemaker уже была изменена для использования socat, как описано в [этой статье](https://www.suse.com/support/kb/doc/?id=7024128), немедленно переключаться на агент ресурса azure-lb не нужно.

**[1]** конфигурация Pacemaker для IBM DB2 HADR:
<pre><code># Put Pacemaker into maintenance mode
sudo crm configure property maintenance-mode=true
</code></pre>

**[1]** создайте ресурсы IBM DB2:
<pre><code># Replace **bold strings** with your instance name db2sid, database SID, and virtual IP address/Azure Load Balancer.

sudo crm configure primitive rsc_Db2_db2ptr_<b>PTR</b> db2 \
        params instance="<b>db2ptr</b>" dblist="<b>PTR</b>" \
        op start interval="0" timeout="130" \
        op stop interval="0" timeout="120" \
        op promote interval="0" timeout="120" \
        op demote interval="0" timeout="120" \
        op monitor interval="30" timeout="60" \
        op monitor interval="31" role="Master" timeout="60"

# Configure virtual IP - same as Azure Load Balancer IP
sudo crm configure primitive rsc_ip_db2ptr_<b>PTR</b> IPaddr2 \
        op monitor interval="10s" timeout="20s" \
        params ip="<b>10.100.0.10</b>"

# Configure probe port for Azure load Balancer
sudo crm configure primitive rsc_nc_db2ptr_<b>PTR</b> azure-lb port=<b>62500</b>

sudo crm configure group g_ip_db2ptr_<b>PTR</b> rsc_ip_db2ptr_<b>PTR</b> rsc_nc_db2ptr_<b>PTR</b>

sudo crm configure ms msl_Db2_db2ptr_<b>PTR</b> rsc_Db2_db2ptr_<b>PTR</b> \
        meta target-role="Started" notify="true"

sudo crm configure colocation col_db2_db2ptr_<b>PTR</b> inf: g_ip_db2ptr_<b>PTR</b>:Started msl_Db2_db2ptr_<b>PTR</b>:Master

sudo crm configure order ord_db2_ip_db2ptr_<b>PTR</b> inf: msl_Db2_db2ptr_<b>PTR</b>:promote g_ip_db2ptr_<b>PTR</b>:start

sudo crm configure rsc_defaults resource-stickiness=1000
sudo crm configure rsc_defaults migration-threshold=5000
</code></pre>

**[1]** ЗАПУСТИТЕ ресурсы IBM DB2:
* Перевести Pacemaker в режим обслуживания.
<pre><code># Put Pacemaker out of maintenance-mode - that start IBM Db2
sudo crm configure property maintenance-mode=false</pre></code>

**[1]** убедитесь, что кластер имеет состояние "ОК" и все ресурсы запущены. Не важно, на каком узле работают ресурсы.
<pre><code>sudo crm status</code>

# <a name="2-nodes-configured"></a>настроено 2 узла
# <a name="5-resources-configured"></a>5 настроенных ресурсов

# <a name="online--azibmdb01-azibmdb02-"></a>В сети: [azibmdb01 azibmdb02]

# <a name="full-list-of-resources"></a>Полный список ресурсов:

#  <a name="stonith-sbd----stonithexternalsbd-started-azibmdb02"></a>stonith-SBD (stonith: External/SBD): запущено azibmdb02
#  <a name="resource-group-g_ip_db2ptr_ptr"></a>Группа ресурсов: g_ip_db2ptr_PTR
#      <a name="rsc_ip_db2ptr_ptr--ocfheartbeatipaddr2-------started-azibmdb02"></a>rsc_ip_db2ptr_PTR (ОКФ:: пульс: IPaddr2): запущено azibmdb02
#      <a name="rsc_nc_db2ptr_ptr--ocfheartbeatazure-lb------started-azibmdb02"></a>rsc_nc_db2ptr_PTR (ОКФ:: пульс: Azure-фунтов): запущено azibmdb02
#  <a name="masterslave-set-msl_db2_db2ptr_ptr-rsc_db2_db2ptr_ptr"></a>Главный/подчиненный набор: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
#      <a name="masters--azibmdb02-"></a>Образцы: [azibmdb02]
#      <a name="slaves--azibmdb01-"></a>Подчиненные: [azibmdb01]
</pre>

> [!IMPORTANT]
> Необходимо управлять экземпляром Clustered DB2 Pacemaker с помощью средств Pacemaker. При использовании таких команд DB2, как db2stop, Pacemaker обнаруживает это действие как сбой ресурса. При выполнении обслуживания можно перевести узлы или ресурсы в режим обслуживания. Pacemaker приостанавливает ресурсы мониторинга, а затем можно использовать обычные команды администрирования DB2.


### <a name="configure-azure-load-balancer"></a>Настройка Azure Load Balancer
Чтобы настроить Azure Load Balancer, рекомендуется использовать [номер SKU Azure Load Balancer (цен. Категория "Стандартный")](../../../load-balancer/load-balancer-overview.md) , а затем выполнить следующие действия.

> [!NOTE]
> Номер SKU Load Balancer (цен. категория "Стандартный") имеет ограничения на доступ к общедоступным IP-адресам с узлов, расположенных под Load Balancer. Сведения о том, как разрешить этим узлам доступ к общедоступным IP-адресам, см. в статье подключение к общедоступной [конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. Категория "Стандартный") в сценариях с высоким уровнем доступности SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md) .

> [!IMPORTANT]
> Плавающий IP-адрес не поддерживается для вторичной IP-конфигурации NIC в сценариях балансировки нагрузки. Дополнительные сведения см. в статье [ограничения балансировщика нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, разверните вторую сетевую карту.  

1. Создайте интерфейсный пул IP-адресов:

   а. В портал Azure откройте Azure Load Balancer, выберите **интерфейсный пул IP-адресов** и нажмите кнопку **Добавить**.

   b. Введите имя нового пула IP-адресов внешнего интерфейса (например, **DB2-Connection**).

   c. Установите статическое **назначение** и введите IP-адрес **Virtual-IP** , определенный в начале.

   d. Щелкните **ОК**.

   д) Когда пул интерфейсных IP-адресов будет создан, запишите его IP-адрес.

1. Создайте пул серверной части:

   а. В портал Azure откройте Azure Load Balancer, выберите **серверные пулы** и нажмите кнопку **Добавить**.

   b. Введите имя нового пула серверной части (например, **DB2-Серверная** часть).

   c. Щелкните **Добавить виртуальную машину**.

   d. Выберите группу доступности или виртуальные машины, на которых размещена база данных IBM DB2, созданная на предыдущем шаге.

   д) Выберите виртуальные машины кластера IBM DB2.

   е) Щелкните **ОК**.

1. Создание пробы работоспособности:

   а. В портал Azure откройте Azure Load Balancer, выберите **зонды работоспособности** и нажмите кнопку **Добавить**.

   b. Введите имя новой проверки работоспособности (например, **DB2-HP**).

   c. Выберите **TCP** в качестве протокола и порта **62500**. Установите для параметра **интервал** значение **5** и установите для параметра **порог неработоспособности** значение **2**.

   d. Щелкните **ОК**.

1. Создайте правила балансировки нагрузки.

   а. В портал Azure откройте Azure Load Balancer, выберите **правила балансировки нагрузки**, а затем нажмите кнопку **Добавить**.

   b. Введите имя нового правила Load Balancer (например, **DB2-SID**).

   c. Выберите интерфейсный IP-адрес, пул внутренних серверов и пробы работоспособности, созданные ранее (например, **DB2-** Front).

   d. Установите для параметра **протокол** значение **TCP** и введите порт *связи с базой данных* порта.

   д) Увеличьте **время ожидания** до 30 минут.

   е) Не забудьте **включить плавающий IP-адрес**.

   ж. Щелкните **ОК**.


### <a name="make-changes-to-sap-profiles-to-use-virtual-ip-for-connection"></a>Внесение изменений в профили SAP для использования виртуального IP-адреса для подключения
Чтобы подключиться к основному экземпляру конфигурации HADR, уровень приложений SAP должен использовать виртуальный IP-адрес, определенный и настроенный для Azure Load Balancer. Требуются следующие изменения:

/сапмнт/ \<SID> /профиле/дефаулт. PFL
<pre><code>SAPDBHOST = db-virt-hostname
j2ee/dbhost = db-virt-hostname
</code></pre>

/сапмнт/ \<SID> /global/db6/db2cli.ini
<pre><code>Hostname=db-virt-hostname
</code></pre>



## <a name="install-primary-and-dialog-application-servers"></a>Установка основных и диалоговых серверов приложений

При установке основного и диалогового сервера приложений для конфигурации DB2 HADR используйте имя виртуального узла, выбранное для конфигурации. 

Если вы выполнили установку до создания конфигурации DB2 HADR, внесите изменения, как описано в предыдущем разделе, а также для стеков SAP Java.

### <a name="abapjava-or-java-stack-systems-jdbc-url-check"></a>ABAP + Java или Java стековая проверка URL-адреса JDBC

Используйте средство настройки J2EE для проверки или обновления URL-адреса JDBC. Поскольку средство настройки J2EE является графическим средством, необходимо установить X Server:
 
1. Войдите на основной сервер приложений экземпляра J2EE и выполните:   `sudo /usr/sap/*SID*/*Instance*/j2ee/configtool/configtool.sh`
1. В левом фрейме выберите **хранилище безопасности**.
1. В правом окне выберите ключевые JDBC/Pool/ \<SAPSID> /УРЛ..
1. Измените имя узла в URL-адресе JDBC на имя виртуального узла.
     `jdbc:db2://db-virt-hostname:5912/TSP:deferPrepares=0`
1. Выберите **Добавить**.
1. Чтобы сохранить изменения, щелкните значок диска в левом верхнем углу.
1. Закройте средство настройки.
1. Перезапустите экземпляр Java.

## <a name="configure-log-archiving-for-hadr-setup"></a>Настройка архивирования журналов для установки HADR
Чтобы настроить архивацию журналов DB2 для программы установки HADR, рекомендуется настроить базу данных как в основной, так и в резервной, чтобы иметь возможность автоматического получения журналов из всех расположений архива журналов. Как основная, так и резервная база данных должна иметь возможность получения файлов архивов журналов из всех расположений архивов журналов, в которые один из экземпляров базы данных может архивировать файлы журналов. 

Архивирование журналов выполняется только базой данных-источником. При изменении ролей HADR серверов баз данных или при возникновении сбоя новая база данных-источник отвечает за архивирование журналов. Если вы настроили несколько расположений архивов журналов, журналы могут быть архивированы дважды. В случае локального или удаленного обнаружения может также потребоваться вручную скопировать Архивные журналы с старого сервера-источника в активное расположение журнала нового сервера-источника.

Рекомендуется настроить общую общую папку NFS, в которой журналы записываются из обоих узлов. Общий ресурс NFS должен быть высокодоступным. 

Можно использовать существующие общие ресурсы NFS высокой доступности для транспорта или каталога профиля. Дополнительные сведения см. в разделе:

- [Обеспечение высокого уровня доступности NFS на виртуальных машинах Azure в SUSE Linux Enterprise Server][nfs-ha] 
- [Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure на SUSE Linux Enterprise Server с Azure NetApp Files для приложений SAP](./high-availability-guide-suse-netapp-files.md)
- [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) (для создания общих папок NFS)


## <a name="test-the-cluster-setup"></a>Проверка настройки кластера

В этом разделе описывается, как можно протестировать установку DB2 HADR. В *каждом тесте предполагается, что вы вошли в систему как пользователь root* , а первичная база данных IBM DB2 работает на виртуальной машине *azibmdb01* .

Начальное состояние всех тестовых случаев описано здесь: (crm_mon-r или CRM Status).

- **состояние CRM** — моментальный снимок состояния Pacemaker во время выполнения 
- **crm_mon-r** — это непрерывный вывод состояния Pacemaker

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   Promoting azibmdb01
     Slaves: [ azibmdb02 ]
</code></pre>

Исходное состояние в системе SAP описано в разделе Transaction DBACOCKPIT > Configuration > Overview (Общие сведения о конфигурации), как показано на следующем рисунке:

![DBACockpit — Предварительная миграция](./media/dbms-guide-ha-ibm/hadr-sap-mgr-org.png)




### <a name="test-takeover-of-ibm-db2"></a>Тестирование перенаправление IBM DB2


> [!IMPORTANT] 
> Перед началом теста убедитесь в том, что:
> * Pacemaker не имеет невыполненных действий (состояние CRM).
> * Отсутствуют ограничения расположения (оставшиеся тесты миграции)
> * Синхронизация IBM DB2 HADR работает. Проверка с помощью пользовательской DB2\<sid> <pre><code>db2pd -hadr -db \<DBSID></code></pre>


Перенесите узел, на котором выполняется первичная база данных DB2, выполнив следующую команду:
<pre><code>crm resource migrate msl_<b>Db2_db2ptr_PTR</b> azibmdb02</code></pre>

После завершения миграции выходные данные состояния CRM выглядят следующим образом:
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Slaves: [ azibmdb01 ]
</code></pre>

Исходное состояние в системе SAP описано в разделе Transaction DBACOCKPIT > Configuration > Overview (Общие сведения о конфигурации), как показано на следующем рисунке:

![DBACockpit — после миграции](./media/dbms-guide-ha-ibm/hadr-sap-mgr-post.png)

При переносе ресурсов с помощью "перенос ресурсов CRM" создаются ограничения расположения. Ограничения расположения должны быть удалены. Если ограничения расположения не удаляются, ресурс не может восстановить размещение, или вы можете столкнуться с нежелательными такеоверс. 

Перенесите ресурс обратно в *azibmdb01* и очистите ограничения расположения
<pre><code>crm resource migrate msl_<b>Db2_db2ptr_PTR</b> azibmdb01
crm resource clear msl_<b>Db2_db2ptr_PTR</b>
</code></pre>

- **миграция ресурсов CRM \<res_name> \<host> :** создает ограничения расположения и может вызвать проблемы с перенаправление
- **ресурс CRM Clear \<res_name>**: очищает ограничения расположения
- **Очистка \<res_name> ресурсов CRM**: очищает все ошибки ресурса.

### <a name="test-the-fencing-agent"></a>Тестирование агента ограждения

В этом случае мы протестируем SBD ограждение, которое рекомендуется выполнять при использовании SUSE Linux.

<pre><code>
azibmdb01:~ # ps -ef|grep sbd
root       2374      1  0 Feb05 ?        00:00:17 sbd: inquisitor
root       2378   2374  0 Feb05 ?        00:00:40 sbd: watcher: /dev/disk/by-id/scsi-36001405fbbaab35ee77412dacb77ae36 - slot: 0 - uuid: 27cad13a-0bce-4115-891f-43b22cfabe65
root       2379   2374  0 Feb05 ?        00:01:51 sbd: watcher: Pacemaker
root       2380   2374  0 Feb05 ?        00:00:18 sbd: watcher: Cluster

azibmdb01:~ # kill -9 2374
</code></pre>

Необходимо перезагрузить узел кластера *azibmdb01* . Основная роль HADR для IBM DB2 будет перемещена в *azibmdb02*. Когда *azibmdb01* возвращается в режим «в сети», экземпляр DB2 будет перемещен в роль экземпляра базы данных-получателя. 

Если служба Pacemaker не запускается автоматически на предыдущей перезагруженной основной реплике, не забудьте запустить ее вручную с помощью:

<code><pre>sudo service pacemaker start</code></pre>

### <a name="test-a-manual-takeover"></a>Тестирование перенаправление вручную

Вы можете протестировать ручной перенаправление, остановите службу Pacemaker на узле *azibmdb01* :
<pre><code>service pacemaker stop</code></pre>

состояние на *azibmdb02*
<pre><code>
2 nodes configured
5 resources configured

Online: [ azibmdb02 ]
OFFLINE: [ azibmdb01 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]
</code></pre>

После отработки отказа службу можно запустить снова в *azibmdb01*.
<pre><code>service pacemaker start</code></pre>


### <a name="kill-the-db2-process-on-the-node-that-runs-the-hadr-primary-database"></a>Закрытие процесса DB2 на узле, на котором выполняется база данных источника HADR

<pre><code>#Kill main db2 process - db2sysc
azibmdb01:~ # ps -ef|grep db2s
db2ptr    34598  34596  8 14:21 ?        00:00:07 db2sysc 0

azibmdb01:~ # kill -9 34598
</code></pre>

Экземпляр DB2 завершится ошибкой, и Pacemaker сообщит о следующем состоянии:

<pre><code>
2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Slaves: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=157, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:28:19 2019', queued=40ms, exec=223ms

</code></pre>

Pacemaker перезапустит экземпляр базы данных DB2 на том же узле или отправит отказ на узел, на котором выполняется экземпляр базы данных-получателя, и появится сообщение об ошибке.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=157, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:28:19 2019', queued=40ms, exec=223ms
</code></pre>


### <a name="kill-the-db2-process-on-the-node-that-runs-the-secondary-database-instance"></a>Завершить процесс DB2 на узле, на котором выполняется экземпляр базы данных-получателя

<pre><code>azibmdb02:~ # ps -ef|grep db2s
db2ptr    65250  65248  0 Feb11 ?        00:09:27 db2sysc 0

azibmdb02:~ # kill -9</code></pre>

Узел возвращается в значение Failed и сообщает об ошибке
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   FAILED azibmdb02
     Masters: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_monitor_30000 on azibmdb02 'not running' (7): call=144, status=complete, exitreason='',
last-rc-change='Tue Feb 12 14:36:59 2019', queued=0ms, exec=0ms</code></pre>

Экземпляр DB2 перезапускается во вторичной роли, которой он был назначен ранее.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_monitor_30000 on azibmdb02 'not running' (7): call=144, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:36:59 2019', queued=0ms, exec=0ms</code></pre>



### <a name="stop-db-via-db2stop-force-on-the-node-that-runs-the-hadr-primary-database-instance"></a>Завершение работы базы данных с помощью db2stop Force на узле, на котором выполняется экземпляр базы данных HADR

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>

Как пользователь DB2 \<sid> EXECUTE команда db2stop Force:
<pre><code>azibmdb01:~ # su - db2ptr
azibmdb01:db2ptr> db2stop force</code></pre>

Обнаружен сбой
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   FAILED azibmdb01
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=201, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:45:25 2019', queued=1ms, exec=150ms</code></pre>

Экземпляр базы данных-получателя HADR DB2 был повышен до первичной роли
<pre><code> nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_start_0 on azibmdb01 'unknown error' (1): call=205, stat
us=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:45:27 2019', queued=0ms, exec=865ms</pre></code>


### <a name="crash-vm-with-restart-on-the-node-that-runs-the-hadr-primary-database-instance"></a>Сбой виртуальной машины с перезапуском на узле, на котором выполняется экземпляр базы данных HADR

<pre><code>#Linux kernel panic - with OS restart
azibmdb01:~ # echo b > /proc/sysrq-trigger</code></pre>

Pacemaker будет повышать уровень вторичного экземпляра до роли основного экземпляра. Старый основной экземпляр будет перемещен во вторичную роль после перезагрузки виртуальной машины после перезапуска виртуальной машины, после чего все службы будут полностью восстановлены.

<pre><code> nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>



### <a name="crash-the-vm-that-runs-the-hadr-primary-database-instance-with-halt"></a>Сбой виртуальной машины, на которой выполняется экземпляр базы данных HADR, с "Halt"

<pre><code>#Linux kernel panic - halts OS
azibmdb01:~ # echo b > /proc/sysrq-trigger</code></pre>

В этом случае Pacemaker обнаружит, что узел, на котором выполняется экземпляр базы данных-источника, не отвечает.

<pre><code>2 nodes configured
5 resources configured

Node azibmdb01: UNCLEAN (online)
Online: [ azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>

Следующим шагом является проверка на наличие *разделенного мозгового* случая. После того, как работающий узел определил, что узел, на котором выполнялся последний экземпляр базы данных-источника, не работает, выполняется отработка отказа ресурсов.
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb02 ]
OFFLINE: [ azibmdb01 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ] </code></pre>


В случае "остановки" узла неисправный узел необходимо перезапустить через средства управления Azure (в портал Azure, PowerShell или Azure CLI). После того как неисправный узел снова вернется в режим «в сети», он запустит экземпляр DB2 во вторичной роли.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Slaves: [ azibmdb01 ]</code></pre>

## <a name="next-steps"></a>Дальнейшие действия
- [Высокодоступная архитектура и сценарии для SAP NetWeaver](./sap-high-availability-architecture-scenarios.md)
- [Настройка Pacemaker на SUSE Linux Enterprise Server в Azure](./high-availability-guide-suse-pacemaker.md)

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[1612105]:https://launchpad.support.sap.com/#/notes/1612105

[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/
[db2-hadr-11.1]:https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html
[db2-hadr-10.5]:https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html
[dbms-db2]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_ibm
[sles-pacemaker]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-pacemaker
[sap-instfind]:https://help.sap.com/viewer/9e41ead9f54e44c1ae1a1094b0f80712/ALL/en-US/576f5c1808de4d1abecbd6e503c9ba42.html
[nfs-ha]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs
[sles-ha-guide]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP4/
[ascs-ha]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md
[azr-sap-plancheck]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-deployment-checklist
