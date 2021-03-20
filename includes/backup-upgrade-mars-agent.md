---
title: Вопросы об агенте службы Azure Backup
description: В этих сведениях объясняется, почему необходимо обновить агент Azure Backup и где можно загрузить обновление.
services: backup
cloud: ''
suite: ''
author: dcurwin
manager: carmonm
ms.service: backup
ms.tgt_pltfrm: <optional>
ms.devlang: <optional>
ms.topic: article
ms.date: 03/03/2020
ms.author: dacurwin
ms.openlocfilehash: bd298f758d6109b908db01dd2ae3b97e5e2f714a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "78673178"
---
## <a name="upgrade-the-mars-agent"></a>Обновление агента MARS

Версии агента Службы восстановления Microsoft Azure (MARS) ниже 2.0.9083.0 имеют зависимость от службы контроля доступа Azure. Агент MARS также называется агентом Azure Backup.

В 2018 Майкрософт не является [устаревшей службой контроля доступа Azure](../articles/active-directory/azuread-dev/active-directory-acs-migration.md). Начиная с 19 марта 2018 во всех версиях агента MARS ниже 2.0.9083.0 будут возникать сбои резервного копирования. Чтобы избежать сбоев резервного копирования или устранить их, [установите последнюю версию агента MARS](https://support.microsoft.com/help/4538314/update-for-azure-backup-for-microsoft-azure-recovery-services-agent). Чтобы указать серверы, для которых требуется обновление агента MARS, выполните действия, описанные в разделе [обновление агента службы восстановления Microsoft Azure (MARS)](../articles/backup/upgrade-mars-agent.md).

Агент MARS используется для резервного копирования файлов и папок и данных состояния системы в Azure. System Center DPM и Azure Backup Server используют агент MARS для резервного копирования данных в Azure.
