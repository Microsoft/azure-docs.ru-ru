---
title: Часто задаваемые вопросы о Azure NetApp Files | Документация Майкрософт
description: Ознакомьтесь с часто задаваемыми вопросами о Azure NetApp Files, например о сети, безопасности, производительности, управлении емкостью и переносе и защите данных.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/09/2021
ms.author: b-juche
ms.openlocfilehash: 6d9d56a7f6d1e265508081f735e2dbc379f195fb
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102552037"
---
# <a name="faqs-about-azure-netapp-files"></a>Часто задаваемые вопросы о Azure NetApp Files

В этой статье содержатся ответы на часто задаваемые вопросы о Azure NetApp Files. 

## <a name="networking-faqs"></a>Вопросы и ответы о сети

### <a name="does-the-nfs-data-path-go-over-the-internet"></a>Выполняет ли путь к данным NFS через Интернет?  

Нет. Путь к данным NFS не переходит через Интернет. Azure NetApp Files — это собственная служба Azure, которая развертывается в виртуальной сети Azure (VNet), где доступна эта служба. Azure NetApp Files использует делегированную подсеть и подготавливает сетевой интерфейс непосредственно в виртуальной сети. 

Дополнительные сведения см. [в разделе рекомендации по Azure NetApp Files планировании сети](./azure-netapp-files-network-topologies.md) .  

### <a name="can-i-connect-a-vnet-that-i-already-created-to-the-azure-netapp-files-service"></a>Можно ли подключить уже созданную виртуальную сеть к службе Azure NetApp Files?

Да, можно подключить виртуальных сетей, созданный вами к службе. 

Дополнительные сведения см. [в разделе рекомендации по Azure NetApp Files планировании сети](./azure-netapp-files-network-topologies.md) .  

### <a name="can-i-mount-an-nfs-volume-of-azure-netapp-files-using-dns-fqdn-name"></a>Можно ли подключить том NFS Azure NetApp Files, используя полное доменное имя DNS?

Да, можно, если создать необходимые записи DNS. Azure NetApp Files предоставляет IP-адрес службы для подготовленного тома. 

> [!NOTE] 
> При необходимости Azure NetApp Files может развернуть дополнительные IP-адреса для службы.  Может потребоваться периодически обновлять записи DNS.

### <a name="can-i-set-or-select-my-own-ip-address-for-an-azure-netapp-files-volume"></a>Можно ли задать или выбрать собственный IP-адрес для Azure NetApp Filesного тома?  

Нет. Назначение IP-адресов для Azure NetApp Files томов является динамическим. Назначение статических IP-адресов не поддерживается. 

### <a name="does-azure-netapp-files-support-dual-stack-ipv4-and-ipv6-vnet"></a>Поддерживает ли Azure NetApp Files виртуальную сеть с двумя стеками (IPv4 и IPv6)?

Нет, Azure NetApp Files в настоящее время не поддерживает виртуальную сеть с двумя стеками (IPv4 и IPv6).  
 
## <a name="security-faqs"></a>Часто задаваемые вопросы о безопасности

### <a name="can-the-network-traffic-between-the-azure-vm-and-the-storage-be-encrypted"></a>Может ли шифроваться сетевой трафик между виртуальной машиной Azure и хранилищем?

Трафик данных между клиентами Нфсв 4.1 и Azure NetApp Filesными томами можно шифровать с помощью протокола Kerberos с шифрованием AES-256. Дополнительные сведения см. в разделе [Настройка шифрования Kerberos нфсв 4.1 для Azure NetApp Files](configure-kerberos-encryption.md) .   

Трафик между клиентами NFSv3 или SMB3 и Azure NetApp Files томами не шифруется. Тем не менее трафик от виртуальной машины Azure (с клиентом NFS или SMB) на Azure NetApp Files является безопасным, как и любой другой трафик Azure-ВМ-VM. Этот трафик является локальным для сети центра обработки данных Azure. 

