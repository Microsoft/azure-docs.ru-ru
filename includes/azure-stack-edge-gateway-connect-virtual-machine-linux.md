---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/03/2020
ms.author: alkohli
ms.openlocfilehash: b3d4ec54d6db88a04f7aca46c0c96fa2d4d17ac7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101730638"
---
Подключитесь к виртуальной машине с помощью частного IP-адреса, переданного во время создания виртуальной машины.

1. Откройте сеанс SSH для подключения с IP-адресом.

   `ssh -l <username> <ip address>`

1. В командной строке введите пароль, который использовался при создании виртуальной машины.

   Если необходимо указать ключ SSH, используйте эту команду.

   `ssh -i c:/users/Administrator/.ssh/id_rsa Administrator@5.5.41.236`

   Ниже приведен пример выходных данных при подключении к виртуальной машине.

    ```powershell
    PS C:\07-30-2020\linux> ssh -l Administrator 10.126.68.186
    Administrator@10.126.68.186's password:
    Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 5.0.0-1027-azure x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
      System information as of Thu Jul 30 22:56:11 UTC 2020
    
      System load:  0.0               Processes:           105
      Usage of /:   5.6% of 28.90GB   Users logged in:     0
      Memory usage: 12%               IP address for eth0: 10.126.68.186
      Swap usage:   0%
    
     * Are you ready for Kubernetes 1.19? It's nearly here! Try RC3 with
       sudo snap install microk8s --channel=1.19/candidate --classic
    
       https://www.microk8s.io/ has docs and details.
    
    68 packages can be updated.
    0 updates are security updates.
    
    
    *** System restart required ***
    
    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.
    
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.
    
    Administrator@mylinuxvm:
    ```

1. Если во время создания виртуальной машины вы использовали общедоступный IP-адрес, то для подключения к виртуальной машине его можно использовать. Чтобы получить общедоступный IP-адрес, выполните следующую команду: 

   ```powershell
   $publicIp = Get-AzureRmPublicIpAddress -Name <Public IP> -ResourceGroupName <Resource group name>
   ```

   В этом случае общедоступный IP-адрес совпадает с частным IP-адресом, который был передан во время создания виртуального сетевого интерфейса.
