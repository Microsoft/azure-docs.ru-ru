---
title: Резервное копирование виртуальных машин VMware с помощью Azure Backup Server
description: Из этой статьи вы узнаете, как использовать Azure Backup Server для резервного копирования виртуальных машин VMware, работающих на сервере VMware vCenter или ESXi.
ms.topic: conceptual
ms.date: 05/24/2020
ms.openlocfilehash: db5e5c4bdac64e2faf5babb107ecec61a02d6468
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96002959"
---
# <a name="back-up-vmware-vms-with-azure-backup-server"></a>Резервное копирование виртуальных машин VMware с помощью Azure Backup Server

В этой статье объясняется, как выполнять резервное копирование виртуальных машин VMware, запущенных на VMware ESXi узлах или vCenter Server в Azure с помощью Azure Backup Server (MABS).

В статье описывается выполнение следующих задач:

- настроить безопасный канал так, чтобы Azure Backup Server мог обмениваться данными с серверами VMware по протоколу HTTPS;
- настроить учетную запись VMware, которую использует Azure Backup Server для доступа к серверу VMware;
- добавить учетные данные учетной записи в Azure Backup;
- добавить сервер vCenter или ESXi в Azure Backup Server;
- настроить группу защиты, которая содержит необходимые для резервного копирования виртуальные машины VMware, указать параметры резервного копирования и планировать его выполнение.

## <a name="supported-vmware-features"></a>Поддерживаемые функции VMware

MABS предоставляет следующие возможности при резервном копировании виртуальных машин VMware:

- Резервное копирование без агента: MABS не требует установки агента на сервере vCenter или ESXi, чтобы создать резервную копию виртуальной машины. Вместо этого просто укажите IP-адрес или полное доменное имя (FQDN) и учетные данные входа, используемые для проверки подлинности сервера VMware с помощью MABS.
- Облачная интегрированная архивация: MABS защищает рабочие нагрузки на диск и облако. Рабочий процесс резервного копирования и восстановления MABS помогает управлять долгосрочным хранением и автономным резервным копированием.
- Обнаружение и защита виртуальных машин, управляемых с помощью vCenter: MABS обнаруживает и защищает виртуальные машины, развернутые на сервере VMware (vCenter или ESXi Server). При увеличении размера развертывания используйте vCenter для управления средой VMware. MABS также обнаруживает виртуальные машины под управлением vCenter, что позволяет защищать крупные развертывания.
- Автоматическая защита на уровне папок. vCenter позволяет организовать виртуальные машины в папки виртуальных машин. MABS обнаруживает эти папки и позволяет защищать виртуальные машины на уровне папок и включает все вложенные папки. При защите папок MABS не только защищает виртуальные машины в этой папке, но также защищает виртуальные машины, добавленные позже. MABS обнаруживает новые виртуальные машины на ежедневной основе и защищает их автоматически. При Организации виртуальных машин в рекурсивных папках MABS автоматически обнаруживает и защищает новые виртуальные машины, развернутые в рекурсивных папках.
- MABS защищает виртуальные машины, хранящиеся на локальном диске, в сетевой файловой системе (NFS) или в хранилище кластера.
- MABS защищает виртуальные машины, перенесенные для балансировки нагрузки. так как виртуальные машины переносятся для балансировки нагрузки, MABS автоматически обнаруживает и возобновляет защиту виртуальных машин.
- MABS может восстанавливать файлы и папки с виртуальной машины Windows без восстановления всей виртуальной машины, что позволяет быстрее восстанавливать необходимые файлы.

## <a name="prerequisites-and-limitations"></a>Предварительные условия и ограничения

Прежде чем применять резервное копирование виртуальных машин VMware, изучите следующий список ограничений и необходимых компонентов.

- Если вы использовали MABS для защиты сервера vCenter Server (работающего в Windows) в качестве Windows Server, использующего полное доменное имя сервера, этот сервер vCenter нельзя защитить как сервер VMware, используя полное доменное имя сервера.
  - В качестве обходного пути можно использовать статический IP-адрес vCenter Server.
  - Если вы хотите использовать полное доменное имя, необходимо отключить защиту в качестве сервера Windows, удалить агент защиты, а затем добавить его в качестве сервера VMware с помощью полного доменного имени.
- Если для управления серверами ESXi в среде используется vCenter, добавьте vCenter (и не ESXi) в группу защиты MABS.
- Вы не можете создавать резервные копии моментальных снимков пользователей перед первой MABS резервной копией. После завершения первой резервной копии MABS можно создать резервные копии пользовательских моментальных снимков.
- MABS не может защитить виртуальные машины VMware с транзитными дисками и сопоставлениями физических необработанных устройств (дисками).
- MABS не может обнаружить или защитить VMware vApp.
- MABS не может защищать виртуальные машины VMware с помощью существующих моментальных снимков.

## <a name="before-you-start"></a>Перед началом работы

