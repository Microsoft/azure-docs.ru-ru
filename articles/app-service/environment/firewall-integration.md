---
title: Блокировка исходящего трафика
description: Узнайте, как выполнить интеграцию с Брандмауэром Azure для защиты исходящего трафика из Среды службы приложений.
author: ccompy
ms.assetid: 955a4d84-94ca-418d-aa79-b57a5eb8cb85
ms.topic: article
ms.date: 09/24/2020
ms.author: ccompy
ms.custom: seodec18, references_regions
ms.openlocfilehash: ec506546b52a2d137d448f07f4b7a6827c01b4d2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100594127"
---
# <a name="locking-down-an-app-service-environment"></a>Блокирование среды службы приложений

Среда Службы приложений (ССП) имеет ряд внешних зависимостей, доступ к которым необходим для правильной работы. ССП находится в пользовательской виртуальной сети Azure (VNet). Пользователи должны разрешить трафик зависимостей ССП, что является проблемой для тех пользователей, которым нужно заблокировать все выходы из своей виртуальной сети.

Существует ряд входящих конечных точек, которые используются для управления Средой службы приложений. Входящий трафик управления невозможно отправлять через устройство брандмауэра. Исходные адреса этого трафика известны и опубликованы в документе [Адреса управления среды Службы приложений](./management-addresses.md). Доступен также тег службы с именем AppServiceManagement, который можно использовать с группами безопасности сети для защиты входящего трафика.

Исходящие зависимости СПП почти полностью определены с помощью FQDN, которые не содержат статических адресов. Отсутствие статических адресов означает, что для блокировки исходящего из Среды службы приложений трафика нельзя использовать группу безопасности сети. Адреса часто меняются, поэтому создать правила, основанные на текущем разрешении, и использовать их для создания NSG невозможно. 

Решение для защиты исходящих адресов заключается в использовании устройства брандмауэра, которое может контролировать исходящий трафик на основе доменных имен. Брандмауэр Azure может ограничить исходящий трафик HTTP и HTTPS на основе FQDN назначения.  

## <a name="system-architecture"></a>Архитектура системы

Развертывание Среды службы приложений с исходящим трафиком, проходящим через устройство брандмауэра, требует изменения маршрутов в подсети Среды службы приложений. Маршруты работают на уровне протокола IP. Если вы не уделяете достаточного внимания определению маршрутов, вы можете принудительно задать получение трафика ответов TCP с другого адреса. Если адрес ответа отличается от адреса, на который был направлен трафик, возникает проблема, которая называется ассиметричной маршрутизацией и которая приводит к разрыву подключения TCP.

Вам нужно определить маршруты, чтобы входящий трафик к Среде службы приложений мог отправлять ответы по тем же адресам, с которых поступил трафик. Маршруты также должны быть определены для входящих запросов управления и входящих запросов приложений.

Входящий и исходящий трафик Среды службы приложений должен удовлетворять следующим требованиям:

* Трафик к SQL Azure, службе хранилища и концентратору событий не поддерживается при использовании устройства брандмауэра. Такой трафик должен направляться непосредственно в эти службы. Для этого вы можете настроить конечные точки этих служб. 
* Определите таблицы маршрутизации, которые обеспечат отправку входящего трафика управления обратно.
* Определите таблицы маршрутизации, которые обеспечат отправку входящего трафика приложений обратно. 
* Весь остальной трафик, поступающий от Среды службы приложений, можно отправлять на устройство брандмауэра, настроив правило таблицы маршрутизации.

![Сетевые подключения Среды службы приложений с Брандмауэром Azure][5]

## <a name="locking-down-inbound-management-traffic"></a>Блокировка входящего трафика управления

Если вашей подсети Среды службы приложений еще не назначена группа безопасности сети, создайте ее. В группе безопасности сети задайте первое правило, разрешающее трафик от тега службы с именем AppServiceManagement на портах 454 и 455. Правило, разрешающее доступ из тега AppServiceManagement, — это единственное, что требуется для управления Средой службы приложений с общедоступных IP-адресов. Адреса, расположенные за таким тегом службы, используются только для администрирования Службы приложений Azure. Трафик управления, который проходит через эти подключения, шифруется и защищается с помощью сертификатов аутентификации. Типичный трафик на этом канале включает такие данные, как инициированные клиентом команды и пробы работоспособности. 

