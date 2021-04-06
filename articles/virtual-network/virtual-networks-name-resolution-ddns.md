---
title: Использование DDNS для регистрации имен узлов в Azure | Документация Майкрософт
description: Узнайте, как установить DDNS для регистрации имен узлов на собственных DNS-серверах.
services: dns
documentationcenter: na
author: subsarma
manager: vitinnan
editor: ''
ms.assetid: c315961a-fa33-45cf-82b9-4551e70d32dd
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/23/2017
ms.author: subsarma
ms.openlocfilehash: ad91eb94aedcdd0e4e715162e3ae064a1d2fb1ea
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98220428"
---
# <a name="use-dynamic-dns-to-register-hostnames-in-your-own-dns-server"></a>Использование DDNS для регистрации имен узлов на собственном DNS-сервере

[Azure предоставляет разрешение имен](virtual-networks-name-resolution-for-vms-and-role-instances.md) для виртуальных машин и экземпляров ролей. Если разрешение имен должно выходить за рамки возможностей стандартных DNS-серверов Azure, вы можете предоставить свои DNS-серверы. Это дает возможность настроить свое решение DNS в соответствии с конкретными потребностями. Например, вам может понадобиться доступ к локальным ресурсам через контроллер домена Active Directory.

Если пользовательские DNS-серверы размещены как виртуальные машины Azure, вы можете направлять запросы имени узла (в той же виртуальной сети) в Azure для разрешения имен узлов. Если вы не хотите использовать этот вариант, можно зарегистрировать имена узлов своих виртуальных машин на DNS-сервере с помощью DDNS (динамическая система доменных имен). Azure не может (не имеет учетных данных) регистрировать записи непосредственно на DNS-серверах, поэтому часто требуется прибегать к альтернативным вариантам. Ниже приведены некоторые распространенные сценарии и альтернативные варианты.

## <a name="windows-clients"></a>Клиенты Windows
Не входящие в домен клиенты Windows пытаются выполнить небезопасные обновления DDNS при загрузке или изменении своего IP-адреса. DNS-именем является имя узла и основной DNS-суффикс. Azure оставляет основной DNS-суффикс пустым, но вы можете определить его на виртуальной машине с помощью [пользовательского интерфейса](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc794784(v=ws.10)) или [PowerShell](/powershell/module/dnsclient/set-dnsclient).

Входящие в домен клиенты Windows регистрируют свои IP-адреса на контроллере домена, используя защищенную службу DDNS. Процесс присоединения к домену задает основной DNS-суффикс на стороне клиента, создавая и поддерживая отношение доверия.

## <a name="linux-clients"></a>Клиенты Linux
Клиенты Linux обычно не регистрируются на DNS-сервере при запуске — предполагается, что это делает DHCP-сервер. DHCP-серверы Azure не имеют учетных данных для регистрации записей на вашем DNS-сервере. Для отправки обновлений DDNS можно использовать инструмент `nsupdate`, который входит в пакет Bind. Так как протокол DDNS является стандартизованным, инструмент `nsupdate` можно применять даже в том случае, если Bind не используется на DNS-сервере.

Чтобы создавать и поддерживать записи имени узла на DNS-сервере, можно использовать обработчики, предоставляемые DHCP-клиентом. Во время цикла DHCP клиент выполняет скрипты в расположении */etc/dhcp/dhclient-exit-hooks.d/* . Эти обработчики можно использовать для регистрации нового IP-адреса с помощью `nsupdate`. Пример:

```bash
#!/bin/sh
requireddomain=mydomain.local

# only execute on the primary nic
if [ "$interface" != "eth0" ]
then
    return
fi

# When you have a new IP, perform nsupdate
if [ "$reason" = BOUND ] || [ "$reason" = RENEW ] ||
   [ "$reason" = REBIND ] || [ "$reason" = REBOOT ]
then
   host=`hostname`
   nsupdatecmds=/var/tmp/nsupdatecmds
     echo "update delete $host.$requireddomain a" > $nsupdatecmds
     echo "update add $host.$requireddomain 3600 a $new_ip_address" >> $nsupdatecmds
     echo "send" >> $nsupdatecmds

     nsupdate $nsupdatecmds
fi
```

Для выполнения безопасных обновлений DDNS можно также использовать команду `nsupdate`. Например, при использовании DNS-сервера Bind создается пара открытого и закрытого ключей (`http://linux.yyz.us/nsupdate/`). DNS-сервер настраивается (`http://linux.yyz.us/dns/ddns-server.html`) с использованием открытой части ключа, что делает возможным проверку подписи запроса. Чтобы предоставить пару ключей для `nsupdate`, используйте параметр `-k` для проверки подписи запросов на обновление DDNS.

При использовании DNS-сервера Windows в `nsupdate` можно выполнять аутентификацию Kerberos с параметром `-g` (недоступно в версии `nsupdate` для Windows). Чтобы использовать Kerberos, воспользуйтесь `kinit` для загрузки учетных данных. Например, можно загрузить учетные данные из [KEYTAB-файла](https://www.itadmintools.com/2011/07/creating-kerberos-keytab-files.html), после чего `nsupdate -g` извлечет учетные данные из кэша.

При необходимости в виртуальные машины можно добавить суффикс поиска DNS. DNS-суффикс указан в файле */etc/resolv.conf* . Большинство дистрибутивов Linux управляет содержимым этого файла автоматически, поэтому его обычно нельзя изменить. Тем не менее, вы можете переопределить суффикс с помощью команды `supersede` DHCP-клиента. Чтобы переопределить суффикс, добавьте следующую строку в файл */etc/dhcp/dhclient.conf*.

```
supersede domain-name <required-dns-suffix>;
```