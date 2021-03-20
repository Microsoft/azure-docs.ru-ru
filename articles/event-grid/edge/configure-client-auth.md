---
title: Настройка проверки подлинности входящих вызовов клиента с помощью службы "Сетка событий Azure" IoT Edge | Документация Майкрософт
description: Сведения о возможных конфигурациях проверки подлинности клиента для модуля "Сетка событий".
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 9525da4204e270d033f0c2354318bd71874eaf54
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91290778"
---
# <a name="configure-client-authentication-of-incoming-calls"></a>Настройка проверки подлинности входящих вызовов клиента

В этом разделе приведены примеры возможных конфигураций проверки подлинности клиента для модуля "Сетка событий". Модуль "Сетка событий" поддерживает два типа проверки подлинности клиента:

* Подпись общего доступа (SAS) на основе ключей
* На основе сертификатов

Все возможные конфигурации см. в разделе руководств по [безопасности и проверке подлинности](security-authentication.md) .

## <a name="enable-certificate-based-client-authentication-no-self-signed-certificates"></a>Включение проверки подлинности клиентов на основе сертификатов, без самозаверяющих сертификатов

```json
 {
  "Env": [
    "inbound__clientAuth__sasKeys__enabled=false",
    "inbound__clientAuth__clientCert__enabled=true",
    "inbound__clientAuth__clientCert__source=IoTEdge",
    "inbound__clientAuth__clientCert__allowUnknownCA=false"
  ]
}
 ```

## <a name="enable-certificate-based-client-authentication-allow-self-signed-certificates"></a>Включение проверки подлинности клиента на основе сертификата, разрешение самозаверяющих сертификатов

```json
 {
  "Env": [
    "inbound__clientAuth__sasKeys__enabled=false",
    "inbound__clientAuth__clientCert__enabled=true",
    "inbound__clientAuth__clientCert__source=IoTEdge",
    "inbound__clientAuth__clientCert__allowUnknownCA=true"
  ]
}
```

>[!NOTE]
>Присвойте свойству **inbound__clientAuth__clientCert__allowUnknownCA** **значение true** только в тестовых средах, так как обычно можно использовать самозаверяющие сертификаты. Для рабочих нагрузок рекомендуется присвоить этому свойству **значение false** и сертификаты из центра сертификации (ЦС).

## <a name="enable-certificate-based-and-sas-key-based-client-authentication"></a>Включение проверки подлинности клиента на основе сертификата и SAS-ключа

```json
 {
  "Env": [
    "inbound__clientAuth__sasKeys__enabled=true",
    "inbound__clientAuth__sasKeys__key1=<some-secret1-here>",
    "inbound__clientAuth__sasKeys__key2=<some-secret2-here>",
    "inbound__clientAuth__clientCert__enabled=true",
    "inbound__clientAuth__clientCert__source=IoTEdge",
    "inbound__clientAuth__clientCert__allowUnknownCA=true"
  ]
}
 ```

>[!NOTE]
>Проверка подлинности клиента на основе ключа SAS позволяет модулю, не относящейся к IoT, выполнять операции управления и выполнения, исходя из того, что порты API доступны за пределами IoT Edge сети.
