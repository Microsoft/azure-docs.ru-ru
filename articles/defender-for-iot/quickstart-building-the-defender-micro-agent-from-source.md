---
title: Краткое руководство. Создание микроагента Defender на основе исходного кода (предварительная версия)
description: В этом кратком руководстве вы узнаете о микроагенте, который включает инфраструктуру, которую можно использовать для настройки распространения.
ms.date: 1/18/2021
ms.topic: quickstart
ms.openlocfilehash: a3a8f82989d6abbaf2657e4b45638c77b3b2f704
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/05/2021
ms.locfileid: "106384603"
---
# <a name="quickstart-build-the-defender-micro-agent-from-source-code-preview"></a>Краткое руководство. Создание микроагента Defender на основе исходного кода (предварительная версия)

Микроагент содержит инфраструктуру, которую можно использовать для настройки распределения. Чтобы увидеть список доступных параметров конфигурации, ознакомьтесь с файлом `configs/LINUX_BASE.conf`.

Если вы намерены выполнить единичное распределение, измените базовый файл `.conf`. 

Если требуется несколько распределений, скопируйте базовую конфигурацию и измените значения. 

Чтобы изменить значения, выполните следующие действия.

1. Создайте новый файл `.dist`.

1. Добавьте `CONF_DEFINE_BASE(${g_plat_config_path} LINUX_BASE.conf)` в начало.
 
1. Определите новые значения для всех необходимых параметров, например: 

    `set(ASC_LOW_PRIORITY_INTERVAL 60*60*24)` 

1. При создании укажите ссылку на файл `.dist`. Например, 

    `cmake -DCMAKE_BUILD_TYPE=Debug -Dlog_level=DEBUG -Dlog_level_cmdline:BOOL=ON -DIOT_SECURITY_MODULE_DIST_TARGET=UBUNTU1804 ..` 

## <a name="prerequisites"></a>Предварительные условия

1. Обратитесь к менеджеру по работе с клиентами, чтобы запросить доступ к исходному коду службы "Defender для Интернета вещей".
 
1. Клонируйте или распакуйте исходный код в папку на диске.

1. Перейдите в этот каталог.

1. Извлеките подмодули, используя следующий код:

    ```bash
    git submodule update --init
    ```
    
1. Затем извлеките подмодули для пакета SDK для Интернета вещей Azure с помощью следующего кода: 

    ```bash
    git -C deps/azure-iot-sdk-c/ submodule update –init
    ```
 

1. Добавьте разрешение на выполнение и запустите сценарий установки среды разработки:

    ```bash
    chmod +x scripts/install_development_environment.sh && ./scripts/install_development_environment.sh 
    ```

1. Создайте каталог для выходных данных сборки: 

    ```bash
    mkdir cmake 
    ```

1. Загрузите и установите [VSCode](https://code.visualstudio.com/download ) (необязательно). 

1. Установите [расширение C/C++](https://code.visualstudio.com/docs/languages/cpp ) для VSCode (необязательно). — Нет

## <a name="baseline-configuration-signing"></a>Подписание базовой конфигурации 

Агент по умолчанию проверяет подлинность файлов конфигурации, размещенных на диске, чтобы избежать незаконных изменений.

Этот процесс можно остановить, определив флаг препроцессора `ASC_BASELINE_CONF_SIGN_CHECK_DISABLE`.

Мы не рекомендуем отключать проверку подписи для рабочих сред. 

Если вам нужна другая конфигурация для сценариев использования в рабочей среде, обратитесь за помощью в команду службы "Defender для Интернета вещей". 

## <a name="building-the-defender-iot-micro-agent"></a>Создание микроагента Defender для Интернета вещей 

1. Откройте каталог с помощью VSCode. 

1. Перейдите к каталогу `cmake`. 

1. Выполните следующую команду. 

    ```bash
    cmake -DCMAKE_BUILD_TYPE=Debug -Dlog_level=DEBUG -Dlog_level_cmdline:BOOL=ON -DIOT_SECURITY_MODULE_DIST_TARGET<the appropriate distro configuration file name> .. 
    
    cmake --build . -- -j${env:NPROC}
    ```

    Например: 

    ```bash
    cmake -DCMAKE_BUILD_TYPE=Debug -Dlog_level=DEBUG -Dlog_level_cmdline:BOOL=ON -DIOT_SECURITY_MODULE_DIST_TARGETUBUNTU1804 ..
    
    cmake --build . -- -j${env:NPROC}
    ```

## <a name="next-steps"></a>Дальнейшие действия

[Настройка решения Azure Defender для Интернета вещей](quickstart-configure-your-solution.md).
