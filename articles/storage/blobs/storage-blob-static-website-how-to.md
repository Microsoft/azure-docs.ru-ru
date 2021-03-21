---
title: Размещение статического веб-сайта в службе хранилища Azure
description: Узнайте, как передавать статическое содержимое (HTML, CSS, JavaScript и файлы изображений) непосредственно из контейнера в учетную запись GPv2 службы хранилища Azure.
author: normesta
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.author: normesta
ms.date: 03/04/2020
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 2ba0e836f1be97681d677158f5cb81c5b033e915
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103008507"
---
# <a name="host-a-static-website-in-azure-storage"></a>Размещение статического веб-сайта в службе хранилища Azure

Вы можете предоставлять статическое содержимое (HTML, CSS, JavaScript и файлы изображений) непосредственно из контейнера в учетной записи [общего назначения v2](../common/storage-account-create.md) или [блоккблобстораже](storage-blob-create-account-block-blob.md) . Дополнительные сведения см. в статье [Размещение статических веб-сайтов в службе хранилища Azure](storage-blob-static-website.md).

В этой статье показано, как включить размещение статического веб-сайта, используя портал Azure, Azure CLI или PowerShell.

## <a name="enable-static-website-hosting"></a>Включение размещения статического веб-сайта

Размещение статических веб-сайтов — это функция, которую необходимо включить в учетной записи хранения.

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Чтобы начать, войдите на [портал Azure](https://portal.azure.com/).

2. Найдите учетную запись хранения и отобразите общие сведения о ней.

3. Выберите **Статический веб-сайт** для отображения на странице конфигурации статических веб-сайтов.

4. Выберите **Включено**, чтобы включить размещение статического веб-сайта в учетной записи хранения.

5. В поле **Имя документа индекса** укажите страницу индексов по умолчанию (например, *index.html*). 

   Страница индексов по умолчанию отображается, когда пользователь переходит к корню статического веб-сайта.  

6. В поле **Путь к документу с сообщением об ошибке** укажите страницу ошибки по умолчанию (например, *404.html*). 

   Страница ошибки по умолчанию отображается, когда пользователь пытается перейти на страницу, которой не существует на вашем статическом веб-сайте.

7. Выберите команду **Сохранить**. Теперь портал Azure отображает конечную точку вашего статического веб-сайта. 

    ![Включение размещения статического веб-сайта в учетной записи хранения](media/storage-blob-static-website-host/enable-static-website-hosting.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

<a id="cli"></a>

Размещение статического веб-сайта можно включить с помощью [Интерфейса командной строки Azure (CLI)](/cli/azure/).

1. Сначала откройте [Azure Cloud Shell](../../cloud-shell/overview.md) или, если вы [установили](/cli/azure/install-azure-cli) Azure CLI локально, откройте командное консольное приложение (например, Windows PowerShell).

2. Если удостоверение связано с несколькими подписками, установите активную подписку в качестве подписки учетной записи хранения, в которой будет размещен статический веб-сайт.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

3. Включите размещение статического веб-сайта.

   ```azurecli-interactive
   az storage blob service-properties update --account-name <storage-account-name> --static-website --404-document <error-document-name> --index-document <index-document-name>
   ```

   * Замените значение заполнителя `<storage-account-name>` именем вашей учетной записи хранения.

   * Замените заполнитель `<error-document-name>` именем документа ошибки, который будет отображаться пользователям при запросе браузером несуществующей страницы сайта.

   * Замените заполнитель `<index-document-name>` именем документа индекса. Этим документом обычно является index.html.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

<a id="powershell"></a>

Включить размещение статического веб-сайта можно с помощью модуля Azure PowerShell.

1. Откройте командное окно Windows PowerShell.

2. Убедитесь, что у вас есть модуль Az в Azure PowerShell 0.7 или более поздней версии.

   ```powershell
   Get-InstalledModule -Name Az -AllVersions | select Name,Version
   ```

   Если вам необходимо выполнить установку или обновление, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-Az-ps).

3. Войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

   ```powershell
   Connect-AzAccount
   ```

4. Если удостоверение связано с несколькими подписками, установите активную подписку в качестве подписки учетной записи хранения, в которой будет размещен статический веб-сайт.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

5. Получите контекст учетной записи хранения, определяющий необходимую учетную запись хранения.

   ```powershell
   $storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
   $ctx = $storageAccount.Context
   ```

   * Замените значение заполнителя `<resource-group-name>` именем своей группы ресурсов.

   * Замените значение заполнителя `<storage-account-name>` именем вашей учетной записи хранения.

6. Включите размещение статического веб-сайта.

   ```powershell
   Enable-AzStorageStaticWebsite -Context $ctx -IndexDocument <index-document-name> -ErrorDocument404Path <error-document-name>
   ```

   * Замените заполнитель `<error-document-name>` именем документа ошибки, который будет отображаться пользователям при запросе браузером несуществующей страницы сайта.

   * Замените заполнитель `<index-document-name>` именем документа индекса. Этим документом обычно является index.html.

---

## <a name="upload-files"></a>Upload files 

### <a name="portal"></a>[Портал](#tab/azure-portal)

В данном разделе приведены инструкции по отправке файлов с помощью версии Обозревателя службы хранилища, которая отображается на портале Azure. Однако вы также можете использовать версию [Обозревателя службы хранилища](https://azure.microsoft.com/features/storage-explorer/), которая работает за пределами портала Azure. Вы можете использовать [AzCopy](../common/storage-use-azcopy-v10.md), PowerShell, CLI или любое пользовательское приложение, которое может отправлять файлы в контейнер **$web** учетной записи. Пошаговое руководство по отправке файлов с помощью Visual Studio Code см. в [руководстве по размещению статического веб-сайта в хранилище BLOB-объектов](./storage-blob-static-website-host.md).

1. Выберите **Обозреватель службы хранилища (предварительная версия)** .

2. Разверните узел **Контейнеры BLOB-объектов** и выберите контейнер **$web**.

3. Нажмите кнопку **Отправить**, чтобы отправить файлы.

   ![Upload files](media/storage-blob-static-website/storage-blob-static-website-upload.png)

4. Если требуется, чтобы браузер отображал содержимое файла, убедитесь, что для типа содержимого этого файла задано значение `text/html`. 

   ![Проверка типов содержимого](media/storage-blob-static-website/storage-blob-static-website-content-type.png)

   >[!NOTE]
   > Обозреватель службы хранилища автоматически задает для этого свойства значение `text/html` для наиболее распознаваемых расширений, таких как `.html`. Однако в некоторых случаях это необходимо будет сделать самостоятельно. Если не задать для этого свойства значение `text/html`, вместо отображения содержимого браузер предложит вам загрузить файл. Чтобы задать это свойство, щелкните правой кнопкой мыши файл, а затем выберите **Свойства**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Передайте объекты в контейнер *$web* из исходного каталога.

В этом примере предполагается, что вы выполняете команды из сеанса Azure Cloud Shell.

```azurecli-interactive
az storage blob upload-batch -s <source-path> -d '$web' --account-name <storage-account-name>
```

> [!NOTE] 
> Если в браузере вместо отображения содержимого появится запрос на скачивание файла, вы можете добавить `--content-type 'text/html; charset=utf-8'` в команду. 

* Замените значение заполнителя `<storage-account-name>` именем вашей учетной записи хранения.

* Замените заполнитель `<source-path>` путем к расположению файлов, которые необходимо отправить.

> [!NOTE]
> Если вы используете установку расположения Azure CLI, вы можете использовать путь к любому расположению на локальном компьютере (например, `C:\myFolder`).
>
> При использовании Azure Cloud Shell необходима будет ссылка на общую папку, которая будет видима для Cloud Shell. Это может быть общая папка облачной общей папки или имеющаяся общая папка, подключенная из Cloud Shell. Дополнительные сведения см. в статье [Сохранение файлов в Azure Cloud Shell](../../cloud-shell/persisting-shell-storage.md).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Передайте объекты в контейнер *$web* из исходного каталога.

```powershell
# upload a file
set-AzStorageblobcontent -File "<path-to-file>" `
-Container `$web `
-Blob "<blob-name>" `
-Context $ctx
```

> [!NOTE] 
> Если в браузере вместо отображения содержимого появится запрос на скачивание файла, вы можете добавить `-Properties @{ ContentType = "text/html; charset=utf-8";}` в команду.

* Замените значение заполнителя `<path-to-file>` полным путем к файлу, который необходимо отправить (например, `C:\temp\index.html`).

* Замените значение заполнителя `<blob-name>` именем, которое вы хотите присвоить полученному большому двоичному объекту (например, `index.html`).

---

<a id="portal-find-url"></a>

## <a name="find-the-website-url"></a>Поиск URL-адреса веб-сайта

Вы можете просматривать страницы сайта из браузера, используя общедоступный URL-адрес веб-сайта.

### <a name="portal"></a>[Портал](#tab/azure-portal)

На панели, которая отображается рядом со страницей обзора учетной записи хранения, выберите **Статический веб-сайт**. URL-адрес сайта отображается в поле **Первичная конечная точка**.

![Выбор метрики для статических веб-сайтов службы хранилища Azure](./media/storage-blob-static-website/storage-blob-static-website-url.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Найдите общедоступный URL-адрес статического веб-сайта с помощью следующей команды:

```azurecli-interactive
az storage account show -n <storage-account-name> -g <resource-group-name> --query "primaryEndpoints.web" --output tsv
```

* Замените значение заполнителя `<storage-account-name>` именем вашей учетной записи хранения.

* Замените значение заполнителя `<resource-group-name>` именем своей группы ресурсов.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Найдите общедоступный URL-адрес статического веб-сайта с помощью следующей команды:

```powershell
 $storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -Name "<storage-account-name>"
Write-Output $storageAccount.PrimaryEndpoints.Web
```

* Замените значение заполнителя `<resource-group-name>` именем своей группы ресурсов.

* Замените значение заполнителя `<storage-account-name>` именем вашей учетной записи хранения.

---

<a id="metrics"></a>

## <a name="enable-metrics-on-static-website-pages"></a>Включение метрик на страницах статического веб-сайта

После включения метрик статистика трафика файлов в контейнере **$web** будет отображаться на панели мониторинга метрик.

1. В разделе **Монитор** меню учетной записи хранения щелкните **Метрики**.

   > [!div class="mx-imgBorder"]
   > ![Ссылка на метрики](./media/storage-blob-static-website/metrics-link.png)

   > [!NOTE]
   > Данные метрик создаются с помощью подключения к API для различных метрик. На портале отображаются только элементы API, используемые в течение заданного промежутка времени, чтобы сосредоточиться только на тех элементах, которые возвращают данные. Чтобы убедиться, что можно будет выбрать необходимый элемент API, сначала необходимо расширить временные рамки.

2. Нажмите кнопку интервала времени, выберите временные рамки и щелкните **Применить**.

   ![Диапазон времени метрик для статических веб-сайтов службы хранилища Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-time-range.png)

3. Из раскрывающегося списка *Пространство имен* выберите **BLOB-объекты**.

   ![Пространство имен метрик для статических веб-сайтов службы хранилища Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-namespace.png)

4. Затем выберите метрику **Исходящий трафик**.

   ![Снимок экрана, на котором показана метрика исходящего трафика статических веб-сайтов службы хранилища Azure.](./media/storage-blob-static-website/storage-blob-static-website-metrics-metric.png)

5. Выберите **Сумма** в селекторе *Агрегирование*.

   ![Агрегирование метрики для статических веб-сайтов службы хранилища Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-aggregation.png)

6. Щелкните кнопку **Добавить фильтр** и выберите **Имя API** в селекторе *Свойство*.

   ![Имя API метрики для статических веб-сайтов службы хранилища Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-api-name.png)

7. Установите флажок рядом с полем **GetWebContent** в селекторе *Значения* для заполнения отчета на основе метрик.

   ![Поле GetWebContent метрики для статических веб-сайтов службы хранилища Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-getwebcontent.png)

   >[!NOTE]
   > Флажок **GetWebContent** появляется, только если этот элемент API использовался в течение данного интервала времени. На портале отображаются только элементы API, используемые в течение заданного промежутка времени, чтобы сосредоточиться только на тех элементах, которые возвращают данные. Если не удается найти конкретный элемент API в списке, разверните временные рамки.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как настроить личный домен со статическим веб-сайтом. См. статью [Сопоставление личного домена с конечной точкой хранилища BLOB-объектов Azure](storage-custom-domain-name.md).