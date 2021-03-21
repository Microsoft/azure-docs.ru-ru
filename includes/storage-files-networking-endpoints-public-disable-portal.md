---
title: включить файл
description: включить файл
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 01/25/2021
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 192a16d6f7bb39608b040ac28fe2bedb7a2e2dc3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98798677"
---
Перейдите к учетной записи хранения, для которой вы хотите запретить доступ к общедоступной конечной точке. В содержании для учетной записи хранения выберите **Сеть**.

В верхней части страницы установите переключатель рядом с пунктом **Выбранные сети**. Это действие отобразит ряд скрытых параметров для управления ограничениями общедоступной конечной точки. Установите флажок **Разрешить доверенным службам Майкрософт доступ к этой учетной записи службы**, чтобы доверенные службы Майкрософт (например, Синхронизация файлов Azure) могли обращаться к этой учетной записи хранения.

[![Снимок экрана: колонка "Сеть", где указаны соответствующие ограничения](media/storage-files-networking-endpoints-public-disable-portal/disable-public-endpoint-0.png)](media/storage-files-networking-endpoints-public-disable-portal/disable-public-endpoint-0.png#lightbox)