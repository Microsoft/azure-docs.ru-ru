---
title: Использование эмулятора Азурите для разработки локальных хранилищ Azure
description: Эмулятор с открытым исходным кодом Азурите предоставляет бесплатную локальную среду для тестирования приложений службы хранилища Azure.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/15/2020
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: b5051a5e29f9e34c43963a9a264ee47e2ad7b04a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92490787"
---
# <a name="use-the-azurite-emulator-for-local-azure-storage-development"></a>Использование эмулятора Азурите для разработки локальных хранилищ Azure

Эмулятор с открытым исходным кодом Азурите предоставляет бесплатную локальную среду для тестирования приложений хранилища BLOB-объектов и очередей Azure. Если вы удовлетворены работой приложения локально, переключитесь на использование учетной записи хранения Azure в облаке. Эмулятор обеспечивает кросс-платформенную поддержку в Windows, Linux и macOS.

Азурите — это будущая платформа эмулятора хранения. Азурите заменяет [эмулятор хранения Azure](storage-use-emulator.md). Азурите будет по-прежнему обновляться для поддержки последних версий API службы хранилища Azure.

Существует несколько различных способов установки и запуска Азурите в локальной системе:

  1. [Установка и запуск расширения Азурите Visual Studio Code](#install-and-run-the-azurite-visual-studio-code-extension)
  1. [Установка и запуск Азурите с помощью NPM](#install-and-run-azurite-by-using-npm)
  1. [Установка и запуск образа DOCKER Азурите](#install-and-run-the-azurite-docker-image)
  1. [Клонирование, сборка и запуск Азурите из репозитория GitHub](#clone-build-and-run-azurite-from-the-github-repository)

## <a name="install-and-run-the-azurite-visual-studio-code-extension"></a>Установка и запуск расширения Азурите Visual Studio Code

В Visual Studio Code выберите панель **расширения** и найдите *Азурите* в списке **расширения: MARKETPLACE**.

![Visual Studio Code расширений Marketplace](media/storage-use-azurite/azurite-vs-code-extension.png)

Вы также можете переходить к [Visual Studio Codeу расширения на рынке](https://marketplace.visualstudio.com/items?itemName=Azurite.azurite) в браузере. Нажмите кнопку " **установить** ", чтобы открыть Visual Studio Code и перейти непосредственно на страницу расширения азурите.

Расширение поддерживает следующие Visual Studio Code команды. Чтобы открыть палитру команд, нажмите клавишу F1 в Visual Studio Code. 

   - **Азурите: Clean** — сброс всех данных сохраняемости азурите Services
   - **Азурите: Очистка службы BLOB** -объектов
   - **Азурите: чистая** служба очереди — чистая служба очередей
   - **Азурите: закрыть** — закрыть все службы азурите
   - **Азурите: Close BLOB Service** — Close Service BLOB
   - **Азурите: закрытие службы очередей** — закрытие службы очередей
   - **Азурите: Start** — запуск всех служб азурите
   - **Азурите: Starting Service BLOB** -Start Service BLOB
   - **Азурите: запуск службы** очередей — запуск службы очередей

Чтобы настроить Азурите в Visual Studio Code, выберите Панель расширения. Выберите значок **Управление** (шестеренка) для **азурите**. Выберите **Параметры расширения**.

![Настройка параметров модуля азуритес](media/storage-use-azurite/azurite-configure-extension-settings.png)

Поддерживаются следующие параметры:

   - **Азурите: узел больших двоичных** объектов — конечная точка для прослушивания службы BLOB-объектов. Значение по умолчанию — 127.0.0.1.
   - **Азурите: порт большого двоичного объекта** — порт прослушивания службы BLOB-объектов. Порт по умолчанию — 10000.
   - **Азурите: CERT** — путь к локально доверенному пути к файлу сертификата PEM или PFX для включения режима HTTPS.
   - **Азурите: Debug** — вывод журнала отладки в канал азурите. Значение по умолчанию — **false**.
   - **Азурите: ключ** — путь к файлу с локально доверенным ключом PEM, необходимый, когда **азурите: CERT** указывает на PEM-файл.
   - **Азурите: Location** — путь к расположению рабочей области. Значение по умолчанию — Рабочая папка Visual Studio Code.
   - **Азурите: свободный** — включение неподдерживаемого режима, который игнорирует Неподдерживаемые заголовки и параметры.
   - **Азурите: OAuth** — необязательный уровень OAuth.
   - **Азурите: PWD** — пароль для PFX-файла. Требуется, если **азурите: CERT** указывает на PFX-файл.
   - **Азурите: узел очереди** — конечная точка прослушивания служба очередей. Значение по умолчанию — 127.0.0.1.
   - **Азурите: порт очереди** — порт, прослушиваемый служба очередей. Порт по умолчанию — 10001.
   - **Азурите: автоматический** режим без уведомления отключает журнал доступа. Значение по умолчанию — **false**.
   - **Азурите: пропустить проверку версии API** — пропустить проверку версии API запроса. Значение по умолчанию — **false**.

## <a name="install-and-run-azurite-by-using-npm"></a>Установка и запуск Азурите с помощью NPM

Этот метод установки требует наличия [Node.js версии 8,0 или более поздней](https://nodejs.org) . Диспетчер пакетов node (NPM) — это средство управления пакетами, поставляемое при каждой установке Node.js. После установки Node.js выполните следующую команду, `npm` чтобы установить азурите.

```console
npm install -g azurite
```

После установки Азурите см. раздел [Запуск азурите из командной строки](#run-azurite-from-a-command-line).

## <a name="install-and-run-the-azurite-docker-image"></a>Установка и запуск образа DOCKER Азурите

Используйте [DockerHub](https://hub.docker.com/) для извлечения [последнего образа азурите](https://hub.docker.com/_/microsoft-azure-storage-azurite) с помощью следующей команды:

```console
docker pull mcr.microsoft.com/azure-storage/azurite
```

**Запустите образ DOCKER азурите**:

Следующая команда запускает образ DOCKER Азурите. `-p 10000:10000`Параметр перенаправляет запросы с порта 10000 хоста компьютера на экземпляр DOCKER.

```console
docker run -p 10000:10000 -p 10001:10001 \
    mcr.microsoft.com/azure-storage/azurite
```

**Укажите расположение рабочей области**:

В следующем примере `-v c:/azurite:/data` параметр указывает *c:/азурите* как расположение сохраненных данных азурите. Перед выполнением команды DOCKER необходимо создать каталог *c:/азурите*.

```console
docker run -p 10000:10000 -p 10001:10001 \
    -v c:/azurite:/data mcr.microsoft.com/azure-storage/azurite
```

**Запуск только службы BLOB-объектов**

```console
docker run -p 10000:10000 mcr.microsoft.com/azure-storage/azurite \
    azurite-blob --blobHost 0.0.0.0 --blobPort 10000
```

Дополнительные сведения о настройке Азурите при запуске см. в разделе [Параметры командной строки](#command-line-options).

## <a name="clone-build-and-run-azurite-from-the-github-repository"></a>Клонирование, сборка и запуск Азурите из репозитория GitHub

Этот метод установки требует наличия установленного [Git](https://git-scm.com/) . Клонировать [репозиторий GitHub](https://github.com/azure/azurite) для проекта азурите можно с помощью следующей команды консоли.

```console
git clone https://github.com/Azure/Azurite.git
```

После клонирования исходного кода выполните следующие команды из корня клонированного репозитория, чтобы создать и установить Азурите.

```console
npm install
npm run build
npm install -g
```

После установки и сборки Азурите см. раздел [Запуск азурите из командной строки](#run-azurite-from-a-command-line).

## <a name="run-azurite-from-a-command-line"></a>Запуск Азурите из командной строки

> [!NOTE]
> Азурите невозможно запустить из командной строки, если установлено только расширение Visual Studio Code. Вместо этого используйте палитру команд Visual Studio Code. Дополнительные сведения см. [в статье Установка и запуск расширения азурите Visual Studio Code](#install-and-run-the-azurite-visual-studio-code-extension).

Чтобы немедленно приступить к работе с командной строкой, создайте каталог с именем *к:\азурите*, а затем запустите азурите, выполнив следующую команду:

```console
azurite --silent --location c:\azurite --debug c:\azurite\debug.log
```

Эта команда указывает Азурите хранить все данные в определенном каталоге, *к:\азурите*. Если `--location` параметр не указан, будет использоваться текущий рабочий каталог.

## <a name="command-line-options"></a>Параметры командной строки

В этом разделе описываются параметры командной строки, доступные при запуске Азурите.

### <a name="help"></a>Help

**Необязательно** — получение справки по командной строке с помощью `-h` `--help` параметра или.

```console
azurite -h
azurite --help
```

### <a name="blob-listening-host"></a>Узел прослушивания BLOB-объектов

**Необязательно** . по умолчанию азурите будет ожидать 127.0.0.1 в качестве локального сервера. Используйте `--blobHost` параметр, чтобы задать для адреса требования.

Принимать запросы только на локальном компьютере:

```console
azurite --blobHost 127.0.0.1
```

Разрешить удаленные запросы:

```console
azurite --blobHost 0.0.0.0
```

> [!CAUTION]
> Разрешение удаленных запросов может сделать систему уязвимой для внешних атак.

### <a name="blob-listening-port-configuration"></a>Конфигурация порта прослушивания BLOB-объектов

**Необязательно** . по умолчанию азурите будет прослушивать службу BLOB-объектов через порт 10000. Используйте `--blobPort` параметр, чтобы указать требуемый порт прослушивания.

> [!NOTE]
> После использования настроенного порта необходимо обновить строку подключения или соответствующую конфигурацию в средствах хранения Azure или пакетах SDK.

Настройте порт прослушивания службы BLOB-объектов:

```console
azurite --blobPort 8888
```

Разрешить системе автоматический выбор доступного порта:

```console
azurite --blobPort 0
```

Используемый порт отображается во время запуска Азурите.

### <a name="queue-listening-host"></a>Узел, слушающий очередь

**Необязательно** . по умолчанию азурите будет ожидать 127.0.0.1 в качестве локального сервера. Используйте `--queueHost` параметр, чтобы задать для адреса требования.

Принимать запросы только на локальном компьютере:

```console
azurite --queueHost 127.0.0.1
```

Разрешить удаленные запросы:

```console
azurite --queueHost 0.0.0.0
```

> [!CAUTION]
> Разрешение удаленных запросов может сделать систему уязвимой для внешних атак.

### <a name="queue-listening-port-configuration"></a>Настройка порта для прослушивания очереди

**Необязательно** . по умолчанию азурите будет прослушивать служба очередей через порт 10001. Используйте `--queuePort` параметр, чтобы указать требуемый порт прослушивания.

> [!NOTE]
> После использования настроенного порта необходимо обновить строку подключения или соответствующую конфигурацию в средствах хранения Azure или пакетах SDK.

Настройте порт прослушивания служба очередей:

```console
azurite --queuePort 8888
```

Разрешить системе автоматический выбор доступного порта:

```console
azurite --queuePort 0
```

Используемый порт отображается во время запуска Азурите.

### <a name="workspace-path"></a>Путь к рабочей области

**Дополнительно** -азурите сохраняет данные на локальный диск во время выполнения. Используйте `-l` параметр или, `--location` чтобы указать путь в качестве расположения рабочей области. По умолчанию будет использоваться текущий рабочий каталог процесса. Обратите внимание на строчные буквы "l".

```console
azurite -l c:\azurite
azurite --location c:\azurite
```

### <a name="access-log"></a>журнал доступа;

**Необязательно** . по умолчанию журнал доступа отображается в окне консоли. Отключите отображение журнала доступа с помощью `-s` `--silent` параметра или.

```console
azurite -s
azurite --silent
```
### <a name="debug-log"></a>Журнал отладки

**Необязательно** . журнал отладки содержит подробную информацию о каждой трассировке стека запросов и исключений. Включите журнал отладки, указав допустимый путь к локальному файлу для `-d` `--debug` параметра или.

```console
azurite -d path/debug.log
azurite --debug path/debug.log
```

### <a name="loose-mode"></a>Несвободный режим

**Необязательно** . по умолчанию азурите применяет режим "в режиме" для блокировки неподдерживаемых заголовков и параметров запроса. Отключите режим "в режиме" с помощью параметра `-L` или `--loose` . Обратите внимание на прописную букву "L".

```console
azurite -L
azurite --loose
```
### <a name="version"></a>Версия

**Необязательно** . Отображение установленного номера версии азурите с помощью параметра `-v` или `--version` .

```console
azurite -v
azurite --version
```

### <a name="certificate-configuration-https"></a>Конфигурация сертификата (HTTPS)

**Необязательно** . по умолчанию азурите использует протокол HTTP. Включите режим HTTPS, указав путь к файлу сертификата Privacy Enhanced Mail (PEM) или [обмена личной информацией (PFX)](/windows-hardware/drivers/install/personal-information-exchange---pfx--files) для `--cert` коммутатора.

Если `--cert` для PEM-файла указано значение, необходимо указать соответствующий `--key` параметр.

```console
azurite --cert path/server.pem --key path/key.pem
```

Если `--cert` для PFX-файла указано значение, необходимо указать соответствующий `--pwd` параметр.

```console
azurite --cert path/server.pfx --pwd pfxpassword
```

Подробные сведения о создании файлов PEM и PFX см. в разделе [Настройка HTTPS](https://github.com/Azure/Azurite/blob/master/README.md#https-setup).

### <a name="oauth-configuration"></a>Конфигурация OAuth

**Необязательно** . Включите проверку подлинности OAuth для азурите с помощью `--oauth` параметра.

```console
azurite --oauth basic --cert path/server.pem --key path/key.pem
```

> [!NOTE]
> Для OAuth требуется конечная точка HTTPS. Убедитесь, что протокол HTTPS включен, предоставляя `--cert` параметр вместе с `--oauth` параметром.

Азурите поддерживает обычную проверку подлинности, указывая `basic` параметр для `--oauth` переключателя. Азурите выполняет обычную проверку подлинности, например проверку поступающего токена носителя, проверку издателя, аудиторию и срок действия. Азурите не проверяет подпись или разрешения маркера.

### <a name="skip-api-version-check"></a>Пропустить проверку версии API

**Необязательно** . при запуске азурите проверяет допустимость запрошенной версии API. Следующая команда пропускает проверку версии API:

```console
azurite --skipApiVersionCheck
```


## <a name="authorization-for-tools-and-sdks"></a>Авторизация для средств и пакетов SDK

Подключайтесь к Азурите из пакетов SDK или средств службы хранилища Azure, например [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/), используя любую стратегию проверки подлинности. Требуется проверка подлинности. Азурите поддерживает авторизацию с помощью OAuth, общего ключа и подписанного URL-адресом (SAS). Азурите также поддерживает анонимный доступ к открытым контейнерам.

Если вы используете пакеты SDK для Azure, запустите Азурите с `--oauth basic and --cert --key/--pwd` параметрами.

### <a name="well-known-storage-account-and-key"></a>Хорошо известная учетная запись хранения и ключ

Азурите принимает одну и ту же известную учетную запись и ключ, используемые устаревшим эмулятором хранения Azure.

- Имя учетной записи: `devstoreaccount1`
- Ключ учетной записи: `Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==`

### <a name="custom-storage-accounts-and-keys"></a>Пользовательские учетные записи хранения и ключи

Азурите поддерживает пользовательские имена и ключи учетных записей хранения, задавая `AZURITE_ACCOUNTS` переменную среды в следующем формате: `account1:key1[:key2];account2:key1[:key2];...` .

Например, используйте настраиваемую учетную запись хранения с одним ключом:

```cmd
set AZURITE_ACCOUNTS="account1:key1"
```

```bash
export AZURITE_ACCOUNTS="account1:key1"
```

Или используйте несколько учетных записей хранения с двумя ключами:

```cmd
set AZURITE_ACCOUNTS="account1:key1:key2;account2:key1:key2"
```

```bash
export AZURITE_ACCOUNTS="account1:key1:key2;account2:key1:key2"
```

По умолчанию азурите обновляет имена пользовательских учетных записей и ключи из переменной среды каждую минуту. С помощью этой функции можно динамически поворачивать ключ учетной записи или добавлять новые учетные записи хранения без перезапуска Азурите.

> [!NOTE]
> `devstoreaccount1`При задании пользовательских учетных записей хранения учетная запись хранения по умолчанию отключена.

### <a name="connection-strings"></a>Строки подключения

Самый простой способ подключиться к Азурите из приложения — настроить строку подключения в файле конфигурации приложения, который ссылается на ярлык *UseDevelopmentStorage = true*. Ниже приведен пример строки подключения в файле *app.config* .

```xml
<appSettings>
  <add key="StorageConnectionString" value="UseDevelopmentStorage=true" />
</appSettings>
```

#### <a name="http-connection-strings"></a>Строки подключения HTTP

В [пакеты SDK](https://aka.ms/azsdk) или средства Azure можно передать следующие строки подключения, например Azure CLI 2,0 или обозреватель службы хранилища.

Полная строка подключения:

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;`

Чтобы подключиться только к службе BLOB-объектов, строка подключения:

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;`

Для подключения к службе очередей используется строка подключения:

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;`

#### <a name="https-connection-strings"></a>Строки подключения HTTPS

Полная строка подключения HTTPS:

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;`

Чтобы использовать только службу BLOB-объектов, строка подключения HTTPS имеет следующее значение:

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;`

Чтобы использовать только службу очереди, строка подключения HTTPS имеет следующее значение:

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;`

Если вы использовали `dotnet dev-certs` для создания самозаверяющего сертификата, используйте следующую строку подключения.

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://localhost:10000/devstoreaccount1;QueueEndpoint=https://localhost:10001/devstoreaccount1;`

Обновите строку подключения при использовании [пользовательских учетных записей хранения и ключей](#custom-storage-accounts-and-keys).

Дополнительные сведения см. в статье [Настройка строк подключения службы хранилища Azure](storage-configure-connection-string.md).

### <a name="azure-sdks"></a>Пакеты Azure SDK

Чтобы использовать Азурите с пакетами [SDK для Azure](https://aka.ms/azsdk), используйте параметры OAuth и HTTPS.

```console
azurite --oauth basic --cert certname.pem --key certname-key.pem
```

#### <a name="azure-blob-storage"></a>хранилище BLOB-объектов Azure

Затем можно создать экземпляр Блобконтаинерклиент, используются службой blobserviceclient или Блобклиент.

```csharp
// With container URL and DefaultAzureCredential
var client = new BlobContainerClient(
    new Uri("https://127.0.0.1:10000/devstoreaccount1/container-name"), new DefaultAzureCredential()
  );

// With connection string
var client = new BlobContainerClient(
    "DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;", "container-name"
  );

// With account name and key
var client = new BlobContainerClient(
    new Uri("https://127.0.0.1:10000/devstoreaccount1/container-name"),
    new StorageSharedKeyCredential("devstoreaccount1", "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==")
  );
```

#### <a name="azure-queue-storage"></a>Хранилище очередей Azure

Можно также создать экземпляр QueueClient или Куеуесервицеклиент.

```csharp
// With queue URL and DefaultAzureCredential
var client = new QueueClient(
    new Uri("https://127.0.0.1:10001/devstoreaccount1/queue-name"), new DefaultAzureCredential()
  );

// With connection string
var client = new QueueClient(
    "DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;", "queue-name"
  );

// With account name and key
var client = new QueueClient(
    new Uri("https://127.0.0.1:10001/devstoreaccount1/queue-name"),
    new StorageSharedKeyCredential("devstoreaccount1", "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==")
  );
```

### <a name="microsoft-azure-storage-explorer"></a>Обозреватель службы хранилища Microsoft Azure

Для просмотра данных, хранящихся в Азурите, можно использовать Обозреватель службы хранилища.

#### <a name="connect-to-azurite-using-http"></a>Подключение к Азурите по протоколу HTTP

В Обозреватель службы хранилища подключитесь к Азурите, выполнив следующие действия.

 1. Щелкните значок " **Управление учетными записями** "
 1. Выберите **Добавить учетную запись**
 1. Выберите **присоединить к локальному эмулятору**
 1. Щелкните **Далее**.
 1. Измените поле **Отображаемое имя** на нужное имя.
 1. Нажмите кнопку " **Далее** " еще раз
 1. Выбор **подключения**

#### <a name="connect-to-azurite-using-https"></a>Подключение к Азурите с помощью HTTPS

По умолчанию Обозреватель службы хранилища не открывает конечную точку HTTPS, использующую самозаверяющий сертификат. Если вы используете Азурите с протоколом HTTPS, скорее всего, вы используете самозаверяющий сертификат. В обозреватель службы хранилища импортируйте SSL-сертификаты с помощью  ->    ->   диалогового окна Изменение сертификатов SSL.

##### <a name="import-certificate-to-storage-explorer"></a>Импорт сертификата в Обозреватель службы хранилища

1. Найдите сертификат на локальном компьютере.
1. В обозреватель службы хранилища перейдите к разделу **изменение**  ->  **SSL-сертификатов**  ->  **Импорт сертификатов** и импорт сертификата.

Если вы не импортируете сертификат, вы получите следующее сообщение об ошибке:

`unable to verify the first certificate` или `self signed certificate in chain`

##### <a name="add-azurite-via-https-connection-string"></a>Добавление Азурите с помощью строки подключения HTTPS

Чтобы добавить Азурите HTTPS в Обозреватель службы хранилища, выполните следующие действия.

1. Выбор **обозревателя**
1. Выберите **Локальный подключенный &**
1. Щелкните правой кнопкой мыши **учетные записи хранения** и выберите **подключиться к службе хранилища Azure**.
1. Выберите **использовать строку подключения**
1. Выберите **Далее**.
1. Введите значение в поле **Отображаемое имя** .
1. Введите [строку подключения HTTPS](#https-connection-strings) из предыдущего раздела этого документа
1. Щелкните **Далее**.
1. Выбор **подключения**

## <a name="workspace-structure"></a>Структура рабочей области

При инициализации Азурите в расположении рабочей области могут быть созданы следующие файлы и папки.

- `__blobstorage__` — Каталог, содержащий сохраняемые двоичные данные службы BLOB-объектов Азурите
- `__queuestorage__` — Каталог, содержащий сохраняемые двоичные данные службы очередей Азурите
- `__azurite_db_blob__.json` — Файл метаданных службы BLOB-объектов азурите
- `__azurite_db_blob_extent__.json` — Файл метаданных экстента службы BLOB-объектов азурите
- `__azurite_db_queue__.json` -Файл метаданных службы очереди азурите
- `__azurite_db_queue_extent__.json` -Файл метаданных экстента службы очередей азурите

Чтобы очистить Азурите, удалите файлы и папки и перезапустите эмулятор.

## <a name="differences-between-azurite-and-azure-storage"></a>Различия между Азурите и хранилищем Azure

Между локальным экземпляром Азурите и учетной записью хранения Azure в облаке существуют функциональные различия.

### <a name="endpoint-and-connection-url"></a>Конечная точка и URL-адрес подключения

Конечные точки службы для Азурите отличаются от конечных точек учетной записи хранения Azure. Локальный компьютер не выполняет разрешение имен доменов, поэтому конечные точки Азурите должны быть локальными адресами.

При адресации ресурса в учетной записи хранения Azure имя учетной записи является частью имени узла URI. Ресурс, к которому выполняется обращение, является частью пути URI:

`<http|https>://<account-name>.<service-name>.core.windows.net/<resource-path>`

Следующий URI является допустимым адресом для большого двоичного объекта в учетной записи хранения Azure:

`https://myaccount.blob.core.windows.net/mycontainer/myblob.txt`

Так как локальный компьютер не выполняет разрешение доменных имен, имя учетной записи является частью пути URI, а не имени узла. Используйте следующий формат URI для ресурса в Азурите:

`http://<local-machine-address>:<port>/<account-name>/<resource-path>`

Для доступа к большому двоичному объекту в Азурите можно использовать следующий адрес:

`http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt`

### <a name="scaling-and-performance"></a>Масштабирование и производительность

Азурите не поддерживает большое число подключенных клиентов. Гарантия производительности не гарантируется. Азурите предназначен для целей разработки и тестирования.

### <a name="error-handling"></a>Обработка ошибок

Азурите соответствует логике обработки ошибок службы хранилища Azure, но существуют различия. Например, сообщения об ошибках могут отличаться, а коды состояния ошибок выровняйте.

### <a name="ra-grs"></a>RA-GRS

Азурите поддерживает геоизбыточную репликацию с доступом на чтение (RA-GRS). Для ресурсов хранилища получите доступ к дополнительному расположению, добавив `-secondary` к имени учетной записи. Например, следующий адрес может использоваться для доступа к большому двоичному объекту с помощью вторичной реплики "только для чтения" в Азурите:

`http://127.0.0.1:10000/devstoreaccount1-secondary/mycontainer/myblob.txt`

### <a name="table-support"></a>Поддержка таблиц

Поддержка таблиц в Азурите в настоящее время находится на стадии разработки и открыта для публикации. Чтобы получить актуальный ход выполнения, проверьте проект [таблицы Азурите v3](https://github.com/Azure/Azurite/wiki/Azurite-V3-Table) .

Для поддержки устойчивых функций требуются таблицы.

## <a name="azurite-is-open-source"></a>Азурите является открытым кодом

Публикации и предложения для Азурите — Добро пожаловать. Перейдите на страницу [проекта Азурите GitHub](https://github.com/Azure/Azurite/projects) или [проблемы GitHub](https://github.com/Azure/Azurite/issues) для вех и рабочих элементов, которые отслеживаются для предстоящих компонентов и исправлений ошибок. Подробные рабочие элементы также отправляются в GitHub.

## <a name="next-steps"></a>Дальнейшие действия

- [Используйте эмулятор хранения Azure для разработки и тестирования](storage-use-emulator.md) документов в старом эмуляторе хранения Azure, который заменяется азурите.
- [Настройка строк подключения службы хранилища Azure](storage-configure-connection-string.md) содержит сведения о том, как собрать допустимую строку подключения к службе хранилища Azure.
