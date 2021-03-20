---
title: Настройка строки подключения
titleSuffix: Azure Storage
description: Настройка строки подключения для учетной записи хранения Azure. Строка подключения содержит сведения, необходимые для авторизации доступа к учетной записи хранения из приложения во время выполнения с помощью авторизации общего ключа.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 10/14/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: d7ca1707c89f03683960822591065143d3f8aa4f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92783884"
---
# <a name="configure-azure-storage-connection-strings"></a>Настройка строк подключения службы хранилища Azure

Строка подключения содержит сведения об авторизации, необходимые приложению для доступа к данным в учетной записи хранения Azure во время выполнения с помощью авторизации общего ключа. Настройка строки подключения позволяет выполнить следующие действия:

* Подключитесь к эмулятору хранения Азурите.
* получить доступ к учетной записи хранения в Azure;
* получить доступ к указанным ресурсам Azure через подписанный URL-адрес (SAS).

Сведения о том, как просмотреть ключи доступа к учетной записи и скопировать строку подключения, см. в статье [Управление ключами доступа учетной записи хранения](storage-account-keys-manage.md).

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

## <a name="store-a-connection-string"></a>Сохранение строки подключения

Приложению требуется доступ к строке подключения во время выполнения, чтобы выполнять авторизацию запросов к службе хранилища Azure. Есть несколько вариантов для хранения строки подключения.

* Строку подключения можно сохранить в переменной среды.
* Приложение, работающее на настольном компьютере или на устройстве, может хранить строку подключения в файле **app.config** или **web.config**. Добавьте строку подключения в раздел **AppSettings** в этих файлах.
* Приложение, работающее в облачной службе Azure, может хранить строку подключения в [файле схемы конфигурации службы Azure (.cscfg)](/previous-versions/azure/reference/ee758710(v=azure.100)). Добавьте строку подключения в раздел **ConfigurationSettings** файла конфигурации службы.

Хранение строки подключения в файле конфигурации упрощает обновление строки подключения для переключения между [эмулятором хранения азурите](../common/storage-use-azurite.md) и учетной записью хранения Azure в облаке. Вам потребуется только изменить строку подключения, чтобы указать целевую среду.

Вы можете использовать [Microsoft Azure Configuration Manager](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/) для доступа к строке подключения в среде выполнения независимо от того, где выполняется приложение.

## <a name="configure-a-connection-string-for-azurite"></a>Настройка строки подключения для Азурите

[!INCLUDE [storage-emulator-connection-string-include](../../../includes/storage-emulator-connection-string-include.md)]

Дополнительные сведения о Азурите см. [в статье Использование эмулятора азурите для локальной разработки службы хранилища Azure](../common/storage-use-azurite.md).

## <a name="configure-a-connection-string-for-an-azure-storage-account"></a>Настройка строки подключения для учетной записи хранения Azure

Чтобы создать строку подключения для учетной записи хранения Azure, используйте формат, указанный ниже. Указывает, следует ли подключаться к учетной записи хранения через HTTPS (рекомендуется) или HTTP. Замените `myAccountName` на имя вашей учетной записи хранения и замените `myAccountKey` на ваш ключ доступа учетной записи:

`DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey`

Например, строка подключения может выглядеть так:

`DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=<account-key>`

Служба хранилища Azure поддерживает в строке подключения как HTTP, так и HTTPS, однако *настоятельно рекомендуется использовать HTTPS*.

