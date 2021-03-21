---
title: Настройка экземпляра и проверки подлинности (с использованием скрипта)
titleSuffix: Azure Digital Twins
description: Сведения о настройке экземпляра службы Digital двойников см. в разделе Создание сценария автоматического развертывания.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: contperf-fy21q3
ms.openlocfilehash: 13f5db52917cd9659438c551ac2f01c477cc1c7b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102201300"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-scripted"></a>Настройка экземпляра Azure Digital двойников и проверки подлинности (с помощью сценария)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

В этой статье описаны действия по **настройке нового экземпляра Azure Digital двойников**, включая создание экземпляра и настройку проверки подлинности. По завершении работы с этой статьей у вас будет готовый экземпляр Azure Digital двойников для начала программирования.

Эта версия этой статьи завершает эти действия, запуская [пример **скрипта автоматического развертывания**](/samples/azure-samples/digital-twins-samples/digital-twins-samples/) , который упрощает процесс. 
* Для просмотра действий CLI вручную, выполняемых сценарием в фоновом режиме, см. версию CLI этой статьи: [*инструкции по настройке экземпляра и проверки подлинности (CLI)*](how-to-set-up-instance-cli.md).
* Чтобы просмотреть действия вручную в соответствии с портал Azure, см. версию портала этой статьи: инструкции по [*настройке экземпляра и проверки подлинности (портал)*](how-to-set-up-instance-portal.md).

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

## <a name="prerequisites-download-the-script"></a>Предварительные требования: скачивание скрипта

Пример скрипта написан на PowerShell. Это часть [**комплексных примеров Azure Digital двойников**](/samples/azure-samples/digital-twins-samples/digital-twins-samples/), которые можно загрузить на компьютер, перейдя по этой ссылке и нажав кнопку *обзора кода* под заголовком. Вы перейдете в репозиторий GitHub для примеров, которые можно скачать как *ZIP*-файлы. Для этого нажмите кнопку *Код* и выберите элемент *Скачать ZIP-файл*.

:::image type="content" source="media/includes/download-repo-zip.png" alt-text="Представление репозитория примеров Digital Twins в GitHub. Выбрана кнопка &quot;Код&quot;, в результате чего отображается небольшое диалоговое окно с выделенной кнопкой &quot;Скачать ZIP-файл&quot;." lightbox="media/includes/download-repo-zip.png":::

На ваш компьютер будет скачана *ZIP*-папка с именем **digital-twins-samples-master.zip**. Перейдите в папку на компьютере и распакуйте ее, чтобы извлечь файлы.

В распакованной папке скрипт развертывания находится в сценариях _Digital-двойников-Samples-master > > **deploy.ps1**_.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="run-the-deployment-script"></a>Выполнение скрипта развертывания

В этой статье используется пример кода Azure Digital двойников для развертывания экземпляра цифрового двойников Azure и автоматической проверки подлинности. Его также можно использовать в качестве отправной точки для написания собственных взаимодействий со сценариями.

