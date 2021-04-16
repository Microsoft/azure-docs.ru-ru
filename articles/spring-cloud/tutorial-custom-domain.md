---
title: Руководство. Сопоставление существующего личного домена с Azure Spring Cloud
description: Сведения о том, как сопоставить существующее настраиваемое DNS-имя с Azure Spring Cloud.
author: MikeDodaro
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 03/19/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 7d01f3d5efeee2db5f468a0fe6217d1ff3c313a7
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107375021"
---
# <a name="tutorial-map-an-existing-custom-domain-to-azure-spring-cloud"></a>Руководство. Сопоставление существующего личного домена с Azure Spring Cloud

**Эта статья применима к:** ✔️ Java ✔️ C#

Технология DNS (служба доменных имен) предназначена для хранения имен узлов в сети. В этом руководстве показано, как с помощью записи CNAME сопоставить домен (например, www.contoso.com). Также мы защитим домен с помощью сертификата и покажем, как применить протокол TLS (или SSL). 

Сертификаты используются для шифрования веб-трафика. Сертификаты TLS/SSL можно хранить в Azure Key Vault. 

## <a name="prerequisites"></a>Предварительные требования
* Приложение, развернутое в Azure Spring Cloud (можно воспользоваться статьей [Краткое руководство. Запуск существующего приложения Azure Spring Cloud с помощью портала Azure](spring-cloud-quickstart.md) или применить любое существующее приложение).
* Доменное имя с доступом к реестру DNS для поставщика домена, например GoDaddy.
* Закрытый сертификат (т. е. ваш самозаверяющий сертификат) стороннего поставщика. Этот сертификат должен соответствовать домену.
* Развернутый экземпляр [Azure Key Vault](../key-vault/general/overview.md).

## <a name="keyvault-private-link-considerations"></a>Рекомендации по использованию Приватного канала с Key Vault

IP-адреса управления для Azure Spring Cloud не являются частью доверенных служб Майкрософт в Azure. Чтобы разрешить Azure Spring Cloud загружать сертификаты из Key Vault, защищенного с помощью подключений частной конечной точки, вам нужно будет добавить следующие IP-адреса в брандмауэр Azure Key Vault:

```
20.53.123.160 52.143.241.210 40.65.234.114 52.142.20.14 20.54.40.121 40.80.210.49 52.253.84.152 20.49.137.168 40.74.8.134 51.143.48.243
```

