---
title: Мониторинг и устранение неполадок со стороны SAP HANA в Azure (крупные экземпляры) | Документация Майкрософт
description: Мониторинг и устранение неполадок со стороны SAP HANA в Azure (крупные экземпляры).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 83743a6985bef8ce6c03e01ed8d10aa740852106
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101668811"
---
# <a name="monitoring-and-troubleshooting-from-hana-side"></a>Мониторинг и устранение неполадок со стороны HANA

Чтобы эффективно анализировать проблемы, связанные с SAP HANA в Azure (крупные экземпляры), полезно сузить круг возможных причин проблемы. Компания SAP опубликовала огромное количество справочных материалов, которые могут вам помочь.

Ответы на часто задаваемые вопросы, связанные с производительностью SAP HANA, можно найти в следующих примечаниях SAP:

- [SAP Note #2222200 – FAQ: SAP HANA Network](https://launchpad.support.sap.com/#/notes/2222200) (Примечание SAP № 2222200. Часто задаваемые вопросы: сеть SAP HANA)
- [SAP Note #2100040 – FAQ: SAP HANA CPU](https://launchpad.support.sap.com/#/notes/0002100040) (Примечание SAP № 2100040. Часто задаваемые вопросы: ЦП SAP HANA)
- [SAP Note #199997 – FAQ: SAP HANA Memory](https://launchpad.support.sap.com/#/notes/2177064) (Примечание SAP № 199997. Часто задаваемые вопросы: память SAP HANA)
- [SAP Note #200000 – FAQ: SAP HANA Performance Optimization](https://launchpad.support.sap.com/#/notes/2000000) (Примечание SAP № 200000. Часто задаваемые вопросы: оптимизация производительности SAP HANA)
- [SAP Note #199930 – FAQ: SAP HANA I/O Analysis](https://launchpad.support.sap.com/#/notes/1999930) (Примечание SAP № 199930. Часто задаваемые вопросы: анализ операций ввода-вывода SAP HANA)
- [SAP Note #2177064 – FAQ: SAP HANA Service Restart and Crashes](https://launchpad.support.sap.com/#/notes/2177064) (Примечание SAP № 2177064. Часто задаваемые вопросы: перезапуск и сбои службы SAP HANA)

## <a name="sap-hana-alerts"></a>Оповещения SAP HANA

Для начала проверьте текущие журналы оповещений SAP HANA. В SAP HANA Studio перейдите в **консоль администрирования: оповещения: показывать: все оповещения**. На этой вкладке показаны все оповещения SAP HANA для определенных значений (объем свободной физической памяти, ЦП и т. д.), которые выходят за пределы установленных минимальных и максимальных пороговых значений. По умолчанию проверки автоматически обновляются каждые 15 минут.

![В SAP HANA Studio выберите Administration Console: Alerts: Show: all alerts (Консоль администрирования: Оповещения: Показать: Все оповещения).](./media/troubleshooting-monitoring/image1-show-alerts.png)

## <a name="cpu"></a>ЦП

Если оповещение запускается из-за неправильно заданного порогового значения, восстановите значение по умолчанию или задайте приемлемое пороговое значение.

![Восстановление значения по умолчанию или задание приемлемого порогового значения](./media/troubleshooting-monitoring/image2-cpu-utilization.png)

Следующие оповещения могут указывать на проблемы с ресурсами ЦП:

- использование ЦП узла (оповещение 5);
- последняя операция точки сохранения (оповещение 28);
- длительность точки сохранения (оповещение 54).

Высокое потребление ресурсов ЦП в базе данных SAP HANA можно заметить по одному из следующих признаков:

- для текущего использования ЦП или использования ЦП в прошлом возникает оповещение 5 (использование ЦП узла);
- отображаемое потребление ресурсов ЦП на экране обзора.

![Отображаемое потребление ресурсов ЦП на экране обзора](./media/troubleshooting-monitoring/image3-cpu-usage.png)

На графике Load (Нагрузка) может быть показан высокий уровень потребления ресурсов ЦП или высокий уровень потребления в прошлом.

![На графике Load (Нагрузка) может быть показан высокий уровень потребления ресурсов ЦП или высокий уровень потребления в прошлом](./media/troubleshooting-monitoring/image4-load-graph.png)

Предупреждение, инициированное из-за высокой загрузки ЦП, может быть вызвано несколькими причинами, включая, помимо прочего, выполнение определенных транзакций, загрузку данных, задания, не отвечающие на запросы, длительные инструкции SQL и производительность запросов (например, с BW в кубах HANA).

Подробные инструкции по устранению неполадок см. в разделе [CPU Related Root Causes and Solutions](https://help.sap.com/saphelp_hanaplatform/helpdata/en/4f/bc915462db406aa2fe92b708b95189/content.htm?frameset=/en/db/6ca50424714af8b370960c04ce667b/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=46&amp;show_children=false) (Основные причины проблем с ЦП и их решения) на сайте SAP HANA.

## <a name="operating-system"></a>Операционная система

Сведения об одной из самых важных проверок SAP HANA на Linux, которая заключается в проверке того, отключены ли Transparent Huge Pages ("прозрачные страницы"), см. в примечании [SAP Note #2131662 – Transparent Huge Pages (THP) on SAP HANA Servers](https://launchpad.support.sap.com/#/notes/2131662) (Примечание SAP № 2131662. Transparent Huge Pages (THP) на серверах SAP HANA).

- Проверить, включены ли Transparent Huge Pages, можно с помощью команды Linux: **cat /sys/kernel/mm/transparent\_hugepage/enabled**.
- Если _always_ заключено в квадратные скобки, как показано ниже, это означает, что Transparent Huge Pages включены: [always] madvise never. Если _never_ заключено в квадратные скобки, как показано ниже, это означает, что Transparent Huge Pages отключены: always madvise [never].

Следующая команда Linux не должна что-либо возвращать: **rpm - qa | grep ulimit.** Если _ulimit_ установлено, удалите его немедленно.

## <a name="memory"></a>Память

Вы можете заметить, что объем памяти, выделенный базой данных SAP HANA, выше, чем ожидалось. Следующие оповещения указывают на неполадки в связи с использованием большого объема памяти:

- использование физической памяти узла (оповещение 1);
- использование памяти сервера доменных имен (оповещение 12);
- общее использование памяти таблиц хранилища столбцов (оповещение 40);
- использование памяти служб (оповещение 43);
- использование памяти основного хранилища таблиц хранилища столбцов (оповещение 45);
- файлы дампа среды выполнения (оповещение 46).

Подробные инструкции по устранению неполадок см. в разделе [Memory Problems](https://help.sap.com/saphelp_hanaplatform/helpdata/en/db/6ca50424714af8b370960c04ce667b/content.htm?frameset=/en/59/5eaa513dde43758b51378ab3315ebb/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=26&amp;show_children=false) (Проблемы с памятью) на сайте SAP HANA.

## <a name="network"></a>Сеть

См. примечание [SAP Note #2081065 – Troubleshooting SAP HANA Network](https://launchpad.support.sap.com/#/notes/2081065) (Примечание SAP № 2081065. Устранение неполадок сети SAP HANA) и выполните действия по устранению неполадок сети, приведенные в этом примечании SAP.

1. Анализ времени кругового пути между сервером и клиентом.
  A. Запустите скрипт SQL [_HANA\_Network\_Clients_](https://launchpad.support.sap.com/#/notes/1969700)_._
  
2. Выполните анализ обмена данными между узлами.
  A. Запустите скрипт SQL [_HANA\_Network\_Services_](https://launchpad.support.sap.com/#/notes/1969700)_._

3. Выполните команду Linux **ifconfig** (в выходных данных показано, есть ли какие-либо потери пакетов).
4. Выполните команду Linux **tcpdump**.

Кроме того, используйте инструмент с открытым исходным кодом [IPERF](https://iperf.fr/) (или что-то похожее), чтобы измерить реальную производительность сети приложения.

Подробные инструкции по устранению неполадок см. в разделе [Network Performance and Connectivity Problems](https://help.sap.com/saphelp_hanaplatform/helpdata/en/a3/ccdff1aedc4720acb24ed8826938b6/content.htm?frameset=/en/dc/6ff98fa36541e997e4c719a632cbd8/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=142&amp;show_children=false) (Проблемы с производительностью сети и подключением) на сайте SAP HANA.

## <a name="storage"></a>Служба хранилища

С точки зрения конечных пользователей приложение (или система в целом) работает медленно, не отвечает или даже может перестать отвечать, если возникают проблемы с производительностью операций ввода-вывода. На вкладке **Volumes** (Тома) в SAP HANA Studio отображаются подключенные тома, а также то, какие тома использует каждая служба.

![На вкладке Volumes (Тома) в SAP HANA Studio отображаются подключенные тома, а также то, какие тома использует каждая служба](./media/troubleshooting-monitoring/image5-volumes-tab-a.png)

В подключенных томах в нижней части экрана можно просматривать сведения о томах, например о файлах и статистике операций ввода-вывода.

![В подключенных томах в нижней части экрана можно просматривать сведения о томах, например о файлах и статистике операций ввода-вывода](./media/troubleshooting-monitoring/image6-volumes-tab-b.png)

Подробные инструкции по устранению неполадок см. в разделах [I/O Related Root Causes and Solutions](https://help.sap.com/saphelp_hanaplatform/helpdata/en/dc/6ff98fa36541e997e4c719a632cbd8/content.htm?frameset=/en/47/4cb08a715c42fe9f7cc5efdc599959/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=55&amp;show_children=false) (Основные причины проблем с операциями ввода-вывода и их решения) и [Disk Related Root Causes and Solutions](https://help.sap.com/saphelp_hanaplatform/helpdata/en/47/4cb08a715c42fe9f7cc5efdc599959/content.htm?frameset=/en/44/3e1db4f73d42da859008df4f69e37a/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=53&amp;show_children=false) (Основные причины проблем с диском и их решения) на сайте SAP HANA.

## <a name="diagnostic-tools"></a>Диагностические средства

Выполните проверку работоспособности SAP HANA с помощью HANA\_Configuration\_Minichecks. Это средство возвращает потенциальные критические технические проблемы, которые должны вызвать оповещения в SAP HANA Studio.

См. примечание [SAP Note #1969700 – SQL statement collection for SAP HANA](https://launchpad.support.sap.com/#/notes/1969700) (Примечание SAP № 1969700. Коллекция инструкций SQL для SAP HANA) и скачайте файл SQL Statements.zip, прикрепленный к этому примечанию. Сохраните этот ZIP-файл на локальном жестком диске.

В SAP HANA Studio на вкладке **сведения о системе** щелкните правой кнопкой мыши столбец **имя** и выберите команду **импортировать инструкции SQL**.

![В SAP HANA Studio на вкладке System Information (Сведения о системе) щелкните правой кнопкой мыши столбец Name (Имя) и выберите Import SQL Statements (Импорт инструкций SQL)](./media/troubleshooting-monitoring/image7-import-statements-a.png)

Выберите файл SQL Statements.zip, хранящийся локально, после чего будет импортирована папка с соответствующими инструкциями SQL. На этом этапе с помощью этих инструкций SQL можно выполнить множество различных диагностических проверок.

Например, чтобы проверить требования к пропускной способности при репликации системы SAP HANA, щелкните правой кнопкой мыши инструкцию **Bandwidth** (Пропускная способность), выбрав **Replication: Bandwidth** (Репликация: Пропускная способность), и щелкните **Open** (Открыть) в консоли SQL.

Откроется полная инструкция SQL, что позволит изменить входные параметры (раздел изменения), а затем выполнить их.

![Откроется полная инструкция SQL, что позволит изменить входные параметры (раздел изменения), а затем выполнить их](./media/troubleshooting-monitoring/image8-import-statements-b.png)

Другой пример — щелкнуть правой кнопкой мыши инструкции, выбрав **Replication: Overview** (Репликация: Обзор). В контекстном меню выберите **выполнить** .

![Другой пример — щелкнуть правой кнопкой мыши инструкции, выбрав Replication: Overview (Репликация: Обзор). Выбор Execute (Выполнить) в контекстном меню](./media/troubleshooting-monitoring/image9-import-statements-c.png)

После этого отобразятся сведения, которые помогут в устранении неполадок.

![После этого отобразятся сведения, которые помогут в устранении неполадок](./media/troubleshooting-monitoring/image10-import-statements-d.png)

Сделайте то же самое для HANA\_Configuration\_Minichecks и проверьте наличие любых меток _X_ в столбце _C_ (Критический).

Пример выходных данных

**HANA\_Configuration\_MiniChecks\_Rev102.01 + 1** для общих проверок SAP HANA.

![HANA\_Configuration\_MiniChecks\_Rev102.01 + 1 для общих проверок SAP HANA](./media/troubleshooting-monitoring/image11-configuration-minichecks.png)

**HANA\_Services\_Overview** для получения общих сведений о том, какие службы SAP HANA запущены в данный момент.

![HANA\_Services\_Overview для получения общих сведений о том, какие службы SAP HANA запущены в данный момент](./media/troubleshooting-monitoring/image12-services-overview.png)

**HANA\_Services\_Statistics** для получения сведений о службе SAP HANA (ЦП, память и т. д.).

![HANA\_Services\_Statistics для получения сведений о службе SAP HANA](./media/troubleshooting-monitoring/image13-services-statistics.png)

**HANA\_Configuration\_Overview\_Rev110 +** для получения общих сведений об экземпляре SAP HANA.

![HANA\_Configuration\_Overview\_Rev110 + для получения общих сведений об экземпляре SAP HANA](./media/troubleshooting-monitoring/image14-configuration-overview.png)

**Hana \_ \_Параметры конфигурации \_ Rev70 +** для проверки параметров SAP HANA.

![HANA\_Configuration\_Parameters\_Rev70 + для проверки параметров SAP HANA](./media/troubleshooting-monitoring/image15-configuration-parameters.png)

**Дальнейшие действия**

- См. статью о [настройке высокого уровня доступности в SUSE с помощью STONITH](ha-setup-with-stonith.md).