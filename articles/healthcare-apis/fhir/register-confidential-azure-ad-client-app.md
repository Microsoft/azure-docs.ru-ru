---
title: Регистрация конфиденциального клиентского приложения в Azure AD — API Azure для FHIR
description: Зарегистрируйте конфиденциальное клиентское приложение в Azure Active Directory, которое проходит проверку подлинности от имени пользователя и запрашивает доступ к приложениям ресурсов.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 8021fb3fa9f11ef895569f48a2ae21b3f7adcd36
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020134"
---
# <a name="register-a-confidential-client-application-in-azure-active-directory"></a>Регистрация конфиденциального клиентского приложения в Azure Active Directory

В этом руководстве вы узнаете, как зарегистрировать конфиденциальное клиентское приложение в Azure Active Directory. 

Регистрация клиентского приложения — это Azure Active Directory представление приложения, которое можно использовать для проверки подлинности от имени пользователя и запроса доступа к [приложениям ресурсов](register-resource-azure-ad-client-app.md). Конфиденциальное клиентское приложение — это приложение, которое может быть доверенным для хранения секрета и представлять этот секрет при запросе маркеров доступа. Примерами конфиденциальных приложений являются приложения на стороне сервера.

Чтобы зарегистрировать новое конфиденциальное приложение на портале, выполните следующие действия.

## <a name="register-a-new-application"></a>Регистрация нового приложения

1. В [портал Azure](https://portal.azure.com)перейдите к **Azure Active Directory**.

1. Щелкните **Регистрация приложений**.

    ![портал Azure. Регистрация нового приложения.](media/how-to-aad/portal-aad-new-app-registration.png)

1. Выберите **Новая регистрация**.

1. Присвойте приложению отображаемое имя.

1. Укажите URL-адрес ответа. Эти сведения можно изменить позже, но если вы знакомы с URL-адресом ответа приложения, введите его сейчас.

    ![Регистрация нового конфиденциального клиентского приложения.](media/how-to-aad/portal-aad-register-new-app-registration-CONF-CLIENT.png)
1. Выберите **Зарегистрировать**.

## <a name="api-permissions"></a>Разрешения API

Теперь, когда приложение зарегистрировано, необходимо выбрать, какие разрешения API должны быть запрошены этим приложением от имени пользователей:

1. Выберите **Разрешения API**.

    ![Конфиденциальный клиент. Разрешения API](media/how-to-aad/portal-aad-register-new-app-registration-CONF-CLIENT-API-Permissions.png)

1. Выберите **Добавить разрешение**.

    Если вы используете API Azure для FHIR, вы добавите разрешение для API здравоохранения Azure, выполнив поиск по API-интерфейсам **Azure для здравоохранения** в разделе API, которые **использует Моя организация**. Его можно найти, только если вы [развернули API Azure для FHIR](fhir-paas-powershell-quickstart.md).

    Если вы ссылаетесь на другое приложение ресурсов, выберите [регистрацию приложения API FHIR](register-resource-azure-ad-client-app.md) , созданную ранее в разделе **Мои API**.


    :::image type="content" source="media/conf-client-app/confidential-client-org-api.png" alt-text="Конфиденциальный клиент. Мои API-интерфейсы Организации" lightbox="media/conf-client-app/confidential-app-org-api-expanded.png":::
    

3. Выберите области (разрешения), которые должны иметь конфиденциальное приложение для запроса от имени пользователя:

    :::image type="content" source="media/conf-client-app/confidential-client-add-permission.png" alt-text="Конфиденциальный клиент. Делегированные разрешения":::

## <a name="application-secret"></a>Секрет приложения

1. Выберите **Сертификаты и секреты**.
1. Выберите **Создать секрет клиента**. 

    ![Конфиденциальный клиент. Секрет приложения](media/how-to-aad/portal-aad-register-new-app-registration-CONF-CLIENT-SECRET.png)

2. Укажите описание и длительность секрета (1 год, 2 года или никогда).

3. После создания он будет отображаться на портале только один раз. Запишите его и храните в безопасном месте.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как зарегистрировать конфиденциальное клиентское приложение в Azure Active Directory. Далее вы можете получить доступ к серверу FHIR, используя POST
 
>[!div class="nextstepaction"]
>[Получение доступа к Azure API для FHIR с помощью Postman](access-fhir-postman-tutorial.md)
