---
author: v-demjoh
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 05/15/2020
ms.author: v-demjoh
ms.openlocfilehash: ec34ba7b2d3bc026376aeb1cb91847d8eac1cac5
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "107073653"
---
## <a name="download-and-install"></a>Загрузите и установите

#### <a name="windows-install"></a>[Установка (Windows)](#tab/windowsinstall)

Вот как установить интерфейс командной строки службы "Речь" в Windows:

1. В Windows для вашей платформы необходим [распространяемый компонент Microsoft Visual C++ для Visual Studio 2019](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads). При первой установке может потребоваться перезагрузка.
1. Установите [пакет SDK для .NET Core 3.1](/dotnet/core/install/windows).
2. Установите CLI службы "Речь" с использованием NuGet. Для этого введите следующую команду:

   ```console
   dotnet tool install --global Microsoft.CognitiveServices.Speech.CLI
   ```
Введите `spx`, чтобы получить справку по интерфейсу командной строки службы "Речь".

> [!NOTE]
> В качестве альтернативы NuGet вы можете скачать и извлечь [ZIP-архив](https://aka.ms/speech/spx-zips.zip) CLI службы "Речь", найти и извлечь свою платформу из каталога `spx-zips` и добавить путь `spx` в системную переменную **PATH**.


### <a name="font-limitations"></a>Ограничения для шрифтов

В Windows в интерфейсе командной строки службы "Речь" могут отображаться только шрифты, доступные для командной строки на локальном компьютере.
[Терминал Windows](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701) поддерживает все шрифты, создаваемые интерфейсом командной строки службы "Речь" в интерактивном режиме.

При выводе результатов в файл текстовый редактор, например Блокнот, или веб-браузер, например Microsoft Edge, будут также отображать все шрифты.

#### <a name="linux-install"></a>[Установка (Linux)](#tab/linuxinstall)

Следующие дистрибутивы Linux поддерживаются для архитектур x64 с использованием CLI службы "Речь":

* CentOS 7/8;
* Debian 9 и 10; 
* Red Hat Enterprise Linux (RHEL) 7/8;
* Ubuntu 16.04/18.04/20.04.

> [!NOTE]
> Дополнительные архитектуры поддерживает пакет SDK службы "Речь" (не CLI службы "Речь"). См. дополнительные сведения о [пакете SDK службы "Речь"](../speech-sdk.md).

Вот как установить интерфейс командной строки службы "Речь" в Linux (ЦП с архитектурой x64):

1. Установите [пакет SDK для .NET Core 3.1](/dotnet/core/install/linux).
2. Установите CLI службы "Речь" с использованием NuGet. Для этого введите следующую команду:

    `dotnet tool install --global Microsoft.CognitiveServices.Speech.CLI`

Введите `spx`, чтобы получить справку по интерфейсу командной строки службы "Речь".

> [!NOTE]
> В качестве альтернативы NuGet вы можете скачать двоичные файлы в [ZIP-архив](https://aka.ms/speech/spx-zips.zip), извлечь `spx-netcore-30-linux-x64.zip` в новый каталог `~/spx`, ввести `sudo chmod +r+x spx` в двоичном файле и добавить путь `~/spx` в системную переменную PATH.


#### <a name="docker-install-windows-linux-macos"></a>[Установка Docker (Windows, Linux, macOS)](#tab/dockerinstall)

Чтобы установить CLI службы "Речь" в контейнере Docker:

1. При необходимости <a href="https://www.docker.com/get-started" target="_blank">установите Docker Desktop</a> для своей платформы.
2. В новой командной строке или в терминале введите следующую команду: .
   ```console   
   docker pull msftspeech/spx
   ```
3. Введите эту команду. Вы должны увидеть справочную информацию по интерфейсу командной строки службы "Речь": .
   ```console 
   docker run -it --rm msftspeech/spx help
   ```

### <a name="mount-a-directory-in-the-container"></a>Подключение каталога в контейнере

Средство командной строки службы "Речь" сохраняет параметры конфигурации в виде файлов и загружает их при выполнении любой команды (за исключением команд справки).
При использовании интерфейса командной строки службы "Речь" в контейнере Docker необходимо подключить локальный каталог из контейнера, чтобы средство могло сохранять или находить параметры конфигурации, а также считывать или записывать любые файлы (например, звуковые файлы с записанной речью), необходимые для выполнения команды.

В Windows введите следующую команду, чтобы создать локальный каталог, который интерфейс командной строки службы "Речь" сможет использовать из контейнера:

`mkdir c:\spx-data`

В Linux или macOS введите следующую команду в терминале, чтобы создать каталог и просмотреть абсолютный путь к нему:

```bash
mkdir ~/spx-data
cd ~/spx-data
pwd
```

При вызове интерфейса командной строки службы "Речь" используется абсолютный путь.

### <a name="run-speech-cli-in-the-container"></a>Запуск интерфейса командной строки службы "Речь" в контейнере

В этой документации показана команда `spx` интерфейса командной строки службы "Речь", используемая при установках, отличных от Docker.
При вызове команды `spx` в контейнере Docker необходимо подключить каталог в контейнере к файловой системе, в которой интерфейс командной строки службы "Речь" может хранить и находить значения конфигурации, а также считывать и записывать файлы.

Команды в Windows:

```console
docker run -it -v c:\spx-data:/data --rm msftspeech/spx
```

В Linux или macOS команды будут выглядеть как в примере ниже. Замените `ABSOLUTE_PATH` на абсолютный путь к подключенному каталогу. Этот путь был возвращен командой `pwd` в предыдущем разделе. 

Если выполнить эту команду перед настройкой ключа и региона, появится сообщение об ошибке, уведомляющее о необходимости настроить ключ и регион:
```console   
sudo docker run -it -v ABSOLUTE_PATH:/data --rm msftspeech/spx
```

Чтобы использовать команду `spx`, установленную в контейнере, всегда вводите полную версию команды, показанную выше, а затем — параметры запроса.
Например, в Windows эта команда задает ключ:

```console
docker run -it -v c:\spx-data:/data --rm msftspeech/spx config @key --set SUBSCRIPTION-KEY
```

Для более расширенного взаимодействия с программой командной строки можно запустить контейнер с интерактивной оболочкой bash, добавив параметр входной точки.
В Windows введите эту команду, чтобы запустить контейнер, который предоставляет интерактивный интерфейс командной строки, где можно ввести несколько команд `spx`:
```console
docker run -it --entrypoint=/bin/bash -v c:\spx-data:/data --rm msftspeech/spx
```

> [!WARNING]
> При запуске CLI службы "Речь" в контейнере Docker нельзя использовать микрофон или динамик компьютера. Но можно выполнять чтение и сохранение звуковых файлов в локальном подключенном каталоге. 

<!-- Need to troubleshoot issues with docker pull image

### Optional: Create a command line shortcut

If you're running the the Speech CLI from a Docker container on Linux or macOS you can create a shortcut. 

Follow these instructions to create a shortcut:
1. Open `.bash_profile` with your favorite text editor. For example:
   ```shell
   nano ~/.bash_profile
   ```
2. Next, add this function to your `.bash_profile`. Make sure you update this function with the correct path to your mounted directory:
   ```shell   
   spx(){
       sudo docker run -it -v ABSOLUTE_PATH:/data --rm msftspeech/spx
   }
   ```
3. Source your profile:
   ```shell
   source ~/.bash_profile
   ```
4. Now instead of running `sudo docker run -it -v ABSOLUTE_PATH:/data --rm msftspeech/spx`, you can just type `spx` followed by arguments. For example: 
   ```shell
   // Get some help
   spx help recognize

   // Recognize speech from an audio file 
   spx recognize --file /mounted/directory/file.wav
   ```

> [!WARNING]
> If you change the mounted directory that Docker is referencing, you need to update the function in `.bash_profile`.
--->
***

## <a name="create-subscription-config"></a>Создание конфигурации подписки

Чтобы начать работу с интерфейсом командной строки службы "Речь", необходимо ввести ключ подписки службы "Речь" и идентификатор региона. Чтобы получить эти учетные данные, выполните инструкции из раздела [Бесплатная пробная подписка на службу "Речь"](../overview.md#try-the-speech-service-for-free).
Указав ключ подписки и идентификатор региона (например, `eastus` и `westus`), выполните следующие команды.

```console
spx config @key --set SUBSCRIPTION-KEY
spx config @region --set REGION
```

Теперь проверку подлинности для подписки можно выполнять для будущих запросов SPX. Если необходимо удалить любое из этих сохраненных значений, выполните `spx config @region --clear` или `spx config @key --clear`.
