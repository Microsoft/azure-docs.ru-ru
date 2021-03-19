---
title: Аутентификация пользователей — .NET с Data Lake Storage 1-го поколения — Azure
description: Узнайте, как реализовать проверку подлинности пользователей в Azure Data Lake Storage 1-го поколения с помощью Azure Active Directory и .NET SDK
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: devx-track-csharp
ms.openlocfilehash: 67ba4f12aec9e987d79109b7197d03301bf40650
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89004788"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-net-sdk"></a>Проверка подлинности пользователей в Azure Data Lake Storage 1-го поколения с помощью .NET SDK
> [!div class="op_single_selector"]
> * [Использование Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [Использование пакета SDK для .NET](data-lake-store-end-user-authenticate-net-sdk.md)
> * [Использование Python](data-lake-store-end-user-authenticate-python.md)
> * [Использование REST API](data-lake-store-end-user-authenticate-rest-api.md)
> 
>  

В этой статье описывается, как использовать пакет SDK для .NET для проверки подлинности пользователей с помощью Azure Data Lake Storage 1-го поколения. См. дополнительные сведения о [проверке подлинности между службами в Data Lake Storage 1-го поколения с помощью .NET SDK](data-lake-store-service-to-service-authenticate-net-sdk.md).

## <a name="prerequisites"></a>Предварительные требования
* **Visual Studio 2013 или более поздней версии**. В приведенных ниже инструкциях используется Visual Studio 2019.

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Создание собственного приложения Active Directory** Вам нужно выполнить инструкции по [аутентификации пользователей в Data Lake Storage 1-го поколения с помощью Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md).

## <a name="create-a-net-application"></a>Создание приложения .NET
1. В Visual Studio выберите меню **файл** , **создать**, а затем **проект**.
2. Выберите **консольное приложение (платформа .NET Framework)**, а затем нажмите кнопку **Далее**.
3. В окне **Имя проекта** введите `CreateADLApplication`, а затем выберите **Создать**.

4. Добавьте пакеты NuGet в проект.

   1. В обозревателе решений щелкните правой кнопкой мыши имя проекта и выберите пункт **Управление пакетами NuGet**.
   2. На вкладке **Диспетчер пакетов NuGet** убедитесь, что для параметра **источник пакета** задано значение **NuGet.org** и установлен флажок **включить предварительные выпуски** .
   3. Найдите и установите следующие пакеты NuGet:

      * `Microsoft.Azure.Management.DataLake.Store`. В этом руководстве используется предварительная версия 2.1.3.
      * `Microsoft.Rest.ClientRuntime.Azure.Authentication`. В этом руководстве используется версия 2.2.12.

        ![Добавление источника NuGet](./media/data-lake-store-get-started-net-sdk/data-lake-store-install-nuget-package.png "Создание новой учетной записи Azure Data Lake")
   4. Закройте **Диспетчер пакетов NuGet**.

5. Открыть **программу. CS**
6. Замените операторы using следующими строками:

    ```csharp
    using System;
    using System.IO;
    using System.Linq;
    using System.Text;
    using System.Threading;
    using System.Collections.Generic;
            
    using Microsoft.Rest;
    using Microsoft.Rest.Azure.Authentication;
    using Microsoft.Azure.Management.DataLake.Store;
    using Microsoft.Azure.Management.DataLake.Store.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```     

## <a name="end-user-authentication"></a>Проверка подлинности конечных пользователей
Добавьте следующий фрагмент кода в клиентское приложение .NET. Замените значения заполнителей значениями, полученными в собственном приложении Azure AD (перечислено как предварительное условие). Этот фрагмент кода позволяет выполнять аутентификацию приложения **интерактивно** с помощью Data Lake Storage 1-го поколения. Это значит, что вам будет предложено ввести учетные данные Azure.

Для удобства использования в следующем фрагменте кода для идентификатора клиента и URI перенаправления используются значения по умолчанию, которые действительны для любой подписки Azure. В следующем фрагменте кода нужно указать только значение для идентификатора клиента. См. инструкции по [получению идентификатор клиента](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
    
- Замените функцию Main() следующим кодом:

    ```csharp
    private static void Main(string[] args)
    {
        //User login via interactive popup
        string TENANT = "<AAD-directory-domain>";
        string CLIENTID = "1950a258-227b-4e31-a9cf-717495945fc2";
        System.Uri ARM_TOKEN_AUDIENCE = new System.Uri(@"https://management.core.windows.net/");
        System.Uri ADL_TOKEN_AUDIENCE = new System.Uri(@"https://datalake.azure.net/");
        string MY_DOCUMENTS = System.Environment.GetFolderPath(System.Environment.SpecialFolder.MyDocuments);
        string TOKEN_CACHE_PATH = System.IO.Path.Combine(MY_DOCUMENTS, "my.tokencache");
        var tokenCache = GetTokenCache(TOKEN_CACHE_PATH);
        var armCreds = GetCreds_User_Popup(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, tokenCache);
        var adlCreds = GetCreds_User_Popup(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, tokenCache);
    }
    ```

Важные сведения о предыдущем фрагменте кода:

* В предыдущем фрагменте используются вспомогательные функции `GetTokenCache` и `GetCreds_User_Popup`. Код этих вспомогательных функций см. [на этой странице GitHub](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options#gettokencache).
* Для быстрого завершения работы с руководством в этом фрагменте кода используется идентификатор клиента собственного приложения, доступный по умолчанию для всех подписок Azure. Таким образом, вы можете **использовать в приложении этот фрагмент в исходном виде**.
* Если вы не хотите использовать свой домен Azure AD и идентификатор клиента приложения, необходимо создать собственное приложение Azure AD и использовать идентификатор клиента Azure AD, идентификатор клиента и URI перенаправления этого приложения. Инструкции см. в статье [Создание приложения Active Directory для проверки подлинности пользователей в Data Lake Storage 1-го поколения](data-lake-store-end-user-authenticate-using-active-directory.md).

  
## <a name="next-steps"></a>Дальнейшие действия
В этой статье вы узнали, как использовать проверку подлинности пользователей, чтобы реализовать проверку подлинности в Data Lake Storage 1-го поколения с помощью .NET SDK. Дополнительные сведения об использовании пакета .NET SDK для работы с Azure Data Lake Storage 1-го поколения см. в следующих статьях.

* [Операции управления учетными записями в Data Lake Storage 1-го поколения c использованием пакета SDK для .NET](data-lake-store-get-started-net-sdk.md)
* [Операции с данными в Data Lake Storage 1-го поколения c использованием SDK для .NET](data-lake-store-data-operations-net-sdk.md)

