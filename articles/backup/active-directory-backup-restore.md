---
title: Резервное копирование и восстановление Active Directory
description: Узнайте, как выполнять резервное копирование и восстановление Active Directory контроллеров домена.
ms.topic: conceptual
ms.date: 07/08/2020
ms.openlocfilehash: 8db2dab605e90e4748b11a632d6651c23d631b6c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98733559"
---
# <a name="back-up-and-restore-active-directory-domain-controllers"></a>Резервное копирование и восстановление Active Directory контроллеров домена

Резервное копирование Active Directory и обеспечение успешного восстановления в случае повреждения, компрометация или аварии является важной частью Active Directory обслуживания.

В этой статье описаны правильные процедуры резервного копирования и восстановления Active Directory контроллеров домена с Azure Backup, будь то виртуальные машины Azure или локальные серверы. В нем обсуждается сценарий, в котором необходимо восстановить состояние всего контроллера домена во время резервного копирования. Чтобы узнать, какой сценарий восстановления подходит для вас, ознакомьтесь с [этой статьей](/windows-server/identity/ad-ds/manage/ad-forest-recovery-determine-how-to-recover).  

>[!NOTE]
> В этой статье не рассматриваются восстановление элементов из [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md). Сведения о восстановлении Azure Active Directory пользователей см. в [этой статье](../active-directory/fundamentals/active-directory-users-restore.md).

## <a name="best-practices"></a>Рекомендации

- Убедитесь, что создается резервная копия по крайней мере одного контроллера домена. При резервном копировании нескольких контроллеров домена убедитесь, что резервное копирование выполняется для всех тех, которые содержат [роли FSMO (гибкие операции с одной главной операцией)](/windows-server/identity/ad-ds/plan/planning-operations-master-role-placement) .

- Регулярно создавайте резервные копии Active Directory. Резервное копирование никогда не должно превышать время существования захоронения (по умолчанию 60 дней), поскольку объекты старше времени жизни захоронения будут "захоронены" и больше не считаются допустимыми.

- Наличие четкого плана аварийного восстановления, включающего инструкции по восстановлению контроллеров домена. Чтобы подготовиться к восстановлению Active Directory леса, ознакомьтесь с [руководством по восстановлению Active Directory леса](/windows-server/identity/ad-ds/manage/ad-forest-recovery-guide).

- Если необходимо восстановить контроллер домена и иметь оставшийся работоспособный контроллер домена в домене, можно создать новый сервер вместо восстановления из резервной копии. Добавьте роль сервера **служб домен Active Directory Services** на новый сервер, чтобы сделать его контроллером домена в существующем домене. Затем данные Active Directory будут реплицироваться на новый сервер. Чтобы удалить предыдущий контроллер домена из Active Directory, выполните действия, описанные [в этой статье](/windows-server/identity/ad-ds/deploy/ad-ds-metadata-cleanup) , чтобы выполнить очистку метаданных.

