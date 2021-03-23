---
title: Создание тома SMB для Azure NetApp Files | Документация Майкрософт
description: В этой статье показано, как создать том SMB3 в Azure NetApp Files. Узнайте о требованиях к Active Directoryным подключениям и доменным службам.
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
ms.topic: how-to
ms.date: 03/01/2021
ms.author: b-juche
ms.openlocfilehash: 6a8de9b373a14eab45df28b28bb3f94314c1d89a
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104801093"
---
# <a name="create-an-smb-volume-for-azure-netapp-files"></a>Создание тома SMB для Azure NetApp Files

Azure NetApp Files поддерживает создание томов с помощью NFS (NFSv3 и Нфсв 4.1), SMB3 или двойного протокола (NFSv3 и SMB). Потребление емкости тома зависит от подготовленной емкости пула. В этой статье показано, как создать том SMB3.

## <a name="before-you-begin"></a>Перед началом 

* Перед началом необходимо настроить пул емкости. См. раздел [Настройка пула емкости](azure-netapp-files-set-up-capacity-pool.md).     
* Подсеть должна быть делегирована службе Azure NetApp Files. См. раздел [Делегирование подсети для Azure NetApp Files](azure-netapp-files-delegate-subnet.md).

## <a name="configure-active-directory-connections"></a>Настройка подключений Active Directory 

Перед созданием тома SMB необходимо создать подключение Active Directory. Если вы еще не настроили Active Directory подключения для файлов Azure NetApp, следуйте инструкциям, описанным в статье [создание Active Directory соединений и управление ими](create-active-directory-connections.md).

## <a name="add-an-smb-volume"></a>Добавление тома SMB

1. В колонке управления пулами емкости щелкните колонку **Тома**. 

    ![Переход к томам](../media/azure-netapp-files/azure-netapp-files-navigate-to-volumes.png)

2. Чтобы создать том щелкните **+ Добавить том**.  
    Откроется окно создания тома.

