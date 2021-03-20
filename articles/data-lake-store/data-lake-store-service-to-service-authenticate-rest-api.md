---
title: Проверка подлинности между службами — Data Lake Storage 1-го поколения — Azure
description: Узнайте, как обеспечить проверку подлинности между службами с помощью Azure Data Lake Storage 1-го поколения и Azure Active Directory использования REST API.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 4947bab58a6d23bcc3e3212a9e3e7dc0c4fa8578
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92106711"
---
# <a name="service-to-service-authentication-with-azure-data-lake-storage-gen1-using-rest-api"></a>Аутентификация между службами в Azure Data Lake Storage 1-го поколения с использованием REST API
> [!div class="op_single_selector"]
> * [Использование Java](data-lake-store-service-to-service-authenticate-java.md)
> * [Использование пакета SDK для .NET](data-lake-store-service-to-service-authenticate-net-sdk.md)
> * [Использование Python](data-lake-store-service-to-service-authenticate-python.md)
> * [Использование REST API](data-lake-store-service-to-service-authenticate-rest-api.md)
> 
> 

Из этой статьи вы узнаете, как использовать REST API для проверки подлинности между службами с помощью Azure Data Lake Storage 1-го поколения. Дополнительные сведения об аутентификации пользователей в Azure Data Lake Storage 1-го поколения с помощью REST API см. в статье [Аутентификация пользователей в Azure Data Lake Storage 1-го поколения с помощью REST API](data-lake-store-end-user-authenticate-rest-api.md).

## <a name="prerequisites"></a>Предварительные условия

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Создайте веб-приложение Azure Active Directory**. Вам нужно выполнить инструкции по [аутентификации между службами в Data Lake Storage 1-го поколения с помощью Azure Active Directory](data-lake-store-service-to-service-authenticate-using-active-directory.md).

## <a name="service-to-service-authentication"></a>Аутентификация между службами

В этом сценарии приложение предоставляет собственные учетные данные для выполнения операций. Для этого необходимо отправить запрос POST, аналогичный следующему:

```console
curl -X POST https://login.microsoftonline.com/<TENANT-ID>/oauth2/token  \
  -F grant_type=client_credentials \
  -F resource=https://management.core.windows.net/ \
  -F client_id=<CLIENT-ID> \
  -F client_secret=<AUTH-KEY>
```

Выходные данные этого запроса будут содержать маркер авторизации (`access-token` в приведенных ниже выходных данных) для дальнейшей передачи в вызовах REST API. Сохраните маркер аутентификации в текстовый файл. Он понадобится вам при выполнении вызовов REST к Data Lake Storage 1-го поколения.

```output
{"token_type":"Bearer","expires_in":"3599","expires_on":"1458245447","not_before":"1458241547","resource":"https://management.core.windows.net/","access_token":"<REDACTED>"}
```

В этой статье используется **неинтерактивный** подход. Дополнительные сведения о неинтерактивном подходе (вызовы между службами) см. в [этой статье](/previous-versions/azure/dn645543(v=azure.100)).

## <a name="next-steps"></a>Дальнейшие действия

В этой статье описывается, как использовать аутентификацию между службами, чтобы реализовать аутентификацию в Data Lake Storage 1-го поколения с помощью REST API. Дополнительные сведения об использовании REST API для работы с Data Lake Storage 1-го поколения см. в следующих статьях.

* [Операции управления учетными записями для Data Lake Storage 1-го поколения с помощью REST API](data-lake-store-get-started-rest-api.md)
* [Операции с данными в Data Lake Storage 1-го поколения c использованием REST API](data-lake-store-data-operations-rest-api.md)