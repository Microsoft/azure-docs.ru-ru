---
title: Аутентификация между службами — Data Lake Storage 1-го поколения — Azure
description: Узнайте, как обеспечить проверку подлинности между службами с помощью Azure Data Lake Storage 1-го поколения Azure Active Directory.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 9dc195f98310e63cbde06885effe86ea3c239249
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91576104"
---
# <a name="service-to-service-authentication-with-azure-data-lake-storage-gen1-using-azure-active-directory"></a>Аутентификация между службами в Azure Data Lake Storage 1-го поколения с помощью Azure Active Directory
> [!div class="op_single_selector"]
> * [Проверка подлинности конечных пользователей](data-lake-store-end-user-authenticate-using-active-directory.md)
> * [Аутентификация между службами](data-lake-store-service-to-service-authenticate-using-active-directory.md)
> 
>  

Azure Data Lake Storage 1-го поколения (ADLS 1-го поколения) использует Azure Active Directory для аутентификации. Прежде чем создать приложение, которое работает с ADLS 1-го поколения, необходимо решить, как для него выполнять аутентификацию в Azure Active Directory (Azure AD). Доступно два основных варианта:

* Проверка подлинности конечных пользователей 
* Аутентификация между службами (в этой статье) 

Оба варианта позволят приложению получить маркер OAuth 2.0, который используется в каждом запросе к Data Lake Storage 1-го поколения.

Из этой статьи вы узнаете, как создать **веб-приложение Azure AD для аутентификации между службами**. Инструкции по настройке приложения Azure AD для аутентификации пользователей см. в статье [Аутентификация пользователей в Data Lake Storage 1-го поколения с помощью Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md).

## <a name="prerequisites"></a>Предварительные требования
* Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

## <a name="step-1-create-an-active-directory-web-application"></a>Шаг 1. Создание веб-приложения Active Directory

Создайте и настройте веб-приложение Azure AD для аутентификации между службами в Azure Data Lake Storage 1-го поколения с помощью Azure Active Directory. Инструкции см. в разделе [Создание приложения Azure AD](../active-directory/develop/howto-create-service-principal-portal.md).

При выполнении инструкций по ссылке выше обязательно выберите тип приложения **Веб-приложение или API**, как показано на следующем снимке экрана:

![Создание веб-приложения](./media/data-lake-store-authenticate-using-active-directory/azure-active-directory-create-web-app.png "Создание веб-приложения")

## <a name="step-2-get-application-id-authentication-key-and-tenant-id"></a>Шаг 2. Получение идентификатора приложения, ключа проверки подлинности и идентификатора клиента
При программном входе необходимо указывать идентификатор приложения. Если приложение работает с использованием собственных учетных данных, потребуется также ключ проверки подлинности.

* Инструкции о том, как получить идентификатор приложения и ключ проверки подлинности (который также называется секретом клиента) для приложения, см. в разделе [Получение идентификатора приложения и ключа проверки подлинности](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).

* Инструкции о том, как получить код клиента, см. в разделе [Получение идентификатора клиента](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).

## <a name="step-3-assign-the-azure-ad-application-to-the-azure-data-lake-storage-gen1-account-file-or-folder"></a>Шаг 3. Назначение приложения Azure AD файлу или папке в учетной записи Azure Data Lake Storage 1-го поколения