Среды службы приложений, созданные на портале с новой подсетью, создаются с группой безопасности сети, которая содержит разрешающее правило для тега AppServiceManagement.  

Среда службы приложений также должна разрешать входящие запросы от тега Load Balancer через порт 16001. Запросы от Load Balancer через порт 16001 — это проверки активности между внешними интерфейсами Load Balancer и Средой службы приложений. Если порт 16001 заблокирован, Среда службы приложений перейдет в неработоспособное состояние.

## <a name="configuring-azure-firewall-with-your-ase"></a>Настройка брандмауэра Azure с помощью СПП 

Шаги по блокировке исходящего трафика существующей Среды службы приложений с помощью Брандмауэра Azure:

1. Включите конечные точки служб для SQL, службы хранилища и концентратора событий в подсети ASE. Чтобы включить конечные точки служб, перейдите на портал сети, перейдите к подсетям и выберите Microsoft.EventHub, Microsoft.SQL и Microsoft.Storage в раскрывающемся списке конечных точек служб. Включив конечные точки служб для SQL Azure, необходимо также настроить конечные точки для всех зависимостей SQL Azure, которые есть у ваших приложений. 

   ![Выбор конечных точек служб][2]
  
1. Создайте подсеть с именем AzureFirewallSubnet в виртуальной сети, в которой находится Среда службы приложений. Следуйте указаниям, приведенным в [документации по Брандмауэру Azure](../../firewall/index.yml), чтобы создать Брандмауэр Azure.

1. В пользовательском интерфейсе Брандмауэра Azure откройте "Правила" > "Коллекция правил приложения" и выберите "Добавить коллекцию правил приложения". Укажите имя, приоритет и в поле "Действие" установите значение "Разрешить". В области тегов полных доменных имен введите имя, задайте * в качестве исходного адреса и выберите значение "Среда службы приложений" и "Центр обновления Windows". 
   
   ![Добавление правила приложения][1]
   
1. В пользовательском интерфейсе Брандмауэра Azure откройте "Правила" > "Коллекция правил сети" и выберите "Добавить коллекцию правил сети". Укажите имя, приоритет и в поле "Действие" установите значение "Разрешить". В разделе правила в разделе IP-адреса укажите имя, выберите протокол **любого**, задайте * для адреса источника и назначения и задайте для портов значение 123. Это правило позволяет системе выполнять синхронизации часов, используя NTP. Таким же образом создайте правило для порта 12000, чтобы упростить диагностику проблем с системой. 

   ![Добавление сетевого правила NTP][3]
   
1. В пользовательском интерфейсе Брандмауэра Azure откройте "Правила" > "Коллекция правил сети" и выберите "Добавить коллекцию правил сети". Укажите имя, приоритет и в поле "Действие" установите значение "Разрешить". В разделе "Правила" для тегов служб укажите имя, в поле "Протокол" выберите **Любой**, в поле адреса источника укажите *, выберите тег службы AzureMonitor, а в поле "Порты назначения" укажите 80, 443. Это правило позволяет системе передавать Azure Monitor сведения о работоспособности и метриках.

   ![Добавление правила сети с тегом службы NTP][6]
   
1. Создайте таблицу маршрутов с помощью адресов управления из документации [Адреса управления среды Службы приложений]( ./management-addresses.md) со следующим прыжком Интернета. Записи таблицы маршрутов необходимы для предотвращения проблем с асимметричной маршрутизацией. Добавьте маршруты для зависимостей IP-адресов, указанных ниже в зависимостях IP-адреса, со следующим прыжком Интернета. Добавьте маршрут виртуального модуля к таблице маршрутов для 0.0.0.0/0, при этом следующий прыжок должен представлять частный IP-адрес вашего Брандмауэра Azure. 

   ![Создание таблицы маршрутов][4]
   
