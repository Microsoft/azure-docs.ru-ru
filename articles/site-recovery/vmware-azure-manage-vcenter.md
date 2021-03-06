---
title: Управление серверами VMware vCenter Server в Azure Site Recovery
description: В этой статье описывается, как добавить VMware vCenter и управлять им для аварийного восстановления виртуальных машин VMware в Azure с помощью Azure Site Recovery.
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: conceptual
ms.date: 12/24/2019
ms.author: ramamill
ms.openlocfilehash: 01aef3aca4f6967b1681bff9598c7dd7a24739cd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84692527"
---
# <a name="manage-vmware-vcenter-server"></a>Управление VMware vCenter Server

В этой статье перечислены действия по управлению VMware vCenter Server в [Azure Site Recovery](site-recovery-overview.md).

## <a name="verify-prerequisites-for-vcenter-server"></a>Проверка предварительных требований для vCenter Server

Необходимые условия для серверов vCenter и виртуальных машин во время аварийного восстановления виртуальных машин VMware в Azure перечислены в [матрице поддержки](vmware-physical-azure-support-matrix.md#replicated-machines).

## <a name="set-up-an-account-for-automatic-discovery"></a>Настройка учетной записи для автоматического обнаружения

При настройке аварийного восстановления для локальных виртуальных машин VMware Site Recovery требуется доступ к узлу vCenter Server или vSphere. Site Recovery сервер обработки может автоматически обнаруживать виртуальные машины и завершать их при необходимости. По умолчанию сервер обработки запускается на Site Recovery сервере конфигурации. Добавьте учетную запись для сервера конфигурации, чтобы подключиться к узлу vCenter Server или vSphere следующим образом:

1. Войдите на сервер конфигурации.
1. Откройте средство сервера конфигурации (_cspsconfigtool.exe_) с помощью ярлыка на рабочем столе.
1. На вкладке **Управление учетной записью** щелкните **Добавить учетную запись**.

   ![add-account](./media/vmware-azure-manage-vcenter/addaccount.png)

1. Укажите сведения об учетной записи и нажмите кнопку **ОК**, чтобы добавить учетную запись. Учетная запись должна иметь привилегии, сводные в таблице разрешений учетной записи.

   > [!NOTE]
   > Синхронизация данных учетной записи с Site Recovery занимает около 15 минут.

### <a name="account-permissions"></a>Разрешения учетной записи

|**Задача** | **Учетная запись** | **Разрешения** | **Сведения**|
|--- | --- | --- | ---|
|**Обнаружение и миграция ВМ (без восстановления размещения)** | Как минимум учетная запись пользователя, доступная только для чтения. | Объект центра обработки данных –> Распространение на дочерний объект, роль Read-only | Пользователь назначается на уровне центра обработки данных и поэтому имеет доступ ко всем объектам в центре обработки данных.<br/><br/> Если нужно ограничить доступ, назначьте дочерним объектам (узлы vSphere, хранилища данных, виртуальные машины и сети) роль **Без доступа** с параметром **Propagate to child** (Распространить на дочерний объект).|
|**Репликация и отработка отказа** | Как минимум учетная запись пользователя, доступная только для чтения. | Объект центра обработки данных –> Распространение на дочерний объект, роль Read-only | Пользователь назначается на уровне центра обработки данных и поэтому имеет доступ ко всем объектам в центре обработки данных.<br/><br/> Чтобы ограничить доступ, назначьте дочерним объектам (узлы vSphere, хранилища данных, виртуальные машины и сети) роль **без доступа** с параметром **распространить на дочерний** объект.<br/><br/> Это полезно для сценария переноса, но не подходит для полной репликации, отработки отказа и восстановления размещения.|
|**Репликация, отработка отказа и восстановление размещения** | Мы рекомендуем создать роль (AzureSiteRecoveryRole) с необходимыми разрешениями, а затем назначить роль пользователю или группе VMware. | Объект центра данных –> распространение на дочерний объект, роль AzureSiteRecoveryRole<br/><br/> Хранилище данных -> выделение места, обзор хранилища данных, низкоуровневые операции с файлами, удаление файлов, обновление файлов виртуальной машины<br/><br/> Сеть -> Назначение сети<br/><br/> Ресурс -> назначение виртуальной машины в пул ресурсов, миграция отключенной виртуальной машины, миграция включенной виртуальной машины<br/><br/> Задачи -> Создание задач, Обновление задач<br/><br/> Виртуальная машина -> конфигурация<br/><br/> Виртуальная машина -> взаимодействие -> ответ на вопрос, подключение устройства, настройка компакт-диска носителя, настройка дискеты носителя, отключение, включение, установка средств VMware<br/><br/> Виртуальная машина -> инвентаризация -> создание, регистрация, отмена регистрации<br/><br/> Виртуальная машина -> подготовка -> разрешение загрузки виртуальной машины, разрешение отправки файлов виртуальной машины<br/><br/> виртуальная машина -> моментальные снимки -> удаление моментальных снимков. | Пользователь назначается на уровне центра обработки данных и поэтому имеет доступ ко всем объектам в центре обработки данных.<br/><br/> Если нужно ограничить доступ, назначьте дочерним объектам (узлы vSphere, хранилища данных, виртуальные машины и сети) роль **Без доступа** с параметром **Propagate to child** (Распространить на дочерний объект).|

## <a name="add-vmware-server-to-the-vault"></a>Добавление сервера VMware в хранилище

При настройке аварийного восстановления для локальных виртуальных машин VMware добавьте узел vCenter Server или vSphere, на котором выполняется обнаружение виртуальных машин в хранилище Site Recovery, следующим образом.

1. В > хранилища серверы конфигурации **инфраструктуры Site Recovery**  >  откройте сервер конфигурации.
1. На странице **сведения** щелкните **vCenter**.
1. В окне **Добавление vCenter** укажите понятное имя узла vSphere или сервера vCenter.
1. Укажите IP-адрес или полное доменное имя сервера.
1. Если серверы VMware настроены на прослушивание запросов через другой порт, оставьте порт 443.
1. Выберите учетную запись, используемую для подключения к серверу VMware vCenter или vSphere ESXi. Нажмите кнопку **ОК**.

## <a name="modify-credentials"></a>Изменение учетных данных

При необходимости можно изменить учетные данные, используемые для подключения к узлу vCenter Server или vSphere, следующим образом:

1. Войдите на сервер конфигурации.
1. Откройте средство сервера конфигурации (_cspsconfigtool.exe_) с помощью ярлыка на рабочем столе.
1. На вкладке **Управление учетными записями** щелкните **Добавить учетную запись**.

   ![add-account](./media/vmware-azure-manage-vcenter/addaccount.png)

1. Укажите сведения о новой учетной записи и нажмите кнопку **ОК**. Учетной записи требуются разрешения, перечисленные в таблице [разрешений учетной записи](#account-permissions) .
1. В > хранилища серверы конфигурации **инфраструктуры Site Recovery**  >  откройте сервер конфигурации.
1. В окне **сведения** нажмите кнопку **обновить сервер**.
1. После завершения задания обновления сервера выберите vCenter Server.
1. В поле **Сводка** выберите только что добавленную учетную запись в **узле VCenter Server/vSphere** и нажмите кнопку **сохранить**.

   ![modify-account](./media/vmware-azure-manage-vcenter/modify-vcente-creds.png)

## <a name="delete-a-vcenter-server"></a>Удаление vCenter Server

1. В > хранилища серверы конфигурации **инфраструктуры Site Recovery**  >  откройте сервер конфигурации.
1. На странице **сведений** выберите сервер vCenter.
1. Нажмите кнопку **Удалить** .

   ![delete-account](./media/vmware-azure-manage-vcenter/delete-vcenter.png)

## <a name="modify-the-ip-address-and-port"></a>Изменение IP-адреса и порта

Можно изменить IP-адрес vCenter Server или порты, используемые для обмена данными между сервером и Site Recovery. По умолчанию Site Recovery обращается к сведениям об узле vCenter Server/vSphere через порт 443.

1. В > хранилище   >  **серверы конфигурации** инфраструктуры Site Recovery щелкните сервер конфигураций, в который добавляется vCenter Server.
1. В **vCenter Servers** щелкните vCenter Server, который необходимо изменить.
1. В поле **Сводка** обновите IP-адрес и порт, а затем сохраните изменения.

   ![add_ip_new_vcenter](media/vmware-azure-manage-vcenter/add-ip.png)

1. Чтобы изменения стали эффективными, подождите 15 минут или [Обновите сервер конфигурации](vmware-azure-manage-configuration-server.md#refresh-configuration-server).

## <a name="migrate-all-vms-to-a-new-server"></a>Перенос всех виртуальных машин на новый сервер

Если вы хотите перенести все виртуальные машины для использования новой vCenter Server, необходимо просто обновить IP-адрес, назначенный vCenter Server. Не добавляйте еще одну учетную запись VMware, так как это может привести к дублированию записей. Обновите адрес следующим образом:

1. В > хранилище   >  **серверы конфигурации** инфраструктуры Site Recovery щелкните сервер конфигураций, в который добавляется vCenter Server.
1. В разделе **серверы vCenter** щелкните vCenter Server, с которого требуется выполнить миграцию.
1. В поле **Сводка** измените IP-адрес на новый vCenter Server и сохраните изменения.
1. Как только IP-адрес будет обновлен, Site Recovery начнет получать сведения об обнаружении виртуальной машины из нового vCenter Server. Это не влияет на текущие действия репликации.

## <a name="migrate-a-few-vms-to-a-new-server"></a>Перенос нескольких виртуальных машин на новый сервер

Если вы хотите перенести только некоторые из реплицируемых виртуальных машин в новый vCenter Server, выполните следующие действия.

1. [Добавьте](#add-vmware-server-to-the-vault) новый vCenter Server на сервер конфигурации.
1. [Отключите репликацию](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) для виртуальных машин, которые будут перемещены на новый сервер.
1. В VMware выполните миграцию виртуальных машин в новую vCenter Server.
1. Снова [включите репликацию](vmware-azure-tutorial.md#enable-replication) для перенесенных виртуальных машин, выбрав новую vCenter Server.

## <a name="migrate-most-vms-to-a-new-server"></a>Перенос большинства виртуальных машин на новый сервер

Если число виртуальных машин, которые требуется перенести на новый vCenter Server, превышает количество виртуальных машин, которые останутся на исходном vCenter Server, выполните следующие действия.

1. [Обновите IP-адрес](#modify-the-ip-address-and-port) , назначенный vCenter Server в параметрах сервера конфигурации, по адресу нового vCenter Server.
1. [Отключите репликацию](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) для нескольких виртуальных машин, оставшихся на старом сервере.
1. [Добавьте старый vCenter Server](#add-vmware-server-to-the-vault) и его IP-адрес на сервер конфигурации.
1. [Повторно включите репликацию](vmware-azure-tutorial.md#enable-replication) для виртуальных машин, оставшихся на старом сервере.

## <a name="next-steps"></a>Дальнейшие действия

Если у вас возникнут проблемы, см. раздел [Устранение неполадок vCenter Server обнаружения](vmware-azure-troubleshoot-vcenter-discovery-failures.md).
