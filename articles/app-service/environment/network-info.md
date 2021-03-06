---
title: Рекомендации по работе с сетями
description: Сведения о сетевом трафике ASE и о настройке групп безопасности сети и определяемых пользователем маршрутов с помощью ASE.
author: ccompy
ms.assetid: 955a4d84-94ca-418d-aa79-b57a5eb8cb85
ms.topic: article
ms.date: 07/27/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 91b6134e7c809a8af75aa1cf23523e352e0a1a0e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95997347"
---
# <a name="networking-considerations-for-an-app-service-environment"></a>Рекомендации по работе с сетями в среде службы приложений #

## <a name="overview"></a>Обзор ##

 [Среда службы приложений][Intro] — это развернутая служба приложений Azure в подсети виртуальной сети Azure. Есть два типа развертывания для среды службы приложений (ASE):

- **Внешняя ASE** — предоставляет приложения, размещенные в ASE, по доступному в Интернете IP-адресу. Дополнительные сведения см. в разделе о [создании внешней среды ASE][MakeExternalASE].
- **ASE с ILB** — предоставляет приложения в ASE по IP-адресу в вашей виртуальной сети Azure. Внутренняя конечная точка является внутренним балансировщиком нагрузки (ILB). Поэтому среда называется ASE с ILB. Дополнительные сведения см. в статье о [создании и использовании ILB для ASE][MakeILBASE].

Все среды ASE, External и ILB имеют общедоступный виртуальный IP-адрес, который используется для входящего трафика управления и как адрес отклонения при выполнении вызовов из ASE в Интернет. Вызовы из ASE, которые переходят в Интернет, оставляют виртуальную сеть через виртуальный IP-адрес, назначенный для ASE. Общедоступным IP-адресом этого виртуального IP-адреса является исходный IP-адрес для всех вызовов из ССП, поступающих в Интернет. Если приложения в вашей среде ASE отправляют вызовы к ресурсам в виртуальной сети или в VPN, то исходный IP-адрес является одним из IP-адресов в подсети, используемой ASE. Среда ASE размещена в виртуальной сети, и ее ресурсы доступны для ASE без дополнительной настройки. Если виртуальная сеть подключена к локальной сети, ее ресурсы также доступны для приложений в среде ССП без дополнительных настроек.

![Внешняя среда ASE][1] 

При использовании внешней ASE общедоступный виртуальный IP-адрес также является конечной точкой, с которой приложения ASE сопоставляют данные для:

* HTTP/S 
* FTP/S
* Веб-развертывание
* Удаленная отладка

![Среда ASE с ILB][2]

Если у вас есть ASE ILB, адрес ILB является конечной точкой для HTTP/S, FTP/S, веб-развертывания и удаленной отладки.

## <a name="ase-subnet-size"></a>Размер подсети ASE ##

После развертывания ASE нельзя изменять размер подсети, используемой для размещения ASE.  В ASE используется адрес для каждой роли инфраструктуры и каждого изолированного экземпляра плана службы приложений.  Кроме того, существует пять адресов, используемых сетью Azure для каждой создаваемой подсети.  В ASE без планов службы приложений перед созданием приложения будут использоваться 12 адресов.  Если это ILB ASE, то перед созданием приложения в этом ASE будет использоваться 13 адресов. При развертывании ССП роли инфраструктуры добавляются для каждых 15 и 20 экземпляров плана Службы приложений.

   > [!NOTE]
   > В подсети должна размещаться исключительно среда ASE. Обязательно выберите достаточно большое адресное пространство для будущего расширения. Впоследствии этот параметр уже нельзя изменить. Мы рекомендуем размер `/24` с 256 адресами.

При масштабировании добавляются новые роли соответствующего размера, а затем рабочие нагрузки перемещаются из текущего размера в целевой. Исходные виртуальные машины удаляются только после переноса рабочих нагрузок. Если у вас есть ASE с экземплярами ASP 100, то потребуется удвоить число виртуальных машин.  Именно по этой причине мы рекомендуем использовать "/24" чтобы вместить все изменения, которые потребуется.  

## <a name="ase-dependencies"></a>Зависимости ASE ##

