---
title: Настройка шифрования Kerberos Нфсв 4.1 для Azure NetApp Files | Документация Майкрософт
description: Описание настройки шифрования Нфсв 4.1 Kerberos для Azure NetApp Files и влияния на производительность.
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
ms.date: 02/18/2021
ms.author: b-juche
ms.openlocfilehash: 6ff87d046c60f588e133010895ec3e7ce08cb71f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101740568"
---
# <a name="configure-nfsv41-kerberos-encryption-for-azure-netapp-files"></a>Настройка шифрования Kerberos NFS версии 4.1 для Azure NetApp Files

Azure NetApp Files поддерживает шифрование клиентов NFS в режимах Kerberos (krb5, krb5i и krb5p) с шифрованием AES-256. В этой статье описываются необходимые конфигурации для использования тома Нфсв 4.1 с шифрованием Kerberos.

## <a name="requirements"></a>Требования

К шифрованию клиента Нфсв 4.1 применяются следующие требования. 

* Подключение домен Active Directory Services (AD DS) для упрощения работы с билетами Kerberos 
* Создание записей A/PTR DNS для клиентских и Azure NetApp Filesных IP-адресов сервера NFS
* Клиент Linux  
    В этой статье приводятся рекомендации для клиентов RHEL и Ubuntu.  Другие клиенты будут работать с аналогичными действиями по настройке. 
* Доступ к NTP-серверу  
    Можно использовать один из часто используемых контроллеров домена контроллера домен Active Directory (AD DC).

## <a name="create-an-nfs-kerberos-volume"></a>Создание тома Kerberos NFS

1.  Выполните действия, описанные в статье [Создание тома NFS для Azure NetApp Files](azure-netapp-files-create-volumes.md) , чтобы создать том нфсв 4.1.   

    На странице Создание тома задайте для параметра версия NFS значение **нфсв 4.1** и установите для параметра Kerberos значение **включено**.

    > [!IMPORTANT] 
    > Вы не можете изменить выбор включения Kerberos после создания тома.

    ![Создание тома Kerberos Нфсв 4.1](../media/azure-netapp-files/create-kerberos-volume.png)  

2. Выберите **Экспорт политики** в соответствии с требуемым уровнем доступа и параметром безопасности (Kerberos 5, Kerberos 5i или Kerberos 5p) для тома.   

    Сведения о влиянии Kerberos на производительность см. [в статье влияние Kerberos на нфсв 4.1](#kerberos_performance).  

    Можно также изменить методы безопасности Kerberos для тома, нажав кнопку Экспорт политики в области навигации Azure NetApp Files.

3.  Щелкните **Проверка + создать** , чтобы создать том нфсв 4.1.

## <a name="configure-the-azure-portal"></a>Настройка портала Azure 

1.  Следуйте инструкциям в разделе [Создание подключения Active Directory](create-active-directory-connections.md).  

    Для использования протокола Kerberos необходимо создать хотя бы одну учетную запись компьютера в Active Directory. Предоставленные сведения об учетной записи используются для создания учетных записей для томов Kerberos *и* нфсв 4.1. Учетная запись этого компьютера создается автоматически во время создания тома.

2.  В разделе **область Kerberos** введите **имя сервера AD** и **IP-адрес KDC** .

    Сервер AD и IP-адрес KDC могут быть одним и тем же сервером. Эти сведения используются для создания учетной записи компьютера-участника-службы, используемой Azure NetApp Files. После создания учетной записи компьютера Azure NetApp Files будет использовать записи DNS-сервера для нахождение дополнительных серверов KDC по мере необходимости. 

    ![Область Kerberos](../media/azure-netapp-files/kerberos-realm.png)
 
3.  Нажмите кнопку **присоединить** , чтобы сохранить конфигурацию.

## <a name="configure-active-directory-connection"></a>Настройка подключения Active Directory 

Конфигурация Нфсв 4.1 Kerberos создает две учетные записи компьютеров в Active Directory:
* Учетная запись компьютера для общих ресурсов SMB
* Учетная запись компьютера для Нфсв 4.1 — вы можете указать эту учетную запись с помощью префикса `NFS-` . 

После создания первого тома Kerberos Нфсв 4.1 задайте тип шифрования для учетной записи компьютера с помощью следующей команды PowerShell:

`Set-ADComputer $NFSCOMPUTERACCOUNT -KerberosEncryptionType AES256`

## <a name="configure-the-nfs-client"></a>Настройка клиента NFS 

Следуйте инструкциям в разделе [Настройка клиента NFS для Azure NetApp Files](configure-nfs-clients.md) , чтобы настроить клиент NFS.  

## <a name="mount-the-nfs-kerberos-volume"></a><a name="kerberos_mount"></a>Подключение тома Kerberos NFS

1. На странице **тома** выберите том NFS, который требуется подключить.

2. Выберите **инструкции по подключению** тома, чтобы отобразить инструкции.

    Пример: 

    ![Инструкции по подключению для томов Kerberos](../media/azure-netapp-files/mount-instructions-kerberos-volume.png)  

3. Создайте каталог (точку подключения) для нового тома.  

4. Задайте для параметра Тип шифрования по умолчанию значение AES 256 для учетной записи компьютера:  
    `Set-ADComputer $NFSCOMPUTERACCOUNT -KerberosEncryptionType AES256 -Credential $ANFSERVICEACCOUNT`

    * Эту команду необходимо выполнить только один раз для каждой учетной записи компьютера.
    * Эту команду можно выполнить из контроллера домена или с компьютера с установленным [RSAT](https://support.microsoft.com/help/2693643/remote-server-administration-tools-rsat-for-windows-operating-systems) . 
    * `$NFSCOMPUTERACCOUNT`Переменная — это учетная запись компьютера, созданная в Active Directory при развертывании тома Kerberos. Это учетная запись с префиксом `NFS-` . 
    * `$ANFSERVICEACCOUNT`Переменная является непривилегированной Active Directory учетной записью пользователя с делегированными элементами управления в подразделении, в котором создана учетная запись компьютера. 

5. Подключите том на узле: 

    `sudo mount -t nfs -o sec=krb5p,rw,hard,rsize=1048576,wsize=1048576,vers=4.1,tcp $ANFEXPORT $ANFMOUNTPOINT`

    * `$ANFEXPORT`Переменная является путем, `host:/export` найденным в инструкциях по подключению.
    * `$ANFMOUNTPOINT`Переменная представляет собой папку, созданную пользователем на узле Linux.

## <a name="performance-impact-of-kerberos-on-nfsv41"></a><a name="kerberos_performance"></a>Влияние Kerberos на производительность NFSv4.1 

Необходимо ознакомиться с возможностями безопасности, доступными для томов Нфсв 4.1, протестированными векторами производительности и ожидаемым влиянием Kerberos на производительность. Дополнительные сведения см. [в статье влияние Kerberos на тома нфсв 4.1 на производительность](performance-impact-kerberos.md) .  

## <a name="next-steps"></a>Дальнейшие действия  

* [Влияние Kerberos на тома Нфсв 4.1 на производительность](performance-impact-kerberos.md)
* [Устранение проблем с Томом Нфсв 4.1 Kerberos](troubleshoot-nfsv41-kerberos-volumes.md)
* [Часто задаваемые вопросы о Azure NetApp Files](azure-netapp-files-faqs.md)
* [Создание тома NFS для Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Создание подключения Active Directory](create-active-directory-connections.md)
* [Настройка клиента NFS для Azure NetApp Files](configure-nfs-clients.md) 
