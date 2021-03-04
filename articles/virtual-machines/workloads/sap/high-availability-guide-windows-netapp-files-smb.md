---
title: ДОСТУПНОСТЬ виртуальных машин Azure для SAP NW в Windows с Azure NetApp Files (SMB) | Документация Майкрософт
description: Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure в Windows с Azure NetApp Files (SMB) для приложений SAP
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 02/18/2021
ms.author: radeltch
ms.openlocfilehash: a4c4631a0a1263e5a5398c44a8570f92571102e8
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102045842"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-windows-with-azure-netapp-filessmb-for-sap-applications"></a>Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure в Windows с Azure NetApp Files (SMB) для приложений SAP

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md

В этой статье описывается, как развернуть, настроить виртуальные машины, установить платформу кластера и установить высокодоступную систему SAP NetWeaver 7,50 на виртуальных машинах Windows с помощью [SMB](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) на [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).  

В этой статье нет подробных сведений об уровне базы данных. Предполагается, что [Виртуальная сеть](../../../virtual-network/virtual-networks-overview.md) Azure уже создана.  

Прежде всего прочитайте следующие примечания и документы SAP:

* [Документация по Azure NetApp Files][anf-azure-doc] 
* Примечание SAP [1928533][1928533], содержащее:  
  * Список размеров виртуальных машин Azure, которые поддерживаются при развертывании программного обеспечения SAP.
  * важные сведения о доступных ресурсах для каждого размера виртуальной машины Azure;
  * сведения о поддерживаемом программном обеспечении SAP и сочетаниях операционных систем и баз данных;
  * Обязательная версия ядра SAP для Windows на Microsoft Azure
