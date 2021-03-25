---
title: Руководство. Создание пользовательских записей Azure DNS для веб-приложения
description: В этом руководстве описано создание записей DNS личного домена для веб-приложения с помощью Azure DNS.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 10/20/2020
ms.author: rohink
ms.openlocfilehash: 369c7dab174f0269797b10635882a6821ade8311
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94952953"
---
# <a name="tutorial-create-dns-records-in-a-custom-domain-for-a-web-app"></a>Руководство. Создание записей DNS для веб-приложения в личном домене 

Службу Azure DNS можно использовать для размещения пользовательского домена для веб-приложений. Например, создается веб-приложение Azure и необходимо, чтобы пользователи получали к нему доступ, используя веб-сайт contoso.com или www\.contoso.com как полное доменное имя (FQDN).

> [!NOTE]
> В этом руководстве веб-сайт contoso.com используется в качестве примера. Подставьте собственное доменное имя вместо contoso.com.

Для этого необходимо создать три записи:

* корневую запись A, указывающую на contoso.com;
* корневую запись TXT для проверки;
* запись CNAME для имени www, которая указывает на запись A.

Помните, что если создать запись A для веб-приложения в Azure, ее необходимо обновить вручную, если основной IP-адрес веб-приложения изменится.

В этом руководстве описано следующее.

> [!div class="checklist"]
> * Создание записи A и TXT для пользовательского домена
> * Создание записи CNAME для личного домена
> * Тестирование новых записей
> * Добавление пользовательских имен узлов для веб-приложения
> * Проверка пользовательских имен узлов

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* У вас должно быть доступное доменное имя, которое можно будет разместить в Azure DNS и использовать для тестирования. У вас должен быть полный контроль над этим доменом, включая возможность определять записи сервера имен (NS) для домена.
* [Создайте приложение службы приложений](../app-service/quickstart-html.md) или используйте приложение, созданное для работы с другим руководством.

* Создайте зону DNS в Azure DNS и делегируйте ее в регистраторе Azure DNS.

   1. Чтобы создать зону DNS, выполните действия, описанные в разделе [Создание зоны DNS](./dns-getstarted-powershell.md).
   2. Для делегирования зоны в Azure DNS выполните действия, описанные в статье [Делегирование зон DNS с помощью Azure DNS](dns-delegate-domain-azure-dns.md).

После создания зоны и ее делегирования в Azure DNS можно создать записи для личного домена.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-an-a-record-and-txt-record"></a>Создание записи A и TXT

Запись A используется для сопоставления имени с IP-адресом. В следующем примере \@ назначается как запись A, используя IPv4-адрес веб-приложения. \@ обычно представляет корневой домен.

### <a name="get-the-ipv4-address"></a>Получение IPv4-адреса

В левой области навигации страницы служб приложений на портале Azure выберите **Личные домены**. 

![Меню личных доменов](../app-service/./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

На странице **Личные домены** скопируйте IPv4-адрес приложения.

![Переход к приложению Azure на портале](../app-service/./media/app-service-web-tutorial-custom-domain/mapping-information.png)

### <a name="create-the-a-record"></a>Создание записи A

```azurepowershell
New-AzDnsRecordSet -Name "@" -RecordType "A" -ZoneName "contoso.com" `
 -ResourceGroupName "MyAzureResourceGroup" -Ttl 600 `
 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "<your web app IP address>")
```

### <a name="create-the-txt-record"></a>Создание записи TXT

Службы приложений используют эту запись только во время настройки, чтобы убедиться, что вы являетесь владельцем личного домена. После проверки и настройки личного домена в службе приложений эту запись TXT можно удалить.

> [!NOTE]
> Если вы хотите проверить имя домена, а не маршрутизацию рабочего трафика к веб-приложению, необходимо только указать запись типа TXT для этого этапа проверки.  Для проверки не требуется запись A или CNAME в дополнение к записи TXT.

```azurepowershell
New-AzDnsRecordSet -ZoneName contoso.com -ResourceGroupName MyAzureResourceGroup `
 -Name "@" -RecordType "txt" -Ttl 600 `
 -DnsRecords (New-AzDnsRecordConfig -Value  "contoso.azurewebsites.net")
```

## <a name="create-the-cname-record"></a>Создание записи CNAME

Если вашим доменом уже управляет Azure DNS (см. статью о [делегировании домена в Azure DNS](dns-domain-delegation.md)), можно использовать следующий пример, чтобы создать запись CNAME для contoso.azurewebsites.net.

Откройте Azure PowerShell и создайте новую запись CNAME. В этом примере будет создан набор записей CNAME со сроком жизни 600 секунд в зоне DNS с именем "contoso.com" и псевдонимом для веб-приложения contoso.azurewebsites.net.

### <a name="create-the-record"></a>Создание записи

```azurepowershell
New-AzDnsRecordSet -ZoneName contoso.com -ResourceGroupName "MyAzureResourceGroup" `
 -Name "www" -RecordType "CNAME" -Ttl 600 `
 -DnsRecords (New-AzDnsRecordConfig -cname "contoso.azurewebsites.net")
```

Ниже приведен пример ответа.

```
    Name              : www
    ZoneName          : contoso.com
    ResourceGroupName : myresourcegroup
    Ttl               : 600
    Etag              : 8baceeb9-4c2c-4608-a22c-229923ee185
    RecordType        : CNAME
    Records           : {contoso.azurewebsites.net}
    Tags              : {}
```

## <a name="test-the-new-records"></a>Тестирование новых записей

Проверить, что записи были созданы правильно, можно, запросив "www.contoso.com" и "contoso.com" с помощью команды nslookup, как показано ниже.

```
PS C:\> nslookup
Default Server:  Default
Address:  192.168.0.1

> www.contoso.com
Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
Name:    <instance of web app service>.cloudapp.net
Address:  <ip of web app service>
Aliases:  www.contoso.com
contoso.azurewebsites.net
<instance of web app service>.vip.azurewebsites.windows.net

> contoso.com
Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
Name:    contoso.com
Address:  <ip of web app service>

> set type=txt
> contoso.com

Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
contoso.com text =

        "contoso.azurewebsites.net"
```
## <a name="add-custom-host-names"></a>Добавление пользовательских имен узлов

Теперь можно добавить пользовательские имена узлов для веб-приложения.

```azurepowershell
set-AzWebApp `
 -Name contoso `
 -ResourceGroupName MyAzureResourceGroup `
 -HostNames @("contoso.com","www.contoso.com","contoso.azurewebsites.net")
```
## <a name="test-the-custom-host-names"></a>Проверка пользовательских имен узлов

Откройте окно браузера и перейдите по адресу `http://www.<your domainname>` и `http://<you domain name>`.

> [!NOTE]
> Убедитесь, что вы добавили префикс `http://`, иначе браузер может попытаться предсказать ваш URL-адрес.

Вы должны увидеть ту же страницу для обоих URL-адресов. Пример:

![Служба приложений в Contoso](media/dns-web-sites-custom-domain/contoso-app-svc.png)


## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы, созданные в этом руководстве, больше не нужны, можно удалить группу ресурсов **myresourcegroup**.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как создать частные зоны Azure DNS.

> [!div class="nextstepaction"]
> [Приступая к работе с частными зонами Azure DNS с помощью PowerShell](private-dns-getstarted-powershell.md)