---
title: Соединитель SAP LaMa для Azure | Документация Майкрософт
description: Управление системами SAP в Azure с помощью SAP LaMa
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: timlt
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/29/2019
ms.author: sedusch
ms.openlocfilehash: 4772fdae06f23430d829fa411068b7af7a85b3dd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101668713"
---
# <a name="sap-lama-connector-for-azure"></a>Соединитель SAP LaMa для Azure

[1877727]:https://launchpad.support.sap.com/#/notes/1877727
[2343511]:https://launchpad.support.sap.com/#/notes/2343511
[2350235]:https://launchpad.support.sap.com/#/notes/2350235
[2562184]:https://launchpad.support.sap.com/#/notes/2562184
[2628497]:https://launchpad.support.sap.com/#/notes/2628497
[2445033]:https://launchpad.support.sap.com/#/notes/2445033
[2815988]:https://launchpad.support.sap.com/#/notes/2815988
[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png
[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md
[hana-ops-guide]:hana-vm-operations.md

> [!NOTE]
> Заявление об общей поддержке. Всегда открывайте инцидент с SAP в компоненте BC-VCM-LVM-HYPERV, если вам нужна поддержка по SAP LaMa или соединителю Azure.

SAP LaMa используется многими клиентами для использования и мониторинга их ландшафта SAP. Начиная c версии 3.0 SP05, SAP LaMa по умолчанию поставляется с соединителем Azure. Вы можете использовать этот соединитель для освобождения и запуска виртуальных машин, копирования и перемещения управляемых дисков и удаления управляемых дисков. Эти основные операции позволяют вам перемещать, копировать, клонировать и обновлять системы SAP с помощью SAP LaMa.

В этом руководстве описывается, как настроить соединитель Azure для SAP LaMa, создать виртуальные машины, которые могут использоваться для установки адаптивных систем SAP, и настроить их.

> [!NOTE]
> Соединитель доступен только в выпуске SAP LaMa Enterprise Edition.

## <a name="resources"></a>Ресурсы

Следующие примечания к SAP актуальны для развертывания SAP LaMa в Azure.

| Номер примечания | Title |
| --- | --- |
| [2343511] |Соединитель Microsoft Azure для управления ландшафтом SAP (LaMa) |
| [2350235] |SAP Landscape Management 3.0 Enterprise Edition |

Также читайте сведения о [SAP LaMa на справочном портале SAP](https://help.sap.com/viewer/p/SAP_LANDSCAPE_MANAGEMENT_ENTERPRISE).

## <a name="general-remarks"></a>Общие замечания

* Не забудьте включить параметр *автоматического создания точки подключения* на вкладке "Setup -> Settings -> Engine" (Настройка -> Параметры -> Подсистема).  
  Если SAP LaMa подключает тома с использованием адаптивных расширений SAP на виртуальной машине, при отключенном приведенном выше параметре должна существовать точка подключения.

* Используйте отдельную подсеть и не указывайте динамические IP-адреса, чтобы предотвратить перехват IP-адреса при развертывании новых виртуальных машин, когда экземпляры SAP не готовы.  
  Если вы используете распределение динамических IP-адресов в подсети, которая также используется SAP LaMa, подготовка системы SAP с SAP LaMa может завершиться сбоем. Если система SAP не готова, IP-адреса не резервируются и могут быть выделены другим виртуальным машинам.

* Если вы входите на управляемые узлы, убедитесь, что не блокируется отключение файловых систем.  
  Если вы входите на виртуальные машины Linux и изменяете рабочий каталог на каталог в точке подключения, например/usr/sap/AH1/ASCS00/exe, том не может быть отключен, а перемещение или неудача завершается сбоем.

* Не забудьте отключить CLOUD_NETCONFIG_MANAGE на виртуальных машинах SUSE SLES Linux. Дополнительные сведения см. в статье [SUSE KB 7023633](https://www.suse.com/support/kb/doc/?id=7023633).

## <a name="set-up-azure-connector-for-sap-lama"></a>Настройка соединителя Azure для SAP LaMa

Соединитель Azure поставляется с SAP LaMa, начиная с версии 3.0 SP05. Мы рекомендуем всегда устанавливать последний пакет поддержки и исправление для SAP LaMa 3.0.

Соединитель Azure использует API Azure Resource Manager для управления ресурсами Azure. SAP лама может использовать субъект-службу или управляемое удостоверение для проверки подлинности в этом API. Если приложение SAP лама работает на виртуальной машине Azure, мы рекомендуем использовать управляемое удостоверение, как описано в разделе [использование управляемого удостоверения для получения доступа к API Azure](lama-installation.md#af65832e-6469-4d69-9db5-0ed09eac126d). Если вы хотите использовать субъект-службу, выполните действия, описанные в разделе [Использование субъекта-службы для получения доступа к API Azure](lama-installation.md#913c222a-3754-487f-9c89-983c82da641e).

### <a name="use-a-service-principal-to-get-access-to-the-azure-api"></a><a name="913c222a-3754-487f-9c89-983c82da641e"></a>Использование субъекта-службы для получения доступа к API Azure

Соединитель Azure может использовать субъект-службу для авторизации Microsoft Azure. Выполните следующие действия, чтобы создать субъект-службу для управления ландшафтом SAP (LaMa).

1. Перейдите на сайт https://portal.azure.com.
1. Откройте колонку "Azure Active Directory".
1. Щелкните "Регистрация приложений".
1. Щелкните "создать регистрацию".
1. Введите имя и щелкните "зарегистрировать".
1. Выберите новое приложение и щелкните Сертификаты & секреты на вкладке Параметры.
1. Создайте новый секрет клиента, введите описание нового ключа, выберите время истечения срока действия секрета и нажмите кнопку Сохранить.
1. Запишите его значение. Он используется в качестве пароля субъекта-службы.
1. Запишите идентификатор приложения. Он используется в качестве имени пользователя субъекта-службы.

У субъекта-службы по умолчанию нет разрешений на доступ к ресурсам Azure. Субъекту-службе необходимо предоставить соответствующие разрешения.

1. Перейдите на страницу https://portal.azure.com.
1. Откройте колонку группы ресурсов.
1. Выберите группу ресурсов, которую нужно использовать.
1. Выберите "Управление доступом (IAM)".
1. Щелкните "Добавить назначение ролей".
1. Выберите роль участника.
1. Введите имя созданного ранее приложения.
1. Щелкните Сохранить
1. Повторите шаги с 3 по 8 для всех групп ресурсов, которые вы хотите использовать в SAP LaMa.

### <a name="use-a-managed-identity-to-get-access-to-the-azure-api"></a><a name="af65832e-6469-4d69-9db5-0ed09eac126d"></a>Использование управляемого удостоверения для получения доступа к API Azure

Чтобы иметь возможность использовать управляемое удостоверение, экземпляр SAP Лама должен работать на виртуальной машине Azure, которой назначена система или пользовательское удостоверение. Дополнительные сведения об управляемых удостоверениях см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../../../active-directory/managed-identities-azure-resources/overview.md) и [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине с помощью портал Azure](../../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md).

По умолчанию управляемое удостоверение не имеет разрешений на доступ к ресурсам Azure. Необходимо предоставить ей разрешения на доступ к ним.

1. Перейдите на страницу https://portal.azure.com.
1. Откройте колонку группы ресурсов.
1. Выберите группу ресурсов, которую нужно использовать.
1. Выберите "Управление доступом (IAM)".
1. Щелкните Add-> добавить назначение ролей.
1. Выберите роль участника.
1. Выберите "виртуальная машина" для "назначение доступа к"
1. Выберите виртуальную машину, на которой работает экземпляр SAP лама
1. Щелкните Сохранить
1. Повторите действия для всех групп ресурсов, которые вы хотите использовать в SAP лама

В конфигурации соединителя SAP лама Azure выберите параметр "использовать управляемое удостоверение", чтобы разрешить использование управляемого удостоверения. Если вы хотите использовать назначенное системой удостоверение, обязательно оставьте поле "имя пользователя" пустым. Если вы хотите использовать удостоверение, назначенное пользователем, введите идентификатор удостоверения в поле имя пользователя.

### <a name="create-a-new-connector-in-sap-lama"></a>Создание нового соединителя в SAP лама

Откройте веб-сайт SAP LaMa и перейдите к разделу инфраструктуры. Перейдите на вкладку "Cloud Managers" (Менеджеры для облака) и щелкните "Добавить". Выберите адаптер для облака Microsoft Azure и нажмите кнопку "Далее". Введите следующие сведения:

* Метка: выберите имя экземпляра соединителя.
* Имя пользователя: идентификатор приложения субъекта-службы или идентификатор назначенного пользователю удостоверения виртуальной машины. Дополнительные сведения см. в разделе [использование системного или назначенного пользователем удостоверения].
* Пароль: ключ или пароль субъекта-службы. Если вы используете систему или назначенное пользователем удостоверение, это поле можно оставить пустым.
* URL-адрес: оставьте адрес по умолчанию `https://management.azure.com/`.
* Интервал мониторинга (секунды): должен быть не менее 300.
* Использовать управляемое удостоверение: SAP лама может использовать систему или назначенное пользователем удостоверение для проверки подлинности в API Azure. См. раздел [использование управляемого удостоверения для получения доступа к API Azure](lama-installation.md#af65832e-6469-4d69-9db5-0ed09eac126d) в этом руководством.
* Идентификатор подписки: идентификатор подписки Azure.
* Идентификатор клиента Azure Active Directory: идентификатор клиента Active Directory.
* Узел прокси-сервера: имя узла прокси-сервера, если SAP LaMa требуется прокси-сервер для подключения к Интернету.
* Порт прокси-сервера: TCP-порт прокси-сервера.
* Изменение типа хранилища для экономии затрат. Включите этот параметр, если адаптер Azure должен изменить тип хранилища управляемых дисков, чтобы снизить затраты, если диски не используются. Для дисков данных, на которые имеются ссылки в конфигурации экземпляра SAP, адаптер изменит тип диска на стандартное хранилище во время подготовки экземпляра и возврата к исходному типу хранилища во время подготовки экземпляра. Если вы останавливаете виртуальную машину в SAP лама, адаптер будет изменять тип хранилища всех подключенных дисков, включая диск операционной системы в хранилище класса Standard. Если запустить виртуальную машину в SAP лама, адаптер изменит тип хранилища обратно на исходный тип хранилища.

Щелкните "Конфигурация теста" для проверки входных данных. Вы должны увидеть сообщение наподобие

"Подключение установлено успешно: подключение к Microsoft Cloud установлено. Найдено 7 групп ресурсов (запрошено только 10 групп)"

в нижней части веб-сайта.

## <a name="provision-a-new-adaptive-sap-system"></a>Подготовка новой адаптивной системы SAP

Можно вручную развернуть виртуальную машину или использовать один из шаблонов Azure в [репозитории быстрого запуска](https://github.com/Azure/azure-quickstart-templates). Он содержит шаблоны для [SAP NetWeaver ASCS](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-ascs), [серверов приложений SAP NetWeaver](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-apps)и [базы данных](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-database). Вы также можете использовать эти шаблоны для предоставления новых узлов в составе системной копии или клона и т. д.

Мы рекомендуем использовать отдельную подсеть для всех виртуальных машин, которыми вы хотите управлять с помощью SAP LaMa, и не использовать динамические IP-адреса для предотвращения "перехвата" IP-адреса при развертывании новых виртуальных машин, когда экземпляры SAP не готовы.

> [!NOTE]
> Если возможно, удалите все расширения виртуальной машины, так как они могут привести к длительным операциям отсоединения дисков от виртуальной машины.

Убедитесь, что пользователь \<hanasid> ADM, \<sapsid> ADM и группа sapsys существуют на целевом компьютере с тем же идентификатором и GID или используйте LDAP. Включите и запустите NFS-сервер на виртуальных машинах, которые должны использоваться для запуска SAP NetWeaver (A)SCS.

### <a name="manual-deployment"></a>Развертывание вручную

SAP LaMa взаимодействует с виртуальной машиной с помощью агента узла SAP. Если вы вручную развертываете виртуальные машины или не используете шаблон Azure Resource Manager из репозитория быстрого запуска, обязательно установите последний агент узла SAP и адаптивные расширения SAP. Дополнительные сведения об обязательных уровнях исправления для Azure см. в примечании к SAP [2343511].

#### <a name="manual-deployment-of-a-linux-virtual-machine"></a>Развертывание виртуальной машины Linux вручную

Создайте виртуальную машину с одной из поддерживаемых операционных систем, перечисленных в примечании к SAP [2343511]. Добавьте дополнительные IP-конфигурации для экземпляров SAP. Каждому экземпляру требуется по крайней мере один IP-адрес. Экземпляр должен быть установлен с использованием виртуального имени узла.

Экземпляру SAP NetWeaver ASCS требуются диски для/сапмнт/ \<SAPSID> ,/usr/SAP/ \<SAPSID> ,/УСР/САП/транс и/usr/SAP/ \<sapsid> ADM. Серверам приложений SAP NetWeaver не нужны дополнительные диски. Все, что связано с экземпляром SAP, должно храниться в ASCS и экспортироваться через NFS. В противном случае в настоящее время невозможно добавить дополнительные серверы приложений с помощью SAP LaMa.

![SAP NetWeaver ASCS в Linux](media/lama/sap-lama-ascs-app-linux.png)

#### <a name="manual-deployment-for-sap-hana"></a>Развертывание вручную для SAP HANA

Создайте виртуальную машину с одной из поддерживаемых операционных систем для SAP HANA, перечисленных в примечании к SAP [2343511]. Добавьте дополнительную IP-конфигурацию для SAP HANA и для клиента HANA.

Для SAP HANA требуются диски /hana/shared, /hana/backup, /hana/data и /hana/log.

![SAP HANA в Linux](media/lama/sap-lama-db-hana.png)

#### <a name="manual-deployment-for-oracle-database-on-linux"></a>Развертывание вручную для Oracle Database в Linux

Создайте виртуальную машину с одной из поддерживаемых операционных систем для баз данных Oracle, перечисленных в примечании к SAP [2343511]. Добавьте дополнительную IP-конфигурацию для базы данных Oracle.

Базе данных Oracle требуются диски для /oracle, /home/oraod1 и /home/oracle.

![Схема, на которой показана база данных Oracle в Linux и необходимые диски.](media/lama/sap-lama-db-ora-lnx.png)

#### <a name="manual-deployment-for-microsoft-sql-server"></a>Развертывание вручную для Microsoft SQL Server

Создайте виртуальную машину с одной из поддерживаемых операционных систем для Microsoft SQL Server, перечисленных в примечании к SAP [2343511]. Добавьте дополнительную IP-конфигурацию для экземпляра SQL Server.

Серверу базы данных SQL Server нужны диски для данных базы данных и файлов журналов и диски для c:\usr\sap.

![База данных Oracle в Linux](media/lama/sap-lama-db-sql.png)

Обязательно установите поддерживаемый драйвер Microsoft ODBC Driver for SQL Server на виртуальной машине, куда вы хотите перенести сервер приложений SAP NetWeaver или которую вы хотите использовать как целевую системную копию или клон.

SAP LaMa не может перемещать сам SQL Server, поэтому для виртуальной машины, куда вы хотите переместить экземпляр базы данных или которую вы хотите использовать в качестве целевой системной копии или клона, требуется предустановленный SQL Server.

### <a name="deploy-virtual-machine-using-an-azure-template"></a>Развертывание виртуальной машины с помощью шаблона Azure

Скачайте следующие последние доступные архивы из [Marketplace ПО SAP](https://support.sap.com/swdc) для операционной системы виртуальных машин:

1. SAPCAR 7.21.
1. АГЕНТ УЗЛА SAP 7.21.
1. АДАПТИВНОЕ РАСШИРЕНИЕ 1.0 ДЛЯ SAP.

Кроме того, скачайте следующие компоненты из [центра загрузки Майкрософт](https://www.microsoft.com/download).

1. Распространяемый пакет Microsoft Visual C++ 2010 (64-разрядная версия) (только для Windows).
1. Microsoft ODBC Driver for SQL Server (только для SQL Server).

Для развертывания шаблона необходимы некоторые компоненты. Самый простой способ сделать их доступными для шаблона — загрузить их в учетную запись Azure и создать подписанный URL-адрес (SAS).

Шаблоны имеют следующие параметры:

* sapSystemId: идентификатор системы SAP. Он используется для создания макета диска (например,/usr/SAP/ \<sapsid> ).

* computerName: имя компьютера новой виртуальной машины. Этот параметр также используется SAP LaMa. При использовании этого шаблона для подготовки новой виртуальной машины как части системной копии SAP LaMa ожидает, пока узел с этим именем компьютера не станет доступным.

* osType: тип операционной системы, которую требуется развернуть.

* dbType: тип базы данных. Этот параметр используется, чтобы определить, сколько дополнительных IP-конфигураций нужно добавить и как должна выглядеть разметка диска.

* sapSystemSize: размер системы SAP, которую вы хотите развернуть. Он используется для определения типа экземпляра и размера виртуальной машины.

* adminUsername: имя пользователя виртуальной машины.

* adminPassword: пароль виртуальной машины. Вы также можете предоставить открытый ключ для SSH.

* sshKeyData: открытый ключ SSH для виртуальных машин. Поддерживается только для операционных систем Linux.

* subnetId: идентификатор требуемой подсети.

* deployEmptyTarget: вы можете развернуть пустой целевой объект, если хотите использовать виртуальную машину в качестве целевого объекта для перемещения экземпляра. В этом случае дополнительные диски или IP-конфигурации не подключаются.

* sapcarLocation: расположение приложения sapcar, соответствующего развертываемой операционной системе. Sapcar используется для извлечения архивов, предоставленных в других параметрах.

* sapHostAgentArchiveLocation: расположение архива агента узла SAP. Агент узла SAP развертывается как часть этого шаблона развертывания.

* sapacExtLocation: расположение адаптивных расширений SAP. В примечании к SAP [2343511] перечислен минимальный требуемый уровень исправлений для Azure.

* vcRedistLocation: расположение среды выполнения виртуального кластера, необходимое для установки адаптивных расширений SAP. Этот параметр требуется только для Windows.

* odbcDriverLocation: расположение для драйвера ODBC, который нужно установить. Поддерживается только драйвер Microsoft ODBC Driver for SQL Server.

* sapadmPassword: пароль для пользователя sapadm.

* sapadmId: идентификатор пользователя Linux для пользователя sapadm. Не требуется для Windows.

* sapsysGid: идентификатор группы Linux для группы sapsys. Не требуется для Windows.

* _artifactsLocation: базовый URI расположения артефактов, необходимых для этого шаблона. После развертывания шаблона с помощью прилагаемых сценариев будет использоваться закрытое расположение в подписке и это значение будет создано автоматически. Требуется, только если вы не выполняете развертывание шаблона из GitHub.

* _artifactsLocationSasToken: sasToken, необходимый для доступа к _artifactsLocation. При развертывании шаблона с помощью сопутствующих сценариев sasToken будет создан автоматически. Требуется, только если вы не выполняете развертывание шаблона из GitHub.

### <a name="sap-hana"></a>SAP HANA

В приведенных ниже примерах предполагается, что вы устанавливаете SAP HANA с системным идентификатором HN1, а систему SAP NetWeaver с системным идентификатором AH1. Виртуальные имена узлов представляют собой hn1-db для экземпляра HANA, ah1-db для клиента HANA, используемого системой SAP NetWeaver, ah1-ascs для SAP NetWeaver ASCS и ah1-di-0 для первого сервера приложений SAP NetWeaver.

#### <a name="install-sap-netweaver-ascs-for-sap-hana-using-azure-managed-disks"></a>Установка SAP NetWeaver ASCS для SAP HANA с помощью управляемых дисков Azure

Перед запуском SAP Software Provisioning Manager (SWPM) вам необходимо подключить IP-адрес виртуального имени узла ASCS. Рекомендуется использовать sapacext. Если вы подключаете IP-адрес с помощью sapacext, после перезагрузки его нужно подключить заново.

![Логотип Linux.][Logo_Linux] Linux

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-ascs -n 255.255.255.128
```

![Эмблема Windows.][Logo_Windows] Windows

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h ah1-ascs -n 255.255.255.128
```

Запустите SWPM и используйте *ah1-ascs* в качестве *имени узла экземпляра ASCS*.

![Логотип Linux.][Logo_Linux] Linux  
Добавьте следующий параметр профиля в профиль агента узла SAP, расположенный в /usr/sap/hostctrl/exe/host_profile. Дополнительные сведения см. в примечании к SAP [2628497].
```
acosprep/nfs_paths=/home/ah1adm,/usr/sap/trans,/sapmnt/AH1,/usr/sap/AH1
```

#### <a name="install-sap-netweaver-ascs-for-sap-hana-on-azure-netappfiles-anf-beta"></a>Установка SAP NetWeaver ASCS для SAP HANA в бета-версии Azure Нетаппфилес (использовании)

> [!NOTE]
> Эта функция еще не общедоступна. Дополнительные сведения см. в примечании к SAP [2815988] (отображаются только для пользователей предварительной версии).
Откройте инцидент SAP в компоненте BC-ВКМ-LVM-HYPERV и отправьте запрос на присоединение к адаптеру хранилища лама для предварительной версии Azure NetApp Files

ИСПОЛЬЗОВАНИИ предоставляет NFS для Azure. В контексте SAP лама это упрощает создание экземпляров ABAP Central Services (ASCS) и последующей установки серверов приложений. Ранее экземпляр ASCS должен действовать как NFS-сервер, а параметр акоспреп/nfs_paths должен быть добавлен в host_profile Hostagent SAP.

#### <a name="anf-is-currently-available-in-these-regions"></a>ИСПОЛЬЗОВАНИИ в настоящее время доступно в следующих регионах:

Восточная Австралия, Центральная часть США, восток США 2, Северная Европа, Юго-Центральный регион США, Западная Европа и Западная часть США 2.

#### <a name="network-requirements"></a>Требования к сети

Для использовании требуется Делегированная подсеть, которая должна быть частью той же ВИРТУАЛЬНОЙ сети, что и серверы SAP. Ниже приведен пример такой конфигурации.
На этом снимке экрана показано создание ВИРТУАЛЬНОЙ сети и первая подсеть.

![Создание виртуальной сети SAP лама для Azure использовании ](media/lama/sap-lama-createvn-50.png)

На следующем шаге создается Делегированная подсеть для Microsoft. NetApp/Volumes.

![SAP лама Добавление делегированной подсети ](media/lama/sap-lama-addsubnet-50.png)

![Список подсетей SAP лама ](media/lama/sap-lama-subnets.png)

Теперь учетная запись NetApp должна быть создана в портал Azure:

![Учетная запись NetApp SAP лама CREATE ](media/lama/sap-lama-create-netappaccount-50.png)

![Учетная запись SAP лама NetApp создана ](media/lama/sap-lama-netappaccount.png)

В учетной записи NetApp пул емкости определяет размер и тип дисков для каждого пула:

![Лама создания пула емкости NetApp SAP ](media/lama/sap-lama-capacitypool-50.png)

![Пул ресурсов SAP лама NetApp создан ](media/lama/sap-lama-capacitypool-list.png)

Теперь тома NFS можно определить. Так как в одном пуле будут храниться тома для нескольких систем, следует выбрать схему именования с самообъяснением. Добавление идентификатора безопасности позволяет группировать связанные тома вместе. Для ASCS и экземпляра AS необходимы следующие подключения: */сапмнт/ \<SID\>*, */usr/SAP/ \<SID\>* и */Хоме/ \<sid\> ADM*. При необходимости */УСР/САП/транс* требуется для центрального каталога транспорта, который по крайней мере используется всеми системами в одной альбомной ориентации.

> [!NOTE]
> На этапе бета-тестирования имена томов должны быть уникальными в пределах подписки.

![SAP лама создание тома 1 ](media/lama/sap-lama-createvolume-80.png)

![SAP лама создание тома 2 ](media/lama/sap-lama-createvolume2-80.png)

![SAP лама создание тома 3 ](media/lama/sap-lama-createvolume3-80.png)

Эти действия также необходимо повторить для других томов.

![Список созданных томов SAP лама ](media/lama/sap-lama-volumes.png)

Теперь эти тома должны быть подключены к системам, где будет выполняться начальная установка с помощью SAP SWPM.

Сначала необходимо создать точки подключения. В этом случае идентификатор безопасности AN1, поэтому необходимо выполнить следующие команды:

```bash
mkdir -p /home/an1adm
mkdir -p /sapmnt/AN1
mkdir -p /usr/sap/AN1
mkdir -p /usr/sap/trans
```
Далее тома использовании будут подключены с помощью следующих команд:

```bash
# sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 9.9.9.132:/an1-home-sidadm /home/an1adm
# sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 9.9.9.132:/an1-sapmnt-sid /sapmnt/AN1
# sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 9.9.9.132:/an1-usr-sap-sid /usr/sap/AN1
# sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 9.9.9.132:/global-usr-sap-trans /usr/sap/trans
```
Команды подключения также могут быть производными от портала. Необходимо настроить локальные точки подключения.

Для проверки используйте команду df-h.

![Уровень ОС точек подключения SAP лама ](media/lama/sap-lama-mounts.png)

Теперь необходимо выполнить установку с SWPM.

Те же действия должны выполняться по крайней мере для одного экземпляра.

После успешной установки система должна быть обнаружена в SAP лама.

Точки подключения должны выглядеть следующим образом для экземпляра ASCS и AS:

![Точки подключения SAP лама в Лама ](media/lama/sap-lama-ascs.png) (это пример. IP-адреса и путь экспорта отличаются от путей, которые использовались ранее).


#### <a name="install-sap-hana"></a>Установка SAP HANA

При установке SAP HANA с помощью средства командной строки hdblcm используйте параметр --hostname для предоставления имени виртуального узла. Необходимо добавить IP-адрес имени виртуального узла базы данных в сетевой интерфейс. Рекомендуется использовать sapacext. Если вы подключаете IP-адрес с помощью sapacext, после перезагрузки его нужно подключить заново.

Добавьте еще одно имя виртуального узла и IP-адрес для имени, используемого на серверах приложений для подключения к клиенту HANA.

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h hn1-db -n 255.255.255.128
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-db -n 255.255.255.128
```

Запустите установку экземпляра базы данных SWPM на виртуальной машине сервера приложений, а не на виртуальной машине HANA. Используйте *ah1-db* в качестве *узла базы данных* в диалоговом окне *Database for SAP System* (База данных для системы SAP).

#### <a name="install-sap-netweaver-application-server-for-sap-hana"></a>Установка сервера приложений SAP NetWeaver для SAP HANA

Перед запуском SAP Software Provisioning Manager (SWPM) вам необходимо подключить IP-адрес имени виртуального узла сервера приложений. Рекомендуется использовать sapacext. Если вы подключаете IP-адрес с помощью sapacext, после перезагрузки его нужно подключить заново.

![Логотип Linux.][Logo_Linux] Linux

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-di-0 -n 255.255.255.128
```

![Эмблема Windows.][Logo_Windows] Windows

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h ah1-di-0 -n 255.255.255.128
```

Рекомендуется использовать параметр профиля SAP NetWeaver dbs/hdb/hdb_use_ident, чтобы установить идентификатор, который используется для поиска ключа в пользовательском магазине HDB. Этот параметр можно добавить вручную после установки экземпляра базы данных с помощью SWPM или запустить SWPM с помощью команды:

```bash
# from https://blogs.sap.com/2015/04/14/sap-hana-client-software-different-ways-to-set-the-connectivity-data/
/sapdb/DVDs/IM_LINUX_X86_64/sapinst HDB_USE_IDENT=SYSTEM_COO
```

Если вы установите его вручную, также необходимо создать новые записи в пользовательском магазине HDB.

```bash
# run as <sapsid>adm
/usr/sap/AH1/hdbclient/hdbuserstore LIST
# reuse the port that was listed from the command above, in this example 35041
/usr/sap/AH1/hdbclient/hdbuserstore SET DEFAULT ah1-db:35041@AH1 SAPABAP1 <password>
```

Используйте *ah1-di-0* в качестве *имени узла экземпляра PAS* в диалоговом окне *Primary Application Server Instance* (Основной экземпляр сервера приложений).

#### <a name="post-installation-steps-for-sap-hana"></a>Действия после установки для SAP HANA

Обязательно создайте резервную копию SYSTEMDB и всех баз данных клиентов, прежде чем пытаться делать копию клиента, перемещать клиент или создавать репликацию системы.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

В приведенных ниже примерах предполагается, что вы устанавливаете систему SAP NetWeaver с системным ID AS1. Имена виртуальных узлов представляют собой as1-db для экземпляра SQL Server, используемого системой SAP NetWeaver, as1-ascs для SAP NetWeaver ASCS и as1-di-0 для первого сервера приложений SAP NetWeaver.

#### <a name="install-sap-netweaver-ascs-for-sql-server"></a>Установка SAP NetWeaver ASCS для SQL Server

Перед запуском SAP Software Provisioning Manager (SWPM) вам необходимо подключить IP-адрес виртуального имени узла ASCS. Рекомендуется использовать sapacext. Если вы подключаете IP-адрес с помощью sapacext, после перезагрузки его нужно подключить заново.

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-ascs -n 255.255.255.128
```

Запустите SWPM и используйте *as1-ascs* в качестве *имени узла экземпляра ASCS*.

#### <a name="install-sql-server"></a>Установка SQL Server

Необходимо добавить IP-адрес имени виртуального узла базы данных в сетевой интерфейс. Рекомендуется использовать sapacext. Если вы подключаете IP-адрес с помощью sapacext, после перезагрузки его нужно подключить заново.

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-db -n 255.255.255.128
```

Запустите установку экземпляра базы данных SWPM на виртуальной машине SQL Server. Используйте SAPINST_USE_HOSTNAME=*as1-db*, чтобы переопределить имя узла, используемое для подключения к SQL Server. Если вы развернули виртуальную машину с помощью шаблона Azure Resource Manager, не забудьте задать для каталога, используемого для файлов данных базы данных, значение *C:\sql\data*, а для файла журнала базы данных — *C:\sql\log*.

Убедитесь, что пользователь *NT AUTHORITY\SYSTEM* имеет доступ к SQL Server и роль сервера *sysadmin*. Дополнительные сведения см. в примечаниях к SAP [1877727] и [2562184].

#### <a name="install-sap-netweaver-application-server"></a>Установка сервера приложений SAP NetWeaver

Перед запуском SAP Software Provisioning Manager (SWPM) вам необходимо подключить IP-адрес имени виртуального узла сервера приложений. Рекомендуется использовать sapacext. Если вы подключаете IP-адрес с помощью sapacext, после перезагрузки его нужно подключить заново.

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-di-0 -n 255.255.255.128
```

Используйте *as1-di-0* в качестве *имени узла экземпляра PAS* в диалоговом окне *Primary Application Server Instance* (Основной экземпляр сервера приложений).

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="errors-and-warnings-during-discover"></a>Ошибки и предупреждения во время обнаружения

* Разрешению SELECT отказано в доступе.
  * [Microsoft][ODBC SQL Server Driver][SQL Server]The SELECT permission was denied on the object 'log_shipping_primary_databases', database 'msdb', schema 'dbo'. (Разрешение SELECT было отклонено в объекте log_shipping_primary_databases базы данных msdb, схема dbo). [SOAPFaultException]  
  The SELECT permission was denied on the object 'log_shipping_primary_databases', database 'msdb', schema 'dbo'. (Разрешение SELECT было отклонено в объекте log_shipping_primary_databases базы данных msdb, схема dbo).
  * Решение  
    Убедитесь, что *NT AUTHORITY\SYSTEM* имеет доступ к SQL Server. См. [примечание к SAP 2562184].


### <a name="errors-and-warnings-for-instance-validation"></a>Ошибки и предупреждения при проверке экземпляра

* Возникло исключение при проверке в пользовательском магазине HDB.  
  * См. средство просмотра журнала.  
    com.sap.nw.lm.aci.monitor.api.validation.RuntimeValidationException: Exception in validator with ID 'RuntimeHDBConnectionValidator' (Validation: 'VALIDATION_HDB_USERSTORE'): Could not retrieve the hdbuserstore (com.sap.nw.lm.aci.monitor.api.validation.RuntimeValidationException: исключение в проверяющем элементе управления с идентификатором RuntimeHDBConnectionValidator (проверка: VALIDATION_HDB_USERSTORE): не удалось получить сведения о hdbuserstore).  
    Пользовательский магазин HANA размещен в неверном расположении.
  * Решение  
    Убедитесь, что путь /usr/sap/AH1/hdbclient/install/installation.ini указан правильно.

### <a name="errors-and-warnings-during-a-system-copy"></a>Ошибки и предупреждения во время копирования системы

* Произошла ошибка при проверке шага подготовки системы.
  * Причина: com.sap.nw.lm.aci.engine.base.api.util.exception.HAOperationException Calling '/usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r' | /usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r
  * Решение  
    Создайте резервную копию всех баз данных в исходной системе HANA.

* Шаг копирования системы *Запуск* экземпляра базы данных.
  * Host Agent Operation '000D3A282BC91EE8A1D76CF1F92E2944' failed (OperationException. FaultCode: '127', Message: 'Command execution failed. (Произошел сбой операции агента узла "000D3A282BC91EE8A1D76CF1F92E2944 (OperationException. FaultCode: "127", сообщение: "Сбой при выполнении команды")). [Microsoft][ODBC SQL Server Driver][SQL Server]User does not have permission to alter database 'AS2', the database does not exist, or the database is not in a state that allows access checks' (У пользователя нет разрешения на изменение базы данных AS2, база данных не существует или база данных не находится в состоянии, позволяющем проверять доступ").
  * Решение  
    Убедитесь, что *NT AUTHORITY\SYSTEM* имеет доступ к SQL Server. См. [примечание к SAP 2562184].

### <a name="errors-and-warnings-during-a-system-clone"></a>Ошибки и предупреждения во время клонирования системы

* Произошла ошибка при попытке зарегистрировать агент экземпляра на шаге *принудительной регистрации и запуска агента экземпляра* сервера приложений или ASCS.
  * Произошла ошибка при попытке зарегистрировать агента экземпляра. (RemoteException: 'Failed to load instance data from profile '\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0':  Cannot access profile '\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0': No such file or directory.') (RemoteException: "Не удалось загрузить данные экземпляра из профиля \as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0. Невозможно получить доступ к профилю \as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0. Нет такого файла или каталога").
  * Решение  
   Убедитесь, что общий ресурс sapmnt в ASCS/SCS имеет полный доступ для SAP_AS1_GlobalAdmin.

* Ошибка на шаге *включения защиты запуска клона*.
  * Failed to open file '\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0' Cause: No such file or directory (Не удалось открыть файл "\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0". Причина: нет такого файла или каталога).
  * Решение  
    Учетной записи компьютера сервера приложений требуется право на запись в профиль.

### <a name="errors-and-warnings-during-create-system-replication"></a>Ошибки и предупреждения во время создания репликации системы

* Исключение при щелчке команды создания репликации системы
  * Причина: com.sap.nw.lm.aci.engine.base.api.util.exception.HAOperationException   Calling '/usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r' | /usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r
  * Решение  
    Проверьте, может ли sapacext быть выполнен как пользователь `<hanasid`>adm.

* Ошибка, когда полная копия не включена на шаге хранения.
  * Произошла ошибка при отправке сообщения об атрибуте контекста для пути IStorageCopyData.storageVolumeCopyList:1 и поля targetStorageSystemId.
  * Решение  
    Игнорируйте предупреждения на шаге и повторите попытку. Эта проблема будет исправлена в новом вспомогательном пакете или исправлении SAP LaMa.

### <a name="errors-and-warnings-during-relocate"></a>Ошибки и предупреждения во время перемещения

* Путь "/usr/sap/AH1" не допускается для повторного экспортирования nfs.
  * Дополнительные сведения см. в примечании к SAP [2628497].
  * Решение  
    Добавьте экспорт ASCS в профиль ASCS HostAgent. См. [примечание к SAP 2628497].

* Функция не реализуется, если перемещается ASCS.
  * Выходные данные команды: exportfs: host:/usr/sap/AX1: "Функция не реализована".
  * Решение  
    Убедитесь, что служба сервера NFS включена на целевой виртуальной машине, куда выполнено перемещение.

### <a name="errors-and-warnings-during-application-server-installation"></a>Ошибки и предупреждения во время установки сервера приложений

* Ошибка при выполнении шага SAPinst: getProfileDir.
  * Ошибка. Последняя ошибка, которая произошла на шаге: Caught ESAPinstException in module call: Validator of step '|NW_DI|ind|ind|ind|ind|0|0|NW_GetSidFromProfiles|ind|ind|ind|ind|getSid|0|NW_readProfileDir|ind|ind|ind|ind|readProfile|0|getProfileDir' reported an error: Node \\\as1-ascs\sapmnt\AS1\SYS\profile does not exist (Перехвачено исключение ESAPinstException в вызове модуля. Проверяющий элемент управления шага "|NW_DI|ind|ind|ind|ind|0|0|NW_GetSidFromProfiles|ind|ind|ind|ind|getSid|0|NW_readProfileDir|ind|ind|ind|ind|readProfile|0|getProfileDir" сообщил об ошибке: узел \\as1-ascs\sapmnt\AS1\SYS\profile не существует). Start SAPinst in interactive mode to solve this problem (Запустите SAPinst в интерактивном режиме, чтобы решить эту проблему).
  * Решение  
    Убедитесь, что SWPM запущен от имени пользователя, имеющего доступ к профилю. Этот пользователь может быть настроен в мастере установки сервера приложений.

* Ошибка при выполнении шага SAPinst: askUnicode
  * Ошибка. Последняя ошибка, которая произошла на шаге: Caught ESAPinstException in module call: Validator of step '|NW_DI|ind|ind|ind|ind|0|0|NW_GetSidFromProfiles|ind|ind|ind|ind|getSid|0|NW_getUnicode|ind|ind|ind|ind|unicode|0|askUnicode' reported an error: Start SAPinst in interactive mode to solve this problem (Перехвачено исключение ESAPinstException в вызове модуля. Проверяющий элемент управления шага "|NW_DI|ind|ind|ind|ind|0|0|NW_GetSidFromProfiles|ind|ind|ind|ind|getSid|0|NW_getUnicode|ind|ind|ind|ind|unicode|0|askUnicode" сообщил об ошибке. Запустите SAPinst в интерактивном режиме, чтобы решить эту проблему).
  * Решение  
    Если вы используете последнее ядро ​​SAP, SWPM не может определить, является ли это система Юникода, используя сервер сообщений ASCS. Дополнительные сведения см. в примечании к SAP [2445033].  
    Эта проблема будет исправлена в новом вспомогательном пакете или исправлении SAP LaMa.  
    Задайте параметр профиля OS_UNICODE=uc в профиле по умолчанию вашей системы SAP, чтобы решить эту проблему.

* Ошибка при выполнении шага SAPinst: dCheckGivenServer
  * Ошибка при выполнении шага SAPinst: Дчеккгивенсервер "Version =" 1.0 "ERROR: (последняя ошибка, сообщаемая шагом: \<p> Установка отменена пользователем. \</p>
  * Решение  
    Убедитесь, что SWPM запущен от имени пользователя, имеющего доступ к профилю. Этот пользователь может быть настроен в мастере установки сервера приложений.

* Ошибка при выполнении шага SAPinst: checkClient
  * Ошибка при выполнении шага SAPinst: Чеккклиент "Version =" 1.0 "ERROR: (последняя ошибка, сообщаемая шагом: \<p> Установка отменена пользователем. \</p>)
  * Решение  
    Убедитесь, что драйвер Microsoft ODBC Driver for SQL Server установлен на виртуальной машине, на которой требуется установить сервер приложений.

* Ошибка при выполнении шага SAPinst: copyScripts
  * Последняя ошибка, о которой сообщается на шаге: "Ошибка при системном вызове". Подробности: Error 13 (0x0000000d) (Permission denied) in execution of system call 'fopenU' with parameter (\\\as1-ascs/sapmnt/AS1/SYS/exe/uc/NTAMD64/strdbs.cmd, w), line (494) in file (\bas/bas/749_REL/bc_749_REL/src/ins/SAPINST/impl/src/syslib/filesystem/syxxcfstrm2.cpp), stack trace: (Ошибка 13 (0x0000000d) (Отказано в доступе) при выполнении системного вызова fopenU с параметром (\\as1-ascs/sapmnt/AS1/SYS/exe/uc/NTAMD64/strdbs.cmd, w), строка (494) в файле (\bas/bas/749_REL/bc_749_REL/src/ins/SAPINST/impl/src/syslib/filesystem/syxxcfstrm2.cpp), трассировка стека:)  
  CThrThread.cpp: 85: CThrThread::threadFunction()  
  CSiServiceSet.cpp: 63: CSiServiceSet::executeService()  
  CSiStepExecute.cpp: 913: CSiStepExecute::execute()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  iaxxcfile.cpp: 183: iastring CIaOsFileConnect::callMemberFunction (iastring const & имя, args_t const & args)  
  iaxxcfile.cpp: 1849: iastring CIaOsFileConnect::newFileStream (args_t const & _args)  
  iaxxbfile.cpp: 773: CIaOsFile::newFileStream_impl(4)  
  syxxcfile.cpp: 233: CSyFileImpl::openStream(ISyFile::eFileOpenMode)  
  syxxcfstrm.cpp: 29: CSyFileStreamImpl::CSyFileStreamImpl(CSyFileStream*,iastring,ISyFile::eFileOpenMode)  
  syxxcfstrm.cpp: 265: CSyFileStreamImpl::open()  
  syxxcfstrm2.cpp: 58: CSyFileStream2Impl::CSyFileStream2Impl(const CSyPath & \\\aw1-ascs/sapmnt/AW1/SYS/exe/uc/NTAMD64/strdbs.cmd, 0x4)  
  syxxcfstrm2.cpp: 456: CSyFileStream2Impl::open()
  * Решение  
    Убедитесь, что SWPM запущен от имени пользователя, имеющего доступ к профилю. Этот пользователь может быть настроен в мастере установки сервера приложений.

* Ошибка при выполнении шага SAPinst: askPasswords
  * Последняя ошибка, о которой сообщается на шаге: "Ошибка при системном вызове". Подробные сведения: Error 5 (0x00000005) (Access is denied.) in execution of system call 'NetValidatePasswordPolicy' with parameter (...), line (359) in file (\bas/bas/749_REL/bc_749_REL/src/ins/SAPINST/impl/src/syslib/account/synxcaccmg.cpp), stack trace: (Ошибка 5 (0x00000005) (В доступе отказано) при выполнении системного вызова NetValidatePasswordPolicy с параметром (...), строка (359) в файле (\bas/bas/749_REL/bc_749_REL/src/ins/SAPINST/impl/src/syslib/account/synxcaccmg.cpp), трассировка стека:)  
  CThrThread.cpp: 85: CThrThread::threadFunction()  
  CSiServiceSet.cpp: 63: CSiServiceSet::executeService()  
  CSiStepExecute.cpp: 913: CSiStepExecute::execute()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  CSiStepExecute.cpp: 764: CSiStepExecute::invokeDialog()  
  DarkModeGuiEngine.cpp: 56: DarkModeGuiEngine::showDialogCalledByJs()  
  DarkModeDialog.cpp: 85: DarkModeDialog::submit()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  iaxxcaccount.cpp: 107: iastring CIaOsAccountConnect::callMemberFunction(iastring const & name, args_t const & args)  
  iaxxcaccount.cpp: 1186: iastring CIaOsAccountConnect::validatePasswordPolicy(args_t const & _args)  
  iaxxbaccount.cpp: 430: CIaOsAccount::validatePasswordPolicy_impl()  
  synxcaccmg.cpp: 297: ISyAccountMgt::PasswordValidationMessage CSyAccountMgtImpl::validatePasswordPolicy(saponazure,*****) const )
  * Решение  
    Не забудьте добавить правило узла на шаге *изоляции* для подключения виртуальной машины к контроллеру домена.

## <a name="next-steps"></a>Дальнейшие действия
* [Руководство по использованию SAP HANA в Azure][hana-ops-guide]
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
