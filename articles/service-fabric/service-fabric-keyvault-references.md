---
title: Service Fabric Azure. Использование Service Fabricных ссылок на KeyVault приложений
description: В этой статье объясняется, как использовать поддержку Кэйваултреференце Service-Fabric для секретов приложений.
ms.topic: article
ms.date: 09/20/2019
ms.openlocfilehash: a0e4ef0decae8cc9ab4dc5f8c69dfef854af81f3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98898602"
---
# <a name="keyvaultreference-support-for-azure-deployed-service-fabric-applications"></a>Поддержка Кэйваултреференце приложений, развернутых в Azure Service Fabric

Распространенной проблемой при создании облачных приложений является безопасное распространение секретов в приложения. Например, может потребоваться развернуть в приложении ключ базы данных, не раскрывая ключ во время конвейера или оператора. Service Fabric поддержка Кэйваултреференце упрощает развертывание секретов в приложениях, просто ссылаясь на URL-адрес секрета, который хранится в Key Vault. Service Fabric будет выполнять получение секрета от имени управляемого удостоверения приложения и активировать приложение с секретом.

> [!NOTE]
> Поддержка Кэйваултреференце для приложений Service Fabric — это общедоступная версия (вне предварительной версии), начиная с Service Fabric версии 7,2 CU5. Рекомендуется выполнить обновление до этой версии перед использованием этой функции.

> [!NOTE]
> Поддержка Кэйваултреференце для приложений Service Fabric поддерживает только секреты с [версиями](../key-vault/general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning) . Секреты без поддержки версий не поддерживаются. Key Vault должны находиться в той же подписке, что и кластер Service Fabric. 

## <a name="prerequisites"></a>Предварительные условия

- Управляемое удостоверение для приложений Service Fabric

    Service Fabric поддержка Кэйваултреференце использует управляемое удостоверение приложения для выборки секретов от имени приложения, поэтому приложение должно быть развернуто с помощью и назначено управляемое удостоверение. Следуйте этому [документу](concepts-managed-identity.md) , чтобы включить управляемое удостоверение для приложения.

- Центральное хранилище секретов (CSS).

    Центральное хранилище секретов (CSS) — это зашифрованный локальный кэш секретов Service Fabric. Эта функция использует CSS для защиты и сохранения секретов после их получения из Key Vault. Для использования этой функции также требуется включить эту необязательную системную службу. Чтобы включить и настроить CSS, следуйте этому [документу](service-fabric-application-secret-store.md) .

- Предоставление управляемому удостоверению приложения разрешения на доступ к keyvault

    Обратитесь к этому [документу](how-to-grant-access-other-resources.md) , чтобы узнать, как предоставить управляемому удостоверению доступ к keyvault. Также обратите внимание, что при использовании управляемого удостоверения, назначенного системой, управляемое удостоверение создается только после развертывания приложения. Это может создать состояние гонки, при котором приложение пытается получить доступ к секрету, прежде чем удостоверение сможет получить доступ к хранилищу. Имя удостоверения, присвоенное системой, будет иметь значение `{cluster name}/{application name}/{service name}` .
    
## <a name="use-keyvaultreferences-in-your-application"></a>Использование Кэйваултреференцес в приложении
Кэйваултреференцес можно использовать несколькими способами.
- [Как переменную среды](#as-an-environment-variable)
- [Подключен как файл в контейнере](#mounted-as-a-file-into-your-container)
- [В качестве ссылки на пароль репозитория контейнеров](#as-a-reference-to-a-container-repository-password)

### <a name="as-an-environment-variable"></a>Как переменную среды

```xml
<EnvironmentVariables>
      <EnvironmentVariable Name="MySecret" Type="KeyVaultReference" Value="<KeyVaultURL>"/>
</EnvironmentVariables>
```

```C#
string secret =  Environment.GetEnvironmentVariable("MySecret");
```

### <a name="mounted-as-a-file-into-your-container"></a>Подключен как файл в контейнере

- Добавление раздела в settings.xml

    Определение `MySecret` параметра с типом `KeyVaultReference` и значением `<KeyVaultURL>`

    ```xml
    <Section Name="MySecrets">
        <Parameter Name="MySecret" Type="KeyVaultReference" Value="<KeyVaultURL>"/>
    </Section>
    ```

- Сослаться на новый раздел в ApplicationManifest.xml в `<ConfigPackagePolicies>`

    ```xml
    <ServiceManifestImport>
        <Policies>
        <IdentityBindingPolicy ServiceIdentityRef="MyServiceMI" ApplicationIdentityRef="MyApplicationMI" />
        <ConfigPackagePolicies CodePackageRef="Code">
            <!--Linux container example-->
            <ConfigPackage Name="Config" SectionName="MySecrets" EnvironmentVariableName="SecretPath" MountPoint="/var/secrets"/>
            <!--Windows container example-->
            <!-- <ConfigPackage Name="Config" SectionName="dbsecrets" EnvironmentVariableName="SecretPath" MountPoint="C:\secrets"/> -->
        </ConfigPackagePolicies>
        </Policies>
    </ServiceManifestImport>
    ```

- Использование секретов из кода службы

    Каждый параметр, указанный в разделе, `<Section  Name=MySecrets>` будет представлять собой файл в папке, на которую указывает EnvironmentVariable секретпас. В приведенном ниже фрагменте кода C# показано, как считывать MySecret из приложения.

    ```C#
    string secretPath = Environment.GetEnvironmentVariable("SecretPath");
    using (StreamReader sr = new StreamReader(Path.Combine(secretPath, "MySecret"))) 
    {
        string secret =  sr.ReadToEnd();
    }
    ```
    > [!NOTE] 
    > Параметр MountPoint управляет папкой, в которой будут подключены файлы, содержащие секретные значения.

### <a name="as-a-reference-to-a-container-repository-password"></a>В качестве ссылки на пароль репозитория контейнеров

```xml
 <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="MyACRUser" Type="KeyVaultReference" Password="<KeyVaultURL>"/>
      </ContainerHostPolicies>
```

## <a name="next-steps"></a>Дальнейшие действия

* [Документация по Azure KeyVault](../key-vault/index.yml)
* [Сведения о центральном хранилище секретов](service-fabric-application-secret-store.md)
* [Сведения об управляемом удостоверении для приложений Service Fabric](concepts-managed-identity.md)