1. Назначьте таблицу маршрутов, созданную в вашей подсети Среды службы приложений.

#### <a name="deploying-your-ase-behind-a-firewall"></a>Развертывание Среды службы приложений за брандмауэром

Шаги по развертыванию Среды службы приложений с защитой брандмауэром аналогичны настройке существующей Среды службы приложений с Брандмауэром Azure, за исключением того, что необходимо будет создать соответствующую подсеть и выполнить предыдущие шаги. Чтобы создать Среду службы приложений в существующей подсети, необходимо [использовать шаблон Resource Manager](./create-from-template.md).

## <a name="application-traffic"></a>Трафик приложения 

Вышеупомянутые шаги позволят вашей СПП работать без проблем. Чтобы удовлетворить потребности приложения, вам все же необходимо выполнить некоторые настройки. С приложениями в СПП, настроенными с помощью брандмауэра Azure, возникают две проблемы.  

- В Брандмауэр Azure или таблицу маршрутов необходимо добавить зависимости приложения. 
- Маршруты должны создаваться для трафика приложения, чтобы избежать проблемы с асимметричной маршрутизацией.

Если ваше приложение имеет зависимости, то их необходимо добавить в брандмауэр Azure. Создайте правила приложения, чтобы разрешать трафик протокола HTTP/HTTPS, и правила сети для всего остального. 

Если вам известен диапазон адресов, из которых будет поступать трафик запроса к приложению, то вы можете добавить его в таблицу маршрутов, назначенную вашей подсети СПП. Если диапазон адресов большой или не задан, то вы можете использовать сетевое устройство, такое как приложение Gateway, которое даст вам один адрес, который необходимо добавить в таблицу маршрутов. Дополнительные сведения о настройке шлюза приложений с подсистемой балансировки нагрузки СПП см. в статье [Интеграция среды Службы приложений с внутренней подсистемой балансировки нагрузки со шлюзом приложений Azure](./integrate-with-application-gateway.md)

Такое использование Шлюза приложений — один из примеров того, как настроить систему. Если вы предпочтете этот вариант, вам понадобится добавить маршрут к таблице маршрутов подсети Среды службы приложений, чтобы ответный трафик, передаваемый в Шлюз приложений, поступал туда напрямую. 

## <a name="logging"></a>Logging 

Брандмауэр Azure может отправлять журналы в службу хранилища Azure, концентратор событий или в журналы Azure Monitor. Чтобы интегрировать приложение с любым поддерживаемым местом назначения, перейдите в раздел "Журналы диагностики"на портале Брандмауэра Azure и включите журналы для требуемого расположения. При интеграции с журналами Azure Monitor вы сможете увидеть ведение журналов с любым трафиком, отправляемым в Брандмауэр Azure. Чтобы просмотреть отклоняемый трафик, откройте раздел "Журналы" на портале Log Analytics и введите следующий запрос. 

```kusto
AzureDiagnostics | where msg_s contains "Deny" | where TimeGenerated >= ago(1h)
```

Интеграция Брандмауэра Azure с журналами Azure Monitor полезна при первом рабочем запуске приложения, когда вам известны не все зависимости приложения. Дополнительные сведения о журналах Azure Monitor см. в статье [Анализ данных журнала в Azure Monitor](../../azure-monitor/logs/log-query-overview.md).
 
## <a name="dependencies"></a>Зависимости

Следующие сведения требуются, только если вы хотите настроить не Брандмауэр Azure, а другой брандмауэр. 

- В разделе "Конечные точки" должны быть указаны конечные точки для включенных служб.
- Зависимости IP-адреса задаются для трафика, не использующего протокол HTTP/HTTPS (TCP и UDP-трафик)
- В устройство брандмауэра можно добавить FQDN конечных точек для протокола HTTP/HTTPS.
- Произвольные конечные точки для HTTP/HTTPS — это зависимости, которые могут меняться вместе со средой СПП в зависимости от количества квалификаторов. 
- Зависимости Linux становятся предметом беспокойства только при развертывании Linux-приложений в СПП. Если вы не развертываете Linux-приложения в СПП, то вам не обязательно добавлять эти адреса в брандмауэр. 

