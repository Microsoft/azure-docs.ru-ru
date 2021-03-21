---
title: Настройка IBM DB2 HADR на виртуальных машинах Azure в RHEL | Документация Майкрософт
description: Установите высокий уровень доступности IBM DB2 LUW на виртуальных машинах Azure (ВМ) RHEL.
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/16/2020
ms.author: juergent
ms.openlocfilehash: 979de230655b4804fa65a4bddb678e21b59c4ea2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101674447"
---
# <a name="high-availability-of-ibm-db2-luw-on-azure-vms-on-red-hat-enterprise-linux-server"></a>Обеспечение высокого уровня доступности IBM DB2 LUW на виртуальных машинах Azure в Red Hat Enterprise Linux Server

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
| [2002167] | Red Hat Enterprise Linux 7.x: установка и обновление |
| [2694118] | Red Hat Enterprise Linux Add-On высокой доступности в Azure |
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
| [Общие сведения о Add-On высокой доступности для Red Hat Enterprise Linux 7][rhel-ha-addon] |
| [Администрирование надстройки для обеспечения высокой доступности][rhel-ha-admin] |
| [Справочник по надстройке для обеспечения высокой доступности][rhel-ha-ref] |
| [Политики поддержки для кластеров высокой доступности RHEL — виртуальные машины Microsoft Azure как члены кластера][rhel-azr-supp]
| [Установка и настройка кластера высокой доступности Red Hat Enterprise Linux 7.4 (и более поздних версий) в Microsoft Azure][rhel-azr-inst]
| [Развертывание СУБД IBM DB2 на Виртуальных машинах Azure для рабочей нагрузки SAP][dbms-db2] |
| [IBM DB2 HADR 11,1][db2-hadr-11.1] |
| [IBM DB2 HADR 10,5][db2-hadr-10.5] |
| [Политика поддержки для кластеров высокой доступности RHEL — управление IBM DB2 для Linux, UNIX и Windows в кластере][rhel-db2-supp]



## <a name="overview"></a>Обзор
Для обеспечения высокой доступности IBM DB2 LUW с HADR устанавливается по крайней мере на двух виртуальных машинах Azure, которые развертываются в [группе доступности Azure](../../windows/tutorial-availability-sets.md) или в [зоны доступности Azure](./sap-ha-availability-zones.md). 

На следующих рисунках показано, как настроить две виртуальные машины Azure сервера базы данных. Обе виртуальные машины Azure сервера базы данных имеют собственное хранилище, подключенное и работающее. В HADR один экземпляр базы данных на одной из виртуальных машин Azure имеет роль основного экземпляра. Все клиенты подключены к основному экземпляру. Все изменения в транзакциях базы данных сохраняются локально в журнале транзакций DB2. По мере сохранения записей журнала транзакций записи передаются через протокол TCP/IP на экземпляр базы данных на втором сервере базы данных, резервном сервере или резервном экземпляре. Резервный экземпляр обновляет локальную базу данных, выполнив накат переданных записей журнала транзакций. Таким образом резервный сервер остается синхронизированным с сервером-источником.

HADR — это только функция репликации. Он не имеет возможности обнаружения сбоев и автоматического перенаправление или отработки отказа. Администратор базы данных должен вручную инициировать перенаправление или передавать на резервный сервер. Чтобы обеспечить автоматическое перенаправление и обнаружение сбоев, можно использовать функцию кластеризации Pacemaker в Linux. Pacemaker отслеживает два экземпляра сервера базы данных. При аварийном завершении экземпляра сервера базы данных Pacemaker инициирует *Автоматический* перенаправление HADR на резервном сервере. Pacemaker также гарантирует, что виртуальный IP-адрес будет назначен новому основному серверу.

![Обзор высокой доступности IBM DB2](./media/high-availability-guide-rhel-ibm-db2-luw/ha-db2-hadr-lb-rhel.png)

