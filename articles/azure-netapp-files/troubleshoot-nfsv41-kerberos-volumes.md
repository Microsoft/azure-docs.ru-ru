---
title: Устранение проблем с Томом Нфсв 4.1 Kerberos для Azure NetApp Files | Документация Майкрософт
description: Описывает сообщения об ошибках и способы их устранения, которые могут помочь при устранении неполадок с Томом Нфсв 4.1 Kerberos для Azure NetApp Files.
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
ms.topic: troubleshooting
ms.date: 01/12/2021
ms.author: b-juche
ms.openlocfilehash: 638607da02b1db4842cdc04f86a4fed1860c243f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98134319"
---
# <a name="troubleshoot-nfsv41-kerberos-volume-issues"></a>Устранение проблем с Томом Нфсв 4.1 Kerberos 

В этой статье описываются способы устранения ошибок, которые могут возникнуть при создании томов Нфсв 4.1 Kerberos и управлении ими. 

## <a name="error-conditions-and-resolutions"></a>Условия и способы устранения ошибок

|     Условия ошибок    |     Способы устранения    |
|-|-|
|`Error allocating volume - Export policy rules does not match kerberosEnabled flag` | Azure NetApp Files не поддерживает протокол Kerberos для томов NFSv3. Протокол Kerberos поддерживается только для протокола Нфсв 4.1.  |
|`This NetApp account has no configured Active Directory   connections`  |  Настройте Active Directory для учетной записи NetApp с полями **IP-адрес KDC** и **имя сервера AD**. Инструкции см. [в разделе настройка портал Azure](configure-kerberos-encryption.md#configure-the-azure-portal) . |
|`Mismatch between KerberosEnabled flag value and ExportPolicyRule's access type parameter values.`  | Azure NetApp Files не поддерживает преобразование тома обычного Нфсв 4.1 в том Kerberos Нфсв 4.1 и наоборот. |
|`mount.nfs: access denied by server when mounting volume <SMB_SERVER_NAME-XXX.DOMAIN_NAME>/<VOLUME_NAME>` <br>  Например, `smb-test-64d9.contoso.com:/nfs41-vol101`. | <ol><li> Убедитесь, что записи A/PTR правильно настроены и существуют в Active Directory для имени сервера `smb-test-64d9.contoso.com` . <br> В клиенте NFS, если `nslookup` `smb-test-64d9.contoso.com` разрешение на IP-адрес IP1 (то есть `10.1.1.68` ), то `nslookup` IP1 должно разрешаться только в одну запись (то есть `smb-test-64d9.contoso.com` ). `nslookup` IP1 не *должны* разрешаться в несколько имен. </li>  <li>Задайте AES-256 для учетной записи компьютера NFS типа `NFS-<Smb NETBIOS NAME>-<few random characters>` On AD с помощью PowerShell или пользовательского интерфейса. <br> Примеры команд. <ul><li>`Set-ADComputer <NFS_MACHINE_ACCOUNT_NAME> -KerberosEncryptionType AES256` </li><li>`Set-ADComputer NFS-SMB-TEST-64 -KerberosEncryptionType AES256` </li></ul> </li> <li>Убедитесь, что время для клиента NFS, AD и Azure NetApp Filesного программного обеспечения хранилища синхронизировано друг с другом и находится в пределах диапазона наклона в пять минут. </li>  <li>Получите билет Kerberos на клиенте NFS с помощью команды `kinit <administrator>` .</li> <li>Сократите имя узла клиента NFS до 15 символов и снова выполните присоединение к сфере. </li><li>Перезапустите клиент NFS и `rpcgssd` службу, как показано ниже. Команда может отличаться в зависимости от операционной системы.<br> RHEL 7: <br> `service nfs restart` <br> `service rpcgssd restart` <br> CentOS 8: <br> `systemctl enable nfs-client.target && systemctl start nfs-client.target` <br> Ubuntu: <br> (Перезапустите `rpc-gssd` службу.) <br> `sudo systemctl start rpc-gssd.service` </ul>| 
|`mount.nfs: an incorrect mount option was specified`   | Эта ошибка может быть связана с проблемой клиента NFS. Перезагрузите клиент NFS.    |
|`Hostname lookup failed`   | Необходимо создать зону обратного просмотра на DNS-сервере, а затем добавить запись PTR для компьютера узла Active Directory в эту зону обратного просмотра. <br> Например, предположим, что IP-адрес компьютера AD — это `10.1.1.4` , имя узла компьютера AD (как было найдено с помощью команды hostname) `AD1` , а доменным именем является `contoso.com` . Запись типа PTR, добавленная в зону обратного просмотра, должна иметь значение `10.1.1.4 -> AD1.contoso.com` . |
|`Volume creation fails due to unreachable DNS server`  | Доступны два возможных решения: <br> <ul><li> Эта ошибка означает, что служба DNS недоступна. Причиной может быть неверный IP-адрес DNS или проблемы с сетью. Проверьте IP-адрес DNS, указанный в подключении AD, и убедитесь, что IP-адрес указан правильно. </li> <li> Убедитесь, что Active Directory и том находятся в одном и том же регионе и в одной виртуальной сети. Если они находятся в разных виртуальных сетей, убедитесь, что между двумя виртуальных сетей установлен пиринг между виртуальными сетями. </li></ul> |
|Создание тома Kerberos для Нфсв 4.1 завершается ошибкой, как показано в следующем примере: <br> `Failed to enable NFS Kerberos on LIF "svm_e719cde8d6d0413fbd6adac0636cdecb_7ad0b82e_73349613". Failed to bind service principal name on LIF "svm_e719cde8d6d0413fbd6adac0636cdecb_7ad0b82e_73349613". SecD Error: server create fail join user auth.` |IP-адрес KDC неправильный, и создан том Kerberos. Обновите IP-адрес KDC с помощью правильного адреса. <br> После обновления IP-адреса KDC эта ошибка не исчезнет. Необходимо повторно создать том. |

## <a name="next-steps"></a>Дальнейшие действия  

* [Настройка шифрования Kerberos NFS версии 4.1 для Azure NetApp Files](configure-kerberos-encryption.md)
