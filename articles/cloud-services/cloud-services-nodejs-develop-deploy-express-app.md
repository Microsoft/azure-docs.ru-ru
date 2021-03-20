---
title: Создание и развертывание приложения Node.js Express в облачных службах Azure (классическая модель)
description: Используйте этот учебник для создания нового приложения с помощью модуля Express, который предоставляет платформу MVC для создания веб-приложений Node.js.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: e15af589b3a3c496738c97c0c2c6429ba708ba7e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98743344"
---
# <a name="build-and-deploy-a-nodejs-web-application-using-express-on-an-azure-cloud-services-classic"></a>Создание и развертывание Node.js веб-приложения с помощью Express в облачных службах Azure (классическая модель)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

Node.js включает минимальный набор функциональных возможностей в базовой среде выполнения.
Разработчики часто используют сторонние модули для получения дополнительной функциональности при разработке приложения Node.js. В этом руководстве будет создано новое приложение с помощью модуля [Express](https://github.com/expressjs/express), который предоставляет платформу MVC для создания веб-приложений Node.js.

Снимок экрана завершенного приложения приведен ниже:

![Веб-браузер, отображающий приветствие модуля Express в Azure](./media/cloud-services-nodejs-develop-deploy-express-app/node36.png)

## <a name="create-a-cloud-service-project"></a>Создание проекта облачной службы
[!INCLUDE [install-dev-tools](../../includes/install-dev-tools.md)]

Чтобы создать новый проект облачной службы с именем expressapp, выполните следующие действия.

1. В **меню Пуск** или на **начальном экране** найдите **Windows PowerShell**. Щелкните правой кнопкой мыши **Windows PowerShell** и выберите **Запуск от имени администратора**.

    ![Значок Azure PowerShell](./media/cloud-services-nodejs-develop-deploy-express-app/azure-powershell-start.png)
2. Перейдите в каталог **c:\\node**, а затем введите указанные ниже команды для создания решения с именем **expressapp** и рабочей роли с именем **WebRole1**.

   ```powershell
   PS C:\node> New-AzureServiceProject expressapp
   PS C:\Node\expressapp> Add-AzureNodeWebRole
   PS C:\Node\expressapp> Set-AzureServiceProjectRole WebRole1 Node 0.10.21
   ```

   > [!NOTE]
   > По умолчанию **Add-AzureNodeWebRole** использует предыдущую версию Node.js. Показанная выше инструкция **Set-AzureServiceProjectRole** предписывает Azure использовать Node v0.10.21.  Обратите внимание, что параметры зависят от регистра.  Можно проверить, выбрана ли правильная версия Node.js, выбрав свойство **engines** в **WebRole1\package.json**.
>
>

## <a name="install-express"></a>Установка модуля Express
1. Установите генератор Express, выполнив следующую команду:

    ```powershell
    PS C:\node\expressapp> npm install express-generator -g
    ```

    Результат команды npm должен соответствовать показанному ниже результату.

    ![Windows PowerShell с выходными данными команды npm install express.](./media/cloud-services-nodejs-develop-deploy-express-app/express-g.png)
2. Перейдите в каталог **WebRole1** и используйте команду express для создания нового приложения.

    ```powershell
    PS C:\node\expressapp\WebRole1> express
    ```

    Вам будет предложено перезаписать более раннюю версию приложения. Для продолжения введите **y** или **yes**. Модуль Express сформирует файл app.js и структуру папок для создания вашего приложения.

    ![Результат команды express](./media/cloud-services-nodejs-develop-deploy-express-app/node23.png)
3. Чтобы установить дополнительные зависимости, определенные в файле package.json, введите следующую команду:

    ```powershell
    PS C:\node\expressapp\WebRole1> npm install
    ```

   ![Результат команды npm install](./media/cloud-services-nodejs-develop-deploy-express-app/node26.png)
4. Скопируйте файл **bin/www** в **server.js** с помощью следующей команды: Таким образом, облачная служба сможет найти точку входа в это приложение.

    ```powershell
    PS C:\node\expressapp\WebRole1> copy bin/www server.js
    ```

   После выполнения команды файл **server.js** должен содержаться в каталоге WebRole1.
5. В файле **server.js** удалите одну из точек (символ ".") из строки, приведенной ниже.

    ```js
    var app = require('../app');
    ```

   После сделанных изменений строка должна выглядеть следующим образом.

    ```js
    var app = require('./app');
    ```

   Это необходимо сделать, так как мы перенесли файл (ранее **bin/www**) в тот же каталог, в котором должен находиться файл приложения. После внесения изменений сохраните файл **server.js** .
6. Выполнив следующую команду, запустите приложение в эмуляторе Azure:

    ```powershell
    PS C:\node\expressapp\WebRole1> Start-AzureEmulator -launch
    ```

    ![Веб-страница, содержащая приветствие модуля express.](./media/cloud-services-nodejs-develop-deploy-express-app/node28.png)

## <a name="modifying-the-view"></a>Изменение представления
Теперь измените представление для отображения сообщения «Вас приветствует Express в Azure».

1. Чтобы открыть файл index.jade, введите следующую команду:

    ```powershell
    PS C:\node\expressapp\WebRole1> notepad views/index.jade
    ```

   ![Содержимое файла index.jade.](./media/cloud-services-nodejs-develop-deploy-express-app/getting-started-19.png)

   Jade — это обработчик представлений по умолчанию, используемый приложениями Express. Дополнительные сведения о подсистеме представлений Jade см. в разделе [http://jade-lang.com][http://jade-lang.com] .
2. Измените последнюю строку текста, добавив слова **in Azure**.

   ![Файл index.jade, последняя строка содержит текст: p Welcome to \#{title} in Azure.](./media/cloud-services-nodejs-develop-deploy-express-app/node31.png)
3. Сохраните файл и выйдите из Блокнота.
4. Обновите браузер и увидите изменения.

   ![Окно браузера со страницей приветствия Express в Azure](./media/cloud-services-nodejs-develop-deploy-express-app/node32.png)

После тестирования приложения остановите эмулятор с помощью командлета **Stop-AzureEmulator** .

## <a name="publishing-the-application-to-azure"></a>Публикация приложения в Azure
В окне Azure PowerShell используйте командлет **Publish-AzureServiceProject** , чтобы развернуть приложение в облачной службе.

```powershell
PS C:\node\expressapp\WebRole1> Publish-AzureServiceProject -ServiceName myexpressapp -Location "East US" -Launch
```

После завершения операции развертывания откроется браузер с отображением веб-страницы.

![В веб-браузере отображается страница Express. URL-адрес указывает, что страница теперь размещается в Azure.](./media/cloud-services-nodejs-develop-deploy-express-app/node36.png)

## <a name="next-steps"></a>Дальнейшие действия
Дополнительную информацию см. в [центре разработчиков Node.js](/azure/developer/javascript/).

[Node.js Web Application]: https://www.windowsazure.com/develop/nodejs/tutorials/getting-started/
[Express]: https://expressjs.com/
[http://jade-lang.com]: http://jade-lang.com