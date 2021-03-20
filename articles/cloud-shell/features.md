---
title: Функции Azure Cloud Shell | Документы Майкрософт
description: Общие сведения о функциях в Azure Cloud Shell
services: Azure
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
ms.date: 04/26/2019
ms.author: damaerte
ms.openlocfilehash: a052364b06ac1b9b30cef76db10a79e8ed85b9a3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89470159"
---
# <a name="features--tools-for-azure-cloud-shell"></a>Функции и средства для Azure Cloud Shell

[!INCLUDE [features-introblock](../../includes/cloud-shell-features-introblock.md)]

Azure Cloud Shell выполняется в `Ubuntu 16.04 LTS`.

## <a name="features"></a>Компоненты

### <a name="secure-automatic-authentication"></a>Безопасная автоматическая аутентификация

Cloud Shell безопасно и автоматически выполняет проверку подлинности при доступе к учетным записям для Azure CLI и Azure PowerShell.

### <a name="home-persistence-across-sessions"></a>Сохранение каталога $Home между сеансами

Чтобы сохранять файлы между сеансами, при первом запуске Cloud Shell предлагается присоединить общую папку Azure.
По завершении настройки Cloud Shell будет автоматически подключать хранилище (подключенное как `$HOME\clouddrive`) для всех будущих сеансов.
Кроме того, ваш каталог `$HOME` сохраняется в виде IMG-файла в файловом ресурсе Azure.
Файлы вне `$HOME` и состояние компьютера не сохраняются между сеансами. Следуйте рекомендациям при хранении секретов, например ключей SSH. Для таких служб, как [Azure Key Vault, предусмотрены руководства по настройке](../key-vault/general/manage-with-cli2.md#prerequisites).

[Дополнительные сведения о сохранении файлов в Cloud Shell.](persisting-shell-storage.md)

### <a name="azure-drive-azure"></a>Диск Azure (Azure:)

PowerShell в Cloud Shell предоставляет диск Azure ( `Azure:` ). Вы можете переключиться на диск Azure с помощью `cd Azure:` и вернуться к домашнему каталогу с помощью `cd  ~` .
Диск Azure упрощает обнаружение ресурсов Azure и перемещение по ним, включая вычислительные ресурсы, сетевые ресурсы, ресурсы хранилища и т. д., предоставляя возможности навигации как у файловой системы.
Для управления этими ресурсами можно воспользоваться привычными [командлетами Azure PowerShell](/powershell/azure), независимо от используемого диска.
Любые изменения, внесенные в ресурсы Azure непосредственно на портале Azure или с помощью командлетов Azure PowerShell, отражаются на диске Azure.  Для обновления ресурсов можно запустить `dir -Force`.

![Снимок экрана Azure Cloud Shell инициализации и списка ресурсов каталога.](media/features-powershell/azure-drive.png)

### <a name="manage-exchange-online"></a>Управление Exchange Online

PowerShell в Cloud Shell содержит частную сборку модуля Exchange Online.  Выполните команду `Connect-EXOPSSession` , чтобы получить командлеты Exchange.

![Снимок экрана Azure Cloud Shell выполнение команд Connect-EXOPSSession и Get-User.](media/features-powershell/exchangeonline.png)

 Выполнить `Get-Command -Module tmp_*`
> [!NOTE]
> Имя модуля должно начинаться с `tmp_` , если установлены модули с таким же префиксом, также отображаются их командлеты. 

![Снимок экрана Azure Cloud Shell, выполняющего команду Get-Command-Module tmp_ *.](media/features-powershell/exchangeonlinecmdlets.png)

### <a name="deep-integration-with-open-source-tooling"></a>Глубокая интеграция со средствами с открытым кодом

В Cloud Shell предварительно настроена аутентификация для таких средств с открытым кодом, как Terraform, Ansible и Chef InSpec. Попробуйте поработать с ней при помощи пошаговых руководств с примерами.

## <a name="tools"></a>Инструменты

|Category   |Имя   |
|---|---|
|Средства Linux            |bash<br> zsh<br> sh<br> tmux<br> dig<br>               |
|Инструменты Azure            |[Azure CLI](https://github.com/Azure/azure-cli) и [классический интерфейс командной строки Azure](https://github.com/Azure/azure-xplat-cli)<br> [AzCopy](../storage/common/storage-use-azcopy-v10.md)<br> [CLI службы "Функции Azure"](https://github.com/Azure/azure-functions-core-tools)<br> [Интерфейс командной строки Service Fabric](../service-fabric/service-fabric-cli.md)<br> [Batch Shipyard](https://github.com/Azure/batch-shipyard)<br> [blobxfer](https://github.com/Azure/blobxfer)|
|Текстовые редакторы           |code (редактор Cloud Shell)<br> vim<br> nano<br> emacs    |
|Система управления версиями         |git                    |
|Инструменты сборки            |make<br> maven<br> npm<br> pip         |
|Контейнеры             |[Виртуальная машина Docker](https://github.com/docker/machine)<br> [Kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/).<br> [Helm](https://github.com/kubernetes/helm)<br> [Интерфейс командной строки DC/OS](https://github.com/dcos/dcos-cli)         |
|Базы данных              |Клиент MySQL<br> Клиент PostgreSQL<br> [Программа sqlcmd](/sql/tools/sqlcmd-utility)<br> [mssql-scripter](https://github.com/Microsoft/sql-xplat-cli) |
|Другие                  |Клиент iPython<br> [Интерфейс командной строки Cloud Foundry](https://github.com/cloudfoundry/cli)<br> [Terraform](https://www.terraform.io/docs/providers/azurerm/)<br> [Ansible](https://www.ansible.com/microsoft-azure)<br> [Chef InSpec](https://www.chef.io/inspec/)<br> [Puppet](https://puppet.com/docs/bolt/latest/bolt.html)<br> [HashiCorp Packer](https://www.packer.io/)<br> [Интерфейс командной строки Office 365](https://pnp.github.io/office365-cli/)|

## <a name="language-support"></a>Поддержка языков

|Язык   |Версия   |
|---|---|
|.NET Core  |[3.1.302](https://github.com/dotnet/core/blob/master/release-notes/3.1/3.1.6/3.1.302-download.md)       |
|Go         |1,9        |
|Java       |1.8        |
|Node.js    |8.16.0      |
|PowerShell |[7.0.0](https://github.com/PowerShell/powershell/releases)       |
|Python     |2.7 и 3.5 (по умолчанию)|

## <a name="next-steps"></a>Дальнейшие действия
[Быстрый запуск Bash в Cloud Shell](quickstart.md) <br>
[Быстрый запуск PowerShell в Cloud Shell](quickstart-powershell.md) <br>
[Справочник команд Azure CLI](/cli/azure/) <br>
[Дополнительные сведения об Azure PowerShell](/powershell/azure/) <br>