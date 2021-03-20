---
title: Обзор Azure Cloud Shell | Документация Майкрософт
description: Обзор Azure Cloud Shell.
services: ''
documentationcenter: ''
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/20/2020
ms.author: damaerte
ms.openlocfilehash: f824bddf833a1e2c01a3b779abc2c5252d8e0547
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89468663"
---
# <a name="overview-of-azure-cloud-shell"></a>Обзор Azure Cloud Shell

Azure Cloud Shell — это интерактивная, аутентифицированная, доступная в браузере оболочка для управления ресурсами Azure. Она предоставляет гибкие возможности при выборе оболочки, соответствующей вашим методам работы, будь то Bash или PowerShell.

Доступ к Cloud Shell можно получить тремя способами:

- **Прямая ссылка**: Откройте браузер в [https://shell.azure.com](https://shell.azure.com) .

- **Портал Azure**: выберите значок Cloud Shell на [портал Azure](https://portal.azure.com):

    ![Значок запуска Cloud Shell из портал Azure](media/overview/portal-launch-icon.png)

- **Фрагменты кода**: в [docs.Microsoft.com]() и [Microsoft Learn](/learn/)нажмите кнопку **попробовать** , которая появляется с Azure CLI и Azure PowerShell фрагментами кода:

    ```azurecli-interactive
    az account show
    ```

    ```azurepowershell-interactive
    Get-AzSubscription
    ```

    Кнопка **попробовать** . открывает Cloud Shell непосредственно вместе с документацией с помощью bash (для фрагментов кода Azure CLI) или PowerShell (для Azure PowerShell фрагментов кода).

    Чтобы выполнить команду, воспользуйтесь командой **Копировать** в фрагменте кода, нажмите **клавиши CTRL** + **SHIFT** + **v** (Windows/Linux) или **cmd** + **SHIFT** + **v** (macOS), чтобы вставить команду, а затем нажмите клавишу **Ввод**.

## <a name="features"></a>Компоненты

### <a name="browser-based-shell-experience"></a>Оболочка на основе браузера

Cloud Shell предоставляет доступ к браузерному интерфейсу командной строки, созданному с учетом задач управления Azure. Используйте Cloud Shell для работы из локального компьютера с неограниченными возможностями, которые может обеспечить только облако.

### <a name="choice-of-preferred-shell-experience"></a>Выбор предпочтительной оболочки

Пользователи могут выбрать между bash или PowerShell.

1. Выберите **Cloud Shell**.

    ![Значок Cloud Shell](media/overview/overview-cloudshell-icon.png)

2. Выберите **Bash** или **PowerShell**.

    ![Выберите bash или PowerShell](media/overview/overview-choices.png)

    После первого запуска можно использовать раскрывающийся список тип оболочки для переключения между Bash и PowerShell:

    ![Элемент управления "раскрывающийся список" для выбора bash или PowerShell](media/overview/select-shell-drop-down.png)

### <a name="authenticated-and-configured-azure-workstation"></a>Настроенная и аутентифицированная рабочая станция Azure

Службой Cloud Shell управляет Майкрософт, поэтому в ней присутствуют популярные программы командной строки и реализована поддержка различных языков. Кроме того, Cloud Shell автоматически безопасно проходит проверку подлинности, тем самым обеспечивая мгновенный доступ к ресурсам с помощью Azure CLI или командлетов Azure PowerShell.

Просмотрите полный список [средств, установленных в Cloud Shell.](features.md#tools)

### <a name="integrated-cloud-shell-editor"></a>Интегрированный редактор Cloud Shell

Cloud Shell предоставляет интегрированный графический текстовый редактор на основе редактора Monaco с открытым кодом. Просто создайте и измените файлы конфигурации, выполнив `code .` для размеренного развертывания с помощью Azure CLI 2.0 или Azure PowerShell.

[Подробнее о редакторе Cloud Shell.](using-cloud-shell-editor.md)

### <a name="multiple-access-points"></a>Несколько точек доступа

Cloud Shell представляет собой гибкий инструмент, который можно использовать с помощью таких элементов:

* [portal.azure.com](https://portal.azure.com)
* [shell.azure.com](https://shell.azure.com);
* [документации по Azure CLI](/cli/azure).
* [Документация по Azure PowerShell](/powershell/azure/)
* [Мобильное приложение Azure](https://azure.microsoft.com/features/azure-portal/mobile-app/)
* [Расширение учетной записи Azure для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account).

### <a name="connect-your-microsoft-azure-files-storage"></a>Подключение хранилища файлов Microsoft Azure

Cloud Shell машины являются временными, но файлы сохраняются двумя способами: через образ диска и через подключенный файловый ресурс с именем `clouddrive` . При первом запуске Cloud Shell предлагает создать группу ресурсов, учетную запись хранения и файловый ресурс Azure от вашего имени. Это одноразовое действие, которое автоматически применяется для всех сеансов. Можно сопоставить отдельный файловый ресурс, который будет использоваться Bash и PowerShell в Cloud Shell.

Дополнительные сведения см. в статье как подключить [новую или существующую учетную запись хранения](persisting-shell-storage.md) или узнать о [механизмах сохранения, используемых в Cloud Shell](persisting-shell-storage.md#how-cloud-shell-storage-works).

> [!NOTE]
> Брандмауэр службы хранилища Azure не поддерживается для учетных записей хранения Cloud Shell.

## <a name="concepts"></a>Основные понятия

* Cloud Shell работает на временном узле, предоставляемом для каждого сеанса и для каждого пользователя отдельно.
* Время ожидания Cloud Shell истекает через 20 минут при отсутствии интерактивных действий.
* Для Cloud Shell требуется подключение файлового ресурса Azure.
* Cloud Shell использует один и тот же файловый ресурс Azure для Bash и PowerShell.
* Cloud Shell назначается один компьютер на учетную запись пользователя.
* Cloud Shell сохраняет каталог $Home с помощью образа размером 5 ГБ, размещенного в общей папке.
* Разрешения задаются как для обычного пользователя Linux в Bash.

Узнайте больше о возможностях [Bash в Cloud Shell](features.md) и [PowerShell в Cloud Shell](./features.md).

## <a name="pricing"></a>Цены

За использование компьютера, на котором размещена среда Cloud Shell, плата не взимается. На этом компьютере должен быть подключен файловый ресурс Azure. Применяются расходы на обычное хранение.

## <a name="next-steps"></a>Дальнейшие действия

[Краткое руководство по использованию Bash в Azure Cloud Shell](quickstart.md) <br>
[Краткое руководство по использованию PowerShell в Azure Cloud Shell (предварительная версия)](quickstart-powershell.md)