---
title: Локальные параметры брандмауэра и прокси-сервера для службы "Синхронизация файлов Azure" | Документация Майкрософт
description: Узнайте, Синхронизация файлов Azure параметры локального прокси-сервера и брандмауэра. Проверьте сведения о конфигурации для портов, сетей и специальных подключений к Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 3/02/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: f0dbe7f32f14eb4da3d591811d619eb2e9bea397
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101729646"
---
# <a name="azure-file-sync-proxy-and-firewall-settings"></a>Параметры брандмауэра и прокси-сервера службы "Синхронизация файлов Azure"
Служба "Синхронизация файлов Azure" подключает локальные серверы к службе файлов Azure, обеспечивая синхронизацию нескольких сайтов и распределение данных по уровням облака. Таким образом локальный сервер должен быть подключен к Интернету. Администратор отдела ИТ должен выбрать наилучший путь подключения сервера к облачным службам Azure.

В этой статье описываются особые требования и возможности, позволяющие создать безопасное подключение сервера к службе "Синхронизация файлов Azure".

Перед изучением этого практического руководства рекомендуется ознакомиться с [рекомендациями по настройке сети для службы "Синхронизация файлов Azure"](storage-sync-files-networking-overview.md).

## <a name="overview"></a>Обзор
Служба "Синхронизация файлов Azure" действует как служба оркестрации между Windows Server, вашим файловым ресурсом Azure и несколькими другими службами Azure, которая синхронизирует данные, как описано в группе синхронизации. Чтобы служба "Синхронизация файлов Azure" работала правильно, необходимо настроить серверы для взаимодействия со следующими службами Azure.

- Хранилище Azure
- Синхронизация файлов Azure
- Azure Resource Manager
- Службы аутентификации

> [!Note]  
> Агент службы "Синхронизация файлов Azure" на сервере Windows инициирует все запросы к облачным службам, поэтому в брандмауэре следует учитывать только исходящий трафик. <br /> Ни одна из служб Azure не инициирует подключение к агенту службы "Синхронизация файлов Azure".

## <a name="ports"></a>Порты
Служба "Синхронизация файлов Azure" перемещает файлы данных и метаданные только по протоколу HTTPS, и порт 443 должен быть открыт для исходящего трафика.
В результате шифруется весь трафик.

## <a name="networks-and-special-connections-to-azure"></a>Сети и специальные подключения к Azure
Агент службы "Синхронизация файлов Azure" не накладывает какие-либо требования в отношении специальных каналов, например [ExpressRoute](../../expressroute/expressroute-introduction.md), в Azure.

