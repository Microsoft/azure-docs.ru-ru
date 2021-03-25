---
ms.openlocfilehash: b1be5e903994f0a2c4a3d457f0c2bb5572a889c3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98663035"
---
## <a name="add-secret-manager"></a>Добавление диспетчера секретов

Средство "Диспетчер секретов" хранит конфиденциальные данные для разработки вне вашего дерева проектов. Этот подход помогает предотвратить случайный обмен секретами приложений в исходном коде. Чтобы разрешить использование Диспетчера секретов в проекте ASP.NET Core, выполните следующие действия:

#### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

Перейдите в корневой каталог проекта и выполните следующую команду, чтобы включить хранилище секретов в проекте:

```dotnetcli
dotnet user-secrets init
```

Элемент `UserSecretsId`, содержащий GUID, добавляется в файл с расширением *.csproj*.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
    
    <PropertyGroup>
        <TargetFramework>net5.0</TargetFramework>
        <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
    </PropertyGroup>

</Project>
```

#### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

Перейдите в корневой каталог проекта и выполните следующую команду, чтобы включить хранилище секретов в проекте:

```dotnetcli
dotnet user-secrets init
```

Элемент `UserSecretsId`, содержащий GUID, добавляется в файл с расширением *.csproj*.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
    
    <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
    </PropertyGroup>

</Project>
```

#### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

1. Откройте *CSPROJ*-файл.

1. Добавьте элемент `UserSecretsId` в файл с расширением *.csproj*, как показано здесь. Вы можете использовать тот же GUID или заменить это значение собственным.

    ```xml
    <Project Sdk="Microsoft.NET.Sdk.Web">
    
        <PropertyGroup>
            <TargetFramework>netcoreapp2.1</TargetFramework>
            <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
        </PropertyGroup>
    
        <ItemGroup>
            <PackageReference Include="Microsoft.AspNetCore.App" />
            <PackageReference Include="Microsoft.AspNetCore.Razor.Design" Version="2.1.2" PrivateAssets="All" />
        </ItemGroup>
    
    </Project>
    ```
    
1. Сохраните *CSPROJ*-файл.

---

> [!TIP]
> Дополнительные сведения о Диспетчере секретов см. в статье [Надежное хранение секретов приложений при разработке в ASP.NET Core](/aspnet/core/security/app-secrets).
