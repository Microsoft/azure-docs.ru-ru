---
title: Ошибки имени зарезервированного ресурса
description: Описывается, как устранить ошибки при предоставлении имени ресурса, который включает зарезервированное слово.
ms.topic: troubleshooting
ms.date: 11/08/2017
ms.openlocfilehash: e76f4bf9bfee7de6e7523d69acf1388d2dd80e93
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75477633"
---
# <a name="resolve-reserved-resource-name-errors"></a>Ошибки имен зарезервированных ресурсов Azure

В этой статье описаны ошибки, которые возникают при развертывании ресурса, в имени которого содержится зарезервированное слово.

## <a name="symptom"></a>Симптом

При развертывании ресурсов, которые доступны через другую общедоступную конечную точку, может появиться следующая ошибка:

```
Code=ReservedResourceName;
Message=The resource name <resource-name> or a part of the name is a trademarked or reserved word.
```

## <a name="cause"></a>Причина

Ресурсы, которые имеют общедоступную конечную точку, не могут содержать зарезервированные слова или товарные знаки в имени.

Следующие слова являются зарезервированными:

* ACCESS;
* AZURE;
* BING;
* BIZSPARK;
* BIZTALK;
* CORTANA;
* DIRECTX;
* DOTNET;
* DYNAMICS;
* EXCEL
* EXCHANGE
* FOREFRONT;
* GROOVE;
* HOLOLENS;
* HYPERV;
* KINECT;
* LYNC;
* MSDN
* O365
* OFFICE;
* OFFICE365;
* ONEDRIVE;
* ONENOTE;
* OUTLOOK;
* POWERPOINT;
* SHAREPOINT
* SKYPE;
* VISIO;
* VISUALSTUDIO.

Следующие слова нельзя использовать ни в качестве целого слова, ни как часть строки в имени:

* Имя_для_входа
* MICROSOFT;
* WINDOWS
* XBOX.

## <a name="solution"></a>Решение

Предоставьте имя, которое не содержит ни одного из зарезервированных слов.