>[!NOTE]
>Azure Backup не включает восстановление на уровне элементов для Active Directory. Если вы хотите восстановить удаленные объекты и можете получить доступ к контроллеру домена, используйте [Active Directory корзину](/windows-server/identity/ad-ds/get-started/adac/introduction-to-active-directory-administrative-center-enhancements--level-100-#ad_recycle_bin_mgmt). Если этот метод недоступен, вы можете использовать резервную копию контроллера домена для восстановления удаленных объектов с помощью средства **ntdsutil.exe** , как описано [здесь](https://support.microsoft.com/help/840001/how-to-restore-deleted-user-accounts-and-their-group-memberships-in-ac).
>
>Сведения о выполнении полномочного восстановления SYSVOL см. в [этой статье](/windows-server/identity/ad-ds/manage/ad-forest-recovery-authoritative-recovery-sysvol).

## <a name="backing-up-azure-vm-domain-controllers"></a>Резервное копирование контроллеров домена виртуальных машин Azure

Если контроллер домена является виртуальной машиной Azure, можно выполнить резервное копирование сервера с помощью [резервного копирования виртуальных машин Azure](backup-azure-vms-introduction.md).

Ознакомьтесь с [вопросами эксплуатации виртуальных контроллеров домена](/windows-server/identity/ad-ds/get-started/virtual-dc/virtualized-domain-controllers-hyper-v#operational-considerations-for-virtualized-domain-controllers) , чтобы обеспечить успешную архивацию (и будущие операции восстановления) контроллеров домена виртуальной машины Azure.

## <a name="backing-up-on-premises-domain-controllers"></a>Резервное копирование локальных контроллеров домена

Чтобы создать резервную копию локального контроллера домена, необходимо выполнить резервное копирование данных о состоянии системы сервера.

- Если вы используете режим MARS, выполните следующие [инструкции](backup-azure-system-state.md).
- Если вы используете MABS (Azure Backup Server), выполните следующие [инструкции](backup-mabs-system-state-and-bmr.md).

>[!NOTE]
> Восстановление локальных контроллеров домена (из состояния системы или с виртуальных машин) в облако Azure не поддерживается. Если вы хотите выполнить отработку отказа из локальной среды Active Directory в Azure, рассмотрите возможность использования [Azure Site Recovery](../site-recovery/site-recovery-active-directory.md).

## <a name="restoring-active-directory"></a>Восстановление Active Directory

Active Directory данные можно восстановить в одном из двух режимов: **полномочное** или **неполномочное**. При полномочном восстановлении восстановленные Active Directory данные будут переопределять данные, находящиеся на других контроллерах домена в лесу.

Однако в этом сценарии выполняется перестроение контроллера домена в существующем домене, поэтому необходимо выполнить **неполномочное** восстановление.

Во время восстановления сервер будет запущен в режиме восстановления служб каталогов (DSRM). Необходимо указать пароль администратора для режима восстановления служб каталогов.

>[!NOTE]
>Если пароль DSRM забыт, его можно сбросить с помощью [этих инструкций](/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc754363(v=ws.11)).

### <a name="restoring-azure-vm-domain-controllers"></a>Восстановление контроллеров домена виртуальной машины Azure

Сведения о восстановлении контроллера домена виртуальной машины Azure см. в статье [восстановление виртуальных машин контроллера домена](backup-azure-arm-restore-vms.md#restore-domain-controller-vms).

Если вы восстанавливаете одну виртуальную машину контроллера домена или несколько виртуальных машин контроллера домена в одном домене, восстановите их как любую другую виртуальную машину. Кроме того, доступен режим восстановления служб каталогов, следовательно, все сценарии восстановления Active Directory также являются приемлемыми.

Если необходимо восстановить виртуальную машину одного контроллера домена в конфигурации с несколькими доменами, восстановите диски и создайте виртуальную машину [с помощью PowerShell](backup-azure-vms-automation.md#restore-the-disks).

Если вы восстанавливаете последний оставшийся контроллер домена в домене или восстанавливаете несколько доменов в одном лесу, рекомендуется [восстановление леса](/windows-server/identity/ad-ds/manage/ad-forest-recovery-single-domain-in-multidomain-recovery).

>[!NOTE]
> Виртуализированные контроллеры домена, начиная с Windows 2012, используют [меры безопасности на основе виртуализации](/windows-server/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100#virtualization-based-safeguards). С учетом этих мер служба Active Directory понимает, является ли восстановленная виртуальная машина контроллером домена, и выполняет необходимые действия для восстановления данных Active Directory.

### <a name="restoring-on-premises-domain-controllers"></a>Восстановление локальных контроллеров домена

Чтобы восстановить локальный контроллер домена, следуйте указаниям по восстановлению состояния системы в Windows Server, используя рекомендации по специальным рекомендациям [по восстановлению состояния системы на контроллере домена](backup-azure-restore-system-state.md#special-considerations-for-system-state-recovery-on-a-domain-controller).

## <a name="next-steps"></a>Дальнейшие действия

- [Таблица поддержки для Azure Backup](backup-support-matrix.md)