### <a name="can-the-storage-be-encrypted-at-rest"></a>Может ли хранилище быть зашифровано неактивных данных?

Все Azure NetApp Filesные тома шифруются с использованием стандарта FIPS 140-2. Все ключи управляются службой Azure NetApp Files. 

### <a name="how-are-encryption-keys-managed"></a>Как осуществляется управление ключами шифрования? 

Управление ключами для Azure NetApp Files обрабатывается службой. Для каждого тома создается уникальный ключ шифрования данных XTS-AES-256. Иерархия ключей шифрования используется для шифрования и защиты всех ключей томов. Эти ключи шифрования никогда не отображаются или не передаются в виде незашифрованного формата. Ключи шифрования немедленно удаляются при удалении тома.

Поддержка управляемых клиентом ключей (создание собственных ключей) с помощью выделенного HSM-модуля Azure доступна на управляемом уровне в восточной части США, юго-центральном регионе США, Западная часть США 2 и в регионах US Gov (Вирджиния). Доступ можно запросить по адресу [anffeedback@microsoft.com](mailto:anffeedback@microsoft.com) . После того как емкость станет доступной, запросы будут утверждены.

### <a name="can-i-configure-the-nfs-export-policy-rules-to-control-access-to-the-azure-netapp-files-service-mount-target"></a>Можно ли настроить правила политики экспорта NFS для контроля доступа к целевому объекту подключения службы Azure NetApp Files?

Да, можно настроить до пяти правил в одной политике экспорта NFS.

### <a name="does-azure-netapp-files-support-network-security-groups"></a>Поддерживает ли Azure NetApp Files группы безопасности сети?

Нет, в настоящее время нельзя применять группы безопасности сети к делегированной подсети Azure NetApp Files или к сетевым интерфейсам, созданным службой.

### <a name="can-i-use-azure-rbac-with-azure-netapp-files"></a>Можно ли использовать Azure RBAC с Azure NetApp Files?

Да, Azure NetApp Files поддерживает функции Azure RBAC.

## <a name="performance-faqs"></a>Вопросы и ответы по производительности

### <a name="what-should-i-do-to-optimize-or-tune-azure-netapp-files-performance"></a>Что делать, чтобы оптимизировать или настроить производительность Azure NetApp Files?

В соответствии с требованиями к производительности можно выполнять следующие действия. 
- Убедитесь, что размер виртуальной машины соответствует требованиям.
- Включите ускоренную сеть для виртуальной машины.
- Выберите требуемый уровень обслуживания и размер для пула емкости.
- Создайте том с требуемым размером квоты для емкости и производительности.

### <a name="how-do-i-convert-throughput-based-service-levels-of-azure-netapp-files-to-iops"></a>Разделы справки преобразовывать уровни обслуживания на основе пропускной способности Azure NetApp Files в операции ввода-вывода?

Можно преобразовать МБ/с в операции ввода-вывода с помощью следующей формулы:  

`IOPS = (MBps Throughput / KB per IO) * 1024`

### <a name="how-do-i-change-the-service-level-of-a-volume"></a>Разделы справки изменить уровень обслуживания тома?

Уровень обслуживания существующего тома можно изменить, переместив том в другой пул мощностей, который использует требуемый [уровень обслуживания](azure-netapp-files-service-levels.md) для тома. См. раздел [Динамическое изменение уровня обслуживания тома](dynamic-change-volume-service-level.md). 

### <a name="how-do-i-monitor-azure-netapp-files-performance"></a>Разделы справки монитор Azure NetApp Files производительность?

Azure NetApp Files предоставляет метрики производительности тома. Можно также использовать Azure Monitor для мониторинга метрик использования для Azure NetApp Files.  Список метрик производительности для Azure NetApp Files см. в разделе [метрики для Azure NetApp Files](azure-netapp-files-metrics.md) .

### <a name="whats-the-performance-impact-of-kerberos-on-nfsv41"></a>Каковы последствия производительности Kerberos на Нфсв 4.1?