* примечание к SAP [2015553][2015553], в котором описываются предварительные требования к SAP при развертывании программного обеспечения SAP в Azure;
* примечание к SAP [2178632][2178632], содержащее подробные сведения обо всех доступных метриках мониторинга для SAP в Azure;
* примечание к SAP [1999351][1999351], содержащее дополнительные сведения об устранении неполадок, связанных с расширением для расширенного мониторинга Azure для SAP;
* В примечании SAP [2287140](https://launchpad.support.sap.com/#/notes/2287140) перечислены предварительные требования для поддерживаемого SAP протокола SMB 3. x.
* В примечании SAP [2802770](https://launchpad.support.sap.com/#/notes/2802770) содержится информация об устранении проблем, возникающих при использовании AL11 транзакций SAP в Windows 2012 и 2016.
* В примечании SAP [1911507](https://launchpad.support.sap.com/#/notes/1911507) содержится информация о функции прозрачной отработки отказа для файлового ресурса в Windows Server с протоколом SMB 3,0.
* В примечании SAP [662452](https://launchpad.support.sap.com/#/notes/662452) содержится рекомендация (деактивация создания имени 8,3) для устранения проблем с производительностью файловой системы и ошибками при доступе к данным.
* [Установка в Azure высокодоступной системы SAP NetWeaver в отказоустойчивом кластере Windows с файловым ресурсом для экземпляров SAP ASCS/SCS](./sap-high-availability-installation-wsfc-file-share.md) 
* [Архитектура высокого уровня доступности и сценарии для SAP NetWeaver на виртуальных машинах Azure](./sap-high-availability-architecture-scenarios.md)
* [Добавление порта пробы в конфигурацию кластера ASCS](sap-high-availability-installation-wsfc-file-share.md)
* [Установка экземпляра (A) SCS в отказоустойчивом кластере](https://www.sap.com/documents/2017/07/f453332f-c97c-0010-82c7-eda71af511fa.html)
* [Создание тома SMB для Azure NetApp Files](../../../azure-netapp-files/create-active-directory-connections.md#requirements-for-active-directory-connections)
* [Приложения NetApp SAP в Microsoft Azure. Использование Azure NetApp Files][anf-sap-applications-azure]

> [!IMPORTANT]
> ВНИМАНИЕ! имейте в виду, что установка системы SAP с SWPM в общем ресурсе SMB, размещенной на [Azure NetApp Files][anf-azure-doc] томе SMB, может завершиться ошибкой установки для недостаточных разрешений, таких как "варнингперм не определен". Чтобы избежать этой ошибки, пользователь, от имени которого выполняется SWPM контекста, должен обладать повышенными привилегиями "Администратор домена" во время установки системы SAP.  

## <a name="overview"></a>Обзор

В SAP разработан новый подход и альтернатива общим дискам кластера для кластеризации экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows. Вместо использования общих дисков кластера один из них может использовать файловый ресурс SMB для развертывания файлов глобального узла SAP. Azure NetApp Files поддерживает SMBv3 (вместе с NFS) с ACL NTFS с помощью Active Directory. Azure NetApp Files автоматически становится высокодоступным (как и служба PaaS). Эти функции делают Azure NetApp Files отличным вариантом для размещения общего файлового ресурса SMB для глобального SAP.  
Поддерживаются [службы доменов Azure Active Directory (AD)](../../../active-directory-domain-services/overview.md) и [домен Active Directory Services (AD DS)](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) . С Azure NetApp Files можно использовать существующие Active Directory контроллеры домена. Контроллеры домена могут находиться в Azure как виртуальные машины или локально с помощью ExpressRoute или S2S VPN. В этой статье мы будем использовать контроллер домена на виртуальной машине Azure.  
Для обеспечения высокого уровня доступности (HA) центральных служб SAP NetWeaver требуется общее хранилище. Чтобы добиться этого в Windows, необходимо создать либо кластер SOFS, либо использовать общий диск кластера s/w, например SIOS. Теперь обеспечить высокий уровень доступности SAP NetWeaver можно с помощью общего хранилища, развернутого в службе Azure NetApp Files. Использование Azure NetApp Files для общего хранилища устраняет необходимость в SOFS или SIOS.  

> [!NOTE]
> Кластеризация экземпляров SAP ASCS/SCS с файловым ресурсом поддерживается для продуктов SAP NetWeaver 7.40 (и более поздней версии) с ядром SAP 7.49 (и более поздней версии).  

![Архитектура высокой доступности SAP ASCS/SCS с общим ресурсом SMB](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb.png)

Ниже перечислены необходимые условия для файлового ресурса SMB.
* Протокол SMB 3.0 (или более поздней версии).
* Возможность установки Active Directory списков управления доступом (ACL) для Active Directory групп пользователей и объекта Computer $ Computer.
* Общая папка должна быть доступна с высокой доступностью.

Общая папка для центральных служб SAP в этой эталонной архитектуре предлагается Azure NetApp Files:

![Архитектура высокой доступности SAP ASCS/SCS со сведениями общего ресурса SMB](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-detail.png)

## <a name="create-and-mount-smb-volume-for-azure-netapp-files"></a>Создание и подключение тома SMB для Azure NetApp Files

Выполните следующие действия, как подготовка к использованию Azure NetApp Files.  

1. Выполните действия, чтобы [зарегистрироваться для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md)  
2. Создайте учетную запись Azure NetApp, выполнив действия, описанные в статье [Создание учетной записи NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md) .  
3. Настройте пул ресурсов, следуя инструкциям в разделе [Настройка пула ресурсов](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md) .
4. Azure NetApp Files ресурсы должны находиться в делегированной подсети. Следуйте инструкциям в разделе [Делегирование подсети, чтобы Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md) создать делегированную подсеть.  

   > [!IMPORTANT]
   > Перед созданием тома SMB необходимо создать подключения Active Directory. Ознакомьтесь с [требованиями к Active Directoryным подключениям](../../../azure-netapp-files/create-active-directory-connections.md#requirements-for-active-directory-connections).  

5. Создайте подключение Active Directory, как описано в разделе [Создание подключения Active Directory](../../../azure-netapp-files/create-active-directory-connections.md#create-an-active-directory-connection) .  
6. Создайте SMB Azure NetApp Files том SMB, следуя инструкциям в разделе [Добавление тома SMB](../../../azure-netapp-files/azure-netapp-files-create-volumes-smb.md#add-an-smb-volume) .  
7. Подключите том SMB к виртуальной машине Windows.

> [!TIP]
> Инструкции по подключению тома Azure NetApp Files, если на [портале Azure](https://portal.azure.com/#home) вы переходите к объекту Azure NetApp Files, щелкните колонку **тома** , а затем — **инструкции по подключению**.  

## <a name="prepare-the-infrastructure-for-sap-ha-by-using-a-windows-failover-cluster"></a>Подготовка инфраструктуры для SAP высокого уровня доступности с помощью отказоустойчивого кластера Windows 

1. [Задайте правила балансировки нагрузки ASCS/SCS для внутреннего балансировщика нагрузки Azure](./sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716).
2. [Добавление виртуальных машин Windows в домен](./sap-high-availability-infrastructure-wsfc-shared-disk.md#e69e9a34-4601-47a3-a41c-d2e11c626c0c).
3. [Добавление записей реестра на обоих узлах кластера экземпляра SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-shared-disk.md#661035b2-4d0f-4d31-86f8-dc0a50d78158)
4. [Настройка отказоустойчивого кластера Windows Server для экземпляра SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-shared-disk.md#0d67f090-7928-43e0-8772-5ccbf8f59aab)
5. При использовании Windows Server 2016 рекомендуется настроить [Azure Cloud следящий](/windows-server/failover-clustering/deploy-cloud-witness).


## <a name="install-sap-ascs-instance-on-both-nodes"></a>Установка экземпляра SAP ASCS на обоих узлах

Вам потребуется следующее программное обеспечение от SAP:
   * Средство установки диспетчера подготовки программного обеспечения SAP (SWPM) версии SPS25 или более поздней.
   * SAP ядра 7,49 или более поздней версии
   * Создайте имя виртуального узла (сетевое имя кластера) для кластеризованного экземпляра SAP ASCS/SCS, как описано в разделе [Создание имени виртуального узла для кластеризованного экземпляра SAP ASCS/SCS](./sap-high-availability-installation-wsfc-shared-disk.md#a97ad604-9094-44fe-a364-f89cb39bf097).

> [!NOTE]
> Кластеризация экземпляров SAP ASCS/SCS с файловым ресурсом поддерживается для продуктов SAP NetWeaver 7.40 (и более поздней версии) с ядром SAP 7.49 (и более поздней версии).  

### <a name="install-an-ascsscs-instance-on-the-first-ascsscs-cluster-node"></a>Установка экземпляра ASCS/SCS на первом узле кластера ASCS/SCS

1. Установите экземпляр SAP ASCS/SCS на первом узле кластера. Запустите средство установки SAP SWPM, а затем перейдите к разделу: **продукт**  >  **СУБД** > установка > сервер приложений ABAP (или Java) > High-Availability System > ASCS/SCS экземпляр > первый узел кластера.  

2. Выберите **кластер общих** папок в качестве конфигурации общего ресурса кластера в SWPM.  
3. При появлении запроса на шаге **Параметры кластера системы SAP** введите имя узла для Azure NetApp Files общего ресурса SMB, который вы уже создали как **имя узла общего файлового ресурса**.  В этом примере имя узла общего ресурса SMB — **анфсмб-9562**. 

   > [!IMPORTANT]
   > Если средство проверки готовности к установке SWPM показывает, что условие функции непрерывной доступности не выполнено, его можно устранить, следуя инструкциям в разделе [отложенные сообщения об ошибках при попытке доступа к общей папке, которая больше не существует в Windows](https://support.microsoft.com/help/2820470/delayed-error-message-when-you-try-to-access-a-shared-folder-that-no-l).  

   > [!TIP]
   > Если средство проверки готовности к установке SWPM отображает условие размера подкачки не выполнено, можно изменить размер подкачки, перейдя к Мой компьютер>свойства системы>параметры производительности> расширенный> виртуальной памяти> изменения.  

4. Настройте ресурс кластера SAP, `SAP-SID-IP` порт пробы, с помощью PowerShell. Выполните эту настройку на одном из узлов кластера SAP ASCS/SCS, как описано в разделе [Настройка порта пробы](./sap-high-availability-installation-wsfc-shared-disk.md#10822f4f-32e7-4871-b63a-9b86c76ce761).

### <a name="install-an-ascsscs-instance-on-the-second-ascsscs-cluster-node"></a>Установка экземпляра ASCS/SCS на второй узел кластера ASCS/SCS

1. Установите экземпляр SAP ASCS/SCS на втором узле кластера. Запустите средство установки SAP SWPM, а затем перейдите к **продукту**  >  **СУБД** > установка > сервер приложений ABAP (или Java) > High-Availability System > ASCS/SCS, > дополнительный узел кластера.  

### <a name="install-a-dbms-instance-and-sap-application-servers"></a>Установка экземпляра СУБД и серверов приложений SAP

Завершите установку SAP, установив:

   * Экземпляр СУБД  
   * Основной сервер приложений SAP;  
   * Дополнительный сервер приложений SAP  

## <a name="test-the-sap-ascsscs-instance-failover"></a>Тестирование отработки отказа экземпляра SAP ASCS/SCS 

### <a name="fail-over-from-cluster-node-a-to-cluster-node-b-and-back"></a>Отработка отказа с узла A на узел B кластера б и обратно
В этом сценарии тестирования мы будем называть узел кластера sapascs1 как узел A, а кластер sapascs2 node — узлом B.

1. Убедитесь, что ресурсы кластера работают на узле A. ![ Рис. 1. ресурсы отказоустойчивого кластера Windows Server, работающие на узле A до выполнения теста отработки отказа](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-1.png)  

2. Перезапустите узел A кластера. Ресурсы кластера SAP будут перемещены на узел B кластера. ![ Рис. 2. ресурсы отказоустойчивого кластера Windows Server, работающие на узле B после теста отработки отказа](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-2.png)  


## <a name="lock-entry-test"></a>Проверка записи блокировки

1. Убедитесь, что сервер репликации очереди SAP (ERS) активен.  
2. Войдите в систему SAP, выполните транзакцию SU01 и откройте идентификатор пользователя в режиме изменения. Это приведет к созданию записи блокировки SAP.  
3. После входа в систему SAP Отобразите запись о блокировке, перейдя к транзакции ST12.  
4. Отработка отказа ресурсов ASCS с узла A на узел B кластера.  
5. Убедитесь, что запись блокировки, созданная перед отработкой отказа ресурсов кластера SAP ASCS/SCS, сохраняется.  

![Рис. 3. запись блокировки сохраняется после теста отработки отказа](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-3.png)  

Дополнительные сведения см. [в разделе Устранение неполадок при отработке отказа в очереди в ASCS с ERS](https://wiki.scn.sap.com/wiki/display/SI/Troubleshooting+for+Enqueue+Failover+in+ASCS+with+ERS)
## <a name="next-steps"></a>Дальнейшие действия

* [Планирование и реализация виртуальных машин Azure для SAP][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* Узнайте, как установить высокий уровень доступности и спланировать аварийное восстановление SAP. 
* HANA в Azure (крупные экземпляры) см. [в разделе SAP HANA (крупные экземпляры) высокий уровень доступности и аварийное восстановление в Azure](hana-overview-high-availability-disaster-recovery.md).
* Дополнительные сведения об установке высокого уровня доступности и плана для аварийного восстановления SAP HANA на виртуальных машинах Azure см. в статье [Высокий уровень доступности SAP HANA на виртуальных машинах Azure][sap-hana-ha].