1. Выполните вход на [портал Azure](https://portal.azure.com). Откройте учетную запись Data Lake Storage 1-го поколения, которую необходимо связать с ранее созданным приложением Azure Active Directory.
2. В колонке учетной записи Data Lake Storage 1-го поколения щелкните **Azure Data Explorer**.
   
    ![Создание каталогов в учетной записи Data Lake Storage 1-го поколения](./media/data-lake-store-authenticate-using-active-directory/adl.start.data.explorer.png "Создание каталогов в учетной записи озера данных")
3. В колонке **Обозреватель данных** щелкните файл (или папку), к которому необходимо предоставить доступ для приложения Azure AD, а затем нажмите кнопку **Доступ**. Чтобы настроить доступ к файлу, необходимо нажать кнопку **Доступ** в колонке **Предварительный просмотр файла**.
   
    ![Настройка списков управления доступом в файловой системе озера данных](./media/data-lake-store-authenticate-using-active-directory/adl.acl.1.png "Настройка списков управления доступом в файловой системе озера данных")
4. В колонке **Доступ** перечислены стандартные и пользовательские варианты доступа, уже назначенные корню. Щелкните значок **Добавить** , чтобы добавить ACL пользовательского уровня.
   
    ![Перечисление стандартных и пользовательских сценариев доступа](./media/data-lake-store-authenticate-using-active-directory/adl.acl.2.png "Перечисление стандартных и пользовательских сценариев доступа")
5. Щелкните значок **Добавить**, чтобы открыть колонку **Добавить настраиваемый доступ**. В этой колонке щелкните **Выберите пользователя или группу**, а затем в колонке **Выберите пользователя или группу** найдите созданное ранее приложение Azure Active Directory. Если у вас много групп для поиска, воспользуйтесь текстовым полем в верхней части для фильтрации по имени группы. Выберите группу, которую необходимо добавить, и нажмите кнопку **Выбрать**.
   
    ![Добавление группы](./media/data-lake-store-authenticate-using-active-directory/adl.acl.3.png "Добавление группы")
6. Щелкните **Выберите разрешения**, выберите необходимые разрешения, а затем укажите, назначить ли разрешения как список ACL по умолчанию, список ACL для доступа или оба этих параметра. Нажмите кнопку **OK**.
   
    ![Снимок экрана: Добавление колонки настраиваемого доступа с вызываемым параметром SELECT Permissions, а также колонка разрешения SELECT с вызываемым параметром ОК.](./media/data-lake-store-authenticate-using-active-directory/adl.acl.4.png "Назначение разрешений для группы")
   
    Дополнительные сведения о разрешениях в Data Lake Storage 1-го поколения и списках ACL (по умолчанию и для доступа) см. в статье [Контроль доступа в Data Lake Storage 1-го поколения](data-lake-store-access-control.md).
7. В колонке **Добавить настраиваемый доступ** нажмите кнопку **ОК**. Новые добавленные группы с соответствующими разрешениями перечислены в колонке **доступ** .
   
    ![Снимок экрана колонки доступа с добавленной группой, вызываемой в разделе настраиваемый доступ.](./media/data-lake-store-authenticate-using-active-directory/adl.acl.5.png "Назначение разрешений для группы")

> [!NOTE]
> Если вы планируете ограничить свое приложение Azure Active Directory конкретной папкой, вам также потребуется предоставить этому приложению Azure Active Directory разрешение на **выполнение** в корневом каталоге, чтобы обеспечить доступ для создания файлов с помощью пакета SDK для .NET.

> [!NOTE]
> Если вы хотите использовать пакеты SDK для создания учетной записи Data Lake Storage 1-го поколения, необходимо назначить веб-приложение Azure AD в качестве роли группе ресурсов, в которой создается эта учетная запись.
> 
>

## <a name="step-4-get-the-oauth-20-token-endpoint-only-for-java-based-applications"></a>Шаг 4. Получение конечной точки маркера OAuth 2.0 (только для приложений на основе Java)

1. Войдите на [портал Azure](https://portal.azure.com) и щелкните "Active Directory" в области слева.

2. В левой области щелкните **Регистрация приложений**.

3. В верхней части колонки "Регистрация приложений" щелкните **Конечные точки**.

    ![Снимок экрана Active Directory с параметром Регистрация приложений и вызываемым параметром конечных точек.](./media/data-lake-store-authenticate-using-active-directory/oauth-token-endpoint.png "Конечная точка токена OAuth")

4. Из списка конечных точек скопируйте значение конечной точки маркера OAuth 2.0.

    ![Снимок экрана: колонка конечных точек с вызываемым значком копирования КОНЕЧНОЙ точки ТОКЕНа O AUTH 2 Point O.](./media/data-lake-store-authenticate-using-active-directory/oauth-token-endpoint-1.png "Конечная точка токена OAuth")   

## <a name="next-steps"></a>Дальнейшие действия
В этой статье вы создали веб-приложение Azure AD и собрали информацию, необходимую в клиентских приложениях, созданных с помощью пакета SDK для .NET, Java, Python, REST API и т. д. Теперь вы можете перейти к следующим статьям, в которых рассматривается использование собственного приложения Azure AD для первой проверки подлинности с помощью Data Lake Storage 1-го поколения, а затем выполнение других операций с хранилищем.

* [Аутентификация между службами в Data Lake Storage 1-го поколения с помощью Java](data-lake-store-service-to-service-authenticate-java.md)
* [Аутентификация между службами в Data Lake Storage 1-го поколения с помощью пакета SDK для .NET](data-lake-store-service-to-service-authenticate-net-sdk.md)
* [Аутентификация между службами в Data Lake Storage 1-го поколения с помощью Python](data-lake-store-service-to-service-authenticate-python.md)
* [Аутентификация между службами в Data Lake Storage 1-го поколения с помощью REST API](data-lake-store-service-to-service-authenticate-rest-api.md)