Сведения о параметрах безопасности для Нфсв 4.1, тестировании векторов производительности и ожидаемом влиянии на производительность см. в статье [влияние Kerberos на тома нфсв 4.1](performance-impact-kerberos.md) . 

## <a name="nfs-faqs"></a>Вопросы и ответы по NFS

### <a name="i-want-to-have-a-volume-mounted-automatically-when-an-azure-vm-is-started-or-rebooted--how-do-i-configure-my-host-for-persistent-nfs-volumes"></a>Я хочу автоматически подключить том при запуске или перезагрузке виртуальной машины Azure.  Разделы справки настроить узел для постоянных томов NFS?

Чтобы том NFS автоматически подключаться при запуске или перезагрузке виртуальной машины, добавьте запись в `/etc/fstab` файл на узле. 

Дополнительные сведения см. [в статье подключение или отключение тома для виртуальных машин Windows или Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md) .  

### <a name="why-does-the-df-command-on-nfs-client-not-show-the-provisioned-volume-size"></a>Почему команда DF на клиенте NFS не показывает подготовленный размер тома?

Размер тома, сообщаемый в DF, — это максимальный размер, который может увеличить Azure NetApp Files том. Размер Azure NetApp Filesого тома в команде DF не является отражающим квоту или размер тома.  Вы можете получить Azure NetApp Files размер или квоту тома с помощью портал Azure или API.

### <a name="what-nfs-version-does-azure-netapp-files-support"></a>Какая версия NFS поддерживает Azure NetApp Files?

Azure NetApp Files поддерживает NFSv3 и Нфсв 4.1. Вы можете [создать том](azure-netapp-files-create-volumes.md) с помощью версии NFS. 

### <a name="how-do-i-enable-root-squashing"></a>Разделы справки включить корневую использование параметра Squash?

Можно указать, может ли учетная запись root иметь доступ к тому или нет, используя политику экспорта тома. Дополнительные сведения см. в статье [Настройка политики экспорта для тома NFS](azure-netapp-files-configure-export-policy.md) .

### <a name="can-i-use-the-same-file-path-volume-creation-token-for-multiple-volumes"></a>Можно ли использовать один и тот же путь к файлу (маркер создания тома) для нескольких томов?

Да, можно. Однако путь к файлу должен использоваться либо в другой подписке, либо в другом регионе.   

Например, вы создаете том с именем `vol1` . Затем вы создаете другой том, который также вызывается `vol1` в другом пуле емкости, но в той же подписке и регионе. В этом случае использование одного и того же имени тома `vol1` приведет к ошибке. Чтобы использовать один и тот же путь к файлу, имя должно находиться в другом регионе или подписке.

### <a name="when-i-try-to-access-nfs-volumes-through-a-windows-client-why-does-the-client-take-a-long-time-to-search-folders-and-subfolders"></a>Почему при попытке доступа к томам NFS через клиент Windows для поиска папок и вложенных папок клиент занимает много времени?

Убедитесь, что `CaseSensitiveLookup` в клиенте Windows включен параметр для ускорения поиска папок и вложенных папок:

1. Чтобы включить Касесенситивелукуп, используйте следующую команду PowerShell:   
    `Set-NfsClientConfiguration -CaseSensitiveLookup 1`    
2. Подключите том к серверу Windows Server.   
    Пример   
    `Mount -o rsize=1024 -o wsize=1024 -o mtype=hard \\10.x.x.x\testvol X:*`

### <a name="how-does-azure-netapp-files-support-nfsv41-file-locking"></a>Как Azure NetApp Files поддерживает блокировку файлов Нфсв 4.1? 

Для клиентов Нфсв 4.1 Azure NetApp Files поддерживает механизм блокировки файлов Нфсв 4.1, сохраняющий состояние всех блокировок файлов в модели на основе аренды. 

