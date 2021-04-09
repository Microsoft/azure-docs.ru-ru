---
title: Краткое руководство. Создание веб-приложения Node.js в Linux
description: Быстрое развертывание первого приложения Hello World на Node.js в Службе приложений Azure.
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 08/01/2020
ms.custom: mvc, devcenter, seodec18
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 59a2c96987762e6b56cc7b453877cebe3124e443
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102109100"
---
<!-- default for linux -->

## <a name="3-deploy-to-azure-app-service-from-visual-studio-code"></a>3. Развертывание в Службе приложений Azure с помощью Visual Studio Code

1. Откройте папку приложения в Visual Studio Code.

    ```bash
    code .
    ```

1. В обозревателе **Служба приложений Azure** выберите **Войти в Azure...** и следуйте инструкциям. После входа в обозревателе должно отобразиться имя вашей подписки Azure.

    ![Вход в Azure](../media/quickstart-nodejs/sign-in.png)
    <br>
    <details>
    <summary>Устранение неполадок со входом в Azure</summary>
    
    Если при входе в Azure отображается сообщение об ошибке **"Не удается найти подписку с именем [идентификатор подписки]"** , возможно, вы находитесь за прокси-сервером и не можете связаться с API Azure. Укажите в переменных среды `HTTP_PROXY` и `HTTPS_PROXY` параметры прокси-сервера, используя команду терминала `export`.
    
    ```bash
    export HTTPS_PROXY=https://username:password@proxy:8080
    export HTTP_PROXY=http://username:password@proxy:8080
    ```

    [Сообщите о проблеме](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=deploy-app)


1. В обозревателе **Служба приложений Azure** щелкните значок с направленной вверх синей стрелкой, чтобы развернуть приложение в Azure. 

    :::image type="content" source="../media/quickstart-nodejs/deploy.png" alt-text="Снимок экрана Службы приложений Azure в VS Code с выбранным синим значком со стрелкой.":::

1. Выберите каталог `nodejs-docs-hello-world`, который у вас уже открыт.

1. Выберите **Create new Web App** (Создать веб-приложение), которое по умолчанию развертывается в Службе приложений в Linux.

1. Введите глобально уникальное <abbr title="В имени приложения допускаются символы 'a-z', '0-9' и '-'.">name</abbr> для веб-приложения и нажмите клавишу **ВВОД**. 

1. Выберите **версию Node.js**. Мы рекомендуем использовать LTS.

    В канале уведомлений отображаются ресурсы Azure, которые создаются для вашего приложения.

1. Щелкните **Да**, когда появится запрос на обновление конфигурации, чтобы выполнить `npm install` на целевом сервере. Затем приложение развертывается.

    :::image type="content" source="../media/quickstart-nodejs/server-build.png" alt-text="Снимок экрана с запросом на обновление конфигурации на целевом сервере с выбранной кнопкой &quot;Да&quot;.":::

1. Выберите **Да**, когда вам будет предложено обновить рабочую область, чтобы последующие развертывания автоматически ориентировались на то же веб-приложение Службы приложений. 

    :::image type="content" source="../media/quickstart-nodejs/save-configuration.png" alt-text="Снимок экрана с запросом на обновление рабочей области с выбранной кнопкой &quot;Да&quot;.":::




1. По завершении развертывания щелкните **Обзор веб-сайта** в диалоговом окне с предложением просмотреть только что развернутое веб-приложение.

<br/>
<details>
<summary><strong>Устранение неполадок</strong></summary>

Если эти действия не удалось выполнить, проверьте следующее:

* Убедитесь, что приложение прослушивает порт, указанный в переменной среды PORT: `process.env.PORT`.

* Если отображается сообщение об ошибке **Вы не имеете разрешения на просмотр этого каталога пли страницы.** , значит приложение не может нормально запуститься. Просмотрите выходные данные, чтобы найти и исправить ошибку. 

</details>

<br>

[Сообщите о проблеме](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&prepare-your-environment)


<br/>
<hr/>