Служба "Синхронизация файлов Azure" будет использовать все доступные средства связи с Azure, автоматически адаптируясь к различным характеристикам сетей (пропускной способности, задержке и т. д.). Кроме того, она предоставляет административный контроль для точной настройки. Некоторые возможности сейчас недоступны. Если вы хотите настроить определенное поведение, свяжитесь с нами с помощью сайта [UserVoice для службы файлов Azure](https://feedback.azure.com/forums/217298-storage?category_id=180670).

## <a name="proxy"></a>Proxy (Прокси)
Служба "Синхронизация файлов Azure" поддерживает параметры прокси-сервера для конкретных приложений и для всего компьютера.

**Параметры прокси-сервера для конкретного приложения** позволяют настраивать прокси-сервер, специально предназначенный для синхронизация файлов Azure трафика. Параметры прокси-сервера для конкретных приложений поддерживаются в версии агента 4.0.1.0 и более поздних версиях. Их можно настроить во время установки агента или с помощью командлета PowerShell Set-StorageSyncProxyConfiguration.

Команды PowerShell для настройки параметров прокси-сервера для конкретных приложений:
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Set-StorageSyncProxyConfiguration -Address <url> -Port <port number> -ProxyCredential <credentials>
```
Например, если для прокси-сервера требуется проверка подлинности с использованием имени пользователя и пароля, выполните следующие команды PowerShell:

```powershell
# IP address or name of the proxy server.
$Address="127.0.0.1"  

# The port to use for the connection to the proxy.
$Port=8080

# The user name for a proxy.
$UserName="user_name" 

# Please type or paste a string with a password for the proxy.
$SecurePassword = Read-Host -AsSecureString

$Creds = New-Object System.Management.Automation.PSCredential ($UserName, $SecurePassword)

# Please verify that you have entered the password correctly.
Write-Host $Creds.GetNetworkCredential().Password

Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"

Set-StorageSyncProxyConfiguration -Address $Address -Port $Port -ProxyCredential $Creds
```
**Параметры прокси-сервера на уровне компьютера** прозрачны для агента Синхронизация файлов Azure, так как весь трафик сервера направляется через прокси-сервер.

Чтобы настроить параметры прокси-сервера для компьютера, выполните следующие действия: 

1. Настройте параметры прокси-сервера для приложений .NET 

   - Измените следующие два файла:  
     C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config  
     C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config

   - Добавьте раздел <system.net> в файлы machine.config (в раздел <system.serviceModel>).  Измените 127.0.01:8888 на IP-адрес и порт для прокси-сервера. 
     ```
      <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
          <proxy autoDetect="false" bypassonlocal="false" proxyaddress="http://127.0.0.1:8888" usesystemdefault="false" />
        </defaultProxy>
      </system.net>
     ```

2. Задайте параметры прокси-сервера WinHTTP 

   - Выполните следующую команду в командной строке с повышенными привилегиями или PowerShell, чтобы увидеть существующие параметры прокси-сервера:   

     netsh winhttp show proxy

   - Выполните следующую команду в командной строке с повышенными привилегиями или PowerShell, чтобы задать параметры прокси-сервера (измените 127.0.01:8888 на IP-адрес и порт для прокси-сервера):  

     netsh winhttp set proxy 127.0.0.1:8888

3. Перезапустите службу агента синхронизации хранилища, выполнив следующую команду в командной строке с повышенными привилегиями или PowerShell: 

      net stop filesyncsvc

      Примечание. Служба агента синхронизации хранилища (filesyncsvc) автоматически запустится после остановки.

## <a name="firewall"></a>Брандмауэр
Как упоминалось в предыдущем разделе, порт 443 необходимо открыть для исходящего трафика. В зависимости от политик в центре обработки данных, филиале или регионе, возможно необязательное или обязательное наложение дополнительных ограничений трафика, проходящего через этот порт в определенные домены.

В следующей таблице описаны требуемые домены для обмена данными.

| Служба | Конечная точка общедоступного облака | Конечная точка Azure для государственных организаций | Использование |
|---------|----------------|---------------|------------------------------|
| **Azure Resource Manager** | `https://management.azure.com` | https://management.usgovcloudapi.net | Любой вызов пользователя (например, PowerShell) проходит через этот URL-адрес, включая вызов для начальной регистрации сервера. |
| **Azure Active Directory** | https://login.windows.net<br>`https://login.microsoftonline.com` | https://login.microsoftonline.us | Вызовы Azure Resource Manager должны осуществляться пользователем, прошедшим аутентификацию. Чтобы вызовы могли быть выполнены, этот URL-адрес используется для аутентификации пользователей. |
| **Azure Active Directory** | https://graph.microsoft.com/ | https://graph.microsoft.com/ | В ходе развертывания службы "Синхронизация файлов Azure" в подписке Azure Active Directory будет создан субъект-служба. Для этого используется данный URL-адрес. Этот субъект используется для делегирования минимального набора прав службе "Синхронизация файлов Azure". Пользователь, выполняющий начальную настройку службы "Синхронизация файлов Azure", должен иметь привилегии владельца подписки. |
| **Azure Active Directory** | https://secure.aadcdn.microsoftonline-p.com | Используйте URL-адрес общедоступной конечной точки. | Доступ к этому URL-адресу предоставляется библиотекой проверки подлинности Active Directory, которую пользовательский интерфейс регистрации Синхронизация файлов Azure Server использует для входа в систему администратора. |
| **Хранилище Azure** | &ast;.core.windows.net | &ast;. core.usgovcloudapi.net | Когда сервер скачивает файл, он эффективнее перемещает данные, если напрямую обращается к файловому ресурсу Azure в учетной записи хранения. Сервер имеет ключ SAS, предоставляющий доступ только к целевому файловому ресурсу. |
| **Синхронизация файлов Azure** | &ast;.one.microsoft.com<br>&ast;. afs.azure.net | &ast;. afs.azure.us | После начальной регистрации сервер получает URL-адрес экземпляра службы "Синхронизация файлов Azure" в этом регионе. Сервер может использовать этот URL-адрес для прямого и эффективного взаимодействия с экземпляром, выполняющим его синхронизацию. |
| **Microsoft PKI** |  https://www.microsoft.com/pki/mscorp/cps<br>http://crl.microsoft.com/pki/mscorp/crl/<br>http://mscrl.microsoft.com/pki/mscorp/crl/<br>http://ocsp.msocsp.com<br>http://ocsp.digicert.com/<br>http://crl3.digicert.com/ | https://www.microsoft.com/pki/mscorp/cps<br>http://crl.microsoft.com/pki/mscorp/crl/<br>http://mscrl.microsoft.com/pki/mscorp/crl/<br>http://ocsp.msocsp.com<br>http://ocsp.digicert.com/<br>http://crl3.digicert.com/ | После установки агента службы "Синхронизация файлов Azure" с помощью URL-адреса PKI загружаются промежуточные сертификаты, обеспечивающие обмен данными между службой "Синхронизация файлов Azure" и файловым ресурсом Azure. С помощью URL-адреса OCSP проверяется состояние сертификата. |
| **Центр обновления Майкрософт** | &ast;.update.microsoft.com<br>&ast;.download.windowsupdate.com<br>&ast;. ctldl.windowsupdate.com<br>&ast;.dl.delivery.mp.microsoft.com<br>&ast;.emdl.ws.microsoft.com | &ast;.update.microsoft.com<br>&ast;.download.windowsupdate.com<br>&ast;. ctldl.windowsupdate.com<br>&ast;.dl.delivery.mp.microsoft.com<br>&ast;.emdl.ws.microsoft.com | После установки агента Синхронизация файлов Azure Центр обновления Майкрософт URL-адреса используются для загрузки обновлений агента Синхронизация файлов Azure. |

> [!Important]
> При разрешении трафика в &ast; . AFS.Azure.NET трафик может быть доступен только службе синхронизации. Нет других служб Майкрософт, использующих этот домен.
> Если разрешить передачу трафика в &ast;.one.microsoft.com, то с сервера будет передаваться не только трафик для службы синхронизации. В поддоменах могут размещаться многие другие службы Майкрософт.

Если &ast; . AFS.Azure.NET или &ast; . One.Microsoft.com слишком широкий, можно ограничить взаимодействие сервера, разрешая обмен данными только с явными региональными экземплярами службы синхронизации файлов Azure. Выбор экземпляров зависит от региона службы синхронизации хранилища, в котором развернут и зарегистрирован сервер. Этот регион называется "URL-адрес основной конечной точки" в таблице ниже.

В целях обеспечения непрерывности бизнес-процессов и аварийного восстановления (BCDR) вы могли разместить файловые ресурсы Azure в учетной записи глобально избыточного хранилища (GRS). Если это так, тогда в случае продолжительного регионального сбоя будет выполнена отработка отказа файловых ресурсов Azure в связанный регион. Служба "Синхронизация файлов Azure" использует те же региональные связи, что и хранилище. Поэтому при использовании учетных записей хранения GRS необходимо включить дополнительные URL-адреса, чтобы разрешить серверу взаимодействовать с парным регионом для Синхронизация файлов Azure. Приведенная ниже таблица вызывает эту "парный регион". Кроме того, есть URL-адрес профиля диспетчера трафика, который также должен быть включен. Это позволяет гарантировать простое перенаправление трафика в парный регион в случае отработки отказа. Этот адрес называется "URL-адрес обнаружения" в таблице ниже.

| Облако  | Регион | URL-адрес основной конечной точки | Парный регион | URL-адрес обнаружения |
|--------|--------|----------------------|---------------|---------------|
| Общие |Восточная Австралия | HTTPS: \/ /australiaeast01.AFS.Azure.NET<br>HTTPS: \/ /kailani-Aue.One.Microsoft.com | Australia Southeast | HTTPS: \/ /TM-australiaeast01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-Aue.One.Microsoft.com |
| Общие |Australia Southeast | HTTPS: \/ /australiasoutheast01.AFS.Azure.NET<br>HTTPS: \/ /kailani-AUS.One.Microsoft.com | Восточная Австралия | HTTPS: \/ /TM-australiasoutheast01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-AUS.One.Microsoft.com |
| Общие | Brazil South | HTTPS: \/ /brazilsouth01.AFS.Azure.NET | Центрально-южная часть США | HTTPS: \/ /TM-brazilsouth01.AFS.Azure.NET |
| Общие | Центральная Канада | HTTPS: \/ /canadacentral01.AFS.Azure.NET<br>HTTPS: \/ /kailani-CAC.One.Microsoft.com | Восточная Канада | HTTPS: \/ /TM-canadacentral01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-CAC.One.Microsoft.com |
| Общие | Восточная Канада | HTTPS: \/ /canadaeast01.AFS.Azure.NET<br>HTTPS: \/ /kailani-CAE.One.Microsoft.com | Центральная Канада | HTTPS: \/ /TM-canadaeast01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani.CAE.One.Microsoft.com |
| Общие | Центральная Индия | HTTPS: \/ /centralindia01.AFS.Azure.NET<br>HTTPS: \/ /kailani-cIn.One.Microsoft.com | Южная Индия | HTTPS: \/ /TM-centralindia01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-cIn.One.Microsoft.com |
| Общие | Центральная часть США | HTTPS: \/ /centralus01.AFS.Azure.NET<br>HTTPS: \/ /kailani-CUS.One.Microsoft.com | восточная часть США 2 | HTTPS: \/ /TM-centralus01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-CUS.One.Microsoft.com |
| Общие | Восточная Азия | HTTPS: \/ /eastasia01.AFS.Azure.NET<br>HTTPS: \/ /kailani11.One.Microsoft.com | Southeast Asia | HTTPS: \/ /TM-eastasia01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani11.One.Microsoft.com |
| Общие | Восточная часть США | HTTPS: \/ /eastus01.AFS.Azure.NET<br>HTTPS: \/ /kailani1.One.Microsoft.com | Западная часть США | HTTPS: \/ /TM-eastus01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani1.One.Microsoft.com |
| Общие | восточная часть США 2 | HTTPS: \/ /eastus201.AFS.Azure.NET<br>HTTPS: \/ /kailani-ESS.One.Microsoft.com | Центральная часть США | HTTPS: \/ /TM-eastus201.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-ESS.One.Microsoft.com |
| Общие | Северная Германия | HTTPS: \/ /germanynorth01.AFS.Azure.NET | Центрально-Западная Германия | HTTPS: \/ /TM-germanywestcentral01.AFS.Azure.NET |
| Общие | Центрально-Западная Германия | HTTPS: \/ /germanywestcentral01.AFS.Azure.NET | Северная Германия | HTTPS: \/ /TM-germanynorth01.AFS.Azure.NET |
| Общие | Japan East | HTTPS: \/ /japaneast01.AFS.Azure.NET | Западная Япония | HTTPS: \/ /TM-japaneast01.AFS.Azure.NET |
| Общие | Западная Япония | HTTPS: \/ /japanwest01.AFS.Azure.NET | Japan East | HTTPS: \/ /TM-japanwest01.AFS.Azure.NET |
| Общие | Республика Корея, центральный регион | HTTPS: \/ /koreacentral01.AFS.Azure.NET/ | Республика Корея, южный регион | HTTPS: \/ /TM-koreacentral01.AFS.Azure.NET/ |
| Общие | Республика Корея, южный регион | HTTPS: \/ /koreasouth01.AFS.Azure.NET/ | Республика Корея, центральный регион | HTTPS: \/ /TM-koreasouth01.AFS.Azure.NET/ |
| Общие | Центрально-северная часть США | HTTPS: \/ /northcentralus01.AFS.Azure.NET | Центрально-южная часть США | HTTPS: \/ /TM-northcentralus01.AFS.Azure.NET |
| Общие | Северная Европа | HTTPS: \/ /northeurope01.AFS.Azure.NET<br>HTTPS: \/ /kailani7.One.Microsoft.com | Западная Европа | HTTPS: \/ /TM-northeurope01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani7.One.Microsoft.com |
| Общие | Центрально-южная часть США | HTTPS: \/ /southcentralus01.AFS.Azure.NET | Центрально-северная часть США | HTTPS: \/ /TM-southcentralus01.AFS.Azure.NET |
| Общие | Южная Индия | HTTPS: \/ /southindia01.AFS.Azure.NET<br>HTTPS: \/ /kailani-Sin.One.Microsoft.com | Центральная Индия | HTTPS: \/ /TM-southindia01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-Sin.One.Microsoft.com |
| Общие | Southeast Asia | HTTPS: \/ /southeastasia01.AFS.Azure.NET<br>HTTPS: \/ /kailani10.One.Microsoft.com | Восточная Азия | HTTPS: \/ /TM-southeastasia01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani10.One.Microsoft.com |
| Общие | Северная Швейцария | HTTPS: \/ /switzerlandnorth01.AFS.Azure.NET<br>HTTPS: \/ /TM-switzerlandnorth01.AFS.Azure.NET | Западная Швейцария | HTTPS: \/ /switzerlandwest01.AFS.Azure.NET<br>HTTPS: \/ /TM-switzerlandwest01.AFS.Azure.NET |
| Общие | Западная Швейцария | HTTPS: \/ /switzerlandwest01.AFS.Azure.NET<br>HTTPS: \/ /TM-switzerlandwest01.AFS.Azure.NET | Северная Швейцария | HTTPS: \/ /switzerlandnorth01.AFS.Azure.NET<br>HTTPS: \/ /TM-switzerlandnorth01.AFS.Azure.NET |
| Общие | южная часть Соединенного Королевства | HTTPS: \/ /uksouth01.AFS.Azure.NET<br>HTTPS: \/ /kailani-UKS.One.Microsoft.com | западная часть Соединенного Королевства | HTTPS: \/ /TM-uksouth01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-UKS.One.Microsoft.com |
| Общие | западная часть Соединенного Королевства | HTTPS: \/ /ukwest01.AFS.Azure.NET<br>HTTPS: \/ /kailani-UKW.One.Microsoft.com | южная часть Соединенного Королевства | HTTPS: \/ /TM-ukwest01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani-UKW.One.Microsoft.com |
| Общие | центрально-западная часть США | HTTPS: \/ /westcentralus01.AFS.Azure.NET | Западная часть США 2 | HTTPS: \/ /TM-westcentralus01.AFS.Azure.NET |
| Общие | Западная Европа | HTTPS: \/ /westeurope01.AFS.Azure.NET<br>HTTPS: \/ /kailani6.One.Microsoft.com | Северная Европа | HTTPS: \/ /TM-westeurope01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani6.One.Microsoft.com |
| Общие | Западная часть США | HTTPS: \/ /westus01.AFS.Azure.NET<br>HTTPS: \/ /kailani.One.Microsoft.com | Восточная часть США | HTTPS: \/ /TM-westus01.AFS.Azure.NET<br>HTTPS: \/ /TM-kailani.One.Microsoft.com |
| Общие | Западная часть США 2 | HTTPS: \/ /westus201.AFS.Azure.NET | центрально-западная часть США | HTTPS: \/ /TM-westus201.AFS.Azure.NET |
| Государственный сектор | US Gov (Аризона) | HTTPS: \/ /usgovarizona01.AFS.Azure.US | US Gov (Техас) | HTTPS: \/ /TM-usgovarizona01.AFS.Azure.US |
| Государственный сектор | US Gov (Техас) | HTTPS: \/ /usgovtexas01.AFS.Azure.US | US Gov (Аризона) | HTTPS: \/ /TM-usgovtexas01.AFS.Azure.US |

- Если вы используете учетные записи хранения локально избыточного хранилища (LRS) или хранилища, избыточного в пределах зоны (ZRS), необходимо включить только URL-адрес, указанный в разделе "URL-адрес основной конечной точки".

- При использовании учетных записей хранения GRS включите три URL-адреса.

**Пример.** Вы развертываете службу синхронизации хранилища в `"West US"` и регистрируете свой сервер в ней. URL-адреса, позволяющие обмениваться данными с сервером для этого случая:

> - HTTPS: \/ /westus01.AFS.Azure.NET (первичная конечная точка: Западная часть США)
> - HTTPS: \/ /eastus01.AFS.Azure.NET (парный регион отработки отказа: Восточная часть США)
> - HTTPS: \/ /TM-westus01.AFS.Azure.NET (URL-адрес обнаружения основного региона)

### <a name="allow-list-for-azure-file-sync-ip-addresses"></a>Список разрешений для Синхронизация файлов Azure IP-адресов
Синхронизация файлов Azure поддерживает использование [тегов служб](../../virtual-network/service-tags-overview.md), представляющих собой группу ПРЕФИКСОВ IP-адресов для данной службы Azure. Теги службы можно использовать для создания правил брандмауэра, обеспечивающих взаимодействие со службой Синхронизация файлов Azure. Тег службы для Синхронизация файлов Azure имеет значение `StorageSyncService` .

Если вы используете Синхронизация файлов Azure в Azure, можно использовать имя тега службы непосредственно в группе безопасности сети, чтобы разрешить трафик. Дополнительные сведения о том, как это делается, см. в статье [Группы безопасности сети](../../virtual-network/network-security-groups-overview.md).

Если вы используете Синхронизацию файлов Azure в локальной среде, можно с помощью API тегов служб получить конкретные диапазоны IP-адресов для включения в список разрешений брандмауэра. Существует два способа получения этой информации.

- Текущий список диапазонов IP-адресов для всех служб Azure, поддерживающих теги служб, публикуется еженедельно в Центре загрузки Майкрософт в виде документа JSON. Каждое облако Azure имеет собственный документ JSON с диапазонами IP-адресов, относящимися к этому облаку:
    - [Azure Public](https://www.microsoft.com/download/details.aspx?id=56519)
    - [Azure для US Gov организаций.](https://www.microsoft.com/download/details.aspx?id=57063)
    - [Azure для Китая](https://www.microsoft.com/download/details.aspx?id=57062)
    - [Azure для Германии](https://www.microsoft.com/download/details.aspx?id=57064)
- API обнаружения тегов служб (предварительная версия) позволяет получать текущий список тегов служб программными средствами. В предварительной версии API обнаружения тегов служб может возвращать менее актуальную информацию по сравнению с документами JSON, публикуемыми в Центре загрузки Майкрософт. Вы можете использовать поверхность API в соответствии с вашими настройками автоматизации:
    - [REST API](/rest/api/virtualnetwork/servicetags/list)
    - [Azure PowerShell](/powershell/module/az.network/Get-AzNetworkServiceTag)
    - [Azure CLI](/cli/azure/network#az-network-list-service-tags)

Поскольку API обнаружения тегов служб не обновляется так часто, как документы JSON, опубликованные в центре загрузки Майкрософт, мы рекомендуем использовать документ JSON для обновления списка разрешений локального брандмауэра. Это можно сделать следующим образом.

```PowerShell
# The specific region to get the IP address ranges for. Replace westus2 with the desired region code 
# from Get-AzLocation.
$region = "westus2"

# The service tag for Azure File Sync. Do not change unless you're adapting this
# script for another service.
$serviceTag = "StorageSyncService"

# Download date is the string matching the JSON document on the Download Center. 
$possibleDownloadDates = 0..7 | `
    ForEach-Object { [System.DateTime]::Now.AddDays($_ * -1).ToString("yyyyMMdd") }

# Verify the provided region
$validRegions = Get-AzLocation | `
    Where-Object { $_.Providers -contains "Microsoft.StorageSync" } | `
    Select-Object -ExpandProperty Location

if ($validRegions -notcontains $region) {
    Write-Error `
            -Message "The specified region $region is not available. Either Azure File Sync is not deployed there or the region does not exist." `
            -ErrorAction Stop
}

# Get the Azure cloud. This should automatically based on the context of 
# your Az PowerShell login, however if you manually need to populate, you can find
# the correct values using Get-AzEnvironment.
$azureCloud = Get-AzContext | `
    Select-Object -ExpandProperty Environment | `
    Select-Object -ExpandProperty Name

# Build the download URI
$downloadUris = @()
switch($azureCloud) {
    "AzureCloud" { 
        $downloadUris = $possibleDownloadDates | ForEach-Object {  
            "https://download.microsoft.com/download/7/1/D/71D86715-5596-4529-9B13-DA13A5DE5B63/ServiceTags_Public_$_.json"
        }
    }

    "AzureUSGovernment" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/6/4/D/64DB03BF-895B-4173-A8B1-BA4AD5D4DF22/ServiceTags_AzureGovernment_$_.json"
        }
    }

    "AzureChinaCloud" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/9/D/0/9D03B7E2-4B80-4BF3-9B91-DA8C7D3EE9F9/ServiceTags_China_$_.json"
        }
    }

    "AzureGermanCloud" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/0/7/6/076274AB-4B0B-4246-A422-4BAF1E03F974/ServiceTags_AzureGermany_$_.json"
        }
    }

    default {
        Write-Error -Message "Unrecognized Azure Cloud: $_" -ErrorAction Stop
    }
}

