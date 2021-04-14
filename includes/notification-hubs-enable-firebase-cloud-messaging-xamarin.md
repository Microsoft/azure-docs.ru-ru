---
title: Включить файл
description: включить файл
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 08/01/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 45bdd569741dc13181bcaf9e8587a02b3d02c621
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "68728825"
---
1. Войдите в [консоль Firebase](https://firebase.google.com/console/). Создайте проект Firebase, если его еще нет.
2. После создания проекта выберите **Add Firebase to your Android app** (Добавить Firebase в приложение Android). 

    ![Добавление Firebase в приложение Android](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-add-firebase-to-android-app.png)

3. Выполните следующие действия на странице **Add Firebase to your Android app** (Добавление Firebase в приложение Android): 
    1. В поле **Android package name** (Имя пакета Android) введите имя пакета. Например: `tutorials.tutoria1.xamarinfcmapp`. 

        ![Указание имени пакета.](./media/notification-hubs-enable-firebase-cloud-messaging/specify-package-name-fcm-settings.png)
    2. Выберите **Регистрация приложения**.  
    1. Выберите **Download google-services.json** (Скачать Download google-services.json). Затем сохраните файл в папку проекта и щелкните **Далее**. Если вы еще не создали проект Visual Studio, этот шаг можно выполнить после создания проекта. 

        ![Загрузка файла google-services.json.](./media/notification-hubs-enable-firebase-cloud-messaging/download-google-service-button.png)
    6. Выберите **Далее**. 
    7. Выберите **Пропустить этот шаг**. 

        ![Пропуск последнего шага.](./media/notification-hubs-enable-firebase-cloud-messaging/skip-this-step.png)
8. В консоли Firebase щелкните значок шестеренки возле имени проекта. Выберите пункт **Project Settings** (Параметры проекта).

    ![Выбор параметров проекта](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-firebase-console-project-settings.png)
4. Если вы еще не скачали файл **google-services.json**, вы можете скачать его на этой странице. 

    ![Скачайте файл google-services.json на вкладке "Общие".](./media/notification-hubs-enable-firebase-cloud-messaging/download-google-services-json-general-page.png)
1. Переключитесь на вкладку **Обмен сообщениями в облаке** в верхней части. Скопируйте и сохраните **Ключ сервера** для последующего использования. Это значение будет использоваться для настройки концентратора уведомлений.

    ![Копирование ключа сервера](./media/notification-hubs-enable-firebase-cloud-messaging/server-key.png)
