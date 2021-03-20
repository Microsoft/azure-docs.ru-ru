---
title: Управление протоколами и шифрами в службе управления API Azure | Документация Майкрософт
description: Узнайте, как управлять протоколами (TLS) и алгоритмами шифрования (DES) в службе управления API Azure.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 05/29/2019
ms.author: apimpm
ms.openlocfilehash: 043a3d0b63dfc74f587b58b3c2ac42f1a084cc4a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86250317"
---
# <a name="manage-protocols-and-ciphers-in-azure-api-management"></a>Управление протоколами и шифрами в службе управления API Azure

Служба управления API Azure поддерживает несколько версий протокола TLS для сторон клиента и сервера, а также несколько версий шифра 3DES.

В этом руководстве показано, как управлять конфигурациями протоколов и шифров для экземпляра службы управления API Azure.

![Управление протоколами и шифрами в APIM](./media/api-management-howto-manage-protocols-ciphers/api-management-protocols-ciphers.png)

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить шаги в этой статье, необходимо иметь следующее:

* Экземпляр управления API.

## <a name="how-to-manage-tls-protocols-and-3des-cipher"></a>Как управлять протоколами TLS и шифрами 3DES

1. Перейдите к **экземпляру службы управления API** на портале Azure.
2. В меню выберите пункт **параметры протокола** .  
3. Включите или отключите нужные протоколы или шифры.
4. Выберите команду **Сохранить**. Изменения будут применены в течение часа.  

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [протоколе TLS](/dotnet/framework/network-programming/tls).
* См. другие [видео](https://azure.microsoft.com/documentation/videos/index/?services=api-management) об управлении API.
