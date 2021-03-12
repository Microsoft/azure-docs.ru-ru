---
title: Тестирование образа виртуальной машины Azure для Azure Marketplace
description: Узнайте, как протестировать и отправить предложение виртуальной машины Azure в Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: kriti-ms
ms.author: krsh
ms.date: 03/10/2021
ms.openlocfilehash: 9ffba221625c57332cd695125651d92adc11cf60
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103200376"
---
# <a name="test-a-virtual-machine-image"></a>Тестирование образа виртуальной машины

В этом разделе рассматриваются шаги по тестированию образа виртуальной машины для развертывания в Azure Marketplace.

## <a name="deploy-an-azure-vm"></a>Развертывание виртуальной машины Azure

Чтобы развернуть виртуальную машину из общего образа коллекции образов, выполните следующие действия.

1. Переход к версии образа коллекции общих образов
1. Щелкните "создать виртуальную машину".
1. Укажите имя виртуальной машины и выберите размер виртуальной машины.
1. Щелкните Просмотреть и создать. После прохождения проверки нажмите кнопку Создать.

> [!NOTE]
> Если необходимо создать виртуальную машину из VHD-файла, следуйте инструкциям в следующих статьях, [подготовьте шаблон Azure Resource Manager](https://docs.microsoft.com/azure/marketplace/azure-vm-image-test#prepare-an-azure-resource-manager-template) или [разверните виртуальную машину Azure с помощью PowerShell](https://docs.microsoft.com/azure/marketplace/azure-vm-image-test#deploy-an-azure-vm-using-powershell).

В этой статье описывается, как выполнить тестирование и отправку образа виртуальной машины на коммерческой платформе, гарантировав ее соответствие актуальным требованиям к публикации в Azure Marketplace.

Перед отправкой предложения виртуальной машины выполните следующие шаги:

- Развертывание виртуальной машины Azure с помощью обобщенного образа; Ознакомьтесь с разрешениями [Создание виртуальной машины с помощью утвержденной базы](azure-vm-create-using-approved-base.md) или [Создание виртуальной машины с помощью собственного образа](azure-vm-create-using-own-image.md).
- Выполните проверки.

## <a name="run-validations"></a>Выполнение проверок

Выполнять проверки на развернутом образе можно двумя способами.

### <a name="use-certification-test-tool-for-azure-certified"></a>с помощью средств проверки сертификации для Azure Certitfied;

#### <a name="download-and-run-the-certification-test-tool"></a>Загрузка и запуск средства проверки сертификации

Средство проверки сертификации для Azure Certified выполняется на локальном компьютере Windows, но тестирует виртуальную машину Azure на базе Windows или Linux. Оно проверяет, можно ли использовать пользовательский образ виртуальной машины в Microsoft Azure и выполнены ли рекомендации и требования по подготовке виртуального жесткого диска.

1. Скачайте и установите [средство проверки сертификации Azure](https://www.microsoft.com/download/details.aspx?id=44299) последней версии.
2. Откройте средство сертификации и щелкните **Запустить новую проверку**.
3. На экране Test Information (Сведения о проверке) введите **имя теста** для тестового запуска.
4. Выберите платформу для виртуальной машины: **Windows Server** или **Linux**. Выбор платформы влияет на остальные параметры.
5. Если ваша виртуальная машина использует эту службу базы данных, установите флажок **Test for Azure SQL Database** (Тестирование для Базы данных SQL Azure).

#### <a name="connect-the-certification-tool-to-a-vm-image"></a>Подключение средства сертификации к образу виртуальной машины

1. Выберите режим SSH Authentication (Аутентификация по SSH): Аутентификация по паролю или по файлу ключа.
2. Если используется проверка подлинности на основе пароля, введите значения для **DNS-имени виртуальной машины**, **имени пользователя** и **пароля**. Вы также можете изменить указанный по умолчанию SSH Port (Порт SSH).

    :::image type="content" source="media/vm/azure-vm-cert-2.png" alt-text="Отображает выбор сведений о тесте виртуальной машины.":::

3. При использовании проверки подлинности на основе файла ключа введите значения DNS-имени, имени пользователя и расположение закрытого ключа. Также можно предоставить значение Passphrase (Парольная фраза) или изменить указанный по умолчанию SSH Port (Порт SSH).
4. Введите полное доменное имя виртуальной машины (например, MyVMName.ClOudapp.net).
5. Введите **имя пользователя** и **пароль**.

    :::image type="content" source="media/vm/azure-vm-cert-4.png" alt-text="Отображает выбор имени пользователя и пароля виртуальной машины.":::

6. Щелкните **Далее**.

#### <a name="run-a-certification-test"></a>Выполнение теста сертификации

После того как вы задаете значения параметров для образа виртуальной машины в средстве сертификации, выберите Проверить подключение, чтобы создать допустимое подключение к виртуальной машине. Когда подключение будет проверено, нажмите кнопку **Далее**, чтобы начать тестирование. По завершении теста результаты отображаются в таблице. В столбце "Состояние" для каждого теста отображается одно из значений: Pass (Успех), Fail (Неудача) или Warning (Предупреждение). Если хотя бы один из тестов не будет пройден, образ не будет сертифицирован. В этом случае ознакомьтесь с требованиями и сообщениями об ошибках, внесите предложенные изменения и повторно выполните тест.

После завершения автоматической проверки укажите дополнительную информацию об этом образе виртуальной машины, перейдя в окне Questionnaire (Анкета) на вкладки General Assessment (Общая оценка) и Kernel Customization (Настройка ядра), затем щелкните Далее.

Последний экран позволяет предоставить дополнительные сведения, такие как сведения о доступе к SSH для образа виртуальной машины Linux, а также пояснения к неудачным оценкам, если вы ищете исключения.

Наконец, щелкните Generate Report (Сформировать отчет), чтобы скачать результаты проверки и файлы журналов по всем выполненным тестам в комплекте с ответами на вопросы анкеты.
> [!Note]
> У нескольких издателей есть сценарии, в которых виртуальные машины должны быть заблокированы, так как на виртуальной машине установлены такие программы, как брандмауэры. В этом случае Скачайте [сертифицированное средство тестирования](https://aka.ms/AzureCertificationTestTool) и отправьте отчет в [службу поддержки](https://aka.ms/marketplacepublishersupport)центра партнеров.

## <a name="how-to-use-powershell-to-consume-the-self-test-api"></a>Использование PowerShell для использования API Self-Test

### <a name="on-linux-os"></a>В ОС Linux

Вызовите API в PowerShell:

1. Для вызова API используйте команду Invoke-WebRequest.
2. Укажите метод Post и тип содержимого в формате JSON, как показано ниже в примере кода и на снимке экрана.
3. Укажите параметры Body в формате JSON.

В следующем примере показан вызов PowerShell для API:

```POWERSHELL
$accesstoken = "token"
$headers = @{ "Authorization" = "Bearer $accesstoken" }
$DNSName = "<Machine DNS Name>"
$UserName = "<User ID>"
$Password = "<Password>"
$OS = "Linux"
$PortNo = "22"
$CompanyName = "ABCD"
$AppID = "<Application ID>"
$TenantId = "<Tenant ID>"

$body = @{
   "DNSName" = $DNSName
   "UserName" = $UserName
   "Password" = $Password
   "OS" = $OS
   "PortNo" = $PortNo
   "CompanyName" = $CompanyName
   "AppID" = $AppID
   "TenantId" = $TenantId
} | ConvertTo-Json

$body

$uri = "URL"

$res = (Invoke-WebRequest -Method "Post" -Uri $uri -Body $body -ContentType "application/json" -Headers $headers).Content
```

<br>Ниже приведен пример вызова API в PowerShell.

[![Пример с экраном для вызова API в PowerShell.](media/vm/call-api-in-powershell.png)](media/vm/call-api-in-powershell.png#lightbox)

<br>Выполнив предыдущий пример, вы получите ответ в формате JSON, анализ которого даст вам следующие сведения.

```PowerShell
$resVar = $res | ConvertFrom-Json
$actualresult = $resVar.Response | ConvertFrom-Json

Write-Host "OSName: $($actualresult.OSName)"
Write-Host "OSVersion: $($actualresult.OSVersion)"
Write-Host "Overall Test Result: $($actualresult.TestResult)"

For ($i = 0; $i -lt $actualresult.Tests.Length; $i++) {
   Write-Host "TestID: $($actualresult.Tests[$i].TestID)"
   Write-Host "TestCaseName: $($actualresult.Tests[$i].TestCaseName)"
   Write-Host "Description: $($actualresult.Tests[$i].Description)"
   Write-Host "Result: $($actualresult.Tests[$i].Result)"
   Write-Host "ActualValue: $($actualresult.Tests[$i].ActualValue)"
}
```

<br>На этом образце экрана отображаются `$res.Content` подробные сведения о результатах теста в формате JSON:

[![Пример экрана для вызова API в PowerShell с подробными сведениями о результатах теста.](media/vm/call-api-in-powershell-details.png)](media/vm/call-api-in-powershell-details.png#lightbox)

<br>Ниже приведен пример результатов теста JSON, просматриваемых в интерактивном средстве просмотра JSON (например, [Code беаутифи](https://codebeautify.org/jsonviewer) или [JSON Viewer](https://jsonformatter.org/json-viewer)).

![Дополнительные результаты тестирования в интерактивном средстве просмотра JSON.](media/vm/test-results-json-viewer-1.png)

### <a name="on-windows-os"></a>В ОС Windows

Вызовите API в PowerShell:

1. Для вызова API используйте команду Invoke-WebRequest.
2. Метод является POST, а тип содержимого — JSON, как показано в следующем примере кода и на образце экрана.
3. Создайте параметры текста в формате JSON.

В этом примере кода показан вызов PowerShell для API:

```PowerShell
$accesstoken = "Get token for your Client AAD App"
$headers = @{ "Authorization" = "Bearer $accesstoken" }
$Body = @{ 
   "DNSName" = "XXXX.westus.cloudapp.azure.com"
   "UserName" = "XXX"
   "Password" = "XXX@123456"
   "OS" = "Windows"
   "PortNo" = "5986"
   "CompanyName" = "ABCD"
   "AppID" = "XXXX-XXXX-XXXX"
   "TenantId" = "XXXX-XXXX-XXXX"
} | ConvertTo-Json

$res = Invoke-WebRequest -Method "Post" -Uri $uri -Body $Body -ContentType "application/json" –Headers $headers;
$Content = $res | ConvertFrom-Json
```

На этих примерах экранов показан пример вызова API в PowerShell:

**С помощью ключа SSH**:

 :::image type="content" source="media/vm/call-api-with-ssh-key.png" alt-text="Вызов API в PowerShell с помощью ключа SSH.":::

**С паролем**:

 :::image type="content" source="media/vm/call-api-with-password.png" alt-text="Вызов API в PowerShell с паролем.":::

<br>Выполнив предыдущий пример, вы получите ответ в формате JSON, анализ которого даст вам следующие сведения.

```PowerShell
$resVar = $res | ConvertFrom-Json
$actualresult = $resVar.Response | ConvertFrom-Json

Write-Host "OSName: $($actualresult.OSName)"
Write-Host "OSVersion: $($actualresult.OSVersion)"
Write-Host "Overall Test Result: $($actualresult.TestResult)"

For ($i = 0; $i -lt $actualresult.Tests.Length; $i++) {
   Write-Host "TestID: $($actualresult.Tests[$i].TestID)"
   Write-Host "TestCaseName: $($actualresult.Tests[$i].TestCaseName)"
   Write-Host "Description: $($actualresult.Tests[$i].Description)"
   Write-Host "Result: $($actualresult.Tests[$i].Result)"
   Write-Host "ActualValue: $($actualresult.Tests[$i].ActualValue)"
}
```

<br>На этом экране отображаются `$res.Content` подробные сведения о результатах теста в формате JSON:

 :::image type="content" source="media/vm/test-results-json-format.png" alt-text="Подробные сведения о результатах теста в формате JSON.":::

<br>Ниже приведен пример результатов теста, просматриваемых в интерактивном средстве просмотра JSON (например, [Code беаутифи](https://codebeautify.org/jsonviewer) или [JSON Viewer](https://jsonformatter.org/json-viewer)).

![Результаты теста в интерактивном средстве просмотра JSON.](media/vm/test-results-json-viewer-2.png)

## <a name="how-to-use-curl-to-consume-the-self-test-api-on-linux-os"></a>Использование функции "перелистывание" для использования API Self-Test в ОС Linux

В этом примере для выполнения вызова API POST к Azure Active Directory и Self-Host виртуальной машине будет использоваться функция «перелистывание».

1. Запрос маркера Azure AD для проверки подлинности в виртуальной машине с самостоятельным размещением

   Убедитесь, что в запросе с фигурами указаны правильные значения.

   ```JSON
   curl --location --request POST 'https://login.microsoftonline.com/{TENANT_ID}/oauth2/token' \
   --header 'Content-Type: application/x-www-form-urlencoded' \
   --data-urlencode 'grant_type=client_credentials' \
   --data-urlencode 'client_id={CLIENT_ID} ' \
   --data-urlencode 'client_secret={CLIENT_SECRET}' \
   --data-urlencode 'resource=https://management.core.windows.net'
   ```

   Ниже приведен пример ответа из запроса.

   ```JSON
   {
       "token_type": "Bearer",
       "expires_in": "86399",
       "ext_expires_in": "86399",
       "expires_on": "1599663998",
       "not_before": "1599577298",
       "resource": "https://management.core.windows.net",
       "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJS…"
   }
   ```

2. Отправка запроса на виртуальную машину с самотестированием

   Убедитесь, что токен и параметры носителя заменены правильными значениями.

   ```JSON
   curl --location --request POST 'https://isvapp.azurewebsites.net/selftest-vm' \
   --header 'Content-Type: application/json' \
   --header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJS…' \
   --data-raw '{
       "DNSName": "avvm1.eastus.cloudapp.azure.com",
       "UserName": "azureuser",
       "Password": "SECURE_PASSWORD_FOR_THE_SSH_INTO_VM",
       "OS": "Linux",
       "PortNo": "22",
       "CompanyName": "COMPANY_NAME",
       "AppId": "CLIENT_ID_SAME_AS_USED_FOR_AAD_TOKEN ",
       "TenantId": "TENANT_ID_SAME_AS_USED_FOR_AAD_TOKEN"
   }'
   ```

   Пример ответа от вызова API виртуальной машины с самотестированием

   ```JSON
   {
       "TrackingId": "9bffc887-dd1d-40dd-a8a2-34cee4f4c4c3",
       "Response": "{\"SchemaVersion\":1,\"AppCertificationCategory\":\"Microsoft Single VM Certification\",\"ProviderID\":\"050DE427-2A99-46D4-817C-5354D3BF2AE8\",\"OSName\":\"Ubuntu 18.04\",\"OSDistro\":\"Ubuntu 18.04.5 LTS\",\"KernelVersion\":\"5.4.0-1023-azure\",\"KernelFullVersion\":\"Linux version 5.4.0-1023-azure (buildd@lgw01-amd64-053) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #23~18.04.1-Ubuntu SMP Thu Aug 20 14:46:48 UTC 2020\\n\",\"OSVersion\":\"18.04\",\"CreatedDate\":\"09/08/2020 01:13:47\",\"TestResult\":\"Pass\",\"APIVersion\":\"1.5\",\"Tests\":[{\"TestID\":\"48\",\"TestCaseName\":\"Bash History\",\"Description\":\"Bash history files should be cleared before creating the VM image.\",\"Result\":\"Passed\",\"ActualValue\":\"No file Exist\",\"RequiredValue\":\"1024\"},{\"TestID\":\"39\",\"TestCaseName\":\"Linux Agent Version\",\"Description\":\"Azure Linux Agent Version 2.2.41 and above should be installed.\",\"Result\":\"Passed\",\"ActualValue\":\"2.2.49\",\"RequiredValue\":\"2.2.41\"},{\"TestID\":\"40\",\"TestCaseName\":\"Required Kernel Parameters\",\"Description\":\"Verifies the following kernel parameters are set console=ttyS0, earlyprintk=ttyS0, rootdelay=300\",\"Result\":\"Warning\",\"ActualValue\":\"Missing Parameter: rootdelay=300\\r\\nMatched Parameter: console=ttyS0,earlyprintk=ttyS0\",\"RequiredValue\":\"console=ttyS0#earlyprintk=ttyS0#rootdelay=300\"},{\"TestID\":\"41\",\"TestCaseName\":\"Swap Partition on OS Disk\",\"Description\":\"Verifies that no Swap partitions are created on the OS disk.\",\"Result\":\"Passed\",\"ActualValue\":\"No. of Swap Partitions: 0\",\"RequiredValue\":\"swap\"},{\"TestID\":\"42\",\"TestCaseName\":\"Root Partition on OS Disk\",\"Description\":\"It is recommended that a single root partition is created for the OS disk.\",\"Result\":\"Passed\",\"ActualValue\":\"Root Partition: 1\",\"RequiredValue\":\"1\"},{\"TestID\":\"44\",\"TestCaseName\":\"OpenSSL Version\",\"Description\":\"OpenSSL Version should be >=0.9.8.\",\"Result\":\"Passed\",\"ActualValue\":\"1.1.1\",\"RequiredValue\":\"0.9.8\"},{\"TestID\":\"45\",\"TestCaseName\":\"Python Version\",\"Description\":\"Python version 2.6+ is highly recommended. \",\"Result\":\"Passed\",\"ActualValue\":\"2.7.17\",\"RequiredValue\":\"2.6\"},{\"TestID\":\"46\",\"TestCaseName\":\"Client Alive Interval\",\"Description\":\"It is recommended to set ClientAliveInterval to 180. On the application need, it can be set between 30 to 235. \\nIf you are enabling the SSH for your end users this value must be set as explained.\",\"Result\":\"Warning\",\"ActualValue\":\"120\",\"RequiredValue\":\"ClientAliveInterval 180\"},{\"TestID\":\"49\",\"TestCaseName\":\"OS Architecture\",\"Description\":\"Only 64-bit operating system should be supported.\",\"Result\":\"Passed\",\"ActualValue\":\"x86_64\\n\",\"RequiredValue\":\"x86_64,amd64\"},{\"TestID\":\"50\",\"TestCaseName\":\"Security threats\",\"Description\":\"Identifies OS with recent high profile vulnerability that may need patching.  Ignore warning if system was patched as appropriate.\",\"Result\":\"Passed\",\"ActualValue\":\"Ubuntu 18.04\",\"RequiredValue\":\"OS impacted by GHOSTS\"},{\"TestID\":\"51\",\"TestCaseName\":\"Auto Update\",\"Description\":\"Identifies if Linux Agent Auto Update is enabled or not.\",\"Result\":\"Passed\",\"ActualValue\":\"# AutoUpdate.Enabled=y\\n\",\"RequiredValue\":\"Yes\"},{\"TestID\":\"52\",\"TestCaseName\":\"SACK Vulnerability patch verification\",\"Description\":\"Checks if the running Kernel Version has SACK vulnerability patch.\",\"Result\":\"Passed\",\"ActualValue\":\"Ubuntu 18.04.5 LTS,Linux version 5.4.0-1023-azure (buildd@lgw01-amd64-053) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #23~18.04.1-Ubuntu SMP Thu Aug 20 14:46:48 UTC 2020\",\"RequiredValue\":\"Yes\"}]}"
   }
   ```

## <a name="next-steps"></a>Дальнейшие действия

- Войдите в [Центр партнеров](https://partner.microsoft.com/).