Чтобы серверы приложений SAP подключались к базе данных-источнику, требуется имя виртуального узла и виртуальный IP-адрес. В случае отработки отказа серверы приложений SAP будут подключаться к новому экземпляру базы данных-источника. В среде Azure [балансировщик нагрузки Azure](https://microsoft.sharepoint.com/teams/WAG/AzureNetworking/Wiki/Load%20Balancing.aspx) требуется для использования виртуального IP-адреса так, как это требуется для HADR IBM DB2. 

На следующем рисунке представлен обзор высокодоступной настройки системы SAP на основе базы данных IBM DB2, которая поможет вам полностью понять, как IBM DB2 LUW с HADR и Pacemaker соответствует требованиям к высокодоступной настройке системы SAP. В этой статье рассматривается только IBM DB2, но содержатся ссылки на другие статьи о настройке других компонентов системы SAP.

![Обзор полной среды высокой доступности IBM DB2](./media/high-availability-guide-rhel-ibm-db2-luw/end-2-end-ha-rhel.png)


### <a name="high-level-overview-of-the-required-steps"></a>Высокоуровневый обзор необходимых действий
Чтобы развернуть конфигурацию IBM DB2, необходимо выполнить следующие действия.

  + Спланируйте среду.
  + Разверните виртуальные машины.
  + Обновление RHEL Linux и настройка файловых систем.
  + Установите и настройте Pacemaker.
  + Настройка [кластера глустерфс][glusterfs] или [Azure NetApp Files][anf-rhel]
  + Установите [ASCS/ERS в отдельном кластере][ascs-ha-rhel].
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
| Ограждение Azure | , Чтобы избежать ситуаций раздельного мозгового случая. |
| Azure Load Balancer | Использование уровня "базовый" или "Стандартный" (рекомендуется), порт пробы для базы данных DB2 (наш Совет 62500) **пробный порт**. |
| Разрешение имен| Как работает разрешение имен в среде. Настоятельно рекомендуется использовать службу DNS. Можно использовать локальный файл hosts. |
    
Дополнительные сведения о Pacemaker Linux в Azure см. в статье [Настройка Pacemaker на Red Hat Enterprise Linux в Azure][rhel-pcs-azr].

## <a name="deployment-on-red-hat-enterprise-linux"></a>Развертывание на Red Hat Enterprise Linux

Агент ресурсов для IBM DB2 LUW входит в надстройку высокой доступности Red Hat Enterprise Linux Server. Для настройки, описанной в этом документе, следует использовать Red Hat Enterprise Linux для SAP. Azure Marketplace содержит образ для Red Hat Enterprise Linux 7,4 для SAP или более поздней версии, который можно использовать для развертывания новых виртуальных машин Azure. При выборе образа виртуальной машины на виртуальной машине Azure Marketplace следует учитывать различные модели поддержки или служб, предлагаемые Red Hat в Azure Marketplace.

### <a name="hosts-dns-updates"></a>Узлы: обновления DNS
Создайте список всех имен узлов, включая имена виртуальных узлов, и обновите DNS-серверы, чтобы включить правильный IP-адрес для разрешения имен узлов. Если DNS-сервер не существует или вы не можете обновлять и создавать записи DNS, необходимо использовать файлы локального узла отдельных виртуальных машин, участвующих в этом сценарии. Если вы используете записи файлов узлов, убедитесь, что записи применяются ко всем виртуальным машинам в среде системы SAP. Однако рекомендуется использовать DNS, который в идеале расширяется в Azure


### <a name="manual-deployment"></a>Развертывание вручную

Убедитесь, что выбранная ОС поддерживается IBM DB2 LUW для IBM/SAP. Список поддерживаемых версий ОС для виртуальных машин Azure и выпусков DB2 доступен в примечании SAP [1928533]. Список выпусков ОС отдельных версий DB2 доступен в матрице доступности продуктов SAP. Мы настоятельно рекомендуем минимум Red Hat Enterprise Linux 7,4 для SAP из-за улучшений производительности, связанных с Azure, в этой или более поздней версии Red Hat Enterprise Linux.

1. Создайте или выберите группу ресурсов.
1. Создайте или выберите виртуальную сеть и подсеть.
1. Создайте группу доступности Azure или разверните зону доступности.
    + Для группы доступности Установите максимальное число доменов обновления равным 2.
1. Создайте виртуальную машину 1.
    + Используйте Red Hat Enterprise Linux для образа SAP в Azure Marketplace.
    + Выберите группу доступности Azure, созданную на шаге 3, или выберите зону доступности.
1.  Создайте виртуальную машину 2.
    + Используйте Red Hat Enterprise Linux для образа SAP в Azure Marketplace.
    + Выберите группу доступности Azure, созданную на шаге 3, или выберите зону доступности (не та же зона, что и на шаге 3).
1. Добавьте диски данных к виртуальным машинам, а затем проверьте рекомендацию по установке файловой системы в статье [развертывание СУБД IBM DB2 Azure для рабочей нагрузки SAP][dbms-db2].

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

#### <a name="red-hat-firewall-rules"></a>Правила брандмауэра Red Hat
По умолчанию в Red Hat Enterprise Linux включен брандмауэр. 

<pre><code>#Allow access to SWPM tool. Rule is not permanent.
sudo firewall-cmd --add-port=4237/tcp</code></pre>

### <a name="installation-hints-for-setting-up-ibm-db2-luw-with-hadr"></a>Указания по установке IBM DB2 LUW с HADR

Чтобы настроить основной экземпляр базы данных IBM DB2 LUW, выполните следующие действия.

- Используйте параметр Высокая доступность или распределенная.
- Установите SAP ASCS/ERS и экземпляр базы данных.
- Создайте резервную копию только что установленной базы данных.

> [!IMPORTANT] 
> Запишите "порт связи базы данных", заданный во время установки. Он должен иметь один и тот же номер порта для обоих экземпляров базы данных.
>![Определение порта SAP SWPM](./media/high-availability-guide-rhel-ibm-db2-luw/hadr-swpm-db2-port.png)

### <a name="ibm-db2-hadr-settings-for-azure"></a>Настройки IBM DB2 HADR для Azure

   При использовании агента разграничения Azure Pacemaker задайте следующие параметры:

   - Длительность однорангового окна HADR (в секундах) (HADR_PEER_WINDOW) = 240  
   - Значение времени ожидания HADR (HADR_TIMEOUT) = 45

Мы рекомендуем использовать описанные выше параметры на основе первоначальной отработки отказа или тестирования перенаправление. Обязательно протестируйте правильную функциональность отработки отказа и перенаправление с этими параметрами параметров. Поскольку отдельные конфигурации могут различаться, для параметров может потребоваться корректировка. 

> [!NOTE]
> Относится к IBM DB2 с конфигурацией HADR с нормальным запуском. перед запуском экземпляра базы данных-источника необходимо, чтобы резервный экземпляр сервера-получателя или сервера, в котором он выполняется, был запущен.

   
> [!NOTE]
> Для установки и настройки, характерной для Azure и Pacemaker: во время процедуры установки с помощью диспетчера подготовки программного обеспечения SAP существует явный вопрос о высокой доступности для IBM DB2 LUW.
>+ Не выбирайте **IBM DB2 пурескале**.
>+ Не устанавливайте флажок **установить IBM Tivoli System Automation для многоплатформенных платформ**.
>+ Не выбирайте **создать файлы конфигурации кластера**.
>![Параметры уровня доступности SAP SWPM-DB2](./media/high-availability-guide-rhel-ibm-db2-luw/swpm-db2ha-opt.png)


Чтобы настроить резервный сервер базы данных с помощью процедуры копирования однородных систем SAP, выполните следующие действия.

1. Выберите параметр **копирования системы** > **целевые системы**  >  **распределенный**  >  **экземпляр базы данных**.
1. В качестве метода копирования выберите **однородную систему** , чтобы можно было использовать резервное копирование для восстановления резервной копии на экземпляре резервного сервера.
1. Когда вы найдете шаг выхода, чтобы восстановить базу данных для однородной копии системы, выйдите из установщика. Восстановите базу данных из резервной копии основного узла. Все последующие этапы установки уже выполнены на сервере базы данных — источнике.

#### <a name="red-hat-firewall-rules-for-db2-hadr"></a>Правила брандмауэра Red Hat для DB2 HADR
Добавьте правила брандмауэра, чтобы разрешить передачу трафика в DB2 и между DB2 для HADR:
+ Порт связи базы данных. При использовании секций добавьте эти порты.
+ Порт HADR (значение параметра DB2 HADR_LOCAL_SVC)
+ Порт пробы Azure
<pre><code>sudo firewall-cmd --add-port=&lt;port&gt;/tcp --permanent
sudo firewall-cmd --reload</code></pre>

#### <a name="ibm-db2-hadr-check"></a>Проверка IBM DB2 HADR
В демонстрационных целях и процедурах, описанных в этой статье, идентификатор безопасности базы данных — **id2**.

После настройки HADR и его состояния, ПОДКЛЮЧЕНного к основному и резервному узлам, выполните следующую проверку.

<pre><code>
Execute command as db2&lt;sid&gt; db2pd -hadr -db &lt;SID&gt;

#Primary output:
Database Member 0 -- Database ID2 -- Active -- Up 1 days 15:45:23 -- Date 2019-06-25-10.55.25.349375

                            <b>HADR_ROLE = PRIMARY
                          REPLAY_TYPE = PHYSICAL
                        HADR_SYNCMODE = NEARSYNC
                           STANDBY_ID = 1
                        LOG_STREAM_ID = 0
                           HADR_STATE = PEER
                           HADR_FLAGS =
                  PRIMARY_MEMBER_HOST = az-idb01
                     PRIMARY_INSTANCE = db2id2
                       PRIMARY_MEMBER = 0
                  STANDBY_MEMBER_HOST = az-idb02
                     STANDBY_INSTANCE = db2id2
                       STANDBY_MEMBER = 0
                  HADR_CONNECT_STATUS = CONNECTED</b>
             HADR_CONNECT_STATUS_TIME = 06/25/2019 10:55:05.076494 (1561460105)
          HEARTBEAT_INTERVAL(seconds) = 7
                     HEARTBEAT_MISSED = 5
                   HEARTBEAT_EXPECTED = 52
                HADR_TIMEOUT(seconds) = 30
        TIME_SINCE_LAST_RECV(seconds) = 5
             PEER_WAIT_LIMIT(seconds) = 0
           LOG_HADR_WAIT_CUR(seconds) = 0.000
    LOG_HADR_WAIT_RECENT_AVG(seconds) = 598.000027
   LOG_HADR_WAIT_ACCUMULATED(seconds) = 598.000
                  LOG_HADR_WAIT_COUNT = 1
SOCK_SEND_BUF_REQUESTED,ACTUAL(bytes) = 0, 46080
SOCK_RECV_BUF_REQUESTED,ACTUAL(bytes) = 0, 369280
            PRIMARY_LOG_FILE,PAGE,POS = S0000012.LOG, 14151, 3685322855
            STANDBY_LOG_FILE,PAGE,POS = S0000012.LOG, 14151, 3685322855
                  HADR_LOG_GAP(bytes) = 132242668
     STANDBY_REPLAY_LOG_FILE,PAGE,POS = S0000012.LOG, 14151, 3685322855
       STANDBY_RECV_REPLAY_GAP(bytes) = 0
                     PRIMARY_LOG_TIME = 06/25/2019 10:45:42.000000 (1561459542)
                     STANDBY_LOG_TIME = 06/25/2019 10:45:42.000000 (1561459542)
              STANDBY_REPLAY_LOG_TIME = 06/25/2019 10:45:42.000000 (1561459542)
         STANDBY_RECV_BUF_SIZE(pages) = 2048
             STANDBY_RECV_BUF_PERCENT = 0
           STANDBY_SPOOL_LIMIT(pages) = 1000
                STANDBY_SPOOL_PERCENT = 0
                   STANDBY_ERROR_TIME = NULL
                 PEER_WINDOW(seconds) = 300
                      PEER_WINDOW_END = 06/25/2019 11:12:03.000000 (1561461123)
             READS_ON_STANDBY_ENABLED = N


#Secondary output:
Database Member 0 -- Database ID2 -- Standby -- Up 1 days 15:45:18 -- Date 2019-06-25-10.56.19.820474

                            <b>HADR_ROLE = STANDBY
                          REPLAY_TYPE = PHYSICAL
                        HADR_SYNCMODE = NEARSYNC
                           STANDBY_ID = 0
                        LOG_STREAM_ID = 0
                           HADR_STATE = PEER
                           HADR_FLAGS =
                  PRIMARY_MEMBER_HOST = az-idb01
                     PRIMARY_INSTANCE = db2id2
                       PRIMARY_MEMBER = 0
                  STANDBY_MEMBER_HOST = az-idb02
                     STANDBY_INSTANCE = db2id2
                       STANDBY_MEMBER = 0
                  HADR_CONNECT_STATUS = CONNECTED</b>
             HADR_CONNECT_STATUS_TIME = 06/25/2019 10:55:05.078116 (1561460105)
          HEARTBEAT_INTERVAL(seconds) = 7
                     HEARTBEAT_MISSED = 0
                   HEARTBEAT_EXPECTED = 10
                HADR_TIMEOUT(seconds) = 30
        TIME_SINCE_LAST_RECV(seconds) = 1
             PEER_WAIT_LIMIT(seconds) = 0
           LOG_HADR_WAIT_CUR(seconds) = 0.000
    LOG_HADR_WAIT_RECENT_AVG(seconds) = 598.000027
   LOG_HADR_WAIT_ACCUMULATED(seconds) = 598.000
                  LOG_HADR_WAIT_COUNT = 1
SOCK_SEND_BUF_REQUESTED,ACTUAL(bytes) = 0, 46080
SOCK_RECV_BUF_REQUESTED,ACTUAL(bytes) = 0, 367360
            PRIMARY_LOG_FILE,PAGE,POS = S0000012.LOG, 14151, 3685322855
            STANDBY_LOG_FILE,PAGE,POS = S0000012.LOG, 14151, 3685322855
                  HADR_LOG_GAP(bytes) = 0
     STANDBY_REPLAY_LOG_FILE,PAGE,POS = S0000012.LOG, 14151, 3685322855
       STANDBY_RECV_REPLAY_GAP(bytes) = 0
                     PRIMARY_LOG_TIME = 06/25/2019 10:45:42.000000 (1561459542)
                     STANDBY_LOG_TIME = 06/25/2019 10:45:42.000000 (1561459542)
              STANDBY_REPLAY_LOG_TIME = 06/25/2019 10:45:42.000000 (1561459542)
         STANDBY_RECV_BUF_SIZE(pages) = 2048
             STANDBY_RECV_BUF_PERCENT = 0
           STANDBY_SPOOL_LIMIT(pages) = 1000
                STANDBY_SPOOL_PERCENT = 0
                   STANDBY_ERROR_TIME = NULL
                 PEER_WINDOW(seconds) = 1000
                      PEER_WINDOW_END = 06/25/2019 11:12:59.000000 (1561461179)
             READS_ON_STANDBY_ENABLED = N
</code></pre>

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

**[A]** добавьте правило брандмауэра для пробного порта:

<pre><code>sudo firewall-cmd --add-port=<b><probe-port></b>/tcp --permanent
sudo firewall-cmd --reload</code></pre>

## <a name="create-the-pacemaker-cluster"></a>Создание кластера Pacemaker
    
Сведения о создании базового кластера Pacemaker для этого сервера IBM DB2 см. в статье [Настройка Pacemaker на Red Hat Enterprise Linux в Azure][rhel-pcs-azr]. 

## <a name="db2-pacemaker-configuration"></a>Конфигурация Pacemaker DB2

При использовании Pacemaker для автоматической отработки отказа в случае сбоя узла необходимо настроить экземпляры DB2 и Pacemaker соответствующим образом. Этот тип конфигурации описан в этом разделе.

Следующие элементы имеют префикс:

- **[A]**: применимо ко всем узлам
- **[1]**: применимо только к узлу 1 
- **[2]**: применимо только к узлу 2

**[A]** необходимый компонент для настройки Pacemaker:
1. Завершите работу обоих серверов баз данных с помощью пользователя DB2 \<sid> с db2stop.
1. Измените среду оболочки для пользователя DB2 \<sid> на */бин/КШ*:
<pre><code># Install korn shell:
sudo yum install ksh
# Change users shell:
sudo usermod -s /bin/ksh db2&lt;sid&gt;</code></pre>  

### <a name="pacemaker-configuration"></a>Конфигурация Pacemaker

**[1]** конфигурация Pacemaker для IBM DB2 HADR:
<pre><code># Put Pacemaker into maintenance mode
sudo pcs property set maintenance-mode=true 
</code></pre>

**[1]** создайте ресурсы IBM DB2:

При создании кластера на **RHEL 7. x** используйте следующие команды:

<pre><code># Replace <b>bold strings</b> with your instance name db2sid, database SID, and virtual IP address/Azure Load Balancer.
sudo pcs resource create Db2_HADR_<b>ID2</b> db2 instance='<b>db2id2</b>' dblist='<b>ID2</b>' master meta notify=true resource-stickiness=5000

#Configure resource stickiness and correct cluster notifications for master resoruce
sudo pcs resource update Db2_HADR_<b>ID2</b>-master meta notify=true resource-stickiness=5000

# Configure virtual IP - same as Azure Load Balancer IP
sudo pcs resource create vip_<b>db2id2</b>_<b>ID2</b> IPaddr2 ip='<b>10.100.0.40</b>'

# Configure probe port for Azure load Balancer
sudo pcs resource create nc_<b>db2id2</b>_<b>ID2</b> azure-lb port=<b>62500</b>

#Create a group for ip and Azure loadbalancer probe port
sudo pcs resource group add g_ipnc_<b>db2id2</b>_<b>ID2</b> vip_<b>db2id2</b>_<b>ID2</b> nc_<b>db2id2</b>_<b>ID2</b>

#Create colocation constrain - keep Db2 HADR Master and Group on same node
sudo pcs constraint colocation add g_ipnc_<b>db2id2</b>_<b>ID2</b> with master Db2_HADR_<b>ID2</b>-master

#Create start order constrain
sudo pcs constraint order promote Db2_HADR_<b>ID2</b>-master then g_ipnc_<b>db2id2</b>_<b>ID2</b>
</code></pre>

При создании кластера на **RHEL 8. x** используйте следующие команды:

<pre><code># Replace <b>bold strings</b> with your instance name db2sid, database SID, and virtual IP address/Azure Load Balancer.
sudo pcs resource create Db2_HADR_<b>ID2</b> db2 instance='<b>db2id2</b>' dblist='<b>ID2</b>' promotable meta notify=true resource-stickiness=5000

#Configure resource stickiness and correct cluster notifications for master resoruce
sudo pcs resource update Db2_HADR_<b>ID2</b>-clone meta notify=true resource-stickiness=5000

# Configure virtual IP - same as Azure Load Balancer IP
sudo pcs resource create vip_<b>db2id2</b>_<b>ID2</b> IPaddr2 ip='<b>10.100.0.40</b>'

# Configure probe port for Azure load Balancer
sudo pcs resource create nc_<b>db2id2</b>_<b>ID2</b> azure-lb port=<b>62500</b>

#Create a group for ip and Azure loadbalancer probe port
sudo pcs resource group add g_ipnc_<b>db2id2</b>_<b>ID2</b> vip_<b>db2id2</b>_<b>ID2</b> nc_<b>db2id2</b>_<b>ID2</b>

#Create colocation constrain - keep Db2 HADR Master and Group on same node
sudo pcs constraint colocation add g_ipnc_<b>db2id2</b>_<b>ID2</b> with master Db2_HADR_<b>ID2</b>-clone

#Create start order constrain
sudo pcs constraint order promote Db2_HADR_<b>ID2</b>-clone then g_ipnc_<b>db2id2</b>_<b>ID2</b>
</code></pre>

**[1]** ЗАПУСТИТЕ ресурсы IBM DB2:
* Перевести Pacemaker в режим обслуживания.
<pre><code># Put Pacemaker out of maintenance-mode - that start IBM Db2
sudo pcs property set maintenance-mode=false</pre></code>

**[1]** убедитесь, что кластер имеет состояние "ОК" и все ресурсы запущены. Не важно, на каком узле работают ресурсы.
<pre><code>sudo pcs status
2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb01
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb01 ]
     Slaves: [ az-idb02 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb01
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb01

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
</code></pre>

> [!IMPORTANT]
> Необходимо управлять экземпляром Clustered DB2 Pacemaker с помощью средств Pacemaker. При использовании таких команд DB2, как db2stop, Pacemaker обнаруживает это действие как сбой ресурса. При выполнении обслуживания можно перевести узлы или ресурсы в режим обслуживания. Pacemaker приостанавливает ресурсы мониторинга, а затем можно использовать обычные команды администрирования DB2.

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
 
1. Войдите на основной сервер приложений экземпляра J2EE и выполните:
    
    <pre><code>sudo /usr/sap/*SID*/*Instance*/j2ee/configtool/configtool.sh</code></pre>  
    
1. В левом фрейме выберите **хранилище безопасности**.
1. В правой части окна выберите ключ `jdbc/pool/\<SAPSID>/url` .
1. Измените имя узла в URL-адресе JDBC на имя виртуального узла.
    
    <pre><code>jdbc:db2://db-virt-hostname:5912/TSP:deferPrepares=0</code></pre>  
    
1. Выберите **Добавить**.
1. Чтобы сохранить изменения, щелкните значок диска в левом верхнем углу.
1. Закройте средство настройки.
1. Перезапустите экземпляр Java.

## <a name="configure-log-archiving-for-hadr-setup"></a>Настройка архивирования журналов для установки HADR

Чтобы настроить архивацию журналов DB2 для программы установки HADR, рекомендуется настроить базу данных как в основной, так и в резервной, чтобы иметь возможность автоматического получения журналов из всех расположений архива журналов. Как основная, так и резервная база данных должна иметь возможность получения файлов архивов журналов из всех расположений архивов журналов, в которые один из экземпляров базы данных может архивировать файлы журналов. 

Архивирование журналов выполняется только базой данных-источником. При изменении ролей HADR серверов баз данных или при возникновении сбоя новая база данных-источник отвечает за архивирование журналов. Если вы настроили несколько расположений архивов журналов, журналы могут быть архивированы дважды. В случае локального или удаленного обнаружения может также потребоваться вручную скопировать Архивные журналы с старого сервера-источника в активное расположение журнала нового сервера-источника.

Рекомендуется настроить общий общий ресурс NFS или Глустерфс, где журналы записываются из обоих узлов. Общий ресурс NFS или Глустерфс должен быть высокодоступным. 

Вы можете использовать существующие общие папки NFS высокой доступности или Глустерфс для транспорта или каталога профиля. Дополнительные сведения см. в разделе:

- [GlusterFS в виртуальных машинах Azure с Red Hat Enterprise Linux для SAP NetWeaver][glusterfs] 
- [Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure на Red Hat Enterprise Linux с Azure NetApp Files для приложений SAP][anf-rhel]
- [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) (для создания общих папок NFS)

## <a name="test-the-cluster-setup"></a>Проверка настройки кластера

В этом разделе описывается, как можно протестировать установку DB2 HADR. В каждом тесте предполагается, что основной сервер IBM DB2 работает на виртуальной машине *AZ-idb01* . Необходимо использовать пользователя с привилегиями sudo или root (не рекомендуется).

Начальное состояние всех тестовых случаев описано здесь: (crm_mon-r или PC Status).

- **состояние компьютеров** — моментальный снимок состояния Pacemaker во время выполнения 
- **crm_mon-r** — это непрерывный вывод состояния Pacemaker

<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb01
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb01 ]
     Slaves: [ az-idb02 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb01
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb01

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
</code></pre>

Исходное состояние в системе SAP описано в разделе Transaction DBACOCKPIT > Configuration > Overview (Общие сведения о конфигурации), как показано на следующем рисунке:

![DBACockpit — Предварительная миграция](./media/high-availability-guide-rhel-ibm-db2-luw/hadr-sap-mgr-org-rhel.png)

### <a name="test-takeover-of-ibm-db2"></a>Тестирование перенаправление IBM DB2


> [!IMPORTANT] 
> Перед началом теста убедитесь в том, что:
> * Pacemaker не имеет невыполненных действий (состояние ПК).
> * Отсутствуют ограничения расположения (оставшиеся тесты миграции)
> * Синхронизация IBM DB2 HADR работает. Проверка с помощью пользовательской DB2\<sid> <pre><code>db2pd -hadr -db \<DBSID></code></pre>

Перенесите узел, на котором выполняется первичная база данных DB2, выполнив следующую команду:
<pre><code># On RHEL 7.x
sudo pcs resource move Db2_HADR_<b>ID2</b>-master
# On RHEL 8.x
sudo pcs resource move Db2_HADR_<b>ID2</b>-clone --master
</code></pre>

После завершения миграции выходные данные состояния CRM выглядят следующим образом:
<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb01
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb02 ]
     Stopped: [ az-idb01 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb02
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb02

</code></pre>

Исходное состояние в системе SAP описано в разделе Transaction DBACOCKPIT > Configuration > Overview (Общие сведения о конфигурации), как показано на следующем рисунке:

![DBACockpit — после миграции](./media/high-availability-guide-rhel-ibm-db2-luw/hadr-sap-mgr-post-rhel.png)

При переносе ресурсов с "перемещение ресурсов на ПК" создаются ограничения расположения. Ограничения расположения в этом случае препятствуют запуску экземпляра IBM DB2 на AZ-idb01. Если ограничения расположения не удаляются, ресурс не может восстановить размещение.

Удалите узел с ограничением расположения и резервным узлом, который будет запущен с помощью команды AZ-idb01.
<pre><code># On RHEL 7.x
sudo pcs resource clear Db2_HADR_<b>ID2</b>-master
# On RHEL 8.x
sudo pcs resource clear Db2_HADR_<b>ID2</b>-clone</code></pre>

И состояние кластера изменится на:

<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb01
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb02 ]
     Slaves: [ az-idb01 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb02
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb02</code></pre>

![DBACockpit — ограничение расположения удалено](./media/high-availability-guide-rhel-ibm-db2-luw/hadr-sap-mgr-clear-rhel.png)

Перенесите ресурс обратно в *AZ-idb01* и очистите ограничения расположения.
<pre><code># On RHEL 7.x
sudo pcs resource move Db2_HADR_<b>ID2</b>-master az-idb01
sudo pcs resource clear Db2_HADR_<b>ID2</b>-master
# On RHEL 8.x
sudo pcs resource move Db2_HADR_<b>ID2</b>-clone --master
sudo pcs resource clear Db2_HADR_<b>ID2</b>-clone</code></pre>

- **В RHEL 7. x — перемещение ресурсов на компьютерах \<res_name> <host> :** создает ограничения расположения и может вызвать проблемы с перенаправление
- **На RHEL 8. x — перемещение ресурсов на компьютерах \<res_name> --master:** создание ограничений расположения и может вызвать проблемы с перенаправление
- **ресурсы ПК Clear \<res_name>**: очищает ограничения расположения
- **Очистка \<res_name> ресурсов ПК**: очищает все ошибки ресурса.

### <a name="test-a-manual-takeover"></a>Тестирование перенаправление вручную

Чтобы протестировать ручной перенаправление, остановите службу Pacemaker на узле *AZ-idb01* :
<pre><code>systemctl stop pacemaker</code></pre>

состояние ON *AZ-ibdb02*
<pre><code>2 nodes configured
5 resources configured

Node az-idb01: pending
Online: [ az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb02
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb02 ]
     Stopped: [ az-idb01 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb02
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb02

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled</code></pre>

После отработки отказа службу можно запустить снова с помощью команды *AZ-idb01*.
<pre><code>systemctl start  pacemaker</code></pre>


### <a name="kill-the-db2-process-on-the-node-that-runs-the-hadr-primary-database"></a>Закрытие процесса DB2 на узле, на котором выполняется база данных источника HADR

<pre><code>#Kill main db2 process - db2sysc
[sapadmin@az-idb02 ~]$ sudo ps -ef|grep db2sysc
db2ptr    34598  34596  8 14:21 ?        00:00:07 db2sysc 0
[sapadmin@az-idb02 ~]$ sudo kill -9 34598
</code></pre>

Происходит сбой экземпляра DB2, и Pacemaker перемещает главный узел и сообщает следующее состояние:

<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb02
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb02 ]
     Stopped: [ az-idb01 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb02
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb02

Failed Actions:
* Db2_HADR_ID2_demote_0 on az-idb01 'unknown error' (1): call=49, status=complete, exitreason='none',
    last-rc-change='Wed Jun 26 09:57:35 2019', queued=0ms, exec=362ms</code></pre>

Pacemaker перезапустит экземпляр базы данных DB2 на том же узле или отправит отказ на узел, на котором выполняется экземпляр базы данных-получателя, и появится сообщение об ошибке.

### <a name="kill-the-db2-process-on-the-node-that-runs-the-secondary-database-instance"></a>Завершить процесс DB2 на узле, на котором выполняется экземпляр базы данных-получателя

<pre><code>[sapadmin@az-idb02 ~]$ sudo ps -ef|grep db2sysc
db2id2    23144  23142  2 09:53 ?        00:00:13 db2sysc 0
[sapadmin@az-idb02 ~]$ sudo kill -9 23144
</code></pre>

Узел возвращается в значение Failed и сообщает об ошибке
<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb02
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb01 ]
     Slaves: [ az-idb02 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb01
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb01

Failed Actions:
* Db2_HADR_ID2_monitor_20000 on az-idb02 'not running' (7): call=144, status=complete, exitreason='none',
    last-rc-change='Wed Jun 26 10:02:09 2019', queued=0ms, exec=0ms</code></pre>

Экземпляр DB2 перезапускается во вторичной роли, которой он был назначен ранее.

### <a name="stop-db-via-db2stop-force-on-the-node-that-runs-the-hadr-primary-database-instance"></a>Завершение работы базы данных с помощью db2stop Force на узле, на котором выполняется экземпляр базы данных HADR

Как пользователь DB2 \<sid> EXECUTE команда db2stop Force:
<pre><code>az-idb01:db2ptr> db2stop force</code></pre>

Обнаружен сбой:

<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb02
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Slaves: [ az-idb02 ]
     Stopped: [ az-idb01 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Stopped
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Stopped

Failed Actions:
* Db2_HADR_ID2_demote_0 on az-idb01 'unknown error' (1): call=110, status=complete, exitreason='none',
    last-rc-change='Wed Jun 26 14:03:12 2019', queued=0ms, exec=355ms</code></pre>

Экземпляр базы данных-получателя HADR DB2 был повышен до первичной роли.
<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

 rsc_st_azure   (stonith:fence_azure_arm):      Started az-idb02
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb02 ]
     Slaves: [ az-idb01 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb02
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb02

Failed Actions:
* Db2_HADR_ID2_demote_0 on az-idb01 'unknown error' (1): call=110, status=complete, exitreason='none',
    last-rc-change='Wed Jun 26 14:03:12 2019', queued=0ms, exec=355ms</pre></code>


### <a name="crash-the-vm-that-runs-the-hadr-primary-database-instance-with-halt"></a>Сбой виртуальной машины, на которой выполняется экземпляр базы данных HADR, с "Halt"

<pre><code>#Linux kernel panic.
sudo echo b > /proc/sysrq-trigger</code></pre>

В этом случае Pacemaker обнаружит, что узел, на котором выполняется экземпляр базы данных-источника, не отвечает.

<pre><code>2 nodes configured
5 resources configured

Node az-idb01: UNCLEAN (online)
Online: [ az-idb02 ]

Full list of resources:

rsc_st_azure    (stonith:fence_azure_arm):      Started az-idb02
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb01 ]
     Slaves: [ az-idb02 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb01
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb01</code></pre>

Следующим шагом является проверка на наличие *разделенного мозгового* случая. После того, как работающий узел определил, что узел, на котором выполнялся последний экземпляр базы данных-источника, не работает, выполняется отработка отказа ресурсов.

<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb02 ]
OFFLINE: [ az-idb01 ]

Full list of resources:

rsc_st_azure    (stonith:fence_azure_arm):      Started az-idb02
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb02 ]
     Stopped: [ az-idb01 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb02
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb02 </code></pre>


В случае тревоги ядра неисправный узел будет помечен агентом ограждения. После того как неисправный узел вернется в режим «в сети», необходимо запустить кластер Pacemaker по
<pre><code>sudo pcs cluster start</code></pre> Он запускает экземпляр DB2 во вторичной роли.

<pre><code>2 nodes configured
5 resources configured

Online: [ az-idb01 az-idb02 ]

Full list of resources:

rsc_st_azure    (stonith:fence_azure_arm):      Started az-idb02
 Master/Slave Set: Db2_HADR_ID2-master [Db2_HADR_ID2]
     Masters: [ az-idb02 ]
     Slaves: [ az-idb01 ]
 Resource Group: g_ipnc_db2id2_ID2
     vip_db2id2_ID2     (ocf::heartbeat:IPaddr2):       Started az-idb02
     nc_db2id2_ID2      (ocf::heartbeat:azure-lb):      Started az-idb02</code></pre>

## <a name="next-steps"></a>Дальнейшие действия
- [Высокодоступная архитектура и сценарии для SAP NetWeaver](./sap-high-availability-architecture-scenarios.md)
- [Настройка кластера Pacemaker в Red Hat Enterprise Linux в Azure][rhel-pcs-azr]

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[1612105]:https://launchpad.support.sap.com/#/notes/1612105
[2694118]:https://launchpad.support.sap.com/#/notes/2694118


[db2-hadr-11.1]:https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html
[db2-hadr-10.5]:https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html
[dbms-db2]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_ibm

[sap-instfind]:https://help.sap.com/viewer/9e41ead9f54e44c1ae1a1094b0f80712/ALL/en-US/576f5c1808de4d1abecbd6e503c9ba42.html
[rhel-ha-addon]:https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index
[rhel-ha-admin]:https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index
[rhel-ha-ref]:https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index
[rhel-azr-supp]:https://access.redhat.com/articles/3131341
[rhel-azr-inst]:https://access.redhat.com/articles/3252491
[rhel-db2-supp]:https://access.redhat.com/articles/3144221
[ascs-ha-rhel]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-rhel
[glusterfs]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-rhel-glusterfs
[rhel-pcs-azr]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-rhel-pacemaker
[anf-rhel]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-rhel-netapp-files

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md
[azr-sap-plancheck]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-deployment-checklist
