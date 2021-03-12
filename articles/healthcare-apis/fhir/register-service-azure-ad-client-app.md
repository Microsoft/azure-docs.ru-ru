---
title: Регистрация приложения службы в Azure AD с помощью API Azure для FHIR
description: Узнайте, как зарегистрировать клиентское приложение службы в Azure Active Directory.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 6f7bf122b292ca144eac406957f19a13c7ba6662
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020064"
---
# <a name="register-a-service-client-application-in-azure-active-directory"></a>Регистрация клиентского приложения службы в Azure Active Directory

В этой статье вы узнаете, как зарегистрировать клиентское приложение службы в Azure Active Directory. Регистрации клиентских приложений — это Azure Active Directory представления приложений, которые можно использовать для проверки подлинности и получения маркеров. Клиент службы предназначен для использования приложением для получения маркера доступа без интерактивной проверки подлинности пользователя. Он будет иметь определенные разрешения приложения и использовать секрет приложения (пароль) при получении маркеров доступа.

Чтобы создать новый клиент службы, выполните следующие действия.

## <a name="app-registrations-in-azure-portal"></a>Регистрация приложений в портал Azure

1. В [портал Azure](https://portal.azure.com)перейдите к **Azure Active Directory**.

2. Щелкните **Регистрация приложений**.

    ![портал Azure. Регистрация нового приложения.](media/how-to-aad/portal-aad-new-app-registration.png)

3. Выберите **Новая регистрация**.

4. Присвойте клиенту службы отображаемое имя. Клиентские приложения службы обычно не используют URL-адрес ответа.

    :::image type="content" source="media/service-client-app/service-client-registration.png" alt-text="портал Azure. Новая регистрация клиентского приложения службы.":::

5. Выберите **Зарегистрировать**.

## <a name="api-permissions"></a>Разрешения API

Теперь, когда приложение зарегистрировано, необходимо выбрать, какие разрешения API должны быть запрошены этим приложением от имени пользователей:

1. Выберите **Разрешения API**.
1. Выберите **Добавить разрешение**.

    Если вы используете API Azure для FHIR, вы добавите разрешение для API здравоохранения Azure, выполнив поиск по API-интерфейсам **Azure для здравоохранения** в разделе API, которые **использует Моя организация**. 

    Если вы ссылаетесь на другое приложение ресурсов, выберите [регистрацию приложения API FHIR](register-resource-azure-ad-client-app.md) , созданную ранее в разделе **Мои API**.

    :::image type="content" source="media/service-client-app/service-client-org-api.png" alt-text="Конфиденциальный клиент. Мои API-интерфейсы Организации" lightbox="media/service-client-app/service-client-org-api-expanded.png":::

1. Выберите области (разрешения), которые должны иметь конфиденциальное приложение для запроса от имени пользователя:

    :::image type="content" source="media/service-client-app/service-client-add-permission.png" alt-text="Клиент службы. Делегированные разрешения":::

1. Предоставьте согласие для приложения. Если у вас нет необходимых разрешений, обратитесь к администратору Azure Active Directory.

    :::image type="content" source="media/service-client-app/service-client-grant-permission.png" alt-text="Клиент службы. Предоставление согласия":::

## <a name="application-secret"></a>Секрет приложения

Для получения маркера клиенту службы требуется секрет (пароль).

1. Выберите **Сертификаты и секреты**.
2. Выберите **Создать секрет клиента**.

    ![портал Azure. Секрет клиента службы](media/how-to-aad/portal-aad-register-new-app-registration-SERVICE-CLIENT-SECRET.png)

3. Укажите описание и длительность секрета (1 год, 2 года или никогда).

4. После создания секрета он будет отображаться на портале только один раз. Запишите его и храните в безопасном месте.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как зарегистрировать клиентское приложение службы в Azure Active Directory. Затем проверьте доступ к серверу FHIR с помощью процедуры POST.
 
>[!div class="nextstepaction"]
>[Получение доступа к Azure API для FHIR с помощью Postman](access-fhir-postman-tutorial.md)