# Find most recent file
$found = $false 
foreach($downloadUri in $downloadUris) {
    try { $response = Invoke-WebRequest -Uri $downloadUri -UseBasicParsing } catch { }
    if ($response.StatusCode -eq 200) {
        $found = $true
        break
    }
}

if ($found) {
    # Get the raw JSON 
    $content = [System.Text.Encoding]::UTF8.GetString($response.Content)

    # Parse the JSON
    $serviceTags = ConvertFrom-Json -InputObject $content -Depth 100

    # Get the specific $ipAddressRanges
    $ipAddressRanges = $serviceTags | `
        Select-Object -ExpandProperty values | `
        Where-Object { $_.id -eq "$serviceTag.$region" } | `
        Select-Object -ExpandProperty properties | `
        Select-Object -ExpandProperty addressPrefixes
} else {
    # If the file cannot be found, that means there hasn't been an update in
    # more than a week. Please verify the download URIs are still accurate
    # by checking https://docs.microsoft.com/azure/virtual-network/service-tags-overview
    Write-Verbose -Message "JSON service tag file not found."
    return
}
```

Затем можно использовать диапазоны IP-адресов в `$ipAddressRanges` для обновления брандмауэра. Сведения о том, как обновить брандмауэр, см. на веб-сайте брандмауэра или сетевого устройства.

## <a name="test-network-connectivity-to-service-endpoints"></a>Проверка сетевого подключения к конечным точкам службы
После регистрации сервера в службе Синхронизация файлов Azure можно использовать командлет Test-StorageSyncNetworkConnectivity и ServerRegistration.exe для проверки связи со всеми конечными точками (URL-адресами), специфичными для этого сервера. Этот командлет может помочь в устранении неполадок, когда незавершенная связь не позволяет серверу полностью работать с Синхронизация файлов Azure и может использоваться для точной настройки конфигурации прокси-сервера и брандмауэра.

Чтобы запустить проверку сетевого подключения, установите агент Синхронизация файлов Azure версии 9,1 или более поздней и выполните следующие команды PowerShell:
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Test-StorageSyncNetworkConnectivity
```

## <a name="summary-and-risk-limitation"></a>Сводка и ограничение рисков
Списки, приведенные ранее в этом документе, содержат URL-адреса, с которыми в настоящее время взаимодействует служба "Синхронизация файлов Azure". Брандмауэры должны иметь возможность разрешить исходящий трафик для этих доменов. Корпорация Майкрософт прилагает все усилия, чтобы этот список был актуальным.

Настройка домена с ограничивающими правилами брандмауэра может служить мерой повышения безопасности. Если используются такие конфигурации брандмауэров, необходимо помнить, что URL-адреса со временем добавляются и могут измениться. Периодически просматривайте эту статью.

## <a name="next-steps"></a>Дальнейшие действия
- [Планирование развертывания службы синхронизации файлов Azure (предварительная версия)](storage-sync-files-planning.md)
- [Как развернуть службу синхронизации файлов Azure (предварительная версия)](storage-sync-files-deployment-guide.md)
- [Мониторинг Синхронизации файлов Azure](storage-sync-files-monitoring.md)