#### <a name="service-endpoint-capable-dependencies"></a>Зависимости конечной точки службы 

| Конечная точка |
|----------|
| Azure SQL |
| Хранилище Azure |
| концентратору событий Azure |

#### <a name="ip-address-dependencies"></a>Зависимости IP-адреса

| Конечная точка | Сведения |
|----------| ----- |
| \*:123 | Проверка часов NTP. Трафик проверяется в нескольких конечных точках на порте 123. |
| \*:12000 | Этот порт используется для некоторых операций мониторинга системы. Если он заблокирован, некоторые проблемы будет сложнее диагностировать, но Среда службы приложений сохранит работоспособность. |
| 40.77.24.27:80 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 40.77.24.27:443 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 13.90.249.229:80 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 13.90.249.229:443 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 104.45.230.69:80 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 104.45.230.69:443 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 13.82.184.151:80 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 13.82.184.151:443 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |

При подключении Брандмауэра Azure все перечисленные ниже параметры автоматически настраиваются с тегами полных доменных имен. 

#### <a name="fqdn-httphttps-dependencies"></a>Зависимости FQDN протокола HTTP/HTTPS 

| Конечная точка |
|----------|
|graph.microsoft.com:443 |
|login.live.com:443 |
|login.windows.com:443 |
|login.windows.net:443 |
|login.microsoftonline.com:443 |
|client.wns.windows.com:443 |
|definitionupdates.microsoft.com:443 |
|go.microsoft.com:80 |
|go.microsoft.com:443 |
|www.microsoft.com:80 |
|www.microsoft.com:443 |
|wdcpalt.microsoft.com:443 |
|wdcp.microsoft.com:443 |
|ocsp.msocsp.com:443 |
|ocsp.msocsp.com:80 |
|oneocsp.microsoft.com:80 |
|oneocsp.microsoft.com:443 |
|mscrl.microsoft.com:443 |
|mscrl.microsoft.com:80 |
|crl.microsoft.com:443 |
|crl.microsoft.com:80 |
|www.thawte.com:443 |
|crl3.digicert.com:80 |
|ocsp.digicert.com:80 |
|ocsp.digicert.com:443 |
|csc3-2009-2.crl.verisign.com:80 |
|crl.verisign.com:80 |
|ocsp.verisign.com:80 |
|cacerts.digicert.com:80 |
|azperfcounters1.blob.core.windows.net:443 |
|azurewatsonanalysis-prod.core.windows.net:443 |
|global.metrics.nsatc.net:80 |
|global.metrics.nsatc.net:443 |
|az-prod.metrics.nsatc.net:443 |
|antares.metrics.nsatc.net:443 |
|azglobal-black.azglobal.metrics.nsatc.net:443 |
|azglobal-red.azglobal.metrics.nsatc.net:443 |
|antares-black.antares.metrics.nsatc.net:443 |
|antares-red.antares.metrics.nsatc.net:443 |
|prod.microsoftmetrics.com:443 |
|maupdateaccount.blob.core.windows.net:443 |
|clientconfig.passport.net:443 |
|packages.microsoft.com:443 |
|schemas.microsoft.com:80 |
|schemas.microsoft.com:443 |
|management.core.windows.net:443 |
|management.core.windows.net:80 |
|management.azure.com:443 |
|www.msftconnecttest.com:80 |
|shavamanifestcdnprod1.azureedge.net:443 |
|validation-v2.sls.microsoft.com:443 |
|flighting.cp.wd.microsoft.com:443 |
|dmd.metaservices.microsoft.com:80 |
|admin.core.windows.net:443 |
|prod.warmpath.msftcloudes.com:443 |
|prod.warmpath.msftcloudes.com:80 |
|gcs.prod.monitoring.core.windows.net:80|
|gcs.prod.monitoring.core.windows.net:443|
|azureprofileruploads.blob.core.windows.net:443 |
|azureprofileruploads2.blob.core.windows.net:443 |
|azureprofileruploads3.blob.core.windows.net:443 |
|azureprofileruploads4.blob.core.windows.net:443 |
|azureprofileruploads5.blob.core.windows.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azprofileruploads1.blob.core.windows.net:443 |
|azprofileruploads10.blob.core.windows.net:443 |
|azprofileruploads2.blob.core.windows.net:443 |
|azprofileruploads3.blob.core.windows.net:443 |
|azprofileruploads4.blob.core.windows.net:443 |
|azprofileruploads6.blob.core.windows.net:443 |
|azprofileruploads7.blob.core.windows.net:443 |
|azprofileruploads8.blob.core.windows.net:443 | 
|azprofileruploads9.blob.core.windows.net:443 |
|azureprofilerfrontdoor.cloudapp.net:443 |
|settings-win.data.microsoft.com:443 |
|maupdateaccount2.blob.core.windows.net:443 |
|maupdateaccount3.blob.core.windows.net:443 |
|dc.services.visualstudio.com:443 |
|gmstorageprodsn1.blob.core.windows.net:443 |
|gmstorageprodsn1.file.core.windows.net:443 |
|gmstorageprodsn1.queue.core.windows.net:443 |
|gmstorageprodsn1.table.core.windows.net:443 |
|rteventservice.trafficmanager.net:443 |
|ctldl.windowsupdate.com:80 |
|ctldl.windowsupdate.com:443 |
|global-dsms.dsms.core.windows.net:443 |