> [!TIP]
> Данные о строках подключения учетной записи хранения можно найти на [портале Azure](https://portal.azure.com). В   >  колонке меню учетной записи хранения перейдите к **разделу Параметры ключи доступа** , чтобы просмотреть строки подключения для первичных и вторичных ключей доступа.
>

## <a name="create-a-connection-string-using-a-shared-access-signature"></a>Создание строки подключения с помощью подписанного URL-адреса

[!INCLUDE [storage-use-sas-in-connection-string-include](../../../includes/storage-use-sas-in-connection-string-include.md)]

## <a name="create-a-connection-string-for-an-explicit-storage-endpoint"></a>Создание строки подключения для явной конечной точки хранилища

Вы можете указать явные конечные точки службы в строке подключения вместо конечных точек по умолчанию. Чтобы создать строку подключения, определяющую явную конечную точку, укажите полную конечную точку для каждой службы, включая спецификацию протокола (HTTPS (рекомендуется) или HTTP), в следующем формате:

```
DefaultEndpointsProtocol=[http|https];
BlobEndpoint=myBlobEndpoint;
FileEndpoint=myFileEndpoint;
QueueEndpoint=myQueueEndpoint;
TableEndpoint=myTableEndpoint;
AccountName=myAccountName;
AccountKey=myAccountKey
```

Один из сценариев, когда может потребоваться указать явную конечную точку, — если конечная точка хранилища BLOB-объектов сопоставлена с [личным доменом](../blobs/storage-custom-domain-name.md). В этом случае в строке подключения можно указать пользовательскую конечную точку хранилища BLOB-объектов. Вы можете дополнительно указать конечные точки по умолчанию для других служб, если приложение их использует.

Ниже приведен пример строки подключения, в которой указывается явная конечная точка для службы BLOB-объектов.

```
# Blob endpoint only
DefaultEndpointsProtocol=https;
BlobEndpoint=http://www.mydomain.com;
AccountName=storagesample;
AccountKey=<account-key>
```

В следующем примере указываются явные конечные точки для всех служб, включая личный домен для службы BLOB-объектов:

```
# All service endpoints
DefaultEndpointsProtocol=https;
BlobEndpoint=http://www.mydomain.com;
FileEndpoint=https://myaccount.file.core.windows.net;
QueueEndpoint=https://myaccount.queue.core.windows.net;
TableEndpoint=https://myaccount.table.core.windows.net;
AccountName=storagesample;
AccountKey=<account-key>
```

Значения конечных точек из строки подключения применяются для создания универсальных кодов ресурса (URI) для служб хранилищ, а также определяют форму любых URI, возвращаемых в код.

Если сопоставить конечную точку хранилища с личным доменом и не указывать эту конечную точку в строке подключения, то вы не сможете получить доступ к данным в этой службе из своего кода с помощью данной строки подключения.

Дополнительные сведения о настройке пользовательского домена для службы хранилища Azure см. в статье [Преобразование личного домена в конечную точку хранилища BLOB-объектов Azure](../blobs/storage-custom-domain-name.md).

> [!IMPORTANT]
> Значения конечных точек служб в строках подключения должны быть в формате правильно сформированного универсального кода ресурса (URI), включая `https://` (рекомендуется) или `http://`.

### <a name="create-a-connection-string-with-an-endpoint-suffix"></a>Создание строки подключения с суффиксом конечной точки

Чтобы создать строку подключения для службы хранилища в регионах или экземплярах с разными суффиксами конечных точек, например для Azure Китая (21Vianet) или Azure для государственных организаций, используйте следующий формат строки подключения. Укажите, следует ли подключаться к учетной записи хранения через HTTPS (рекомендуется) или HTTP. Замените `myAccountName` именем своей учетной записи хранения, замените `myAccountKey` ключом доступа своей учетной записи и замените `mySuffix` суффиксом URI:

```
DefaultEndpointsProtocol=[http|https];
AccountName=myAccountName;
AccountKey=myAccountKey;
EndpointSuffix=mySuffix;
```

Ниже приведен пример строки подключения для служб хранилища в Azure для Китая (21Vianet).

```
DefaultEndpointsProtocol=https;
AccountName=storagesample;
AccountKey=<account-key>;
EndpointSuffix=core.chinacloudapi.cn;
```

## <a name="parsing-a-connection-string"></a>Анализ строки подключения

[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Использование эмулятора Азурите для разработки локальных хранилищ Azure](../common/storage-use-azurite.md)
* [Клиентские инструменты службы хранилища Azure](storage-explorers.md)
* [Использование подписанных URL-адресов](storage-sas-overview.md)