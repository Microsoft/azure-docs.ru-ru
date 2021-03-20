---
author: paulbouwer
ms.topic: include
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: 1a023475de1ce2891916807632d9ee15e382326c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81735285"
---
## <a name="download-and-install-the-linkerd-linkerd-client-binary"></a>Скачивание и установка двоичного файла клиента Linkerd Linkerd

В оболочке на основе PowerShell в Windows используйте `Invoke-WebRequest` для загрузки выпуска Linkerd следующим образом:

```powershell
# Specify the Linkerd version that will be leveraged throughout these instructions
$LINKERD_VERSION="stable-2.6.0"

# Enforce TLS 1.2
[Net.ServicePointManager]::SecurityProtocol = "tls12"
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -URI "https://github.com/linkerd/linkerd2/releases/download/$LINKERD_VERSION/linkerd2-cli-$LINKERD_VERSION-windows.exe" -OutFile "linkerd2-cli-$LINKERD_VERSION-windows.exe"
```

`linkerd`Клиентский двоичный файл выполняется на клиентском компьютере и позволяет взаимодействовать с сеткой службы Linkerd. Используйте следующие команды для установки `linkerd` двоичного файла клиента Linkerd в оболочке на основе PowerShell в Windows. Эти команды копируют `linkerd` клиентский двоичный файл в папку Linkerd, а затем делают его доступными как непосредственно (в текущей оболочке), так и постоянно (при перезапуске оболочки) с помощью `PATH` . Для выполнения этих команд не требуются повышенные привилегии (администратора), и вам не нужно перезапускать оболочку.

```powershell
# Copy linkerd.exe to C:\Linkerd
New-Item -ItemType Directory -Force -Path "C:\Linkerd"
Copy-Item -Path ".\linkerd2-cli-$LINKERD_VERSION-windows.exe" -Destination "C:\Linkerd\linkerd.exe"

# Add C:\Linkerd to PATH. 
# Make the new PATH permanently available for the current User
$USER_PATH = [environment]::GetEnvironmentVariable("PATH", "User") + ";C:\Linkerd\"
[environment]::SetEnvironmentVariable("PATH", $USER_PATH, "User")
# Make the new PATH immediately available in the current shell
$env:PATH += ";C:\Linkerd\"
```