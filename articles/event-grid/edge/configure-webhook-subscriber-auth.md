---
title: Настройка проверки подлинности подписчика веб-перехватчика IoT Edge службы "Сетка событий Azure" | Документация Майкрософт
description: Настройка аутентификации подписчика веб-перехватчика
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: d78b6e80005e9533ccd5ebfaea853f35ae6004be
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86171658"
---
# <a name="configure-webhook-subscriber-authentication"></a>Настройка аутентификации подписчика веб-перехватчика

В этом разделе приведены примеры возможных конфигураций подписчиков веб-перехватчиков для модуля сетки событий. По умолчанию для подписчиков веб-перехватчика принимаются только конечные точки HTTPS. Модуль сетки событий будет отклонен, если подписчик представляет самозаверяющий сертификат.

## <a name="allow-only-https-subscriber"></a>Разрешить только подписчик HTTPS

```json
 {
  "Env": [
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=false"
  ]
}
 ```

## <a name="allow-https-subscriber-with-self-signed-certificate"></a>Разрешить подписчику HTTPS с самозаверяющим сертификатом

```json
 {
  "Env": [
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=true"
  ]
}
 ```

>[!NOTE]
>Задайте для свойства `outbound__webhook__allowUnknownCA` значение `true` только в тестовых средах, так как обычно можно использовать самозаверяющие сертификаты. Для рабочих нагрузок рекомендуется установить для них значение **false**.

## <a name="allow-https-subscriber-but-skip-certificate-validation"></a>Разрешить подписчику HTTPS, но пропустить проверку сертификата

```json
 {
  "Env": [
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=true",
    "outbound__webhook__allowUnknownCA=false"
  ]
}
 ```

>[!NOTE]
>Задайте для свойства `outbound__webhook__skipServerCertValidation` значение `true` только в тестовых средах, так как вы можете не представлять сертификат, для которого требуется проверка подлинности. Для рабочих нагрузок рекомендуется установить для них значение **false** .

## <a name="allow-both-http-and-https-with-self-signed-certificates"></a>Разрешить как HTTP, так и HTTPS с самозаверяющими сертификатами

```json
 {
  "Env": [
    "outbound__webhook__httpsOnly=false",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=true"
  ]
}
 ```

>[!NOTE]
>Задайте для свойства `outbound__webhook__httpsOnly` значение `false` только в тестовых средах, так как может потребоваться сначала открыть подписчик HTTP. Для рабочих нагрузок рекомендуется установить для них значение **true** .
