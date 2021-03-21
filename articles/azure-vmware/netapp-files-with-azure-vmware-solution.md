---
title: Azure NetApp Files с помощью решения VMware для Azure
description: Используйте Azure NetApp Files с виртуальными машинами Azure VMware для переноса и синхронизации данных между локальными серверами, виртуальными машинами Azure VMware и облачными инфраструктурами.
ms.topic: how-to
ms.date: 02/10/2021
ms.openlocfilehash: 2f2e8fdeb777e7e4b2b4e89c1bb36b51c3083257
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100575428"
---
# <a name="azure-netapp-files-with-azure-vmware-solution"></a>Azure NetApp Files с помощью решения VMware для Azure

В этой статье мы рассмотрим шаги интеграции Azure NetApp Files с рабочими нагрузками на основе решений VMware для Azure. Гостевая операционная система будет работать на виртуальных машинах, обращающихся к Azure NetApp Filesным томам. 

## <a name="azure-netapp-files-overview"></a>Обзор Azure NetApp Files

[Azure NetApp Files](../azure-netapp-files/azure-netapp-files-introduction.md) — это служба Azure для миграции и запуска самых требовательных корпоративных рабочих нагрузок файлов в облаке: баз данных, SAP и высокопроизводительных вычислительных приложений без изменения кода.

### <a name="features"></a>Компоненты
(Службы, где используются Azure NetApp Files.)

