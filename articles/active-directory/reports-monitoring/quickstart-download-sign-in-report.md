---
title: Краткое руководство по загрузке отчета по входу в систему с помощью портала Azure | Документация Майкрософт
description: Узнайте, как загрузить отчет по входу в систему с помощью портала Azure
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 9131f208-1f90-4cc1-9c29-085cacd69317
ms.service: active-directory
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 86bc72f69903134afa3750ad6b72486a713b6cc0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96011920"
---
# <a name="quickstart-download-a-sign-in-report-using-the-azure-portal"></a>Краткое руководство. Загрузка отчета по входу в систему с помощью портала Azure | Документация Майкрософт

Из этого краткого руководства вы узнаете, как загрузить данные о входах своего клиента за последние 24 часа. С портала Azure можно скачать до 250 000 записей. Записи отсортированы по убыванию даты и времени по умолчанию, и вы получите 250 000 последних записей. 

## <a name="prerequisites"></a>Предварительные требования

Вам необходимы:

* клиент Azure Active Directory с лицензией уровня Premium для просмотра отчета о действиях входа; Чтобы обновить выпуск Azure Active Directory, ознакомьтесь со статьей [Регистрация для работы с выпусками Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md). Обратите внимание, если до обновления данные о действиях отсутствовали, данные отобразятся в отчетах через несколько дней после обновления до лицензии Premium.
* пользователь с ролью **администратора безопасности**, **читателя сведений о безопасности**, **читателя отчетов** или **глобального администратора** для этого клиента; а также любой пользователь клиента, который имеет доступ к своим собственным данным для входа в систему.

## <a name="quickstart-download-a-sign-in-report"></a>Краткое руководство. Загрузка отчета по входу в систему

1. Перейдите на [портал Azure](https://portal.azure.com).
2. Выберите **Azure Active Directory** в области навигации слева и с помощью **переключателя** выберите свой каталог Active Directory.
3. На панели мониторинга выберите **Azure Active Directory**, а затем выберите **Входы в систему**. 
4. Выберите **Последние 24 часа** в раскрывающемся списке фильтра **Дата**, затем нажмите кнопку **Применить**, чтобы просмотреть входы в систему за последние 24 часа. 
5. Нажмите кнопку **Загрузить**, выберите **CSV** как формат файла и укажите имя файла для скачивания CSV-файла, содержащего отфильтрованные записи. 

![Отчеты](./media/quickstart-download-sign-in-report/download-sign-ins.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Отчеты о действиях входа на портале Azure Active Directory](concept-sign-ins.md)
* [Хранение отчетов Azure Active Directory](reference-reports-data-retention.md)
* [Задержки в отчетах Azure Active Directory](reference-reports-latencies.md)