#### <a name="wildcard-httphttps-dependencies"></a>Зависимости подстановочного знака протокола HTTP/HTTPS 

| Конечная точка |
|----------|
|gr-Prod-\*.cloudapp.net:443 |
| \*.management.azure.com:443 |
| \*.update.microsoft.com:443 |
| \*.windowsupdate.microsoft.com:443 |
| \*.identity.azure.net:443 |
| \*.ctldl.windowsupdate.com:80 |
| \*.ctldl.windowsupdate.com:443 |

#### <a name="linux-dependencies"></a>Зависимости Linux 

| Конечная точка |
|----------|
|wawsinfraprodbay063.blob.core.windows.net:443 |
|registry-1.docker.io:443 |
|auth.docker.io:443 |
|production.cloudflare.docker.com:443 |
|download.docker.com:443 |
|us.archive.ubuntu.com:80 |
|download.mono-project.com:80 |
|packages.treasuredata.com:80|
|security.ubuntu.com:80 |
|oryx-cdn.microsoft.io:443 |
| \*.cdn.mscr.io:443 |
| \*. data.mcr.microsoft.com:443 |
|mcr.microsoft.com:443 |
|\*. data.mcr.microsoft.com:443 |
|packages.fluentbit.io:80 |
|packages.fluentbit.io:443 |
|apt-mo.trafficmanager.net:80 |
|apt-mo.trafficmanager.net:443 |
|azure.archive.ubuntu.com:80 |
|azure.archive.ubuntu.com:443 |
|changelogs.ubuntu.com:80 |
|13.74.252.37:11371 |
|13.75.127.55:11371 |
|13.76.190.189:11371 |
|13.80.10.205:11371 |
|13.91.48.226:11371 |
|40.76.35.62:11371 |
|104.215.95.108:11371 |

## <a name="us-gov-dependencies"></a>Зависимости регионов US Gov