В соответствии с RFC 3530 Azure NetApp Files определяет один период аренды для всех состояний, удерживаемых клиентом NFS. Если клиент не продлит свою аренду в течение заданного периода, все состояния, связанные с арендой клиента, будут освобождены сервером.  

Например, если клиент, который подключает том, перестает отвечать на запросы или выходит за пределы времени ожидания, блокировки будут освобождены. Клиент может явно или неявно обновить аренду, выполнив такие операции, как чтение файла.   

Льготный период определяет период особой обработки, в которой клиенты могут попытаться восстановить состояние блокировки во время восстановления сервера. Время ожидания по умолчанию для аренды составляет 30 секунд с льготным периодом в 45 секунд. По истечении этого времени Аренда клиента будет освобождена.   

## <a name="smb-faqs"></a>Часто задаваемые вопросы о SMB

### <a name="which-smb-versions-are-supported-by-azure-netapp-files"></a>Какие версии SMB поддерживаются Azure NetApp Files?

Azure NetApp Files поддерживает SMB 2,1 и SMB 3,1 (включая поддержку SMB 3,0).    

### <a name="is-an-active-directory-connection-required-for-smb-access"></a>Требуется Active Directoryое подключение для доступа по протоколу SMB? 

Да, перед развертыванием тома SMB необходимо создать подключение Active Directory. Для успешного подключения указанные контроллеры домена должны быть доступны для делегированной подсети Azure NetApp Files.  Дополнительные сведения см. [в разделе Создание тома SMB](./azure-netapp-files-create-volumes-smb.md) . 

### <a name="how-many-active-directory-connections-are-supported"></a>Сколько Active Directory подключений поддерживается?

Azure NetApp Files не поддерживает множественные подключения Active Directory (AD) в одном *регионе*, даже если подключения к Active Directory находятся в разных учетных записях NetApp. Однако в одной *подписке* можно использовать несколько подключений AD, если они находятся в разных регионах. Если требуется несколько подключений AD в одном регионе, для этого можно использовать отдельные подписки. 

Подключение AD настраивается для каждой учетной записи NetApp; подключение Active Directory доступно только через учетную запись NetApp, в которой он создан.

### <a name="does-azure-netapp-files-support-azure-active-directory"></a>Поддерживает ли Azure NetApp Files Azure Active Directory? 

