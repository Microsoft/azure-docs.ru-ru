---
title: Учебник. Архивация журналов каталога в учетную запись хранения | Документация Майкрософт
description: Узнайте, как настроить службу "Диагностика Azure" для принудительной отправки журналов Azure Active Directory в учетную запись хранения.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 045f94b3-6f12-407a-8e9c-ed13ae7b43a3
ms.service: active-directory
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: fa2cac7c89a2cb1bd99675727bcd51e82c4a18ef
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74007863"
---
# <a name="tutorial-archive-azure-ad-logs-to-an-azure-storage-account"></a>Руководство. Архивация журналов Azure AD в учетную запись хранения Azure

В этом руководстве вы узнаете, как настроить параметры диагностики Azure Monitor для передачи журналов Azure Active Directory (Azure AD) в учетную запись хранения Azure.

## <a name="prerequisites"></a>Предварительные условия 

Для использования этой функции необходимо иметь следующее.

* Подписка Azure с учетной записью хранения Azure. Если у вас нет подписки Azure, можно [зарегистрироваться и получить бесплатную пробную версию](https://azure.microsoft.com/free/).
* Клиент Azure AD.
* Пользователь, который является *глобальным администратором* или *администратором безопасности* для этого клиента Azure AD.

## <a name="archive-logs-to-an-azure-storage-account"></a>Архивация журналов в учетную запись хранения Azure

1. Войдите на [портал Azure](https://portal.azure.com). 

2. Щелкните **Azure Active Directory** > **Действие** > **Журналы аудита**. 

3. Выберите **Параметры экспорта**. 

4. В области **Параметры диагностики** выполните одно из действий ниже:
   * Чтобы изменить имеющиеся параметры, выберите **Настройка параметра**.
   * Чтобы добавить новые параметры, выберите **Add diagnostics setting** (Добавить параметр диагностики).  
     Можно задать не более трех параметров. 

     ![Параметры экспорта](./media/quickstart-azure-monitor-route-logs-to-storage-account/ExportSettings.png)

5. Введите понятное имя параметра, чтобы не забыть, для чего он нужен (например, *Отправить в учетную запись хранения Azure*). 

6. Установите флажок **Архивировать в учетной записи хранения**, а затем выберите **Учетная запись хранения**. 

7. Выберите подписку Azure и учетную запись хранения, в которые нужно отправить журналы.
 
8. Нажмите кнопку **ОК**, чтобы закрыть конфигурацию.

9. Выполните одно из приведенных ниже действий или оба:
    * Установите флажок **AuditLogs**, чтобы отправить журналы аудита в учетную запись хранения. 
    * Установите флажок **SignInLogs**, чтобы отправить журналы входа в учетную запись хранения.

10. С помощью ползунка установите период хранения данных журнала. По умолчанию это значение равно *0*. Это означает, что журналы будут храниться в учетной записи хранения бессрочно. Если вы зададите другое значение, события, старше указанного количества дней, будут автоматически очищены.

11. Нажмите кнопку **Сохранить**, чтобы сохранить параметры.

    ![Параметры диагностики](./media/quickstart-azure-monitor-route-logs-to-storage-account/DiagnosticSettings.png)

12. Примерно через 15 минут проверьте наличие журналов в учетной записи хранения. Перейдите на [портал Azure](https://portal.azure.com), щелкните **Учетные записи хранения**, выберите учетную запись хранения, которая использовалась ранее, и щелкните **Большие двоичные объекты**. Для **журналов аудита** выберите **insights-log-audit**. Для **журналов входа** выберите **insights-logs-signin**.

    ![Учетная запись хранения](./media/quickstart-azure-monitor-route-logs-to-storage-account/StorageAccount.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Interpret the Azure AD audit logs schema in Azure Monitor (preview)](reference-azure-monitor-audit-log-schema.md) (Интерпретация схемы журналов аудита Azure Active Directory в Azure Monitor (предварительная версия))
* [Interpret the Azure AD sign-in logs schema in Azure Monitor (preview)](reference-azure-monitor-sign-ins-log-schema.md) (Интерпретация схемы журналов входа Azure Active Directory в Azure Monitor (предварительная версия))
* [Часто задаваемые вопросы и известные проблемы](concept-activity-logs-azure-monitor.md#frequently-asked-questions)