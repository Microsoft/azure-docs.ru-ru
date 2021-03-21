---
title: Настройка клиента NFS для Azure NetApp Files | Документация Майкрософт
description: Описание настройки клиентов NFS для использования с Azure NetApp Files.
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
ms.date: 11/09/2020
ms.author: b-juche
ms.openlocfilehash: c1cdeaa41dda11f2ab520cf8d31ddb2116587082
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94409575"
---
# <a name="configure-an-nfs-client-for-azure-netapp-files"></a>Настройка клиента NFS для Azure NetApp Files

Конфигурация клиента NFS, описанная в этой статье, является частью программы установки при [настройке шифрования нфсв 4.1 Kerberos](configure-kerberos-encryption.md) или [создании тома с двумя протоколами](create-volumes-dual-protocol.md). Для использования с Azure NetApp Files доступен широкий спектр дистрибутивов Linux. В этой статье описываются конфигурации для двух наиболее часто используемых сред: RHEL 8 и Ubuntu 18,04. 

Независимо от используемой версии Linux требуются следующие конфигурации:
* Настройте NTP-клиент, чтобы избежать проблем с отклонением во времени.
* Настройка записей DNS клиента Linux для разрешения имен.  
    Эта конфигурация должна включать запись "A" (переадресация) и запись PTR (обратная). 
* Для присоединение к домену создайте учетную запись компьютера для клиента Linux в целевом Active Directory (который создается во время выполнения команды JOIN в сфере). 
    > [!NOTE] 
    > `$SERVICEACCOUNT`Переменная, используемая в приведенных ниже командах, должна быть учетной записью пользователя с разрешениями или делегированием для создания учетной записи компьютера в целевом подразделении.

## <a name="rhel-8-configuration"></a>Конфигурация RHEL 8 

В этом разделе описываются конфигурации RHEL, необходимые для шифрования Нфсв 4.1 Kerberos и двойного протокола.  

В примерах этого раздела используется следующее доменное имя и IP-адрес:  

* Имя домена: `contoso.com`
* Частный IP-адрес: `10.6.1.4`

### <a name="rhel-8-configuration-if-you-are-using-nfsv41-kerberos-encryption"></a><a name="rhel8_nfsv41_kerberos"></a>Конфигурация RHEL 8 при использовании шифрования Kerberos версии Нфсв 4.1

1. Настройте `/etc/resolv.conf` с помощью соответствующего DNS-сервера.  

    Пример:  

    `[root@reddoc cbs]# cat /etc/resolv.conf`   
    `search contoso.com`   
    `nameserver 10.6.1.4(private IP)`   

2. Добавьте запись клиента NFS на DNS-сервере для зоны прямого и обратного просмотра DNS.

3. Чтобы проверить DNS, используйте следующие команды из клиента NFS:   

    `# nslookup [hostname/FQDN of NFS client(s)]`   
    `# nslookup [IP address of NFS client(s)]`

4. Установить пакеты:   

    `yum update`   
    `sudo yum -y install realmd sssd adcli samba-common krb5-workstation chrony nfs-utils`

5.  Настройте NTP-клиент.  

    По умолчанию в RHEL 8 используется чрони. Следуя рекомендациям по конфигурации в руководстве по настройке [NTP, используйте `Chrony` набор](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/using-chrony-to-configure-ntp).

6.  Присоединиться к домену Active Directory:  

    `sudo realm join $DOMAIN.NAME -U $SERVICEACCOUNT --computer-ou="OU=$YOUROU"`

    Пример: 

    `sudo realm join CONTOSO.COM -U ad_admin --computer-ou="CN=Computers"`
    
    Убедитесь, что `default_realm` для параметра задана указанная область в `/etc/krb5.conf` .  В противном случае добавьте его в `[libdefaults]` раздел файла, как показано в следующем примере:
    
    `default_realm = CONTOSO.COM`

