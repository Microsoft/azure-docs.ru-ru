---
title: Настройка сред разработки и развертывания Бицеп
description: Настройка сред разработки и развертывания Бицеп
ms.topic: conceptual
ms.date: 03/26/2021
ms.openlocfilehash: 0e62e6a4633bee09fcbe8b783118cc95ccd5702e
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105626107"
---
# <a name="install-bicep-preview"></a>Установка Бицеп (Предварительная версия)

Узнайте, как настроить среду разработки и развертывания Бицеп.

## <a name="development-environment"></a>Среда разработки

Чтобы получить наилучший опыт создания Бицеп, вам потребуется два компонента:

- **Расширение бицеп для Visual Studio Code**. Для создания файлов Бицеп необходим хороший редактор Бицеп. Рекомендуется [Visual Studio Code](https://code.visualstudio.com/) с [расширением бицеп](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep). Эти средства обеспечивают поддержку языка и автозавершение ресурсов. Они помогают создавать и проверять файлы Бицеп. Дополнительные сведения об использовании Visual Studio Code и расширении Бицеп см. в разделе [Краткое руководство. Создание файлов бицеп с помощью Visual Studio Code](./quickstart-create-bicep-use-visual-studio-code.md).
- **БИЦЕП CLI**. Используйте Бицеп CLI для компиляции файлов Бицеп в шаблоны JSON в ARM и декомпиляцию шаблонов JSON ARM в файлы Бицеп. Инструкции по установке см. в разделе [Install БИЦЕП CLI](#install-manually).

## <a name="deployment-environment"></a>Среда развертывания

Для развертывания локальных файлов Бицеп необходимы два компонента:

- **Azure CLI версии 2.20.0 или более поздней или Azure PowerShell версии 5.6.0 или более поздней**. Ознакомьтесь с инструкциями по установке:

  - [Установка Azure PowerShell](/powershell/azure/install-az-ps)
  - [Установка Azure CLI в Windows](/cli/azure/install-azure-cli-windows)
  - [Установка Azure CLI в Linux](/cli/azure/install-azure-cli-linux)
  - [Установка Azure CLI в macOS](/cli/azure/install-azure-cli-macos)

  > [!NOTE]
  > В настоящее время и Azure CLI, и Azure PowerShell могут развертывать только локальные файлы Бицеп. Дополнительные сведения о развертывании файлов Бицеп с помощью Azure CLI см. в разделе [deploy-CLI](./deploy-cli.md#deploy-remote-template). Дополнительные сведения о развертывании файлов Бицеп с помощью Azure PowerShell см. в разделе [deploy-PowerShell]( ./deploy-powershell.md#deploy-remote-template).

- **БИЦЕП CLI**. Бицеп CLI требуется для компиляции файлов Бицеп в шаблоны JSON перед развертыванием. Инструкции по установке см. в разделе [Install БИЦЕП CLI](#install-bicep-cli).

После установки компонентов можно развернуть файл Бицеп с помощью:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateFile <path-to-template-or-bicep> `
  -storageAccountType Standard_GRS
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file <path-to-template-or-bicep> \
  --parameters storageAccountType=Standard_GRS
```

---

## <a name="install-bicep-cli"></a>Установка Бицеп CLI

- Сведения об использовании Бицеп CLI для компиляции и декомпиляции файлов Бицеп см. в разделе [Installing (установка вручную](#install-manually)).
- Сведения об использовании Azure CLI для развертывания файлов Бицеп см. в разделе [использование с Azure CLI](#use-with-azure-cli).
- Сведения об использовании Azure PowerShell для развертывания файлов Бицеп см. в разделе [использование с Azure PowerShell](#use-with-azure-powershell).

### <a name="use-with-azure-cli"></a>Использование с Azure CLI

Если установлена версия Azure CLI 2.20.0 или более поздняя, Бицеп CLI устанавливается автоматически при выполнении команды, зависящей от нее. Пример:

```azurecli
az deployment group create --template-file azuredeploy.bicep --resource-group myResourceGroup
```

или диспетчер конфигурации служб

```azurecli
az bicep ...
```

Вы также можете вручную установить интерфейс командной строки с помощью встроенных команд:

```bash
az bicep install
```

Чтобы выполнить обновление до последней версии, выполните следующие действия.

```bash
az bicep upgrade
```

Чтобы установить определенную версию, выполните следующие действия.

```bash
az bicep install --version v0.3.126
```

> [!IMPORTANT]
> Azure CLI устанавливает отдельную версию интерфейса командной строки Бицеп, которая не конфликтует с другими устанавливаемыми Бицеп, и Azure CLI не добавляет Бицеп CLI в путь. Сведения об использовании Бицеп CLI для компиляции и декомпиляции файлов Бицеп, а также для развертывания файлов Бицеп с помощью Azure PowerShell см. в [статье Установка вручную](#install-manually) или [использование с помощью Azure PowerShell](#use-with-azure-powershell).

Чтобы получить список всех доступных версий Бицеп CLI, выполните следующие действия.

```bash
az bicep list-versions
```

Чтобы отобразить установленные версии, выполните следующие действия.

```bash
az bicep version
```

### <a name="use-with-azure-powershell"></a>Использование с Azure PowerShell

Azure PowerShell у вас еще нет возможности установить CLI Бицеп. Azure PowerShell (v 5.6.0 или более поздней версии) ждет, что интерфейс командной строки Бицеп уже установлен и доступен по пути. Выполните один из [методов установки вручную](#install-manually).

Для развертывания файлов Бицеп требуется Бицеп CLI версии 0.3.1 или более поздней. Чтобы проверить версию Бицеп CLI, выполните следующие действия.

```cmd
bicep --version
```

> [!IMPORTANT]
> Azure CLI устанавливает собственную автономную версию Бицеп CLI. Azure PowerShell развертывание завершается сбоем, даже если для Azure CLI установлены необходимые версии.

После установки интерфейса командной строки Бицеп интерфейс командной строки Бицеп вызывается каждый раз, когда он необходим для командлета развертывания. Пример:

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName myResourceGroup -TemplateFile azuredeploy.bicep
```

### <a name="install-manually"></a>Установка вручную

Следующие методы устанавливают интерфейс командной строки Бицеп и добавляют его в свой путь.

#### <a name="linux"></a>Linux

```sh
# Fetch the latest Bicep CLI binary
curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-linux-x64
# Mark it as executable
chmod +x ./bicep
# Add bicep to your PATH (requires admin)
sudo mv ./bicep /usr/local/bin/bicep
# Verify you can now access the 'bicep' command
bicep --help
# Done!

```

#### <a name="macos"></a>macOS

##### <a name="via-homebrew"></a>через Homebrew

```sh
# Add the tap for bicep
brew tap azure/bicep https://github.com/azure/bicep

# Install the tool
brew install azure/bicep/bicep
```

##### <a name="macos-manual-install"></a>Установка macOS вручную

```sh
# Fetch the latest Bicep CLI binary
curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-osx-x64
# Mark it as executable
chmod +x ./bicep
# Add Gatekeeper exception (requires admin)
sudo spctl --add ./bicep
# Add bicep to your PATH (requires admin)
sudo mv ./bicep /usr/local/bin/bicep
# Verify you can now access the 'bicep' command
bicep --help
# Done!

```

#### <a name="windows"></a>Windows

##### <a name="windows-installer"></a>Установщик Windows

Скачайте и запустите [последнюю версию установщика Windows](https://github.com/Azure/bicep/releases/latest/download/bicep-setup-win-x64.exe). Для установщика не требуются права администратора. После установки Бицеп CLI добавляется в путь пользователя. Закройте и снова откройте все открытые окна командной оболочки, чтобы изменения пути вступили в силу.

##### <a name="chocolatey"></a>Chocolatey

```powershell
choco install bicep
```

##### <a name="winget"></a>винжет

```powershell
winget install -e --id Microsoft.Bicep
```

##### <a name="manual-with-powershell"></a>Ручная с помощью PowerShell

```powershell
# Create the install folder
$installPath = "$env:USERPROFILE\.bicep"
$installDir = New-Item -ItemType Directory -Path $installPath -Force
$installDir.Attributes += 'Hidden'
# Fetch the latest Bicep CLI binary
(New-Object Net.WebClient).DownloadFile("https://github.com/Azure/bicep/releases/latest/download/bicep-win-x64.exe", "$installPath\bicep.exe")
# Add bicep to your PATH
$currentPath = (Get-Item -path "HKCU:\Environment" ).GetValue('Path', '', 'DoNotExpandEnvironmentNames')
if (-not $currentPath.Contains("%USERPROFILE%\.bicep")) { setx PATH ($currentPath + ";%USERPROFILE%\.bicep") }
if (-not $env:path.Contains($installPath)) { $env:path += ";$installPath" }
# Verify you can now access the 'bicep' command.
bicep --help
# Done!
```

## <a name="install-the-nightly-builds"></a>Установка ночных сборок

Если вы хотите испытать последние предварительные версии Бицеп перед их выпуском, см. статью [Установка ночных сборок](https://github.com/Azure/bicep/blob/main/docs/installing-nightly.md).

> [!WARNING]
> Эти предварительные сборки с большей вероятностью имеют известные или неизвестные ошибки.

## <a name="next-steps"></a>Дальнейшие шаги

Приступая к работе с [кратким](./quickstart-create-bicep-use-visual-studio-code.md)руководством по бицеп.
