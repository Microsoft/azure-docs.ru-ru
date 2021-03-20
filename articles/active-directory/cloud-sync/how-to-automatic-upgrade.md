---
title: 'Azure AD Connect агент подготовки облака: автоматическое обновление | Документация Майкрософт'
description: В этой статье описывается встроенная функция автоматического обновления в Azure AD Connect агент подготовки облака.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/02/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 26c0b47dd358826b843143aaf23c469d852e93b3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98614277"
---
# <a name="azure-ad-connect-cloud-provisioning-agent-automatic-upgrade"></a>Azure AD Connect агент подготовки облака: автоматическое обновление

Проверка того, что ваша Azure Active Directory (Azure AD) Connect Cloud Подготовка агента подготовки к работе в облаке, всегда является самой простой функцией автоматического обновления.

Агент устанавливается здесь: "Program Files\azure Multi AD Connect подготовка Agent\AADConnectProvisioningAgent.exe"

Чтобы проверить версию, щелкните исполняемый файл правой кнопкой мыши и выберите пункт Свойства, а затем — сведения.

![Версия файла агента](media/how-to-automatic-upgrade/agent-1.png)

Агент обновления устанавливается здесь: "Program Files\azure Multi AD Connect Подготовка агента Updater\AzureADConnectAgentUpdater.exe"

Чтобы проверить версию, щелкните исполняемый файл правой кнопкой мыши и выберите пункт Свойства, а затем — сведения.

![Версия агента обновления](media/how-to-automatic-upgrade/agent-2.png)

## <a name="uninstall-the-agent"></a>Удаление агента
Чтобы удалить агент, перейдите к разделу **Удаление или изменение программы** и удалите следующее:

- **Microsoft Azure AD Connect Agent Updater;**
- **агент подготовки Microsoft Azure AD Connect;**
- **пакет агента подготовки Microsoft Azure AD Connect.**

![Удаление агента](media/how-to-automatic-upgrade/agent-3.png)

## <a name="next-steps"></a>Дальнейшие действия 

- [Что собой представляет подготовка?](what-is-provisioning.md)
- [Что представляет собой облачная синхронизация Azure AD Connect?](what-is-cloud-sync.md)

