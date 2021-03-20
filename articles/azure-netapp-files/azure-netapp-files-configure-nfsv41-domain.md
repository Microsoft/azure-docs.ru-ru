---
title: Настройка домена Нфсв 4.1 по умолчанию для Azure NetApp Files | Документация Майкрософт
description: Описание настройки клиента NFS для использования Нфсв 4.1 с Azure NetApp Files.
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
ms.date: 10/14/2020
ms.author: b-juche
ms.openlocfilehash: c3c853190d5f63bbe9012727d8b7b7ac91da135f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92072158"
---
# <a name="configure-nfsv41-default-domain-for-azure-netapp-files"></a>Настройка домена по умолчанию NFSv4.1 для Azure NetApp Files

В NFSv4 введена концепция домена проверки подлинности. В настоящее время Azure NetApp Files поддерживает только корневое сопоставление пользователей из службы с клиентом NFS. Чтобы использовать функциональность Нфсв 4.1 с Azure NetApp Files, необходимо обновить клиент NFS.

## <a name="default-behavior-of-usergroup-mapping"></a>Поведение по умолчанию при сопоставлении пользователей и групп

Корневое сопоставление по умолчанию `nobody` имеет значение User, так как домен NFSv4 по `localdomain` умолчанию установлен в. При подключении тома Azure NetApp Files Нфсв 4.1 в качестве привилегированного пользователя вы увидите разрешения для файлов следующим образом:  

![Поведение по умолчанию сопоставления пользователей и групп для Нфсв 4.1](../media/azure-netapp-files/azure-netapp-files-nfsv41-default-behavior-user-group-mapping.png)

Как показано в приведенном выше примере, пользователь `file1` должен быть `root` , но по умолчанию сопоставлен с ним `nobody` .  В этой статье показано, как задать `file1` пользователю `root` , изменив `idmap Domain` значение параметра на `defaultv4iddomain.com` .  

## <a name="steps"></a>Шаги 

1. Измените `/etc/idmapd.conf` файл на клиенте NFS.   
    Раскомментируйте строку `#Domain` (т. е. Удалите `#` из строки) и измените значение `localdomain` на `defaultv4iddomain.com` . 

    Начальная конфигурация: 
    
    ![Начальная конфигурация для Нфсв 4.1](../media/azure-netapp-files/azure-netapp-files-nfsv41-initial-config.png)

    Обновленная конфигурация:
    
    ![Обновленная конфигурация для Нфсв 4.1](../media/azure-netapp-files/azure-netapp-files-nfsv41-updated-config.png)

2. Отключите все подключенные в данный момент тома NFS.
3. Обновите файл `/etc/idmapd.conf`.
4. Перезапустите `rpcbind` службу на узле ( `service rpcbind restart` ) или просто перезагрузите узел.
5. При необходимости подключите тома NFS.   

    См. раздел [подключение или отключение тома для виртуальных машин Windows или Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md). 

В следующем примере показан результат изменения пользователя или группы: 

![Снимок экрана, на котором показан пример итогового изменения пользователя или группы.](../media/azure-netapp-files/azure-netapp-files-nfsv41-resulting-config.png)

Как показано в примере, пользователь или группа теперь изменились с `nobody` на `root` .

## <a name="behavior-of-other-non-root-users-and-groups"></a>Поведение других (не корневых) пользователей и групп

Azure NetApp Files поддерживает локальных пользователей (пользователей, созданных локально на узле), имеющих разрешения, связанные с файлами или папками в томах Нфсв 4.1. Однако служба в настоящее время не поддерживает сопоставление пользователей и групп между несколькими узлами. Таким образом, пользователи, созданные на одном узле, по умолчанию не сопоставляются с пользователями, созданными на другом узле. 

В следующем примере `Host1` имеются три учетные записи тестовых пользователей ( `testuser01` , `testuser02` , `testuser03` ): 

![Снимок экрана, на котором показано, что у host1 есть три учетные записи тестовых пользователей.](../media/azure-netapp-files/azure-netapp-files-nfsv41-host1-users.png)

`Host2`Обратите внимание, что учетные записи тестовых пользователей не были созданы, но один и тот же том подключен на обоих узлах:

![Результирующая конфигурация для Нфсв 4.1](../media/azure-netapp-files/azure-netapp-files-nfsv41-host2-users.png)

## <a name="next-step"></a>Следующий шаг 

[Подключение или отключение тома для виртуальных машин Windows или Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)