Если вы используете Среды службы приложений в регионах US Gov, выполните инструкции из [этого раздела](#configuring-azure-firewall-with-your-ase), чтобы настроить Брандмауэр Azure для Среды службы приложений.

Если вы хотите использовать в регионе US Gov другое устройство (не Брандмауэр Azure): 

* В разделе "Конечные точки" должны быть указаны конечные точки для включенных служб.
* В устройство брандмауэра можно добавить FQDN конечных точек для протокола HTTP/HTTPS.
* Произвольные конечные точки для HTTP/HTTPS — это зависимости, которые могут меняться вместе со средой СПП в зависимости от количества квалификаторов.

ОС Linux недоступна в регионах US Gov и поэтому не указана в качестве дополнительной конфигурации.

#### <a name="service-endpoint-capable-dependencies"></a>Зависимости конечной точки службы ####

| Конечная точка |
|----------|
| Azure SQL |
| Хранилище Azure |
| концентратору событий Azure |

#### <a name="ip-address-dependencies"></a>Зависимости IP-адреса

| Конечная точка | Сведения |
|----------| ----- |
| \*:123 | Проверка часов NTP. Трафик проверяется в нескольких конечных точках на порте 123. |
| \*:12000 | Этот порт используется для некоторых операций мониторинга системы. Если он заблокирован, некоторые проблемы будет сложнее диагностировать, но Среда службы приложений сохранит работоспособность. |
| 40.77.24.27:80 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 40.77.24.27:443 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 13.90.249.229:80 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 13.90.249.229:443 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 104.45.230.69:80 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 104.45.230.69:443 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 13.82.184.151:80 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |
| 13.82.184.151:443 | Требуется для отслеживания проблем со Средой службы приложений и создания оповещений о них. |

#### <a name="dependencies"></a>Зависимости ####

| Конечная точка |
|----------|
| \*.ctldl.windowsupdate.com:80 |
| \*.management.usgovcloudapi.net:80 |
| \*.update.microsoft.com:80 |
|admin.core.usgovcloudapi.net:80 |
|azperfmerges.blob.core.windows.net:80 |
|azperfmerges.blob.core.windows.net:80 |
|azprofileruploads1.blob.core.windows.net:80 |
|azprofileruploads10.blob.core.windows.net:80 |
|azprofileruploads2.blob.core.windows.net:80 |
|azprofileruploads3.blob.core.windows.net:80 |
|azprofileruploads4.blob.core.windows.net:80 |
|azprofileruploads5.blob.core.windows.net:80 |
|azprofileruploads6.blob.core.windows.net:80 |
|azprofileruploads7.blob.core.windows.net:80 |
|azprofileruploads8.blob.core.windows.net:80 |
|azprofileruploads9.blob.core.windows.net:80 |
|azureprofilerfrontdoor.cloudapp.net:80 |
|azurewatsonanalysis.usgovcloudapp.net:80 |
|cacerts.digicert.com:80 |
|client.wns.windows.com:80 |
|crl.microsoft.com:80 |
|crl.verisign.com:80 |
|crl3.digicert.com:80 |
|csc3-2009-2.crl.verisign.com:80 |
|ctldl.windowsupdate.com:80 |
|definitionupdates.microsoft.com:80 |
|download.windowsupdate.com:80 |
|fairfax.warmpath.usgovcloudapi.net:80 |
|flighting.cp.wd.microsoft.com:80 |
|gcwsprodgmdm2billing.queue.core.usgovcloudapi.net:80 |
|gcwsprodgmdm2billing.table.core.usgovcloudapi.net:80 |
|global.metrics.nsatc.net:80 |
|go.microsoft.com:80 |
|gr-gcws-prod-bd3.usgovcloudapp.net:80 |
|gr-gcws-prod-bn1.usgovcloudapp.net:80 |
|gr-gcws-prod-dd3.usgovcloudapp.net:80 |
|gr-gcws-prod-dm2.usgovcloudapp.net:80 |
|gr-gcws-prod-phx20.usgovcloudapp.net:80 |
|gr-gcws-prod-sn5.usgovcloudapp.net:80 |
|login.live.com:80 |
|login.microsoftonline.us:80 |
|management.core.usgovcloudapi.net:80 |
|management.usgovcloudapi.net:80 |
|maupdateaccountff.blob.core.usgovcloudapi.net:80 |
|mscrl.microsoft.com:80
|ocsp.digicert.com:80 |
|ocsp.verisign.com:80 |
|rteventse.trafficmanager.net:80 |
|settings-n.data.microsoft.com:80 |
|shavamafestcdnprod1.azureedge.net:80 |
|shavanifestcdnprod1.azureedge.net:80 |
|v10ortex-win.data.microsoft.com:80 |
|wp.microsoft.com:80 |
|dcpalt.microsoft.com:80 |
|www.microsoft.com:80 |
|www.msftconnecttest.com:80 |
|www.thawte.com:80 |
|\*ctldl.windowsupdate.com:443 |
|\*.management.usgovcloudapi.net:443 |
|\*.update.microsoft.com:443 |
|admin.core.usgovcloudapi.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azprofileruploads1.blob.core.windows.net:443 |
|azprofileruploads10.blob.core.windows.net:443 |
|azprofileruploads2.blob.core.windows.net:443 |
|azprofileruploads3.blob.core.windows.net:443 |
|azprofileruploads4.blob.core.windows.net:443 |
|azprofileruploads5.blob.core.windows.net:443 |
|azprofileruploads6.blob.core.windows.net:443 |
|azprofileruploads7.blob.core.windows.net:443 |
|azprofileruploads8.blob.core.windows.net:443 |
|azprofileruploads9.blob.core.windows.net:443 |
|azureprofilerfrontdoor.cloudapp.net:443 |
|azurewatsonanalysis.usgovcloudapp.net:443 |
|cacerts.digicert.com:443 |
|client.wns.windows.com:443 |
|crl.microsoft.com:443 |
|crl.verisign.com:443 |
|crl3.digicert.com:443 |
|csc3-2009-2.crl.verisign.com:443 |
|ctldl.windowsupdate.com:443 |
|definitionupdates.microsoft.com:443 |
|download.windowsupdate.com:443 |
|fairfax.warmpath.usgovcloudapi.net:443 |
|gcs.monitoring.core.usgovcloudapi.net:443 |
|flighting.cp.wd.microsoft.com:443 |
|gcwsprodgmdm2billing.queue.core.usgovcloudapi.net:443 |
|gcwsprodgmdm2billing.table.core.usgovcloudapi.net:443 |
|global.metrics.nsatc.net:443 |
|go.microsoft.com:443 |
|gr-gcws-prod-bd3.usgovcloudapp.net:443 |
|gr-gcws-prod-bn1.usgovcloudapp.net:443 |
|gr-gcws-prod-dd3.usgovcloudapp.net:443 |
|gr-gcws-prod-dm2.usgovcloudapp.net:443 |
|gr-gcws-prod-phx20.usgovcloudapp.net:443 |
|gr-gcws-prod-sn5.usgovcloudapp.net:443 |
|login.live.com:443 |
|login.microsoftonline.us:443 |
|management.core.usgovcloudapi.net:443 |
|management.usgovcloudapi.net:443 |
|maupdateaccountff.blob.core.usgovcloudapi.net:443 |
|mscrl.microsoft.com:443 |
|ocsp.digicert.com:443 |
|ocsp.msocsp.com:443 |
|ocsp.msocsp.com:80 |
|oneocsp.microsoft.com:80 |
|oneocsp.microsoft.com:443 |
|ocsp.verisign.com:443 |
|rteventservice.trafficmanager.net:443 |
|settings-win.data.microsoft.com:443 |
|shavamanifestcdnprod1.azureedge.net:443 |
|shavamanifestcdnprod1.azureedge.net:443 |
|v10.vortex-win.data.microsoft.com:443 |
|wdcp.microsoft.com:443 |
|wdcpalt.microsoft.com:443 |
|www.microsoft.com:443 |
|www.msftconnecttest.com:443 |
|www.thawte.com:443 |
|global-dsms.dsms.core.usgovcloudapi.net:443 |

<!--Image references-->
[1]: ./media/firewall-integration/firewall-apprule.png
[2]: ./media/firewall-integration/firewall-serviceendpoints.png
[3]: ./media/firewall-integration/firewall-ntprule.png
[4]: ./media/firewall-integration/firewall-routetable.png
[5]: ./media/firewall-integration/firewall-topology.png
[6]: ./media/firewall-integration/firewall-ntprule-monitor.png
