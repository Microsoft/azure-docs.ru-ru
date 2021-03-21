---
title: Создание пула узлов Виртуального рабочего стола Windows с помощью PowerShell (Azure)
description: Сведения о том, как создать пул узлов в Виртуальном рабочем столе Windows с использованием командлетов PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 10/02/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 9ec900f0537030d3ed0d1c875e8125806159bd51
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98251460"
---
# <a name="create-a-windows-virtual-desktop-host-pool-with-powershell"></a>Создание пула узлов виртуальных рабочих столов Windows с помощью PowerShell

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows с объектами Azure Resource Manager для Виртуального рабочего стола Windows. Если вы используете Виртуальный рабочий стол Windows (классический) без объектов Azure Resource Manager, ознакомьтесь с [этой статьей](./virtual-desktop-fall-2019/create-host-pools-powershell-2019.md).

Пулы узлов — это коллекция, состоящая из одной или нескольких идентичных виртуальных машин в средах клиента Виртуального рабочего стола Windows. Каждый пул узлов может быть связан с несколькими группами RemoteApp, одной группой классических приложений и несколькими узлами сеансов.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что вы уже выполнили инструкции по [настройке модуля PowerShell](powershell-module.md).

## <a name="use-your-powershell-client-to-create-a-host-pool"></a>Создание пула узлов с помощью клиента PowerShell

Выполните следующий командлет, чтобы войти в среду Виртуального рабочего стола Windows:

```powershell
New-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -WorkspaceName <workspacename> -HostPoolType <Pooled|Personal> -LoadBalancerType <BreadthFirst|DepthFirst|Persistent> -Location <region> -DesktopAppGroupName <appgroupname>
```

Этот командлет создает пул узлов, рабочую область и группу классических приложений. Также он регистрирует группу классических приложений в рабочей области. Вы можете создать новую рабочую область с помощью того же командлета или использовать имеющуюся.

Выполните следующий командлет, чтобы создать маркер регистрации, который разрешит узлу сеансов подключиться к пулу узлов, и сохранить его в новый файл на локальном компьютере. Срок действия маркера регистрации можно указать с помощью параметра -ExpirationHours.

>[!NOTE]
>Срок действия маркера не может быть меньше часа или больше одного месяца. Если вы укажете значение *-ExpirationTime* за пределами этих лимитов, командлет не создаст маркер.

```powershell
New-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
```

Например, если вы хотите создать маркер со сроком действия два часа, выполните следующий командлет:

```powershell
New-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> -ExpirationTime $((get-date).ToUniversalTime().AddHours(2).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
```

После этого выполните следующий командлет, чтобы добавить пользователей Azure Active Directory в группу классических приложений по умолчанию для пула узлов.

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <hostpoolname+"-DAG"> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Выполните следующий командлет, чтобы добавить группы пользователей Azure Active Directory в группу классических приложений по умолчанию для пула узлов:

