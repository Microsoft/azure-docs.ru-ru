---
title: Устранение неполадок с представлениями корпоративных затрат в Azure
description: Узнайте, как устранить все проблемы, связанные с представлением организационных затрат в пределах портала Azure.
author: bandersmsft
ms.reviewer: amberb
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: troubleshooting
ms.date: 08/20/2019
ms.author: banders
ms.custom: seodec18
ms.openlocfilehash: d78621697d53f429624dc22a8e91ba4367dcfd18
ms.sourcegitcommit: 97c48e630ec22edc12a0f8e4e592d1676323d7b0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2021
ms.locfileid: "101093971"
---
# <a name="troubleshoot-enterprise-cost-views"></a>Устранение неполадок в представлении затрат на условиях соглашения Enterprise

В рамках регистрации на предприятии с помощью некоторых настроек можно ограничить возможность пользователей просматривать затраты.  Этими параметрами управляет администратор регистрации. Если же регистрацию не приобрели напрямую через корпорацию Майкрософт, параметрами управляет партнер.  В этой статье содержится обзор параметров, а также их взаимосвязи с регистрацией. Эти параметры не зависят от ролей Azure.

## <a name="enable-access-to-costs"></a>Включение доступа к затратам

При поиске сведений о затратах вы можете получить сообщение об ошибке авторизации или об *отключении представлений затрат в регистрации*. С вами такое случилось?
![Снимок экрана, показывающий "не авторизовано" в поле текущей стоимости для подписки.](./media/enterprise-mgmt-grp-troubleshoot-cost-view/unauthorized.png)

Это может быть вызвано одной из следующих причин:

1. Вы приобрели Azure через корпоративного участника, который еще не предоставил вам сведения о ценах. Свяжитесь с партнером, чтобы обновить параметр цен на портале [Enterprise Portal](https://ea.azure.com).
2. Если вы являетесь непосредственным клиентом EA, доступны несколько возможностей:
    * Вы являетесь владельцем учетной записи, и администратор регистрации отключил параметр **просмотра затрат для владельца учетной записи**.  
    * Вы являетесь администратором отдела, и администратор регистрации отключил **возможность просмотра затрат для администратора отдела**.
    * Обратитесь к администратору регистрации для получения доступа. Администратор регистрации может обновить параметры на портале [Enterprise Portal](https://ea.azure.com/manage/enrollment).

      ![Снимок экрана, показывающий параметры для просмотра затрат на портале Enterprise Portal.](./media/enterprise-mgmt-grp-troubleshoot-cost-view/ea-portal-settings.png)

## <a name="asset-is-unavailable"></a>Ресурс недоступен

Если при попытке получить доступ к подписке или группе управления вы получили сообщение об ошибке, констатирующее, что **ресурс недоступен**, значит вам не назначена определенная роль, чтобы просмотреть этот элемент.  

![Снимок экрана: сообщение "ресурс недоступен".](./media/enterprise-mgmt-grp-troubleshoot-cost-view/asset-not-found.png)

Чтобы получить доступ, попросите администратора подписки Azure или группы управления. Дополнительные сведения см. в статье [Назначение ролей Azure с помощью портала Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Дальнейшие действия
- Если у вас есть вопросы или вам нужна помощь, [создайте запрос в службу поддержки](https://go.microsoft.com/fwlink/?linkid=2083458).
