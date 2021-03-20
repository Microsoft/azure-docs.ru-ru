---
title: включить файл
description: включить файл
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: include
ms.date: 09/12/2018
ms.author: raynew
ms.openlocfilehash: f4cd9cad3813378fcdc3f06e8ab1c28eced4f93c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95999197"
---
Имя | Коммерческий URL-адрес | Государственный URL-адрес | Описание
--- | --- | --- | ---
Azure Active Directory | ``login.microsoftonline.com`` | ``login.microsoftonline.us`` | Используется для управления доступом и удостоверениями с помощью Azure Active Directory. 
Резервное копирование | ``*.backup.windowsazure.com`` | ``*.backup.windowsazure.us`` | Используется для передачи данных репликации и координации.
Репликация | ``*.hypervrecoverymanager.windowsazure.com`` | ``*.hypervrecoverymanager.windowsazure.us``  | Используется для операций управления репликацией и координации.
Память | ``*.blob.core.windows.net`` | ``*.blob.core.usgovcloudapi.net``  | Используется для доступа к учетной записи хранения, в которой хранятся реплицируемые данные.
Телеметрия (необязательно) | ``dc.services.visualstudio.com`` | ``dc.services.visualstudio.com`` | Используется для телеметрии.
Синхронизация времени | ``time.windows.com`` | ``time.nist.gov`` | Используется для проверки синхронизации времени системы с глобальным временем во всех развертываниях.


