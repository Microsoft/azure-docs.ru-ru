---
title: Настройка удостоверения — служба "Сетка событий Azure" IoT Edge | Документация Майкрософт
description: Настройка удостоверения модуля сетки событий
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 2418c8518bbf298a102d4f29b4a973f6121de2eb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86171692"
---
# <a name="configure-identity-for-the-event-grid-module"></a>Настройка удостоверения для модуля "Сетка событий"

В этой статье показано, как настроить удостоверение для сетки на границе. По умолчанию модуль "Сетка событий" представляет свой сертификат удостоверений, настроенный управляющей программой IoT. Сетка событий на границе представляет сертификат удостоверения с исходящими вызовами при доходе доставки событий. Затем подписчик может проверить модуль сетки событий, который отправил событие перед принятием.

Все возможные конфигурации см. в разделе руководств по [безопасности и проверке подлинности](security-authentication.md) .

## <a name="always-present-identity-certificate"></a>Сертификат удостоверения всегда представлен
Ниже приведен пример конфигурации для постоянного представления сертификата идентификации в исходящих вызовах. 

```json
 {
  "Env": [
    "outbound__clientAuth__clientCert__enabled=true",
    "outbound__clientAuth__clientCert__source=IoTEdge"
  ]
}
 ```

## <a name="dont-present-identity-certificate"></a>Не представлять сертификат удостоверения
Ниже приведен пример конфигурации для отсутствия сертификата удостоверения для исходящих вызовов. 

```json
 {
  "Env": [
    "outbound__clientAuth__clientCert__enabled=false"
  ]
}
 ```