### <a name="ase-inbound-dependencies"></a>Входящие зависимости ССП ###

Для работы ASE ASE необходимо открыть следующие порты:

| Использование | Исходный тип | Кому |
|-----|------|----|
| Управление | Адреса управления службой приложений | Подсеть ASE: 454, 455 |
|  Внутренний обмен данными ASE | Подсеть ASE: все порты | Подсеть ASE: все порты
|  Разрешение входящего трафика Azure Load Balancer | Azure Load Balancer | Подсеть ASE: 16001

Есть два других порта, которые могут отображаться как открытые при сканировании портов, 7654 и 1221. Они отвечают на IP-адрес и ничего больше. При необходимости их можно заблокировать. 

Помимо системы мониторинга, входящий административный трафик обеспечивает контроль и управление ССП. Исходные IP-адреса для этого трафика перечислены в документе [Адреса управления среды Службы приложений][ASEManagement]. Конфигурация безопасности сети должна разрешать доступ из адресов управления ASE на портах 454 и 455. Если доступ с этих адресов блокируется, ССП утратит работоспособность, а затем приостановит работу. На порты 454 и 455 TCP-трафик должен поступать с одного виртуального IP-адреса, иначе возникнет проблема с асимметричной маршрутизацией. 

В подсети ASE существует много портов, используемых для взаимодействия внутренних компонентов, и они могут изменяться. Для этого необходимо, чтобы все порты в подсети ASE были доступны из нее. 

Для взаимодействия между балансировщиком нагрузки Azure и подсетью ASE необходимо открыть как минимум порты 454, 455 и 16001. Порт 16001 используется для трафика проверки активности между балансировщиком нагрузки и ASE. Если вы используете ILB ASE, то можете заблокировать трафик до портов 454, 455, 16001.  При использовании внешнего ASE необходимо учитывать нормальные порты доступа к приложениям.  

Другие порты, с которыми необходимо уделить внимание, — это порты приложений:

| Использование | Порты |
|----------|-------------|
|  HTTP/HTTPS  | 80, 443 |
|  FTP/FTPS    | 21, 990, 10001-10020 |
|  Удаленная отладка в Visual Studio  |  4020, 4022, 4024 |
|  Служба веб-развертывание | 8172 |

Если заблокировать порты приложения, ASE по-прежнему сможет работать, но ваше приложение может не быть.  Если вы используете IP-адреса, назначенные приложению, с внешним ASE, вам нужно разрешить трафик с IP, назначенных вашим приложениям, в подсеть ASE на портах, показанных на странице > IP-адреса портала ASE.

### <a name="ase-outbound-dependencies"></a>Исходящие зависимости ССП ###

Что касается доступа исходящего трафика, среда ASE зависит от различных внешних систем. Эти зависимости системы определяются при помощи DNS-имен и не сопоставляются с фиксированным набором IP-адресов. Среде ASE требуется доступ исходящего трафика из подсети ASE для всех внешних IP-адресов через различные порты. 

ASE взаимодействует с доступными адресами в Интернете на следующих портах:

| Использование | Порты |
|-----|------|
| DNS | 53 |
| NTP | 123 |
| CRL, обновления Windows, зависимости Linux, службы Azure | 80/443 |
| Azure SQL | 1433 | 
| Мониторинг | 12000 |

Исходящие зависимости перечислены в документе, описывающем [блокировку среда службы приложений исходящем трафике](./firewall-integration.md). Если у ССП нет доступа к этим зависимостям, среда прекращает работу. Если это происходит достаточно долго, ASE переходит в состояние приостановки. 

### <a name="customer-dns"></a>DNS пользователя ###

Если в виртуальной сети настроен определенный пользователем DNS-сервер, то она используется для рабочих нагрузок клиента. ASE использует Azure DNS в целях управления. Если виртуальная сеть настроена с DNS-сервером, выбранным пользователем, то DNS-сервер должен быть доступен из подсети, содержащей ASE.

Чтобы проверить разрешение DNS из веб-приложения, можно использовать команду консоли *nameresolver*. Перейдите в окно отладки на сайте scm для приложения или перейдите к приложению на портале и выберите консоль. В командной строке оболочки можно выполнить команду *nameresolver* вместе с DNS-именем, которое вы хотите найти. Получаемый результат такой же, как и тот, что приложение получает во время такого же поиска. При использовании nslookup Поиск выполняется с помощью Azure DNS.