## <a name="import-certificate"></a>Импорт сертификата
### <a name="prepare-your-certificate-file-in-pfx-optional"></a>Подготовка файла сертификата в формате PFX (необязательно)
Azure Key Vault поддерживает импорт частного сертификата в формате PEM и PFX. Если PEM-файл, полученный от поставщика сертификатов, не работает, выполните действия в разделе ниже: [Сохранение сертификата в Key Vault](#save-certificate-in-key-vault), выполните следующие действия, чтобы создать PFX-файл для Azure Key Vault.

#### <a name="merge-intermediate-certificates"></a>Объединение промежуточных сертификатов

Если цепочка сертификатов из центра сертификации содержит несколько сертификатов, их необходимо объединить по порядку.

Чтобы сделать это, откройте каждый полученный сертификат в текстовом редакторе.

Создайте файл для объединенных сертификатов и присвойте ему имя _mergedcertificate.crt_. В текстовом редакторе скопируйте содержимое каждого сертификата в этот файл. Порядок сертификатов должен соответствовать порядку в цепочке сертификатов, начиная вашим сертификатом и заканчивая корневым сертификатом. Это должно выглядеть следующим образом:

```
-----BEGIN CERTIFICATE-----
<your entire Base64 encoded SSL certificate>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 1>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 2>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded root certificate>
-----END CERTIFICATE-----
```

#### <a name="export-certificate-to-pfx"></a>Экспорт сертификата в PFX-файл

Экспортируйте объединенный TLS/SSL-сертификат с закрытым ключом, с помощью которого был создан запрос сертификата.

Если запрос сертификата был создан с помощью OpenSSL, то вы создали файл закрытого ключа. Чтобы экспортировать сертификат в PFX-файл, выполните следующую команду. Замените заполнители _&lt;private-key-file>_ и _&lt;merged-certificate-file>_ путями к закрытому ключу и объединенному файлу сертификатов.

```bash
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

При появлении запроса определите пароль для экспорта. Позже этот пароль потребуется при передаче TLS/SSL-сертификата в Azure Key Vault.

Если вы создали запрос на сертификат с помощью IIS или _Certreq.exe_, установите сертификат на локальный компьютер, а затем [экспортируйте его в PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).

### <a name="save-certificate-in-key-vault"></a>Сохранение сертификата в Key Vault
Для импорта сертификата требуется, чтобы на диске существовал файл в кодировке PEM или PFX, и у вас был доступ к закрытому ключу. 
#### <a name="portal"></a>[Портал](#tab/Azure-portal)
Чтобы передать сертификат в хранилище ключей, сделайте следующее:
1. Перейдите к экземпляру хранилища ключей.
1. В области навигации слева щелкните **Сертификаты**.
1. В меню сверху щелкните **Создать или импортировать**.
1. В диалоговом окне **Создание сертификата** в разделе **Метод создания сертификата** выберите значение `Import`.
1. В разделе **Отправка файла сертификата** найдите и выберите расположение сертификата.
1. В поле **Пароль** введите закрытый ключ сертификата.
1. Нажмите кнопку **Создать**.

    ![Импорт сертификата 1](./media/custom-dns-tutorial/import-certificate-a.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

```azurecli
az keyvault certificate import --file <path to .pfx file> --name <certificate name> --vault-name <key vault name> --password <export password>
```
---

### <a name="grant-azure-spring-cloud-access-to-your-key-vault"></a>Предоставление Azure Spring Cloud доступа к хранилищу ключей

Чтобы предоставить Azure Spring Cloud доступ к хранилищу ключей перед импортом сертификата, сделайте следующее:
#### <a name="portal"></a>[Портал](#tab/Azure-portal)
1. Перейдите к экземпляру хранилища ключей.
1. В области навигации слева щелкните **Политика доступа**.
1. В меню сверху щелкните **Добавить политику доступа**.
1. Введите сведения и нажмите кнопку **Добавить**, а затем **Сохранить**.

| Разрешение секрета | Разрешение сертификата | Выбор субъекта |
|--|--|--|
| Получить, перечислить | Получить, перечислить | Управление доменом Azure Spring Cloud |

![Импорт сертификата 2](./media/custom-dns-tutorial/import-certificate-b.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

Предоставьте Azure Spring Cloud доступ на чтение к хранилищу ключей, заменив `<key vault resource group>` и `<key vault name>` в следующей команде.
```
az keyvault set-policy -g <key vault resource group> -n <key vault name>  --object-id 938df8e2-2b9d-40b1-940c-c75c33494239 --certificate-permissions get list --secret-permissions get list
``` 
---

### <a name="import-certificate-to-azure-spring-cloud"></a>Импорт сертификата в Azure Spring Cloud
#### <a name="portal"></a>[Портал](#tab/Azure-portal)
1. Перейдите к экземпляру службы. 
1. На странице приложения в области навигации слева выберите **Параметры TLS/SSL**.
1. Теперь щелкните **Импорт сертификата Key Vault**.

    ![Импорт сертификата](./media/custom-dns-tutorial/import-certificate.png)

1. После успешного импорта сертификат появится в списке **Сертификаты закрытых ключей**.

    ![Сертификат закрытого ключа](./media/custom-dns-tutorial/key-certificates.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

```
az spring-cloud certificate add --name <cert name> --vault-uri <key vault uri> --vault-certificate-name <key vault cert name>
```

Чтобы отобразить список импортированных сертификатов, выполните следующее.

```
az spring-cloud certificate list --resource-group <resource group name> --service <service name>
```
---

> [!IMPORTANT] 
> Чтобы защитить личный домен с помощью этого сертификата, необходимо создать привязку сертификата к этому домену. Следуйте инструкциям в этом разделе: [Добавление SSL-привязки](#add-ssl-binding)

## <a name="add-custom-domain"></a>Добавление личного домена
Пользовательское DNS-имя можно сопоставить c Azure Spring Cloud с помощью записи CNAME. 

> [!NOTE] 
> Запись типа A не поддерживается. 

### <a name="create-the-cname-record"></a>Создание записи CNAME
Перейдите к поставщику DNS и добавьте запись CNAME для сопоставления домена с адресом <имя_службы>.azuremicroservices.io. Здесь <имя_службы> обозначает имя вашего экземпляра Azure Spring Cloud. Для этих целей поддерживаются подстановочные знаки в именах доменов и поддоменов. После добавления записи CNAME страница управления записями DNS будет выглядеть примерно так: 

![Страница записей DNS](./media/custom-dns-tutorial/dns-records.png)

## <a name="map-your-custom-domain-to-azure-spring-cloud-app"></a>Сопоставление личного домена с приложением Azure Spring Cloud
Если у вас нет приложения в Azure Spring Cloud, выполните инструкции, приведенные в статье [Краткое руководство. Запуск существующего приложения Azure Spring Cloud с помощью портала Azure](./spring-cloud-quickstart.md).

#### <a name="portal"></a>[Портал](#tab/Azure-portal)
Перейдите к странице приложения.

1. Выберите **Личный домен**.
2. Теперь щелкните **Добавить личный домен**. 

    ![Личный домен](./media/custom-dns-tutorial/custom-domain.png)

3. Введите полное доменное имя, для которого вы добавили запись CNAME, например www.contoso.com. Убедитесь, что для параметра Hostname указан тип записи CNAME (<имя_службы>.azuremicroservices.io)
4. Щелкните **Проверить**, чтобы активировать кнопку **Добавить**.
5. Нажмите кнопку **Добавить**.

    ![Добавление личного домена](./media/custom-dns-tutorial/add-custom-domain.png)

У одного приложения может быть несколько доменов, но каждый домен может быть сопоставлен только с одним приложением. После успешного сопоставления личного домена с приложением вы увидите его в таблице личных доменов.

![Таблица личных доменов](./media/custom-dns-tutorial/custom-domain-table.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)
```
az spring-cloud app custom-domain bind --domain-name <domain name> --app <app name> --resource-group <resource group name> --service <service name>
```

Чтобы отобразить список личных доменов, выполните приведенные ниже действия.
```
az spring-cloud app custom-domain list --app <app name> --resource-group <resource group name> --service <service name>
```
---

> [!NOTE]
> Метка **Небезопасно** для личного домена означает, что он еще не привязан к SSL-сертификату. На любой HTTPS-запрос из браузера к такому личному домену возвращается сообщение об ошибке или предупреждение.

## <a name="add-ssl-binding"></a>Добавление привязки SSL

#### <a name="portal"></a>[Портал](#tab/Azure-portal)
В таблице личных доменов выберите **Добавить SSL-привязку**, как показано на предыдущем рисунке.  
1. В поле **Сертификат** выберите или импортируйте сертификат.
1. Выберите команду **Сохранить**.

    ![Добавление SSL-привязки 1](./media/custom-dns-tutorial/add-ssl-binding.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)
```
az spring-cloud app custom-domain update --domain-name <domain name> --certificate <cert name> --app <app name> --resource-group <resource group name> --service <service name>
```
---

После успешного добавления привязки SSL состояние домена станет "безопасным": **Работоспособно**. 

![Добавление SSL-привязки 2](./media/custom-dns-tutorial/secured-domain-state.png)

## <a name="enforce-https"></a>Принудительное использование HTTPS
По умолчанию любой пользователь по-прежнему может обращаться к приложению по протоколу HTTP, но вы можете перенаправить все HTTP-запросы на порт HTTPS.
#### <a name="portal"></a>[Портал](#tab/Azure-portal)
Для этого на странице приложения в области навигации слева выберите **Личный домен**. Здесь для параметра **Только HTTPS** установите значение *True*.

![Добавление SSL-привязки 3](./media/custom-dns-tutorial/enforce-http.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)
```
az spring-cloud app update -n <app name> --resource-group <resource group name> --service <service name> --https-only
```
---
По завершении операции перейдите по любому из URL-адресов HTTPS, которые указывают на ваше приложение. Обратите внимание, что URL-адреса HTTP теперь не работают.

## <a name="see-also"></a>См. также раздел
* [Что такое хранилище ключей Azure?](../key-vault/general/overview.md)
* [Импорт сертификата](../key-vault/certificates/certificate-scenarios.md#import-a-certificate)
* [Запуск приложения Spring Cloud с помощью Azure CLI](./spring-cloud-quickstart.md)