Ниже приведены действия по запуску скрипта развертывания в Cloud Shell.
1. Перейдите в окно [Azure Cloud Shell](https://shell.azure.com/) в браузере. Выполните вход с помощью следующей команды:
    ```azurecli-interactive
    az login
    ```
    Если в CLI можно запустить браузер по умолчанию, откроется браузер со страницей входа. В противном случае самостоятельно откройте в браузере страницу *https://aka.ms/devicelogin* и введите код авторизации, отображаемый в терминале.
 
2. На панели значков Cloud Shell убедитесь, что Cloud Shell настроена для запуска версии PowerShell.

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-powershell.png" alt-text="Окно Cloud Shell, в котором отображается выбор версии PowerShell":::

1. Выберите значок "Отправить/скачать файлы" и нажмите кнопку "Отправить".

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-upload.png" alt-text="Окно Cloud Shell, показывающее значок отправки":::

    Перейдите к файлу _**deploy.ps1**_ на компьютере (в папке _Digital-двойников-Samples-Master > scripts > **deploy.ps1**_) и нажмите кнопку "Открыть". Файл будет отправлен в Cloud Shell, чтобы его можно было запустить в окне Cloud Shell.

4. Запустите сценарий, отправив `./deploy.ps1` команду в окне Cloud Shell. Вы можете скопировать приведенную ниже команду (Напомним, что для вставки в Cloud Shell можно использовать **сочетание клавиш CTRL + SHIFT + v** в Windows и Linux или **CMD + SHIFT + v** в macOS. Вы также можете использовать контекстное меню.

    ```azurecli-interactive
    ./deploy.ps1
    ```

    Скрипт создаст экземпляр Azure Digital двойников и присвоит пользователю Azure роль *владельца данных Digital двойников* в экземпляре.

    Так как сценарий выполняется с помощью действий автоматической установки, вам будет предложено передать следующие значения:
    * Для экземпляра: *идентификатор подписки* Azure для использования.
    * Для экземпляра: *Расположение* , в котором вы хотите развернуть экземпляр. Чтобы узнать, какие регионы поддерживают Azure Digital двойников, посетите страницу [*продукты Azure, доступные по регионам*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
    * Для экземпляра: имя *группы ресурсов* . Вы можете использовать существующую группу ресурсов или ввести новое имя, чтобы создать его.
    * Для экземпляра: *имя* вашего экземпляра Digital двойников для Azure. Если в вашей подписке есть другой экземпляр Azure Digital двойников в регионе, где уже используется указанное имя, вам будет предложено выбрать другое имя.

Ниже приведен фрагмент журнала вывода из скрипта:

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/deployment-script-output.png" alt-text="Cloud Shell окно, показывающее журнал входных и выходных данных при выполнении скрипта развертывания" lightbox="media/how-to-set-up-instance/cloud-shell/deployment-script-output.png":::

Если сценарий завершается успешно, окончательная печать будет говорить `Deployment completed successfully` . В противном случае выполняйте сообщение об ошибке и повторно запустите скрипт. Он обходит уже выполненные действия и снова запросит входные данные в том месте, где вы остановились.

> [!NOTE]
> В настоящее время сценарий назначает требуемую роль управления в Azure Digital двойников (*владелец данных Azure Digital двойников*) тому же пользователю, который запускает скрипт из Cloud Shell. Если вам нужно назначить эту роль другому пользователю, который будет управлять экземпляром, это можно сделать сейчас с помощью портал Azure ([инструкции](how-to-set-up-instance-portal.md#set-up-user-access-permissions)) или CLI ([инструкции](how-to-set-up-instance-cli.md#set-up-user-access-permissions)).

>[!NOTE]
>В настоящее время существует **известная ошибка** установки сценариев, в которой некоторые пользователи (особенно пользователи в личных [учетных записях Майкрософт (MSAS)](https://account.microsoft.com/account)) могут найти **назначение ролей для _владельца данных Azure Digital двойников_ не было создано**.
>
>Вы можете проверить назначение ролей в разделе [*Проверка назначения роли пользователя*](#verify-user-role-assignment) далее в этой статье. при необходимости настройте назначение ролей вручную с помощью [портал Azure](how-to-set-up-instance-portal.md#set-up-user-access-permissions) или [CLI](how-to-set-up-instance-cli.md#set-up-user-access-permissions).
>
>Дополнительные сведения об этой проблеме см. в статье [*Устранение неполадок: известные проблемы в Azure Digital Twins*](troubleshoot-known-issues.md#missing-role-assignment-after-scripted-setup).

## <a name="verify-success-and-collect-important-values"></a>Проверка успешности и получение важных значений

Чтобы проверить создание ресурсов и разрешений, настроенных сценарием, можно просмотреть их в [портал Azure](https://portal.azure.com) с помощью приведенных ниже инструкций. Если вы не можете проверить успешность какого бы то ни было действия, повторите шаг. Эти действия можно выполнить по отдельности с помощью [портал Azure](how-to-set-up-instance-portal.md) или инструкций [CLI](how-to-set-up-instance-cli.md) .

В этом разделе также показано, как найти важные значения в ресурсах, настроенных скриптом, который может потребоваться при продолжении работы с экземпляром Digital двойников для Azure. Эти значения следует сохранить в надежном месте или вернуться к этому разделу, чтобы найти их позже по мере необходимости. Если другие пользователи будут программировать на экземпляре, вы также должны использовать эти значения совместно с ними.

### <a name="verify-instance"></a>Проверить экземпляр

Чтобы проверить, создан ли экземпляр, перейдите на [страницу цифровой двойников Azure](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) в портал Azure. Вы можете перейти на эту страницу самостоятельно, выполнив поиск по запросу *Azure Digital двойников* на панели поиска портала.

На этой странице перечислены все экземпляры Azure Digital двойников. Найдите имя созданного экземпляра в списке.

Если проверка завершилась неудачно, можно повторить попытку создания экземпляра с помощью [портала](how-to-set-up-instance-portal.md#create-the-azure-digital-twins-instance) или [интерфейса командной строки](how-to-set-up-instance-cli.md#create-the-azure-digital-twins-instance).

### <a name="collect-instance-values"></a>Сбор значений экземпляров

Выберите имя своего экземпляра на [странице Azure Digital двойников](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) , чтобы открыть страницу *обзора* экземпляра. Запомните его *имя*, *группу ресурсов* и *имя узла*. Эти данные могут потребоваться позже для обнаружения экземпляра и подключения к нему.

:::image type="content" source="media/how-to-set-up-instance/portal/instance-important-values.png" alt-text="Выделение важных значений на странице обзора экземпляра":::

### <a name="verify-user-role-assignment"></a>Проверка назначения роли пользователя

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

> [!NOTE]
> Помните, что сценарий в настоящее время назначает эту требуемую роль тому же пользователю, который запускает скрипт из Cloud Shell. Если вам нужно назначить эту роль другому пользователю, который будет управлять экземпляром, это можно сделать сейчас с помощью портал Azure ([инструкции](how-to-set-up-instance-portal.md#set-up-user-access-permissions)) или CLI ([инструкции](how-to-set-up-instance-cli.md#set-up-user-access-permissions)).

Если проверка завершилась неудачно, можно также повторить собственное назначение ролей с помощью [портала](how-to-set-up-instance-portal.md#set-up-user-access-permissions) или [интерфейса командной строки](how-to-set-up-instance-cli.md#set-up-user-access-permissions).

## <a name="next-steps"></a>Следующие шаги

Вытестируйте отдельные REST API вызовы в экземпляре с помощью команд CLI Azure Digital двойников: 
* [AZ DT Справочник](/cli/azure/ext/azure-iot/dt)
* [*Практическое руководство. Использование CLI для Azure Digital Twins*](how-to-use-cli.md)

Или см. раздел как подключить клиентское приложение к экземпляру с помощью кода проверки подлинности:
* [*Пошаговое руководство. Написание кода проверки подлинности приложения*](how-to-authenticate-client.md)