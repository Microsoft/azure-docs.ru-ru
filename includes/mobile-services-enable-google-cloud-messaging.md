---
title: включить файл
description: включить файл
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 04/06/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: b32ff0926aa2b797bdb8dfcbb01cf845080a0ce9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93376188"
---
1. Перейдите к [консоли Google Cloud](https://console.developers.google.com/cloud-resource-manager)и войдите в нее с помощью учетной записи Google. 
2. Выберите **Создать проект** на панели инструментов. 
   
    ![Создание нового проекта](./media/mobile-services-enable-google-cloud-messaging/mobile-services-google-new-project.png)   
3. В поле **Project name** (Имя проекта) введите имя проекта и нажмите кнопку **Create** (Создать).
4. На панели инструментов щелкните значок **оповещений** и выберите проект в списке. Откроется панель мониторинга проекта. Вы также можете перейти непосредственно к панели мониторинга, используя URL-адрес `https://console.developers.google.com/home/dashboard?project=<YOUR PROJECT NAME>`.

    ![Выбор проекта в окне оповещений](./media/mobile-services-enable-google-cloud-messaging/alert-new-project.png)
5. Запишите **номер проекта**, который отображается на плитке **Project info** (Сведения о проекте) панели мониторинга. 

    ![Идентификатор проекта](./media/mobile-services-enable-google-cloud-messaging/project-number.png)
6. На плитке **APIs** (API-интерфейсы) панели мониторинга щелкните **Go to APIs overview** (Перейти к обзору API-интерфейсов). 

    ![Ссылка на страницу обзора API](./media/mobile-services-enable-google-cloud-messaging/go-to-api-overview.png)
7. На странице **API** щелкните **Enable APIs and services** (Включить API-интерфейсы и службы). 

    ![Кнопка "Enable APIs and Services" (Включить API-интерфейсы и службы)](./media/mobile-services-enable-google-cloud-messaging/enable-api-services-button.png)
8. Найдите и выберите **Google Cloud Messaging**. 

    ![Поиск и выбор Google Cloud Messaging](./media/mobile-services-enable-google-cloud-messaging/search-select-gcm.png)
9. Чтобы включить Google Cloud Messaging для проекта, нажмите кнопку **Enable** (Включить).

    ![Включение Google Cloud Messaging](./media/mobile-services-enable-google-cloud-messaging/enable-gcm-button.png)
10. Щелкните **Create credentials** (Создать учетные данные) на панели инструментов. 

    ![Кнопка "Create credentials" (Создать учетные данные)](./media/mobile-services-enable-google-cloud-messaging/create-credentials-button.png)
11. На странице **Add credentials to your project** (Добавление учетных данных в проект) щелкните ссылку на **ключ API**. 

    ![Добавление учетных данных](./media/mobile-services-enable-google-cloud-messaging/api-key-button.png)    
12. На странице **API key** (Ключ API) нажмите кнопку **Create** (Создать) или Save (Сохранить). В следующем примере выбран параметр **IP addresses** (IP-адреса) и для разрешенных IP-адресов указан диапазон **0.0.0.0/0**. Задайте соответствующие ограничения ключа API. 

    ![Страница ключа API — кнопка Create (Создать)](./media/mobile-services-enable-google-cloud-messaging/api-key-create-button.png)
13. Скопируйте **ключ API** в буфер обмена и сохраните его в каком-нибудь расположении. 

    ![Копирование ключа API](./media/mobile-services-enable-google-cloud-messaging/copy-api-key.png)
   
    Этот ключ API службы Azure будут использовать для аутентификации в службе GCM и отправки push-уведомлений от имени вашего приложения. Чтобы вернуться на панель мониторинга проекта, используйте URL-адрес `https://console.developers.google.com/home/dashboard?project=<YOUR PROJECT NAME>`.