- Убедитесь, что вы используете версию vCenter/ESXi, которая поддерживается для резервного копирования. См. таблицу поддержки [здесь](./backup-mabs-protection-matrix.md).
- Убедитесь, что Azure Backup Server настроено. Если это еще не сделано, перед началом изучите [это](backup-azure-microsoft-azure-backup.md). Azure Backup Server следует запускать с последними обновлениями.
- Убедитесь, что открыты следующие сетевые порты:
  - TCP 443 между MABS и vCenter;
  - TCP 443 и TCP 902 между MABS и узлом ESXi.

## <a name="create-a-secure-connection-to-the-vcenter-server"></a>Создание безопасного подключения к серверу vCenter

По умолчанию Azure Backup Server взаимодействует с серверами VMware по протоколу HTTPS. Чтобы настроить соединение HTTPS, скачайте сертификат Центра сертификации (ЦС) VMware и импортируйте его в Azure Backup Server.

### <a name="before-you-begin"></a>Перед началом

- Если вы не хотите использовать протокол HTTPS, можно [отключить проверку сертификата HTTPS для всех серверов VMware](backup-azure-backup-server-vmware.md#disable-https-certificate-validation).
- Обычно к серверу vCenter или ESXi можно подключиться из браузера на компьютере Azure Backup Server через веб-клиент vSphere. В первый раз подключение не будет защищено и отобразится следующее.
- Важно понимать, как Azure Backup Server обрабатывает резервные копии.
  - Первым шагом будет создание Azure Backup Server резервной копии данных в локальном хранилище дисков. Azure Backup Server использует пул хранилища, набор дисков и томов, на которых Azure Backup Server хранит точки восстановления диска для защищенных данных. Пул хранения может быть хранилищем, подключенным напрямую (DAS), оптоволоконным каналом сети SAN или устройством или сетью хранения данных iSCSI. Важно убедиться в наличии достаточного объема хранилища для локального резервного копирования данных виртуальных машин VMware.
  - Затем Azure Backup Server создает резервные копии в Azure из локального хранилища дисков.
  - Чтобы выяснить, какой объем хранилища необходим, см. [этот раздел](/system-center/dpm/create-dpm-protection-groups#figure-out-how-much-storage-space-you-need). Информация предназначена для DPM, но ее также можно использовать для сервера Azure Backup Server.

### <a name="set-up-the-certificate"></a>Настройка сертификата

Настройте безопасный канал следующим образом.

1. В браузере на сервере Azure Backup Server введите URL-адрес веб-клиента vSphere. Если страница входа не отображается, проверьте параметры подключения и прокси-сервера из браузера.

    ![Веб-клиент vSphere](./media/backup-azure-backup-server-vmware/vsphere-web-client.png)

2. На странице входа веб-клиента vSphere выберите **Загрузить Сертификаты доверенного корневого ЦС**.

    ![Скачивание сертификата доверенного корневого ЦС.](./media/backup-azure-backup-server-vmware/vmware-download-ca-cert-prompt.png)

3. Загружается файл с именем **download**. В зависимости от браузера появится сообщение с предложением открыть или сохранить файл.

    ![Скачивание сертификата ЦС](./media/backup-azure-backup-server-vmware/download-certs.png)

4. Сохраните файл на компьютере Azure Backup Server с расширением ZIP.

5. Щелкните правой кнопкой мыши **download.zip**  >  **извлечь все**. Содержимое сжатого ZIP-файла будет извлечено в папку **certs**, которая вмещает следующие элементы.
   - Файл корневого сертификата с расширением, которое начинается с нумерованной последовательности, например, .0 и .1.
   - Файл списка отзыва сертификатов имеет расширение, которое начинается с такой последовательности, как .r0 или .r1. Файл списка отзыва сертификатов связан с сертификатом.

    ![Загруженные сертификаты](./media/backup-azure-backup-server-vmware/extracted-files-in-certs-folder.png)

6. В папке **certs** щелкните правой кнопкой мыши файл корневого сертификата и выберите **Переименовать**.

    ![Переименование корневого сертификата](./media/backup-azure-backup-server-vmware/rename-cert.png)

7. Измените расширение файла корневого сертификата на CRT и подтвердите. Значок файла изменится на значок корневого сертификата.

8. Щелкните правой кнопкой мыши этот корневой сертификат и во всплывающем меню выберите **Установить сертификат**.

9. В **мастере импорта сертификатов** выберите **локальный компьютер** в качестве места назначения для сертификата, а затем нажмите кнопку **Далее**. Если отобразится запрос разрешения на внесение изменений на компьютер, разрешите внести изменения.

    ![Страница приветствия мастера](./media/backup-azure-backup-server-vmware/certificate-import-wizard1.png)

10. На странице **хранилище сертификатов** выберите **поместить все сертификаты в следующее хранилище**, а затем нажмите кнопку **Обзор** , чтобы выбрать хранилище сертификатов.

    ![Хранилище сертификатов](./media/backup-azure-backup-server-vmware/cert-import-wizard-local-store.png)

11. В области **Выбор хранилища сертификатов** выберите **Доверенные корневые центры сертификации** в качестве папки назначения для сертификатов, а затем нажмите кнопку **ОК**.

    ![Папка назначения для сертификата](./media/backup-azure-backup-server-vmware/certificate-store-selected.png)

12. На **странице Завершение работы мастера импорта сертификатов** проверьте папку и нажмите кнопку **Готово**.

    ![Проверка наличия сертификата в нужной папке](./media/backup-azure-backup-server-vmware/cert-wizard-final-screen.png)

13. После подтверждения импорта сертификата войдите на vCenter Server, чтобы подтвердить, что подключение защищено.

### <a name="disable-https-certificate-validation"></a>Отключить проверку сертификата HTTPS

Если в вашей организации есть безопасные границы и вы не хотите использовать протокол HTTPS между серверами VMware и Azure Backup Server компьютером, отключите HTTPS следующим образом:

1. Скопируйте этот текст и вставьте его в простой текстовый файл.

    ```text
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager\VMWare]
    "IgnoreCertificateValidation"=dword:00000001
    ```

2. На компьютере Azure Backup Server сохраните файл с именем **DisableSecureAuthentication.reg**.

3. Дважды щелкните этот файл, чтобы активировать параметр реестра.

## <a name="create-a-vmware-role"></a>Создание роли VMware

Для Azure Backup Server необходима учетная запись пользователя с разрешениями для доступа к серверу v-Center Server или узлу ESXi. Создайте роль VMware с определенными привилегиями, а затем свяжите учетную запись пользователя с ролью.

1. Войдите на сервер vCenter Server (или, если вы не используете vCenter Server, на узел ESXi).
2. На панели **Навигатор** выберите **Администрирование**.

    ![Администрирование](./media/backup-azure-backup-server-vmware/vmware-navigator-panel.png)

3. В области **административные**  >  **роли** щелкните значок Добавить роль (символ +).

    ![Добавление роли](./media/backup-azure-backup-server-vmware/vmware-define-new-role.png)

4. В окне **Создание**  >  **имени роли** роли введите *BackupAdminRole*. Это может быть любое имя, описывающее назначение этой роли.

5. Выберите привилегии, как описано в таблице ниже, а затем нажмите кнопку **ОК**.  В списке на панели **Роли** появится новая роль.
   - Щелкните значок рядом с родительской меткой, чтобы развернуть родительский элемент и просмотреть дочерние привилегии.
   - Чтобы выбрать привилегии виртуальной машины, необходимо использовать несколько уровней иерархии родителей-потомков.
   - Вам не нужно выбирать все дочерние привилегии, входящие в родительскую.

    ![Иерархия привилегий родителей-потомков](./media/backup-azure-backup-server-vmware/cert-add-privilege-expand.png)

### <a name="role-permissions"></a>Разрешения ролей

В следующей таблице перечислены привилегии, которые нужно назначить создаваемой учетной записи пользователя:

| Привилегии для учетной записи пользователя vCenter 6.5                          | Привилегии для учетной записи пользователя vCenter 6.7                            |
|----------------------------------------------------------------------------|----------------------------------------------------------------------------|
| Кластер хранилища данных. Настройка кластера хранилища данных                           | Кластер хранилища данных. Настройка кластера хранилища данных                           |
| Datastore.AllocateSpace                                                    | Datastore.AllocateSpace                                                    |
| Datastore.Browse datastore                                                 | Datastore.Browse datastore                                                 |
| Datastore.Low-level file operations                                        | Datastore.Low-level file operations                                        |
| Global.Disable methods                                                     | Global.Disable methods                                                     |
| Global.Enable methods                                                      | Global.Enable methods                                                      |
| Global.Licenses                                                            | Global.Licenses                                                            |
| Global.Log event                                                           | Global.Log event                                                           |
| Global.Manage custom attributes                                            | Global.Manage custom attributes                                            |
| Global.Set custom attribute                                                | Global.Set custom attribute                                                |
| Host.Local operations.Create virtual machine                               | Host.Local operations.Create virtual machine                               |
| Network.Assign network                                                     | Network.Assign network                                                     |
| Resource. Assign virtual machine to resource pool                          | Resource. Assign virtual machine to resource pool                          |
| vApp.Add virtual machine                                                   | vApp.Add virtual machine                                                   |
| vApp.Assign resource pool                                                  | vApp.Assign resource pool                                                  |
| vApp.Unregister                                                            | vApp.Unregister                                                            |
| VirtualMachine.Configuration. Добавление или удаление устройства                         | VirtualMachine.Configuration. Добавление или удаление устройства                         |
| Virtual machine.Configuration.Disk lease                                   | Virtual machine.Configuration.Acquire disk lease                           |
| Virtual machine.Configuration.Add new disk                                 | Virtual machine.Configuration.Add new disk                                 |
| Virtual machine.Configuration.Advanced                                     | Virtual machine.Configuration.Advanced configuration                       |
| Virtual machine.Configuration.Disk change tracking                         | Virtual machine.Configuration.Toggle disk change tracking                  |
| Virtual machine.Configuration.Host USB device                              | Virtual machine.Configuration.Configure Host USB device                    |
| Virtual machine.Configuration.Extend virtual disk                          | Virtual machine.Configuration.Extend virtual disk                          |
| Virtual machine.Configuration.Query unowned files                          | Virtual machine.Configuration.Query unowned files                          |
| Virtual machine.Configuration.Swapfile placement                           | Virtual machine.Configuration.Change Swapfile placement                    |
| Virtual machine.Guest Operations.Guest Operation Program Execution         | Virtual machine.Guest Operations.Guest Operation Program Execution         |
| Virtual machine.Guest Operations.Guest Operation Modifications             | Virtual machine.Guest Operations.Guest Operation Modifications             |
| Virtual machine.Guest Operations.Guest Operation Queries                   | Virtual machine.Guest Operations.Guest Operation Queries                   |
| Virtual machine.Interaction.Device connection                            | Virtual machine.Interaction.Device connection                            |
| Virtual machine.Interaction.Guest operating system management by VIX API | Virtual machine.Interaction.Guest operating system management by VIX API |
| Virtual machine.Interaction.Power Off                                    | Virtual machine.Interaction.Power Off                                    |
| Virtual machine.Inventory.Create new                                      | Virtual machine.Inventory.Create new                                      |
| Virtual machine.Inventory.Remove                                          | Virtual machine.Inventory.Remove                                          |
| Virtual machine.Inventory.Register                                        | Virtual machine.Inventory.Register                                        |
| Virtual machine.Provisioning.Allow disk access                            | Virtual machine.Provisioning.Allow disk access                            |
| Virtual machine.Provisioning.Allow file access                            | Virtual machine.Provisioning.Allow file access                            |
| Virtual machine.Provisioning.Allow read-only disk access                  | Virtual machine.Provisioning.Allow read-only disk access                  |
| Virtual machine.Provisioning.Allow virtual machine download               | Virtual machine.Provisioning.Allow virtual machine download               |
| Virtual machine.Snapshot management. Create snapshot                      | Virtual machine.Snapshot management. Create snapshot                      |
| Virtual machine.Snapshot management.Remove Snapshot                       | Virtual machine.Snapshot management.Remove Snapshot                       |
| Virtual machine.Snapshot management.Revert to snapshot                    | Virtual machine.Snapshot management.Revert to snapshot                    |

> [!NOTE]
> В следующей таблице перечислены привилегии для учетных записей пользователей vCenter 6.0 и vCenter 5.5.

| Привилегии для учетной записи пользователя vCenter 6.0 | Привилегии для учетной записи пользователя vCenter 5.5 |
| --- | --- |
| Datastore.AllocateSpace | Network.Assign |
| Global.Manage custom attributes | Datastore.AllocateSpace |
| Global.Set custom attribute | VirtualMachine.Config.ChangeTracking |
| Host.Local operations.Create virtual machine | VirtualMachine.State.RemoveSnapshot |
| Network. Assign network | VirtualMachine.State.CreateSnapshot |
| Resource. Assign virtual machine to resource pool | VirtualMachine.Provisioning.DiskRandomRead |
| Virtual machine.Configuration.Add new disk | VirtualMachine.Interact.PowerOff |
| Virtual machine.Configuration.Advanced | VirtualMachine.Inventory.Create |
| Virtual machine.Configuration.Disk change tracking | VirtualMachine.Config.AddNewDisk |
| Virtual machine.Configuration.Host USB device | VirtualMachine.Config.HostUSBDevice |
| Virtual machine.Configuration.Query unowned files | VirtualMachine.Config.AdvancedConfig |
| Virtual machine.Configuration.Swapfile placement | VirtualMachine.Config.SwapPlacement |
| Virtual machine.Interaction.Power Off | Global.ManageCustomFields |
| Virtual machine.Inventory. Create new |   |
| Virtual machine.Provisioning.Allow disk access |   |
| Virtual machine.Provisioning. Allow read-only disk access |   |
| Virtual machine.Snapshot management.Create snapshot |   |
| Virtual machine.Snapshot management.Remove Snapshot |   |

## <a name="create-a-vmware-account"></a>Создание учетной записи VMware

1. На панели **навигатор** VCenter Server выберите **Пользователи и группы**. Создайте учетную запись на соответствующем узле ESXi в том случае, если вы не используете vCenter Server.

    ![Пользователи и группы](./media/backup-azure-backup-server-vmware/vmware-userandgroup-panel.png)

    Появится панель **Пользователи и группы vCenter** .

2. На панели **Пользователи и группы vCenter** перейдите на вкладку **Пользователи** и щелкните значок Добавить пользователей (символ +).

    ![Панель vCenter Users and Groups (Пользователи и группы vCenter)](./media/backup-azure-backup-server-vmware/usersandgroups.png)

3. В диалоговом окне **Новый пользователь** добавьте сведения о пользователе, а затем щелкните **ОК**. В этой процедуре имя пользователя — BackupAdmin.

    ![Диалоговое окно New User (Новый пользователь)](./media/backup-azure-backup-server-vmware/vmware-new-user-account.png)

4. Чтобы связать учетную запись пользователя с ролью, на панели **Навигатор** выберите **глобальные разрешения**. На панели **глобальные разрешения** выберите вкладку **Управление** , а затем щелкните значок Добавить (символ "+").

    ![Панель Global Permissions (Глобальные разрешения)](./media/backup-azure-backup-server-vmware/vmware-add-new-perms.png)

5. В области **глобальные разрешения root — Добавление разрешения** нажмите кнопку **Добавить** , чтобы выбрать пользователя или группу.

    ![Выбор пользователя или группы](./media/backup-azure-backup-server-vmware/vmware-add-new-global-perm.png)

6. В окне **Select Users/Groups Выбрать пользователей или группы** (Выбрать пользователей или группы) выберите **BackupAdmin** > **Добавить**. В разделе **Users** (Пользователи) для учетной записи пользователя используется формат *домен\имя пользователя*. Если вы хотите использовать другой домен, выберите его из списка **доменов**. Нажмите кнопку **ОК** , чтобы добавить выбранных пользователей в диалоговое окно **Добавление разрешения** .

    ![Добавление пользователя BackupAdmin](./media/backup-azure-backup-server-vmware/vmware-assign-account-to-role.png)

7. В разделе **Назначенная роль** в раскрывающемся списке выберите **BackupAdminRole** > **ОК**.

    ![Назначение пользователя роли](./media/backup-azure-backup-server-vmware/vmware-choose-role.png)

На вкладке **Manage** (Управление) на панели **Global Permissions** (Глобальные разрешения) в списке появятся новая учетная запись пользователя и связанная роль.

## <a name="add-the-account-on-azure-backup-server"></a>Добавление учетной записи в Azure Backup Server

1. Откройте Azure Backup Server. Если не удается найти значок на рабочем столе, откройте Microsoft Azure Backup из списка приложений.

    ![Значок Azure Backup Server](./media/backup-azure-backup-server-vmware/mabs-icon.png)

2. В консоли Azure Backup Server выберите **Управление**  >   **рабочие серверы**  >  **Управление VMware**.

    ![Консоль сервера Azure Backup Server](./media/backup-azure-backup-server-vmware/add-vmware-credentials.png)

3. В диалоговом окне **Управление учетными данными** выберите **Добавить**.

    ![Диалоговое окно «Управление учетными данными»](./media/backup-azure-backup-server-vmware/mabs-manage-credentials-dialog.png)

4. В поле **Добавить учетные данные** введите имя и описание для новых учетных данных и укажите имя пользователя и пароль, определенные на сервере VMware. Имя *Contoso Vcenter credential* используется для идентификации учетных данных в этой процедуре. Если сервер VMware и Azure Backup Server размещены в разных доменах, укажите домен в имени пользователя.

    ![Диалоговое окно добавления учетных данных на сервере Azure Backup Server](./media/backup-azure-backup-server-vmware/mabs-add-credential-dialog2.png)

5. Нажмите кнопку **Добавить** , чтобы добавить новые учетные данные.

    ![Добавить новые учетные данные](./media/backup-azure-backup-server-vmware/new-list-of-mabs-creds.png)

## <a name="add-the-vcenter-server"></a>Добавление vCenter Server

Добавление vCenter Server в Azure Backup Server

1. В консоли Azure Backup Server выберите **Управление**  >  **рабочие серверы**  >  **Добавить**.

    ![Открытие мастера добавления рабочего сервера](./media/backup-azure-backup-server-vmware/add-vcenter-to-mabs.png)

2. В **мастере добавления рабочего сервера**  >  **выберите страница Тип рабочего сервера** , выберите **серверы VMware** и нажмите кнопку **Далее**.

    ![Мастер добавления рабочего сервера](./media/backup-azure-backup-server-vmware/production-server-add-wizard.png)

3. На шаге **Выбрать компьютеры в строке**  **Server Name/IP Address Имя или IP-адрес сервера** (Имя или IP-адрес сервера) укажите полное доменное имя или IP-адрес сервера VMware. Если один и тот же сервер vCenter управляет всеми серверами ESXi, то укажите имя vCenter. В противном случае добавьте узел ESXi.

    ![Указывание сервера VMware](./media/backup-azure-backup-server-vmware/add-vmware-server-provide-server-name.png)

4. В поле **SSL-порт** введите порт, который использовался для взаимодействия с сервером VMware. 443 – порт по умолчанию, но если ваш сервер VMware прослушивает другой порт, его можно изменить.

5. В поле **указания учетных данных** выберите созданные учетные данные.

    ![Указание учетных данных](./media/backup-azure-backup-server-vmware/identify-creds.png)

6. Нажмите кнопку **Добавить** , чтобы добавить сервер VMware в список серверы. Выберите **Далее**.

    ![Добавление сервера VMware и учетных данных](./media/backup-azure-backup-server-vmware/add-vmware-server-credentials.png)

7. На странице **Сводка** выберите **Добавить** , чтобы добавить сервер VMware в Azure Backup Server. Новый сервер добавляется сразу, на сервере VMware агент не нужен.

    ![Добавление сервера VMware в Azure Backup Server](./media/backup-azure-backup-server-vmware/tasks-screen.png)

8. Проверьте параметры на странице **Готово**.

   ![Страница "Готово"](./media/backup-azure-backup-server-vmware/summary-screen.png)

Если у вас есть несколько узлов ESXi, которыми сервер vCenter не управляет, или несколько экземпляров vCenter Server, для добавления серверов необходимо снова запустить мастер.

## <a name="configure-a-protection-group"></a>Настройка группы защиты

Добавьте виртуальные машины VMware для резервного копирования. Группы защиты собирают несколько виртуальных машин и применяют одинаковые параметры хранения данных и резервного копирования ко всем виртуальным машинам в группе.

1. В консоли Azure Backup Server выберите **Защита**, > **создать**.

    ![Открытие мастера создания групп защиты](./media/backup-azure-backup-server-vmware/open-protection-wizard.png)

1. На странице приветствия мастера **создания новой группы защиты** нажмите кнопку **Далее**.

    ![Диалоговое окно мастера создания групп защиты](./media/backup-azure-backup-server-vmware/protection-wizard.png)

1. На странице **Выбор типа группы защиты** выберите **серверы** , а затем нажмите кнопку **Далее**. Появится страница **Выбор элементов группы**.

1. В окне **Выбор элементов группы** выберите виртуальные машины (или папки виртуальных машин), для которых требуется создать резервную копию. Выберите **Далее**.

    - Когда вы выберите папку, виртуальные машины или папки внутри этой папки также будут выбраны для резервного копирования. Можно отменить выбор папок или виртуальных машин, для которых вы не хотите создавать резервную копию.
1. Это сделать невозможно, если уже выполняется резервное копирование виртуальной машины или папки. Это гарантирует, что дублирование точек восстановления для виртуальной машины не создается.

    ![Выберите членов группы](./media/backup-azure-backup-server-vmware/server-add-selected-members.png)

1. На странице **Select Data Protection Method** (Выбор метода защиты данных) введите имя для группы защиты и параметры защиты. Чтобы создать резервную копию Azure, установите краткосрочную защиту **Диска** и разрешите защиту в сети. Выберите **Далее**.

    ![Выберите метод защиты данных](./media/backup-azure-backup-server-vmware/name-protection-group.png)

1. В окне **Specify Short-Term Goals** (Выбор краткосрочных целей) укажите, как долго вы хотите хранить резервную копию данных на диске.
   - В окне **Диапазон хранения** укажите количество дней для хранения точек восстановления диска.
   - В окне **Частота синхронизации** укажите частоту создания точек восстановления диска.
       - Если вы не хотите задавать интервал резервного копирования, можно проверить **непосредственно перед точкой восстановления** , чтобы резервное копирование выполнялось непосредственно перед планированием каждой точки восстановления.
       - Краткосрочные резервные копии являются полными резервными копиями, а не добавочным.
       - Выберите **изменить** , чтобы изменить время и даты при краткосрочных резервных копиях.

         ![Выберите краткосрочные цели](./media/backup-azure-backup-server-vmware/short-term-goals.png)

1. В окне **Проверить выделение дискового пространства** проверьте дисковое пространство для резервных копий виртуальных машин. Для виртуальных машин.

   - Рекомендуемые дисковые пространства основаны на указанном диапазоне периода удержания, типе рабочей нагрузки и размере защищенных данных. Внесите необходимые изменения, а затем нажмите кнопку **Далее**.
   - **Размер данных** — размер данных в группе защиты.
   - **Место на диске** — рекомендуемый объем дискового пространства для группы защиты. Если вы хотите изменить этот параметр, следует выделить общее пространство, которое немного больше, чем предполагаемое значение для каждого из источников данных.
   - **Совместное размещение данных** — если вы включите совместное размещение, несколько источников данных в защите могут сопоставиться с отдельной репликой и томом точек восстановления. Совместное размещение поддерживается не для всех рабочих нагрузок.
   - **Автоматическое увеличение:** Если включить этот параметр, то если данные в защищенной группе будут увеличивать начальное выделение, Azure Backup Server попытается увеличить размер диска на 25 процентов.
   - **Дополнительные сведения о пуле носителей** — этот параметр отображает состояние пула носителей, включая общий размер диска и оставшийся.

    ![Просмотр выделения дискового пространства](./media/backup-azure-backup-server-vmware/review-disk-allocation.png)

1. На странице **Выбор метода создания реплики** укажите, как следует выполнять начальное резервное копирование, а затем нажмите кнопку **Далее**.
   - Значения по умолчанию — **Автоматически по сети** и **Сейчас**.
   - Если вы используете значения по умолчанию, укажите время низкой нагрузки. Выберите параметр **Позже** и укажите день и время.
   - Для больших объемов данных или неоптимальных условий сети лучше выбрать автономную репликацию данных с использованием съемных носителей.

    ![Выберите метод создания реплики](./media/backup-azure-backup-server-vmware/replica-creation.png)

1. На странице **Consistency Check Options** (Параметры проверки согласованности) выберите, как и когда автоматически запускать проверки на согласованность. Выберите **Далее**.
      - В случае несогласованности данных реплики вы можете запустить проверку на согласованность, либо эта проверка может выполняться по расписанию.
      - Если настраивать автоматическую проверку согласованности не требуется, можно выполнить проверку вручную. Чтобы это сделать, щелкните правой кнопкой мыши группу защиты, а затем нажмите **Perform Consistency Check** (Выполнить проверку согласованности).

1. На странице **Specify Online Protection Data** (Указание данных для оперативной защиты) выберите виртуальные машины или папки виртуальной машины для создания резервной копии. Вы можете выбрать элементы по отдельности или выбрать **пункт Выбрать все** , чтобы выбрать все элементы. Выберите **Далее**.

    ![Выбор оперативной защиты данных](./media/backup-azure-backup-server-vmware/select-data-to-protect.png)

1. На странице **Specify Online Backup Schedule** (Укажите расписание архивации в сети) укажите частоту резервного копирования данных из локального хранилища в Azure.

    - Точки восстановления для данных в облаке будут созданы согласно с расписанием. Выберите **Далее**.
    - После создания точки восстановления она передается в хранилище служб восстановления в Azure.

    ![Выбор расписания оперативного резервного копирования](./media/backup-azure-backup-server-vmware/online-backup-schedule.png)

1. На странице **Specify Online Retention Policy** (Выбор политику хранения в сети) укажите, каким образом точки восстановления, созданные из ежедневных, еженедельных, ежемесячных и ежегодных резервных копий, должны храниться в Azure. затем нажмите кнопку **Далее**.

    - Для хранения данных в Azure нет временных ограничений.
    - Единственным ограничением является то, что на защищенный экземпляр у вас может быть не более 9999 точек восстановления. В этом примере защищенным экземпляром является сервер VMware.

    ![Выбор политики оперативного хранения](./media/backup-azure-backup-server-vmware/retention-policy.png)

1. На странице **Сводка** проверьте параметры и нажмите кнопку **создать группу**.

    ![Сводка параметров и элементов группы защиты](./media/backup-azure-backup-server-vmware/protection-group-summary.png)

## <a name="vmware-parallel-backups"></a>Параллельные операции резервного копирования VMware

>[!NOTE]
> Эта функция применима к MABS v3 UR1.

В более ранних версиях MABS параллельные резервные копии выполнялись только в группах защиты. При использовании MABS v3 UR1 все резервные копии виртуальных машин VMWare в одной группе защиты являются параллельными, что приводит к ускорению резервного копирования виртуальных машин. Все задания разностной репликации VMWare выполняются параллельно. По умолчанию число заданий, выполняемых параллельно, равно 8.

Количество заданий можно изменить с помощью раздела реестра, как показано ниже (отсутствует по умолчанию, его необходимо добавить):

**Путь к разделу**: `Software\Microsoft\Microsoft Data Protection Manager\Configuration\ MaxParallelIncrementalJobs\VMWare`<BR>
**Тип ключа**: DWORD (32-bit).

> [!NOTE]
> Вы можете увеличить количество заданий. Если задать номер задания 1, задания репликации будут выполняться последовательно. Увеличивая число, необходимо учитывать производительность VMWare. Оцените количество используемых ресурсов и дополнительные сведения об использовании на сервере VMWare vSphere, а также определите число заданий разностной репликации, выполняемых параллельно. Кроме того, это изменение повлияет только на вновь созданные группы защиты. Для существующих групп защиты необходимо временно добавить еще одну виртуальную машину в группу защиты. В этом случае необходимо соответствующим образом обновить конфигурацию группы защиты. После завершения процедуры эту виртуальную машину можно удалить из группы защиты.

## <a name="vmware-vsphere-67"></a>VMware vSphere 6.7

Чтобы создать резервную копию vSphere 6,7, выполните следующие действия.

- Включение TLS 1,2 на сервере MABS

>[!NOTE]
>В VMWare 6,7 в качестве протокола связи включен протокол TLS.

- Задайте разделы реестра следующим образом:

```text
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v2.0.50727]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v2.0.50727]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001
```

## <a name="exclude-disk-from-vmware-vm-backup"></a>Исключение диска из резервной копии виртуальной машины VMware

> [!NOTE]
> Эта функция применима к MABS v3 UR1.

С помощью MABS v3 UR1 можно исключить конкретный диск из резервной копии виртуальной машины VMware. **ExcludeDisk.ps1** скрипта конфигурации находится в папке `C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin folder` .

Чтобы настроить исключение диска, выполните следующие действия.

### <a name="identify-the-vmware-vm-and-disk-details-to-be-excluded"></a>Поиск сведений о виртуальной машине и диске VMWare, которые будут исключены

  1. В консоли VMware перейдите к параметрам виртуальной машины, для которых требуется исключить диск.
  2. Выберите диск, который необходимо исключить, и запишите путь к этому диску.

        Например, чтобы исключить жесткий диск 2 из TestVM4, путь к жесткому диску 2 — **[datastore1] TestVM4/TestVM4\_1.vmdk**.

        ![Жесткий диск для исключения](./media/backup-azure-backup-server-vmware/test-vm.png)

### <a name="configure-mabs-server"></a>Настройка сервера MABS

Перейдите на сервер MABS, на котором виртуальная машина VMware настроена для защиты, чтобы настроить исключение диска.

  1. Получите сведения о узле VMware, который защищен на сервере MABS.

        ```powershell
        $psInfo = get-DPMProductionServer
        $psInfo
        ```

        ```output
        ServerName   ClusterName     Domain            ServerProtectionState
        ----------   -----------     ------            ---------------------
        Vcentervm1                   Contoso.COM       NoDatasourcesProtected
        ```

  2. Выберите узел VMware и укажите защиту для узла VMware.

        ```powershell
        $vmDsInfo = get-DPMDatasource -ProductionServer $psInfo[0] -Inquire
        $vmDsInfo
        ```

        ```output
        Computer     Name     ObjectType
        --------     ----     ----------
        Vcentervm1  TestVM2      VMware
        Vcentervm1  TestVM1      VMware
        Vcentervm1  TestVM4      VMware
        ```

  3. Выберите виртуальную машину, для которой требуется исключить диск.

        ```powershell
        $vmDsInfo[2]
        ```

        ```output
        Computer     Name      ObjectType
        --------     ----      ----------
        Vcentervm1   TestVM4   VMware
        ```

  4. Чтобы исключить диск, перейдите в `Bin` папку и запустите сценарий *ExcludeDisk.ps1* со следующими параметрами:

        > [!NOTE]
        > Перед выполнением этой команды необходимо запустить службу DPMRA на сервере MABS. В противном случае скрипт возвращает значение Success, но не обновляет список исключений. Перед остановкой службы убедитесь, что нет выполняющихся заданий.

     **Чтобы добавить или удалить диск из исключения, выполните следующую команду:**

      ```powershell
      ./ExcludeDisk.ps1 -Datasource $vmDsInfo[0] [-Add|Remove] "[Datastore] vmdk/vmdk.vmdk"
      ```

     **Пример**:

     Чтобы добавить исключение диска для TestVM4, выполните следующую команду:

       ```powershell
      C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -Add "[datastore1] TestVM4/TestVM4\_1.vmdk"
       ```

      ```output
       Creating C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin\excludedisk.xml
       Disk : [datastore1] TestVM4/TestVM4\_1.vmdk, has been added to disk exclusion list.
      ```

  5. Убедитесь, что диск добавлен для исключения.

     **Чтобы просмотреть существующее исключение для конкретных виртуальных машин, выполните следующую команду:**

        ```powershell
        ./ExcludeDisk.ps1 -Datasource $vmDsInfo[0] [-view]
        ```

     **Пример**

        ```powershell
        C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -view
        ```

        ```output
        <VirtualMachine>
        <UUID>52b2b1b6-5a74-1359-a0a5-1c3627c7b96a</UUID>
        <ExcludeDisk>[datastore1] TestVM4/TestVM4\_1.vmdk</ExcludeDisk>
        </VirtualMachine>
        ```

     После настройки защиты для этой виртуальной машины исключенный диск не будет перечислен во время защиты.

        > [!NOTE]
        > Если вы выполняете эти действия для уже защищенной виртуальной машины, необходимо запустить проверку согласованности вручную после добавления диска для исключения.

### <a name="remove-the-disk-from-exclusion"></a>Удаление диска из исключения

Чтобы удалить диск из исключения, выполните следующую команду:

```powershell
C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -Remove "[datastore1] TestVM4/TestVM4\_1.vmdk"
```

## <a name="next-steps"></a>Дальнейшие действия

Для решения проблем с устранением неполадок при настройке резервного копирования см. статью [Устранение неполадок Azure Backup Server](./backup-azure-mabs-troubleshoot.md).