7. Перезапустите все службы NFS:  
 
    `systemctl start nfs-*`   
    `systemctl restart rpc-gssd.service`

    Перезагрузка предотвращает состояние ошибки `“mount.nfs: an incorrect mount option was specified”` при подключении Kerberos.

8. Выполните `kinit` команду с учетной записью пользователя, чтобы получить билеты: 
 
    `sudo kinit $SERVICEACCOUNT@DOMAIN`  

    Пример:   

    `sudo kinit ad_admin@CONTOSO.COM`


### <a name="rhel-8-configuration-if-you-are-using-dual-protocol"></a>Конфигурация RHEL 8 при использовании двух протоколов

Следующие шаги являются необязательными. Эти действия необходимо выполнить, только если в клиенте NFS используется сопоставление пользователей. 

1. Выполните все действия, описанные в разделе [Конфигурация RHEL 8, если вы используете раздел шифрования Kerberos версии нфсв 4.1](#rhel8_nfsv41_kerberos) .   

2. Добавьте статическую запись DNS в файл/etc/hosts, чтобы использовать полное доменное имя (FQDN) для AD вместо использования IP-адреса в файле конфигурации SSSD:  

    `cat /etc/hosts`   
    `10.6.1.4 winad2016.contoso.com`

3. Добавьте дополнительный раздел для доменов, чтобы разрешить идентификаторы на сервере AD LDAP:    

    `[root@reddoc cbs]# cat /etc/sssd/sssd.conf`   
    `[sssd]`   
    `domains = contoso.com, contoso-ldap (new entry added for LDAP as id_provider)`   
    `config_file_version = 2`   
    `services = nss, pam, ssh, sudo (ensure nss is present in this list)`   
 
    `[domain/contoso-ldap] (Copy the following lines. Modify as per your domain name.)`   
    `auth_provider = krb5`   
    `chpass_provider = krb5`   
    `id_provider = ldap`   
    `ldap_search_base = dc=contoso,dc=com(your domain)`   
    `ldap_schema = rfc2307bis`   
    `ldap_sasl_mech = GSSAPI`   
    `ldap_user_object_class = user`   
    `ldap_group_object_class = group`   
    `ldap_user_home_directory = unixHomeDirectory`   
    `ldap_user_principal = userPrincipalName`   
    `ldap_account_expire_policy = ad`   
    `ldap_force_upper_case_realm = true`   
    `ldap_user_search_base = cn=Users,dc=contoso,dc=com (based on your domain)`   
    `ldap_group_search_base = cn=Users,dc=contoso,dc=com (based on your domain)`   
    `ldap_sasl_authid = REDDOC$ (ensure $ at the end you can get this from “klist -kte” command)`   
    `krb5_server = winad2016.contoso.com (same as AD address which is added in /etc/hosts)`   
    `krb5_realm = CONTOSO.COM (domain name in caps)`   
    `krb5_kpasswd = winad2016.contoso.com (same as AD address which is added in /etc/hosts)`   
    `use_fully_qualified_names = false`   
 
    `[domain/contoso.com]  (Do not edit or remove any of the following information. This information is automatically generated during the realm join process.)`   
    `ad_domain = contoso.com`   
    `krb5_realm = CONTOSO.COM`   
    `realmd_tags = manages-system joined-with-adcli`   
    `cache_credentials = True`   
    `id_provider = ad`   
    `krb5_store_password_if_offline = True`   
    `default_shell = /bin/bash`   
    `ldap_id_mapping = True`   
    `use_fully_qualified_names = True`   
    `fallback_homedir = /home/%u@%d`   
    `access_provider = ad`   

4. Убедитесь `/etc/nsswitch.conf` , что у вас есть `sss` запись:   

    `cat /etc/nsswitch.conf`   
    `passwd: sss files systemd`   
    `group: sss files systemd`   
    `netgroup: sss files`   

5. Перезапустите `sssd` службу и очистите кэш:   

    `service sssd stop`   
    `rm -f /var/lib/sss/db/*`   
    `service sssd start`   
 
6. Проверьте, что клиент интегрирован с LDAP-сервером:   

    `[root@red81 cbs]# id ldapuser1`   
    `uid=1234(ldapuser1) gid=1111(ldapgroup1) groups=1111(ldapgroup1)`   

## <a name="ubuntu-configuration"></a>Конфигурация Ubuntu   

В этом разделе описываются конфигурации Ubuntu, необходимые для шифрования Kerberos версии Нфсв 4.1 и двойного протокола. 

В примерах этого раздела используется следующее доменное имя и IP-адрес:  

* Имя домена: `contoso.com`
* Частный IP-адрес: `10.6.1.4`

1. Настройте `/etc/resolv.conf` с помощью соответствующего DNS-сервера:

    `root@ubuntu-rak:/home/cbs# cat /etc/resolv.conf`   
    `search contoso.com`   
    `nameserver <private IP address of DNS server>`   

2. Добавьте запись клиента NFS на DNS-сервере для зоны прямого и обратного просмотра DNS.
 
    Чтобы проверить DNS, используйте следующие команды из клиента NFS:

    `# nslookup [hostname/FQDN of NFS client(s)]`   
    `# nslookup [IP address of NFS client(s)]`   

3. Установить пакеты:
 
    `apt-get update`   
    `apt-get install -y realmd packagekit sssd adcli samba-common chrony krb5-user nfs-common`
    
    При появлении запроса введите `$DOMAIN.NAME` `CONTOSO.COM` в качестве области Kerberos по умолчанию (используя прописные буквы, например).

4. Перезапустите службу `rpc-gssd.service` : 

    `sudo systemctl start rpc-gssd.service`

5. Ubuntu 18,04 по умолчанию использует чрони. Следуя рекомендациям по настройке в [Ubuntu Бионик: использование чрони для настройки NTP](https://ubuntu.com/blog/ubuntu-bionic-using-chrony-to-configure-ntp).

6. Присоединиться к домену Active Directory:   
 
    `sudo realm join $DOMAIN.NAME -U $SERVICEACCOUNT --computer-ou="OU=$YOUROU"`
 
    Пример:    
    `sudo realm join CONTOSO.COM -U ad_admin --computer-ou="CN=Computers"`

7. Выполните `kinit` с помощью пользователя, чтобы получить билеты: 
 
    `sudo kinit $SERVICEACCOUNT`   
 
    Пример:    
    `sudo kinit ad_admin`  

### <a name="ubuntu-configuration-if-you-are-using-dual-protocol"></a>Конфигурация Ubuntu при использовании двух протоколов  

Следующие шаги являются необязательными.  Эти действия необходимо выполнить, только если вы хотите использовать сопоставление пользователей на клиенте NFS:  

1. Выполните следующую команду, чтобы обновить установленные пакеты:   
    `sudo apt update && sudo apt install libnss-ldap libpam-ldap ldap-utils nscd`

    В следующем примере используются примеры значений. Когда команда запросит ввод данных, необходимо ввести входные данные в соответствии с вашей средой. 

    `base dc=contoso,dc=com uri ldap://10.20.0.4:389/ ldap_version 3 rootbinddn cn=admin,cn=Users,dc=contoso,dc=com pam_password ad`   

2. Выполните следующую команду, чтобы перезапустить и включить службу:

    `sudo systemctl restart nscd && sudo systemctl enable nscd`   

В следующем примере производится запрос к серверу LDAP AD из клиента Ubuntu LDAP для пользователя LDAP `‘hari1’` : 

`root@cbs-k8s-varun4-04:/home/cbs# getent passwd hari1`   
`hari1:*:1237:1237:hari1:/home/hari1:/bin/bash`   


## <a name="next-steps"></a>Дальнейшие действия  

* [Создание тома NFS для Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Создание тома с двумя протоколами для Azure NetApp Files](create-volumes-dual-protocol.md)