При изменении параметра DNS, в виртуальной сети которого находится ССП, необходимо перезагрузить ССП. Чтобы избежать перезагрузки ССП, настоятельно рекомендуется перед созданием среды ССП настроить параметры DNS для виртуальной сети.  

<a name="portaldep"></a>

## <a name="portal-dependencies"></a>Зависимости портала ##

Помимо функциональных зависимостей ASE есть несколько дополнительных элементов, относящихся к интерфейсу портала. Некоторые возможности на портале Azure зависят от прямого доступа к _сайту диспетчера служб_. Для каждого приложения в службе приложений Azure предусмотрено два URL-адреса. Первый URL-адрес предназначен для доступа к вашему приложению. Второй URL-адрес предназначен для доступа к сайту диспетчера служб, который также называется _Консоль Kudu_. Ниже перечислены компоненты, использующие сайт диспетчера служб.

-   Веб-задания
-   Функции
-   Потоковая передача журналов
-   Kudu
-   Расширения
-   Обозреватель процессов
-   Консоль

При использовании ILB ASE сайт SCM недоступен извне виртуальной сети. Некоторые возможности не будут работать на портале приложений, так как им требуется доступ к сайту SCM приложения. Вы можете подключиться к сайту SCM напрямую, не используя портал. 

Если ILB ASE — имя домена *contoso.appserviceenvironment.NET* , а имя приложения — *TestApp*, приложение достигается в *TestApp.contoso.appserviceenvironment.NET*. Сайт SCM, который перемещается с ним, находится по адресу *TestApp.SCM.contoso.appserviceenvironment.NET*.

## <a name="ase-ip-addresses"></a>IP-адреса ASE ##

В среде ASE есть несколько IP-адресов, на которые следует обратить внимание. Их можно сформулировать следующим образом.

- **Общедоступный IP-адрес для входящего трафика**. Используется для трафика приложений во внешней среде ASE и трафика управления в обоих средах (внешняя ASE и ASE с ILB).
- **Общедоступный IP-адрес для исходящего трафика**. Используется в качестве IP-адреса для исходящих подключений ASE из виртуальной сети, которым не назначен маршрут VPN.
- **IlB IP-адрес**. IP-адрес ilB существует только в ilB ASE.
- **Присвоенные приложению адреса с SSL на основе IP**. Могут применяться, только если используется внешняя ASE и настроен SSL на основе IP-адреса.

Все эти IP-адреса отображаются в портал Azure из пользовательского интерфейса ASE. Если вы используете ASE с ILB, то отображается IP-адрес для ILB.

   > [!NOTE]
   > Эти IP-адреса не изменятся до тех пор, пока ССП активна и работает.  Если ССП приостанавливает и восстанавливает работу, адреса, используемые ССП, изменятся. Обычное состояние, при котором ССП приостанавливает работу, — это "заблокирован административный доступ входящего трафика" или "запрещен доступ к зависимости ССП". 

![IP-адреса][3]

### <a name="app-assigned-ip-addresses"></a>Присвоенные приложению IP-адреса ###

Внешняя ASE позволяет назначать IP-адреса отдельным приложениям. В ASE с ILB такая возможность не предусмотрена. Дополнительные сведения о настройке собственного IP-адреса в приложении см. [в статье Защита настраиваемого DNS-имени с помощью привязки TLS/SSL в службе приложений Azure](../configure-ssl-bindings.md).

Если у приложения есть собственный адрес с SSL на основе IP, ASE резервирует два порта для сопоставления с этим IP-адресом. Один порт используется для трафика HTTP, а другой — для трафика HTTPS. Эти порты указаны в разделе IP-адресов пользовательского интерфейса ASE. Порты должны быть открыты для трафика, поступающего с виртуального IP-адреса, иначе приложения будут недоступны. Это требование важно помнить при настройке групп безопасности сети (NSG).

## <a name="network-security-groups"></a>группы сетевой безопасности; ##

