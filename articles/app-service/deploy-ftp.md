---
title: Развертывание содержимого с помощью FTP/S
description: Узнайте, как развернуть приложение в службе приложений Azure с помощью FTP или FTPS. Повысьте безопасность веб-сайта, отключив незашифрованный FTP.
ms.assetid: ae78b410-1bc0-4d72-8fc4-ac69801247ae
ms.topic: article
ms.date: 02/26/2021
ms.reviewer: dariac
ms.custom: seodec18
ms.openlocfilehash: c7427a1f8f528fdf405b22c4e91941ea7a915ffa
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102045808"
---
# <a name="deploy-your-app-to-azure-app-service-using-ftps"></a>Развертывание приложения в службе приложений Azure с помощью FTP или FTPS

В этой статье показано, как с помощью FTP или FTPS развернуть веб-приложение, серверную частью мобильного приложения или приложение API в [службе приложений Azure](./overview.md).

Конечная точка FTP или FTPS для приложения уже активна. Чтобы обеспечить развертывание через FTP или FTPS, не требуется никаких настроек.

> [!NOTE]
> Страница « **центр разработки» (классическая)** в портал Azure, которая является старым процессом развертывания, будет признана устаревшей в марте 2021. Это изменение не повлияет на существующие параметры развертывания в приложении, и вы можете продолжить управление развертыванием приложения на странице **центра развертывания** .

## <a name="get-deployment-credentials"></a>Получить учетные данные развертывания

1. Следуйте инструкциям в статье [Настройка учетных данных развертывания для службы приложений Azure](deploy-configure-credentials.md) , чтобы скопировать учетные данные области приложения или задать учетные данные области пользователя. Подключиться к конечной точке FTP/S приложения можно с помощью любой учетной записи.

1. Создание имени пользователя FTP в следующем формате в зависимости от выбранной области учетных данных:

    | Область приложения | Область пользователя |
    | - | - |
    |`<app-name>\$<app-name>`|`<app-name>\<deployment-user>`|

    ---

    В службе приложений конечная точка FTP/S является общей для всех приложений. Так как учетные данные области пользователя не связаны с конкретным ресурсом, необходимо добавить имя пользователя в начале имени приложения, как показано выше.

## <a name="get-ftps-endpoint"></a>Получить конечную точку FTP/S
    
# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