Поддерживаются [службы доменов Azure Active Directory (AD)](../active-directory-domain-services/overview.md) и [домен Active Directory Services (AD DS)](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) . С Azure NetApp Files можно использовать существующие Active Directory контроллеры домена. Контроллеры домена могут размещаться в Azure как виртуальные машины или локально с помощью ExpressRoute или S2S VPN. В настоящее время Azure NetApp Files не поддерживает присоединение к AD для [Azure Active Directory](https://azure.microsoft.com/resources/videos/azure-active-directory-overview/) .

При использовании Azure NetApp Files с доменными службами Azure Active Directory путь подразделения при настройке Active Directory для учетной записи NetApp будет `OU=AADDC Computers`.

### <a name="what-versions-of-windows-server-active-directory-are-supported"></a>Какие версии Windows Server Active Directory поддерживаются?

Azure NetApp Files поддерживает версии домен Active Directory служб Windows Server 2008r2SP1-2019.

### <a name="why-does-the-available-space-on-my-smb-client-not-show-the-provisioned-size"></a>Почему доступное пространство на моем клиенте SMB не показывает подготовленный размер?

Размер тома, сообщаемый SMB-клиентом, — это максимальный размер, который может увеличить Azure NetApp Files том. Размер тома Azure NetApp Files, как показано на SMB-клиенте, не отражен в размере квоты или размера тома. Вы можете получить Azure NetApp Files размер или квоту тома с помощью портал Azure или API.

### <a name="im-having-issues-connecting-to-my-smb-share-what-should-i-do"></a>Возникли проблемы при подключении к общему ресурсу SMB.   Что следует делать?

Рекомендуется установить максимальную погрешность синхронизации компьютерных часов до пяти минут. Дополнительные сведения см. в разделе [Максимальная погрешность синхронизации часов компьютера](/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj852172(v=ws.11)). 

<!--
### Does Azure NetApp Files support LDAP signing? 

Yes, Azure NetApp Files supports LDAP signing by default. This functionality enables secure LDAP lookups between the Azure NetApp Files service and the user-specified [Active Directory Domain Services domain controllers](/windows/win32/ad/active-directory-domain-services). For more information, see [ADV190023 | Microsoft Guidance for Enabling LDAP Channel Binding and LDAP Signing](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190023).
--> 

## <a name="capacity-management-faqs"></a>Вопросы и ответы по управлению емкостью

### <a name="how-do-i-monitor-usage-for-capacity-pool-and-volume-of-azure-netapp-files"></a>Разделы справки отслеживать использование пула емкости и объема Azure NetApp Files? 

Azure NetApp Files предоставляет метрики емкости пула и использования томов. Можно также использовать Azure Monitor для мониторинга использования Azure NetApp Files. Дополнительные сведения см. в разделе [метрики для Azure NetApp Files](azure-netapp-files-metrics.md) . 

### <a name="can-i-manage-azure-netapp-files-through-azure-storage-explorer"></a>Можно ли управлять Azure NetApp Files с помощью Обозреватель службы хранилища Azure?

Нет. Azure NetApp Files не поддерживается Обозреватель службы хранилища Azure.

### <a name="how-do-i-determine-if-a-directory-is-approaching-the-limit-size"></a>Разделы справки определить, приближается ли размер каталога к размеру?

Можно использовать `stat` команду клиента, чтобы определить, приближается ли каталог к максимальному размеру метаданных каталога (320 МБ).

Для каталога 320 МБ количество блоков равно 655360, а размер каждого блока составляет 512 байт.  (Т. е. 320x1024x1024/512.)  

Примеры:

```console
[makam@cycrh6rtp07 ~]$ stat bin
File: 'bin'
Size: 4096            Blocks: 8          IO Block: 65536  directory

[makam@cycrh6rtp07 ~]$ stat tmp
File: 'tmp'
Size: 12288           Blocks: 24         IO Block: 65536  directory
 
[makam@cycrh6rtp07 ~]$ stat tmp1
File: 'tmp1'
Size: 4096            Blocks: 8          IO Block: 65536  directory
```


## <a name="data-migration-and-protection-faqs"></a>Часто задаваемые вопросы по переносу и защите данных

### <a name="how-do-i-migrate-data-to-azure-netapp-files"></a>Разделы справки перенести данные в Azure NetApp Files?
Azure NetApp Files предоставляет тома NFS и SMB.  Для переноса данных в службу можно использовать любое средство копирования на основе файлов. 

NetApp предлагает решение на основе SaaS, [NetApp облачную синхронизацию](https://cloud.netapp.com/cloud-sync-service).  Решение позволяет реплицировать данные NFS или SMB в Azure NetApp Files экспорта NFS или общих ресурсах SMB. 

Можно также использовать широкий набор бесплатных средств для копирования данных. Для NFS можно использовать средства рабочих нагрузок, такие как [rsync](https://rsync.samba.org/examples.html) , для копирования и синхронизации исходных данных в том Azure NetApp Files. Для SMB можно использовать рабочие нагрузки с помощью средства [Robocopy](/windows-server/administration/windows-commands/robocopy) аналогичным образом.  Эти средства также могут выполнять репликацию разрешений для файлов и папок. 

Ниже приведены требования к переносу данных из локальной среды в Azure NetApp Files. 

- Убедитесь, что Azure NetApp Files доступен в целевом регионе Azure.
- Проверьте сетевое подключение между исходным и Azure NetApp Files целевым IP-адресом тома. Обмен данными между локальным и Azure NetApp Files службой поддерживается через ExpressRoute.
- Создайте целевой Azure NetApp Files том.
- Перенесите исходные данные на целевой том с помощью предпочтительного средства копирования файлов.

### <a name="how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region"></a>Разделы справки создать копию тома Azure NetApp Files в другом регионе Azure?
    
Azure NetApp Files предоставляет тома NFS и SMB.  Любое средство копирования на основе файлов можно использовать для репликации данных между регионами Azure. 

NetApp предлагает решение на основе SaaS, [NetApp облачную синхронизацию](https://cloud.netapp.com/cloud-sync-service).  Решение позволяет реплицировать данные NFS или SMB в Azure NetApp Files экспорта NFS или общих ресурсах SMB. 

Можно также использовать широкий набор бесплатных средств для копирования данных. Для NFS можно использовать средства рабочих нагрузок, такие как [rsync](https://rsync.samba.org/examples.html) , для копирования и синхронизации исходных данных в том Azure NetApp Files. Для SMB можно использовать рабочие нагрузки с помощью средства [Robocopy](/windows-server/administration/windows-commands/robocopy) аналогичным образом.  Эти средства также могут выполнять репликацию разрешений для файлов и папок. 

Ниже приведены требования к репликации Azure NetApp Files тома в другой регион Azure. 
- Убедитесь, что Azure NetApp Files доступен в целевом регионе Azure.
- Проверьте сетевое подключение между виртуальных сетей в каждом регионе. В настоящее время глобальный пиринг между виртуальных сетей не поддерживается.  Вы можете установить подключение между виртуальных сетей, связав его с каналом ExpressRoute или с помощью VPN-подключения S2S. 
- Создайте целевой Azure NetApp Files том.
- Перенесите исходные данные на целевой том с помощью предпочтительного средства копирования файлов.

### <a name="is-migration-with-azure-data-box-supported"></a>Поддерживается ли миграция с Azure Data Box?

Нет. Azure Data Box не поддерживает Azure NetApp Files в данный момент. 

### <a name="is-migration-with-azure-importexport-service-supported"></a>Поддерживается ли миграция с поддержкой службы импорта и экспорта Azure?

Нет. Служба импорта и экспорта Azure не поддерживает Azure NetApp Files в настоящее время.

## <a name="product-faqs"></a>Вопросы и ответы по продуктам

### <a name="can-i-use-azure-netapp-files-nfs-or-smb-volumes-with-azure-vmware-solution-avs"></a>Можно ли использовать Azure NetApp Files NFS или томов SMB с решением VMware для Azure (AVS)?

Тома Azure NetApp Files NFS можно подключить на виртуальных машинах Windows или виртуальных машинах Linux. Вы можете сопоставлять Azure NetApp Files общие ресурсы SMB на виртуальных машинах Windows AVS. Дополнительные сведения см. [в статье Azure NetApp Files с помощью решения VMware для Azure]( ../azure-vmware/netapp-files-with-azure-vmware-solution.md).  

### <a name="what-regions-are-supported-for-using-azure-netapp-files-nfs-or-smb-volumes-with-azure-vmware-solution-avs"></a>Какие регионы поддерживаются при использовании Azure NetApp Files NFS или томов SMB с решением VMware для Azure (AVS)?

Использование Azure NetApp Files NFS или томов SMB с AVS поддерживается в следующих регионах: Восточная часть США, Западная часть США, Западная Европа и Восточная Австралия.

## <a name="next-steps"></a>Следующие шаги  

- [Microsoft Azure ExpressRoute часто задаваемые вопросы](../expressroute/expressroute-faqs.md)
- [Вопросы и ответы по виртуальная сеть Microsoft Azure](../virtual-network/virtual-networks-faq.md)
- [Создание запроса на поддержку Azure](../azure-portal/supportability/how-to-create-azure-support-request.md)
- [Azure Data Box](../databox/index.yml)
- [Часто задаваемые вопросы о производительности SMB для Azure NetApp Files](azure-netapp-files-smb-performance.md)
