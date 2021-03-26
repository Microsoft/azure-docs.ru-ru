---
title: Необходимые условия для программного доступа к данным аналитики
description: Узнайте о требованиях, которые необходимо выполнить, прежде чем можно будет программным способом получить доступ к данным аналитики для коммерческого рынка.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 1cdd3dba8203ce9e8daeaa963f1722389d89d19d
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105563826"
---
# <a name="prerequisites-to-programmatically-access-analytics-data"></a>Необходимые условия для программного доступа к данным аналитики

Прежде чем вы сможете получить доступ к данным аналитики коммерческих рынков программным способом, необходимо выполнить следующие требования.

## <a name="commercial-marketplace-enrollment"></a>Коммерческая регистрация в Marketplace

Чтобы получить доступ к коммерческим данным аналитики Marketplace программным способом, необходимо зарегистрироваться в программе коммерческого рынка и получить учетную запись центра партнеров. Чтобы узнать, как это сделать, см. статью [создание коммерческой учетной записи Marketplace в центре партнеров](./partner-center-portal/create-account.md).

## <a name="create-azure-active-directory-application"></a>Создание приложения Azure Active Directory

Обычные учетные данные пользователя нельзя использовать для программного доступа к данным аналитики коммерческих рынков. Для доступа к API-интерфейсам аналитики необходимо создать приложение Azure Active Directory (Azure AD) и секретный код. Сведения о создании приложения и секрета Azure AD см. в статье [Краткое руководство. Регистрация приложения на платформе Microsoft Identity](../active-directory/develop/quickstart-register-app.md).

## <a name="associate-the-azure-ad-application-to-the-partner-center-tenant"></a>Связывание приложения Azure AD с клиентом центра партнеров

Приложение Azure AD, созданное в портал Azure, должно быть связано с учетной записью центра партнеров. Для этого необходимо выполнить следующие шаги:

1. Войдите в [Центр партнеров](https://partner.microsoft.com/dashboard).
1. В правом верхнем углу щелкните значок шестеренки, а затем выберите **Параметры учетной записи**.
1. В меню **Параметры учетной записи** выберите **Управление пользователями**.
1. Выберите **приложения Azure AD** и щелкните **+ создать приложение Azure AD**.
1. Выберите приложение Azure AD, созданное в портал Azure, а затем нажмите кнопку **Далее**.
1. Установите флажок **Диспетчер (Windows)** и нажмите кнопку **Добавить**.

    :::image type="content" source="./media/analytics-programmatic-access/azure-ad-roles.png" alt-text="На этой странице показана страница Создание приложения Azure AD с флажками для выбора ролей.":::

## <a name="generate-an-azure-ad-token"></a>Создание маркера Azure AD

Необходимо создать маркер Azure AD с помощью идентификатора приложения (клиента). Этот идентификатор помогает однозначно идентифицировать клиентское приложение на платформе Microsoft Identity и секрет клиента из предыдущего шага. Действия по созданию маркера Azure AD см. в статье [вызовы служб с помощью учетных данных клиента (общий секрет или сертификат)](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md).

> [!NOTE]
> Токен действителен в течение одного часа.

## <a name="next-steps"></a>Дальнейшие действия

- [Парадигма программного доступа](analytics-programmatic-access.md)