```powershell
New-AzRoleAssignment -ObjectId <usergroupobjectid> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <hostpoolname+"-DAG"> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Выполните следующий командлет, чтобы экспортировать маркер регистрации в переменную, которая потребуется нам позже при [регистрации виртуальных машин в пуле узлов Виртуального рабочего стола Windows](#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool).

```powershell
$token = Get-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname>
```

## <a name="create-virtual-machines-for-the-host-pool"></a>Создание виртуальных машин для пула узлов

Теперь можно создать виртуальную машину Azure и присоединить ее к пулу узлов Виртуального рабочего стола Windows.

Создать виртуальную машину можно несколькими способами:

- [Создание виртуальной машины из образа коллекции Azure](../virtual-machines/windows/quick-create-portal.md#create-virtual-machine)
- [Создание виртуальной машины из управляемого образа](../virtual-machines/windows/create-vm-generalized-managed.md)
- [Создание виртуальной машины из неуправляемого образа](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-user-image-data-disks)

>[!NOTE]
>Если вы развертываете виртуальную машину с Windows 7 в качестве ОС узла, процесс создания и развертывания будет немного отличаться. Подробные сведения см. в статье [Развертывание виртуальной машины Windows 7 в службе "Виртуальный рабочий стол Windows"](./virtual-desktop-fall-2019/deploy-windows-7-virtual-machine.md).

После создания виртуальных машин для узла сеансов [примените лицензию Windows к виртуальной машине узла сеансов](./apply-windows-license.md#apply-a-windows-license-to-a-session-host-vm), чтобы не оплачивать дополнительную лицензию для работы виртуальных машин Windows или Windows Server.

## <a name="prepare-the-virtual-machines-for-windows-virtual-desktop-agent-installations"></a>Подготовка виртуальных машин к установке агентов Виртуального рабочего стола Windows

Чтобы подготовить виртуальные машины к установке агентов Виртуального рабочего стола Windows и зарегистрировать эти виртуальные машины в пуле узлов Виртуального рабочего стола Windows, выполните следующие действия.

- Присоедините компьютер к домену. Это позволит сопоставлять для пользователей, входящих в Виртуальный рабочий стол Windows, учетные записи Azure Active Directory с учетными записями Active Directory и успешно разрешать доступ к виртуальной машине.
- Если виртуальная машина работает под управлением ОС Windows Server, необходимо установить роль узла сеансов удаленного рабочего стола. Эта роль позволяет правильно установить агенты Виртуального рабочего стола Windows.

Чтобы успешно присоединиться к домену, выполните следующие действия на каждой виртуальной машине.

1. [Подключитесь к виртуальной машине](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine), указав учетные данные, которые вы использовали при создании этой виртуальной машины.
2. На виртуальной машине откройте **Панель управления** и выберите **Система**.
3. Выберите **Имя компьютера**, щелкните **Изменить параметры** и выберите **Изменить**.
4. Выберите **Домен** и введите имя домена Active Directory в этой виртуальной сети.
5. Выполните аутентификацию с учетной записью домена, которая имеет права на присоединение компьютеров к домену.

    >[!NOTE]
    > Если вы присоединяете виртуальные машины к среде доменных служб Active Directory (Azure AD DS), убедитесь, что пользователь, который присоединяется к домену, также является участником [группы администраторов AAD DC](../active-directory-domain-services/tutorial-create-instance-advanced.md#configure-an-administrative-group).

>[!IMPORTANT]
>Не рекомендуется включать политики или конфигурации, которые отключают установщик Windows. Если отключить установщик Windows, служба не сможет установить обновления агента на узлах сеансов, и узлы сеансов будут работать неправильно.

## <a name="register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool"></a>Регистрация виртуальных машин в пуле узлов Виртуального рабочего стола Windows

Этот процесс ничем не сложнее установки агентов Виртуального рабочего стола Windows.

Чтобы зарегистрировать агенты Виртуального рабочего стола Windows, выполните следующие действия на каждой виртуальной машине.

1. [Подключитесь к виртуальной машине](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine), указав учетные данные, которые вы использовали при создании этой виртуальной машины.
2. Скачайте и установите агент Виртуального рабочего стола Windows.
   - Скачайте [агент Виртуального рабочего стола Windows](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv).
   - Запустите установщик. Когда установщик запросит у вас маркер регистрации, введите значение, полученное от командлета **Get-азввдрегистратионинфо** .
3. Скачайте и установите загрузчик агента Виртуального рабочего стола Windows.
   - Скачайте [загрузчик агента Виртуального рабочего стола Windows](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH).
   - Запустите установщик.

>[!IMPORTANT]
>Чтобы усилить защиту среды Виртуального рабочего стола Windows в Azure, мы рекомендуем не открывать входящий порт 3389 для виртуальных машин. Виртуальный рабочий стол Windows не требует открытия входящего порта 3389, чтобы пользователи могли получить доступ к виртуальным машинам пула узла. Если вам все же нужно открыть порт 3389 для устранения неполадок, мы рекомендуем использовать[JIT-доступ к виртуальным машинам](../security-center/security-center-just-in-time.md). Мы также рекомендуем не назначать виртуальным машинам общедоступные IP-адреса.

## <a name="update-the-agent"></a>Обновление агента

Необходимо обновить агент, если вы используете одну из следующих ситуаций:

- Вы хотите перенести ранее зарегистрированный узел сеанса в новый пул узлов
- Узел сеанса не отображается в пуле узлов после обновления

Чтобы обновить агент, выполните следующие действия.

1. Войдите на виртуальную машину с правами администратора.
2. Перейдите в раздел **службы**, а затем завершите процессы **загрузчика агента** **рдажент** и удаленный рабочий стол.
3. Затем найдите агент и загрузчик MSI. Они либо находятся в папке **к:\деплойажент** , либо в любом месте, в котором он был сохранен при установке.
4. Найдите следующие файлы и удалите их:
     
     - Microsoft. Рдинфра. Рдажент. Installer-x64-веркс. x. x
     - Microsoft. Рдинфра. Рдажентбутлоадер. Installer — x64

   Чтобы удалить эти файлы, щелкните правой кнопкой мыши имя каждого файла, а затем выберите **Удалить**.
5. При необходимости можно также удалить следующие параметры реестра:
     
     - Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\RDInfraAgent
     - Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\RDAgentBootLoader

6. После удаления этих элементов необходимо удалить все связи со старым пулом узлов. Если вы хотите повторно зарегистрировать этот узел в службе, следуйте инструкциям в разделе [регистрация виртуальных машин в пуле узлов виртуальных рабочих столов Windows](create-host-pools-powershell.md#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool).


## <a name="next-steps"></a>Дальнейшие действия

Теперь пул узлов готов, и вы можете заполнить его удаленными приложениями RemoteApp. Дополнительные сведения о том, как управлять приложениями в Виртуальном рабочем столе Windows см. в руководстве по управлению группами приложений.

> [!div class="nextstepaction"]
> [Управление группами приложений](./manage-app-groups.md)