На той же странице управления для приложения, в которое были скопированы учетныеданные развертывания (  >  **учетные данные FTP** центра развертывания), скопируйте **конечную точку FTPS**.

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Выполните команду [AZ webapp Deployment List-Publishing-Profiles](/cli/azure/webapp/deployment#az_webapp_deployment_list_publishing_profiles) . В следующем примере используется [путь жмес](https://jmespath.org/) для извлечения конечных точек FTP/S из выходных данных.

```azurecli-interactive
az webapp deployment list-publishing-profiles --name <app-name> --resource-group <group-name> --query "[?ends_with(profileName, 'FTP')].{profileName: profileName, publishUrl: publishUrl}"
```

Каждое приложение имеет две конечные точки FTP/S, один из которых доступен для чтения и записи, а второй — только для чтения ( `profileName` содержит `ReadOnly` ) и предназначен для сценариев восстановления данных. Чтобы развернуть файлы с помощью FTP, скопируйте URL-адрес конечной точки для чтения и записи.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Выполните команду [Get-азвебапппублишингпрофиле](/powershell/module/az.websites/get-azwebapppublishingprofile) . В следующем примере из выходных данных XML извлекается конечная точка FTP/S.

```azurepowershell-interactive
$xml = [xml](Get-AzWebAppPublishingProfile -Name <app-name> -ResourceGroupName <group-name> -OutputFile null)
$xml.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@publishUrl").value
```

-----

## <a name="deploy-files-to-azure"></a>Развертывание файлов в Azure

1. Подключитесь к приложению из клиента FTP (например, [Visual Studio](https://www.visualstudio.com/vs/community/), [Cyberduck](https://cyberduck.io/) или [WinSCP](https://winscp.net/index.php)), используя полученные сведения.
2. Скопируйте файлы и соответствующую им структуру каталогов в Azure в каталог [**/site/wwwroot**](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure) (или в каталог веб-заданий **/site/wwwroot/App_Data/Jobs/**).
3. Перейдите по URL-адресу приложения, чтобы убедиться, что приложение работает правильно. 

> [!NOTE] 
> В отличие от развертываний [на основе git](deploy-local-git.md) и [ZIP](deploy-zip.md)-развертываний, развертывание по FTP не поддерживает автоматизацию сборки, например: 
>
> - восстановление зависимостей (например, в инструментах автоматизации NuGet, NPM, PIP и Composer);
> - компиляция двоичных файлов .NET;
> - создание файла Web.config (вот [пример для Node.js](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps)).
> 
> Создайте эти необходимые файлы вручную или на локальном компьютере, а затем разверните их вместе с приложением.
>

## <a name="enforce-ftps"></a>Принудительное применение FTPS

Для повышения безопасности следует разрешить только FTP через TLS/SSL. Вы также можете отключить и FTP, и FTPS, если развертывание FTP вам не нужно.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

1. На странице ресурсов приложения в [портал Azure](https://portal.azure.com)выберите **Конфигурация**  >  **Общие параметры** в левой области навигации.

2. Чтобы отключить незашифрованный FTP, выберите **FTPS только** в **состоянии FTP**. Чтобы полностью отключить протоколы FTP и FTPS, выберите значение **отключено**. По завершении щелкните **Сохранить**. Если используется **только FTPS**, необходимо принудительно включить TLS 1,2 или более поздней версии, перейдя в колонку **параметров TLS/SSL** веб-приложения. Протоколы TLS 1.0 и 1.1 не поддерживаются для параметра **Только FTPS**.

    ![Отключение FTP(S)](./media/app-service-deploy-ftp/disable-ftp.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Выполните команду [AZ webapp config Set](/cli/azure/webapp/deployment#az_webapp_deployment_list_publishing_profiles) с `--ftps-state` аргументом.

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <group-name> --ftps-state FtpsOnly
```

Возможные значения для `--ftps-state` : `AllAllowed` (FTP и FTPS), `Disabled` (FTP и FTPS отключены) и `FtpsOnly` (только для FTPS).

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Выполните команду [Set-азвебапп](/powershell/module/az.websites/set-azwebapp) с `-FtpsState` параметром.

```azurepowershell-interactive
Set-AzWebApp -Name <app-name> -ResourceGroupName <group-name> -FtpsState FtpsOnly
```

Возможные значения для `--ftps-state` : `AllAllowed` (FTP и FTPS), `Disabled` (FTP и FTPS отключены) и `FtpsOnly` (только для FTPS).

-----

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="troubleshoot-ftp-deployment"></a>Устранение неполадок с развертыванием FTP

- [Как устранять неполадки с развертыванием FTP?](#how-can-i-troubleshoot-ftp-deployment)
- [Я не могу выполнять FTP и публиковать мой код. Как устранить проблему?](#im-not-able-to-ftp-and-publish-my-code-how-can-i-resolve-the-issue)
- [Как применить пассивный режим для подключения к FTP в Службе приложений Azure?](#how-can-i-connect-to-ftp-in-azure-app-service-via-passive-mode)

#### <a name="how-can-i-troubleshoot-ftp-deployment"></a>Как устранять неполадки с развертыванием FTP?

При устранении неполадок с развертыванием FTP первым делом следует выяснить, связана ли проблема с самим развертыванием или с приложением среды выполнения.

Ошибки развертывания обычно приводят к тому, что в приложении совсем не развертываются файлы или развертываются не те файлы, которые нужны. Для устранения неполадок изучите данные о развертывании FTP или попробуйте применить альтернативный путь развертывания (например, систему управления версиями).

Проблемы с приложением среды выполнения обычно приводят к тому, что приложение ведет себя отличным от ожидаемого образом после правильного развертывания нужных файлов. Для устранения неполадок следует сосредоточиться на поведении кода в среде выполнения или на конкретных сценариях сбоя.

Чтобы отделить проблемы развертывания от проблем среды выполнения, воспользуйтесь рекомендациями из [этой статьи](https://github.com/projectkudu/kudu/wiki/Deployment-vs-runtime-issues).

#### <a name="im-not-able-to-ftp-and-publish-my-code-how-can-i-resolve-the-issue"></a>Мне не удается подключиться по FTP для публикации кода. Как мне решить эту проблему?
Убедитесь, что вы ввели правильное [имя узла](#get-ftps-endpoint) и [учетные данные](#get-deployment-credentials). Также убедитесь, что на вашем компьютере брандмауэр не блокирует следующие порты FTP.

- Порт управляющего соединения FTP: 21, 990
- Порт подключения к данным FTP: 989, 10001–10300.
 
#### <a name="how-can-i-connect-to-ftp-in-azure-app-service-via-passive-mode"></a>Как применить пассивный режим для подключения к FTP в Службе приложений Azure?
Служба приложений Azure поддерживает подключение как в активном, так и в пассивном режимах. Пассивный режим является предпочтительным, так как компьютеры развертывания обычно защищены брандмауэром (встроенным в операционную систему или развернутым отдельно в домашней или корпоративной сети). См. также [пример из документации по WinSCP](https://winscp.net/docs/ui_login_connection). 

## <a name="more-resources"></a>Дополнительные ресурсы

* [Развертывание локального репозитория Git в Службе приложений Azure](deploy-local-git.md)
* [Учетные данные развертывания службы приложений Azure](deploy-configure-credentials.md)
* [Пример. Создание веб-приложения и развертывание файлов с помощью FTP (Azure CLI)](./scripts/cli-deploy-ftp.md).
* [Пример. Отправка файлов в веб-приложение с помощью FTP (PowerShell)](./scripts/powershell-deploy-ftp.md).