[Группы безопасности сети][NSGs] дают возможность управлять сетевым доступом в виртуальной сети. При использовании портала действует неявное правило всеобщего запрета с самым низким приоритетом. Поэтому необходимо создать правила разрешения.

В среде ASE нет доступа к виртуальным машинам, используемым для ее размещения. Они входят в подписку, управляемую корпорацией Майкрософт. Чтобы ограничить доступ к приложениям в ASE, задайте в подсети ASE группы безопасности сети. При этом обратите особое внимание на зависимости ASE. Блокировка любой зависимости останавливает работу ASE.

Группы NSG можно настроить на портале Azure или с помощью PowerShell. Здесь описана настройка на портале Azure. Группы NSG создаются и управляются на портале в разделе **Сеть** как ресурс верхнего уровня.

Для работы ASE требуются записи в NSG, чтобы разрешить трафик:

**Входящий трафик**
* TCP из тега службы IP Аппсервицеманажемент на портах 454 455
* TCP от балансировщика нагрузки через порт 16001
* из подсети ASE в подсеть ASE на всех портах.

**Исходящий трафик**
* UDP ко всем IP-адресам через порт 53
* UDP ко всем IP-адресам через порт 123
* TCP ко всем IP-адресам на портах 80, 443
* TCP на тег службы IP `Sql` на портах 1433
* TCP ко всем IP-адресам через порт 12000
* в подсеть ASE на всех портах

Эти порты не включают порты, которые требуются вашим приложениям для успешного использования. Например, приложению может потребоваться вызвать сервер MySQL через порт 3306. Протокол NTP на порте 123 — это протокол синхронизации времени, используемый операционной системой. Конечные точки NTP не относятся к службам приложений, могут различаться в зависимости от операционной системы и не имеют четко определенного списка адресов. Чтобы предотвратить проблемы синхронизации времени, необходимо разрешить трафик UDP ко всем адресам на порту 123. Исходящий трафик TCP-порта 12000 предназначен для поддержки и анализа системы. Конечные точки являются динамическими и не имеют четко определенного набора адресов.

Стандартные порты доступа для приложения:

| Использование | Порты |
|----------|-------------|
|  HTTP/HTTPS  | 80, 443 |
|  FTP/FTPS    | 21, 990, 10001-10020 |
|  Удаленная отладка в Visual Studio  |  4020, 4022, 4024 |
|  Служба веб-развертывание | 8172 |

Когда учтены требования к входящему и исходящему трафику, группы безопасности сети должны быть оформлены, как показано в этом примере. 

![Правила безопасности для входящего трафика][4]

Правило по умолчанию активирует обмен данными между IP-адресами в виртуальной сети и подсетью ASE. Другое правило по умолчанию позволяет балансировщику нагрузки, который также называют общедоступным виртуальным IP-адресом, обмениваться данными с ASE. Чтобы просмотреть правила по умолчанию, щелкните **Правила по умолчанию** рядом со значком **Добавить**. Если вы поместили правило "запретить все остальное" перед правилами по умолчанию, трафик между виртуальным IP-адресом и ASE будет запрещен. Чтобы запретить трафик, поступающий из виртуальной сети, добавьте собственное правило для разрешения входящего трафика. Укажите для источника значение AzureLoadBalancer, для назначения — значение **Any** (Любое), а для диапазона портов — **\*\**. Поскольку правило группы безопасности сети применяется к подсети ASE, указывать конкретное назначение не требуется.

Если приложению присвоен IP-адрес, то убедитесь, что порты остаются открытыми. Чтобы просмотреть порты, выберите **Среда службы приложений**  >  **IP-адреса**.  

Требуются все элементы, показанные в следующих правилах для исходящего трафика, за исключением последнего элемента. Они обеспечивают доступ к сети для зависимостей ASE, которые упоминались ранее в этой статье. При блокировке любого из этих элементов ASE прекратит работу. Последний элемент в списке позволяет среде ASE обмениваться данными с другими ресурсами в виртуальной сети.

![Правила безопасности для исходящего трафика][5]

