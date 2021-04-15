---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: trbye
ms.openlocfilehash: 4f542970db5cf7015df4e0f276fccd8782d7beee
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105152709"
---
а также как установить [пакет SDK для службы "Речь"](~/articles/cognitive-services/speech-service/speech-sdk.md) для Python. Если вам нужно только имя пакета, чтобы приступить к работе самостоятельно, выполните `pip install azure-cognitiveservices-speech`.

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="supported-operating-systems"></a>Поддерживаемые операционные системы

- Пакет SDK службы "Речь" для Python доступен для таких операционных систем:
  - Windows: x64 и x86.
  - MAC: macOS X версии 10.12 или более поздней.
  - Linux: см. список [поддерживаемых дистрибутивов Linux и целевых архитектур](~/articles/cognitive-services/speech-service/speech-sdk.md).

## <a name="prerequisites"></a>Предварительные требования

- В Windows для вашей платформы необходим [распространяемый компонент Microsoft Visual C++ для Visual Studio 2019](https://support.microsoft.com/en-us/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0). При первой установке может потребоваться перезагрузка.

- Для поддерживаемых платформ Linux потребуется установить определенные библиотеки (`libssl` для поддержки протокола SSL и `libasound2` для поддержки звука). Ниже приведены команды, необходимые для установки правильных версий этих библиотек. Используйте команды для своего дистрибутива.

  - Чтобы установить необходимые пакеты в Ubuntu или Debian, выполните следующие команды:

    ```sh
    sudo apt-get update
    sudo apt-get install build-essential libssl1.0.0 libasound2
    ```

    Если пакет libssl1.0.0 недоступен, установите libssl1.0.x (где x больше 0) или libssl1.1.

  - Чтобы установить необходимые пакеты в RHEL/CentOS, выполните следующие команды:

    ```sh
    sudo yum update
    sudo yum install alsa-lib openssl python3
    ```

> [!NOTE]
> - Если вы используете RHEL/CentOS 7, выполните инструкции по [настройке пакета SDK службы "Речь" в RHEL/CentOS 7](~/articles/cognitive-services/speech-service/how-to-configure-rhel-centos-7.md).
> - Если вы используете RHEL/CentOS 8, выполните инструкции по [настройке OpenSSL для Linux](~/articles/cognitive-services/speech-service/how-to-configure-openssl-linux.md).

- В Windows для вашей платформы необходим [распространяемый компонент Microsoft Visual C++ для Visual Studio 2019](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads). Обратите внимание, что при первой установке может потребоваться перезагрузить Windows, чтобы продолжить работу с данным руководством.
- И, наконец, потребуется [Python 3.6–3.9](https://www.python.org/downloads/). Чтобы проверить установку, откройте командную строку, введите команду `python --version` и проверьте результат. Если установка выполнена правильно, вы получите ответ "Python 3.8.8".

## <a name="install-the-speech-sdk-from-pypi"></a>Установка пакета SDK службы "Речь" из PyPI

Если вы разрабатываете средства в собственной среде, выполните приведенную ниже команду, чтобы установить пакет SDK службы "Речь" из [PyPI](https://pypi.org/). Пользователям Visual Studio Code следует перейти к следующему подразделу, чтобы получить пошаговые инструкции по установке.

```sh
pip install azure-cognitiveservices-speech
```

Если вы используете macOS, вам может потребоваться выполнить следующую команду, чтобы команда `pip` сработала:

```sh
python3 -m pip install --upgrade pip
```

Успешно установив `azure-cognitiveservices-speech` с помощью `pip`, можно использовать пакет SDK для службы "Речь", импортировав пространство имен в проекты Python.

```py
import azure.cognitiveservices.speech as speechsdk
```

## <a name="install-the-speech-sdk-using-visual-studio-code"></a>Установка пакета SDK для службы "Речь" с помощью Visual Studio Code

1. Скачайте и установите последнюю поддерживаемую версию [Python](https://www.python.org/downloads/) для своей платформы (3.6–3.9.).
   - Пользователи Windows должны выбрать вариант добавления Python в переменную среды PATH при установке.
1. Скачайте и установите [Visual Studio Code](https://code.visualstudio.com/Download).
1. Откройте Visual Studio Code и установите расширение Python. В меню выберите **Файл** > **Параметры** > **Расширения**. Найдите **Python** и щелкните **Установить**.

   ![Установка расширения Python](~/articles/cognitive-services/speech-service/media/sdk/qs-python-vscode-python-extension.png)

1. Кроме того, в Visual Studio Code можно установить пакет SDK службы "Речь" для Python из интегрированной командной строки:
   1. Откройте терминал (в раскрывающихся меню выберите **Терминал** > **Новый терминал**).
   1. В открывшемся окне терминала введите команду `python -m pip install azure-cognitiveservices-speech`.

Если вы не знакомы с Visual Studio Code, ознакомьтесь с более детальной [документацией по Visual Studio Code](https://code.visualstudio.com/docs). Дополнительные сведения о Visual Studio Code и Python см. в [учебнике по Visual Studio Code для Python](https://code.visualstudio.com/docs/python/python-tutorial).

## <a name="support-and-updates"></a>Поддержка и обновления

Обновления пакета SDK службы "Речь" для Python распространяются с помощью PyPI, о чем указывается в [заметках о выпуске](~/articles/cognitive-services/speech-service/releasenotes.md).
Если новая версия доступна, вы можете обновить ее с помощью команды `pip install --upgrade azure-cognitiveservices-speech`.
Проверьте, какая версия установлена в данный момент, просмотрев переменную `azure.cognitiveservices.speech.__version__`.

Если у вас возникла проблема или отсутствует функция, ознакомьтесь со статьей [Возможности получения поддержки и справки](../../../../cognitive-services-support-options.md?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext%253fcontext%253d%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext).

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [windows](../quickstart-list.md)]