- **Active Directory подключения**: Azure NetApp Files поддерживает [службы домен Active Directory и Azure Active Directory доменных служб](../azure-netapp-files/create-active-directory-connections.md#decide-which-domain-services-to-use).

- **Общий протокол**: Azure NetApp Files поддерживает протоколы SMB и Network File System (NFS). Такая поддержка означает, что тома могут быть подключены к клиенту Linux и могут быть сопоставлены в клиенте Windows.

- **Решение VMware для Azure**. Azure NetApp Files общие ресурсы можно подключить к виртуальным машинам, созданным в среде решения Azure VMware.

Azure NetApp Files доступен во многих регионах Azure и поддерживает репликацию между регионами. Дополнительные сведения о методах конфигурации Azure NetApp Files см. в разделе [иерархия хранилища Azure NetApp Files](../azure-netapp-files/azure-netapp-files-understand-storage-hierarchy.md).

## <a name="reference-architecture"></a>Эталонная архитектура

На следующей схеме показано подключение через Azure ExpressRoute к частному облаку решения Azure VMware. Среда решения Azure VMware обращается к Azure NetApp Files общему ресурсу, подключенному к виртуальным машинам Azure VMware.

![Схема, на которой показаны файлы NetApp для архитектуры решения VMware для Azure.](media/net-app-files/net-app-files-topology.png)

В этой статье рассматриваются инструкции по настройке, тестированию и проверке Azure NetApp Filesного тома в качестве файлового ресурса для виртуальных машин Azure VMware. В этом сценарии мы использовали протокол NFS. Azure NetApp Files и решение Azure VMware создаются в одном регионе Azure.

## <a name="prerequisites"></a>Предварительные условия 

> [!div class="checklist"]
> * Подписка Azure с включенным Azure NetApp Files
> * Подсеть для Azure NetApp Files
> * Виртуальная машина Linux в решении VMware для Azure
> * Виртуальные машины Windows в решении VMware для Azure

## <a name="regions-supported"></a>Поддерживаемые регионы

Список поддерживаемых регионов можно найти в списке [продуктов Azure по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=netapp,azure-vmware&regions=all).

## <a name="verify-pre-configured-azure-netapp-files"></a>Проверка предварительно настроенных Azure NetApp Files 

Следуйте пошаговым инструкциям в следующих статьях, чтобы создать и подключить Azure NetApp Files тома на виртуальных машинах Azure VMware.

- [Создание учетной записи NetApp](../azure-netapp-files/azure-netapp-files-create-netapp-account.md)
- [Настройка пула емкости](../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md)
- [Создание тома SMB для Azure NetApp Files](../azure-netapp-files/azure-netapp-files-create-volumes-smb.md)
- [Создание тома NFS для Azure NetApp Files](../azure-netapp-files/azure-netapp-files-create-volumes.md)
- [Делегирование подсети службе Azure NetApp Files](../azure-netapp-files/azure-netapp-files-delegate-subnet.md)

Следующие шаги включают проверку предварительно настроенных Azure NetApp Files, созданных в Azure на уровне службы Azure NetApp Files Premium.

1. В портал Azure в разделе **хранилище** выберите **Azure NetApp Files**. Отобразится список настроенных Azure NetApp Files. 

    :::image type="content" source="media/net-app-files/azure-net-app-files-list.png" alt-text="Снимок экрана, показывающий список предварительно настроенных Azure NetApp Files."::: 

2. Выберите настроенную учетную запись NetApp Files, чтобы просмотреть ее параметры. Например, выберите **contoso-anf2**. 

3. Выберите **Пулы ресурсов** , чтобы проверить настроенный пул. 

    :::image type="content" source="media/net-app-files/net-app-settings.png" alt-text="Снимок экрана с параметрами для просмотра пулов емкости и томов настроенной учетной записи NetApp Files.":::

    Откроется страница пулы ресурсов, на которой отображается уровень емкости и обслуживание. В этом примере пул носителей настроен как 4 Тиб с уровнем обслуживания Premium.

4. Выберите **тома** , чтобы просмотреть тома, созданные в пуле ресурсов. (См. предыдущий снимок экрана).

5. Выберите том, чтобы просмотреть его конфигурацию.  

    :::image type="content" source="media/net-app-files/azure-net-app-volumes.png" alt-text="Снимок экрана, показывающий тома, созданные в пуле ресурсов.":::

    Откроется окно, в котором отображаются сведения о конфигурации тома.

    :::image type="content" source="media/net-app-files/configuration-of-volume.png" alt-text="Снимок экрана, показывающий сведения о конфигурации тома.":::

    Вы видите, что анфволуме имеет размер 200 гиб и находится в пуле емкости anfpool1. Он экспортируется как файловый ресурс NFS через 10.22.3.4:/АНФВОЛУМЕ. Один частный IP-адрес из виртуальной сети Azure (VNet) был создан для Azure NetApp Files и пути NFS для подключения к виртуальной машине.

    Дополнительные сведения о Azure NetApp Files производительности объемом по размеру или квоте см. в разделе [рекомендации по производительности для Azure NetApp Files](../azure-netapp-files/azure-netapp-files-performance-considerations.md). 

## <a name="verify-pre-configured-azure-vmware-solution-vm-share-mapping"></a>Проверка предварительно настроенного сопоставления общего ресурса виртуальной машины Azure VMware

Чтобы сделать ресурс Azure NetApp Files доступным для виртуальной машины Azure VMware, необходимо ознакомиться с сопоставлением общего ресурса SMB и NFS. Вы можете подключить их только после настройки томов SMB или NFS, как описано здесь.

- Общий ресурс SMB. Создайте подключение Active Directory перед развертыванием тома SMB. Для успешного подключения указанные контроллеры домена должны быть доступны для делегированной подсети Azure NetApp Files. После настройки Active Directory в учетной записи Azure NetApp Files она будет отображаться как выбираемый элемент при создании томов SMB.

- Общая папка NFS. Azure NetApp Files участвует в создании томов с помощью NFS или двойного протокола (NFS и SMB). Потребление емкости тома зависит от подготовленной емкости пула. NFS можно подключить к серверу Linux с помощью командных строк или записей/etc/fstab.

## <a name="use-cases-of-azure-netapp-files-with-azure-vmware-solution"></a>Варианты использования Azure NetApp Files в решении Azure VMware

Ниже приведены лишь несколько интересных Azure NetApp Files вариантов использования. 
- Управление профилями горизонта
- Управление профилями Citrix
- Управление профилем службы удаленных рабочих столов
- Общие файловые ресурсы в решении VMware для Azure

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как интегрировать Azure NetApp Files с рабочими нагрузками решения Azure VMware, вам может потребоваться изучить следующие темы:

- [Ограничения ресурсов для Azure NetApp Files](../azure-netapp-files/azure-netapp-files-resource-limits.md#resource-limits).
- [Рекомендации по Azure NetApp Filesному планированию сети](../azure-netapp-files/azure-netapp-files-network-topologies.md).
- [Репликация Azure NetApp Files томов между регионами](../azure-netapp-files/cross-region-replication-introduction.md). 
- [Часто задаваемые вопросы о Azure NetApp Files](../azure-netapp-files/azure-netapp-files-faqs.md).