3. В окне Создание тома щелкните **создать** и укажите сведения для следующих полей на вкладке "основы":   
    * **Имя тома**      
        Укажите имя создаваемого тома.   

        Имя тома должно быть уникальным в пределах каждого пула емкости. Длина имени должна быть не менее трех знаков. Имя может состоять из любых буквенно-цифровых символов.   

        Нельзя использовать `default` или `bin` в качестве имени тома.

    * **Пул емкости**  
        Укажите пул емкости, в котором нужно создать том.

    * **Квота**  
        Укажите объем логического хранилища, выделенный для тома.  

        В поле **Доступная квота** отображается объем неиспользуемого пространства выбранного пула емкости, которое можно использовать для создания нового тома. Размер нового тома не должен превышать доступную квоту.  

    * **Пропускная способность (MiB/с)**   
        Если том создается в пуле ресурсов ручного обслуживания вручную, укажите необходимую пропускную способность для тома.   

        Если том создается в пуле ресурсов автоматического качества обслуживания, в этом поле отображается значение (пропускная способность уровня обслуживания "квота x").   

    * **Виртуальная сеть**  
        Укажите виртуальную сеть Azure, из которой будет осуществляться доступ к тому.  

        В указанной виртуальной сети должна быть подсеть, делегированная службе Azure NetApp Files. Доступ к службе Azure NetApp Files можно получить только из той же виртуальной сети либо из виртуальной сети, которая находится в том же расположении, что и том, через пиринговую связь. Доступ к тому также можно получить из локальной сети через Express Route.   

    * **Подсеть**  
        Укажите подсеть, которую нужно использовать для тома.  
        Указанная подсеть должна быть делегирована службе Azure NetApp Files. 
        
        Если вы не делегировали подсеть, можно нажать **Создать** на странице "Создать том". Затем на странице "Создание подсети" укажите сведения о подсети и выберите **Microsoft.NetApp/volumes**, чтобы делегировать подсеть службе Azure NetApp Files. В каждой виртуальной сети можно делегировать только одну подсеть для Azure NetApp Files.   
 
        ![Создание тома](../media/azure-netapp-files/azure-netapp-files-new-volume.png)
    
        ![Создание подсети](../media/azure-netapp-files/azure-netapp-files-create-subnet.png)

    * Если вы хотите применить к тому существующую политику моментальных снимков, щелкните **Показать дополнительные** , чтобы развернуть его, укажите, нужно ли скрыть путь к моментальному снимку, а затем выберите политику моментальных снимков в раскрывающемся меню. 

        Дополнительные сведения о создании политики моментальных снимков см. в разделе [Управление политиками моментальных снимков](azure-netapp-files-manage-snapshots.md#manage-snapshot-policies).

        ![Отобразить расширенный выбор](../media/azure-netapp-files/volume-create-advanced-selection.png)

4. Нажмите **Протокол** и введите следующие сведения.  
    * Выберите **SMB** в качестве типа протокола тома. 
    * В раскрывающемся списке выберите подключение **Active Directory**.
    * Укажите имя общего тома в поле **Имя общего ресурса**.
    * Если вы хотите включить постоянную доступность для тома SMB, установите флажок **Включить постоянную доступность**.    

        > [!IMPORTANT]   
        > Функция непрерывной доступности SMB в настоящее время доступна в общедоступной предварительной версии. Чтобы получить доступ к этой функции, необходимо отправить запрос ваитлист на **[странице отправки общедоступной предварительной версии ваитлист общего доступа к Azure NETAPP Files SMB](https://aka.ms/anfsmbcasharespreviewsignup)**. Перед использованием функции непрерывной доступности дождитесь официального сообщения электронной почты от команды Azure NetApp Files.   
        > 
        > Постоянная доступность должна быть включена только для рабочих нагрузок SQL. Использование общих ресурсов постоянной доступности SMB для рабочих нагрузок, кроме SQL Server, *не* поддерживается. В настоящее время эта функция поддерживается в SQL Server Windows. SQL Server Linux в настоящее время не поддерживается. Если для установки SQL Server используется учетная запись без прав администратора (домена), убедитесь, что учетной записи назначена необходимая привилегия безопасности. Если учетная запись домена не имеет необходимых прав доступа ( `SeSecurityPrivilege` ), и права доступа не могут быть заданы на уровне домена, вы можете предоставить привилегию учетной записи с помощью поля " **Пользователи привилегий безопасности** " Active Directory подключений. См. раздел [Создание подключения Active Directory](create-active-directory-connections.md#create-an-active-directory-connection).

    <!-- [1/13/21] Commenting out command-based steps below, because the plan is to use form-based (URL) registration, similar to CRR feature registration -->
    <!-- 
        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSMBCAShare
        ```

        Check the status of the feature registration: 

        > [!NOTE]
        > The **RegistrationState** may be in the `Registering` state for up to 60 minutes before changing to`Registered`. Wait until the status is `Registered` before continuing.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSMBCAShare
        ```
        
        You can also use [Azure CLI commands](/cli/azure/feature?preserve-view=true&view=azure-cli-latest) `az feature register` and `az feature show` to register the feature and display the registration status. 
    --> 

    ![Снимок экрана, описывающий вкладку протокола создания тома SMB.](../media/azure-netapp-files/azure-netapp-files-protocol-smb.png)

5. Нажмите **Проверка и создание**, чтобы ознакомиться с сведениями о томе.  Затем нажмите кнопку **Создать**, чтобы создать том SMB.

    Созданный том появится на странице "Тома". 
 
    От пула емкости том наследует атрибуты подписки, группы ресурсов и расположения. Состояние развертывания можно отслеживать на вкладке уведомлений.

## <a name="control-access-to-an-smb-volume"></a>Управление доступом к тому SMB  

Управление доступом к тому SMB осуществляется с помощью разрешений.  

### <a name="share-permissions"></a>Разрешения для общего ресурса  

По умолчанию новый том имеет разрешения **"Все" и "Полный доступ"** для общего доступа. Члены группы "Администраторы домена" могут изменять разрешения на общий доступ, используя компонент "Управление компьютером" в учетной записи компьютера, используемой для тома Azure NetApp Files.

![Путь подключения SMB](../media/azure-netapp-files/smb-mount-path.png) 
![Задание разрешений для общего ресурса](../media/azure-netapp-files/set-share-permissions.png) 

### <a name="ntfs-file-and-folder-permissions"></a>Разрешения для файлов и папок NTFS  

Разрешения для файла или папки можно задать на вкладке **Безопасность** свойств объекта в клиенте SMB Windows.
 
![Задание разрешений для файлов и папок](../media/azure-netapp-files/set-file-folder-permissions.png) 

## <a name="next-steps"></a>Дальнейшие действия  

* [Подключение или отключение тома для виртуальных машин Windows или Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)
* [Ограничения ресурсов для службы Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Часто задаваемые вопросы о SMB](./azure-netapp-files-faqs.md#smb-faqs)
* [Устранение неполадок с томами с двумя протоколами или SMB](troubleshoot-dual-protocol-volumes.md)
* [Узнайте об интеграции виртуальной сети для служб Azure](../virtual-network/virtual-network-for-azure-services.md)
* [Установка нового леса Active Directory с помощью Azure CLI](/windows-server/identity/ad-ds/deploy/virtual-dc/adds-on-azure-vm)