Определив группы безопасности сети, присвойте их подсети, в которой размещена ASE. Если вы не помните название виртуальной сети или подсети ASE, можно найти его на странице портала ASE. Чтобы присвоить подсети группу NSG, перейдите к пользовательскому интерфейсу подсети и выберите NSG.

## <a name="routes"></a>Маршруты ##

Принудительное туннелирование применяется при настройке маршрутов в виртуальной сети, чтобы исходящий трафик не направлялся не в Интернет, а куда-то еще, например в шлюз ExpressRoute или на виртуальное устройство.  Если вам нужно настроить ASE таким образом, прочитайте документ о [настройке среда службы приложений с принудительным туннелированием][forcedtunnel].  Этот документ поможет определить параметры, доступные для работы с ExpressRoute и применения принудительного туннелирования.

При создании ASE на портале мы также создадим набор таблиц маршрутов для подсети, создаваемой с использованием ASE.  По этим маршрутам исходящий трафик отправляется непосредственно в Интернет.  
Чтобы создать эти маршруты вручную, сделайте следующее:

1. Перейдите на портал Azure. Выберите **Сетевые**  >  **таблицы маршрутов**.

2. Создайте таблицу маршрутов в том же регионе, где находится виртуальная сеть.

3. В пользовательском интерфейсе таблицы маршрутов выберите **маршруты**  >  **добавить**.

4. Задайте для параметра **Тип следующего прыжка** значение **Интернет**, а для параметра **Префикс адреса** — значение **0.0.0.0/0**. Щелкните **Сохранить**.

    Отобразится примерно следующее:

    ![Функциональные маршруты][6]

5. Создав таблицу маршрутов, перейдите к подсети со средой ASE. Выберите свою таблицу маршрутов из списка на портале. После сохранения изменений отобразится список групп безопасности сети и маршрутов, указанных в подсети.

    ![Группы безопасности сети и маршруты][7]

## <a name="service-endpoints"></a>Конечные точки службы ##

Конечные точки службы позволяют ограничить доступ к мультитенантным службам для набора виртуальных сетей и подсетей. Дополнительные сведения о конечных точках служб для виртуальной сети см. в [этой статье][serviceendpoints]. 

При включении конечных точек службы в ресурс создаются маршруты с более высоким приоритетом. Если вы используете конечные точки службы в любой службе Azure с принудительным туннелированием ASE, трафик к этим службам не будет принудительно туннелным. 

Если конечные точки службы включены в подсеть с экземпляром Azure SQL, все экземпляры Azure SQL, подключенные к этой подсети, должны содержать конечные точки службы. Если вы хотите получить доступ к нескольким экземплярам Azure SQL в той же подсети, нельзя включать конечные точки службы только в один экземпляр Azure SQL. Никакая другая служба Azure не похожа на Azure SQL в отношении конечных точек службы. При включении конечных точек службы в хранилище Azure, доступ к этому ресурсу из подсети блокируется, но он по-прежнему доступен для других учетных записей хранилища Azure, даже если они не содержат конечных точек службы.  

![Конечные точки службы][8]

<!--Image references-->
[1]: ./media/network_considerations_with_an_app_service_environment/networkase-overflow.png
[2]: ./media/network_considerations_with_an_app_service_environment/networkase-overflow2.png
[3]: ./media/network_considerations_with_an_app_service_environment/networkase-ipaddresses.png
[4]: ./media/network_considerations_with_an_app_service_environment/networkase-inboundnsg.png
[5]: ./media/network_considerations_with_an_app_service_environment/networkase-outboundnsg.png
[6]: ./media/network_considerations_with_an_app_service_environment/networkase-udr.png
[7]: ./media/network_considerations_with_an_app_service_environment/networkase-subnet.png
[8]: ./media/network_considerations_with_an_app_service_environment/serviceendpoint.png

<!--Links-->
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/network-security-groups-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[mobileapps]: /previous-versions/azure/app-service-mobile/app-service-mobile-value-prop
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ss-cert.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[ASEManagement]: ./management-addresses.md
[serviceendpoints]: ../../virtual-network/virtual-network-service-endpoints-overview.md
[forcedtunnel]: ./forced-tunnel-support.md
[serviceendpoints]: ../../virtual-network/virtual-network-service-endpoints-overview.md