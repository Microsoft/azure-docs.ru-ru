---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.openlocfilehash: 2f27c50b1d016265c20102521a137bcbb0646115
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185505"
---
При наличии подписанного URL-адреса (SAS), предоставляющего доступ к ресурсам в учетной записи хранения, SAS можно использовать в строке подключения. Так как SAS содержит сведения URI, необходимые для аутентификации запроса, строка подключения с SAS предоставляет протокол, конечную точку службы и учетные данные, необходимые для доступа к ресурсу.

Чтобы создать строку подключения, содержащую подписанный URL-адрес, задайте ее в следующем формате:

```
BlobEndpoint=myBlobEndpoint;
QueueEndpoint=myQueueEndpoint;
TableEndpoint=myTableEndpoint;
FileEndpoint=myFileEndpoint;
SharedAccessSignature=sasToken
```

Несмотря на то что конечные точки службы необязательны, строка подключения должна содержать по крайней мере одну конечную точку.

> [!NOTE]
> По соображениям безопасности с SAS рекомендуется использовать протокол HTTPS.
>
> При указании SAS в строке подключения в файле конфигурации может потребоваться закодировать специальные символы в URL-адресе.
>
>

### <a name="service-sas-example"></a>Пример SAS службы
Ниже приведен пример строки подключения, которая включает подписанный URL-адрес службы для хранилища BLOB-объектов:

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
SharedAccessSignature=sv=2015-04-05&sr=b&si=tutorial-policy-635959936145100803&sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D
```

Еще один пример той же строки подключения с кодированием специальных символов:

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
SharedAccessSignature=sv=2015-04-05&amp;sr=b&amp;si=tutorial-policy-635959936145100803&amp;sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D
```

### <a name="account-sas-example"></a>Пример SAS учетной записи
Ниже приведен пример строки подключения, которая включает подписанный URL-адрес учетной записи для хранилища BLOB-объектов и файлового хранилища: Обратите внимание, что указаны конечные точки для обеих служб.

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
FileEndpoint=https://storagesample.file.core.windows.net;
SharedAccessSignature=sv=2015-07-08&sig=iCvQmdZngZNW%2F4vw43j6%2BVz6fndHF5LI639QJba4r8o%3D&spr=https&st=2016-04-12T03%3A24%3A31Z&se=2016-04-13T03%3A29%3A31Z&srt=s&ss=bf&sp=rwl
```

Еще один пример той же строки подключения с кодированием URL-адреса:

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
FileEndpoint=https://storagesample.file.core.windows.net;
SharedAccessSignature=sv=2015-07-08&amp;sig=iCvQmdZngZNW%2F4vw43j6%2BVz6fndHF5LI639QJba4r8o%3D&amp;spr=https&amp;st=2016-04-12T03%3A24%3A31Z&amp;se=2016-04-13T03%3A29%3A31Z&amp;srt=s&amp;ss=bf&amp;sp=rwl
```

