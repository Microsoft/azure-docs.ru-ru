---
title: Устранение неполадок в приложениях функций Python в функциях Azure
description: Узнайте, как устранять неполадки в функциях Python.
author: Hazhzeng
ms.topic: article
ms.date: 07/29/2020
ms.author: hazeng
ms.custom: devx-track-python
ms.openlocfilehash: 9b9f5d389eda5d74e7e78cfcfa9a46fba7276cbd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87846043"
---
# <a name="troubleshoot-python-errors-in-azure-functions"></a>Устранение неполадок при появлении ошибок Python в Функциях Azure

Ниже приведен список руководств по устранению распространенных проблем в функциях Python.

* [ModuleNotFoundError и ImportError](#troubleshoot-modulenotfounderror)
* [Не удается импортировать "цигрпк"](#troubleshoot-cannot-import-cygrpc)

## <a name="troubleshoot-modulenotfounderror"></a>Устранение неполадок Модуленотфаундеррор

Этот раздел поможет вам устранить ошибки, связанные с модулем, в приложении-функции Python. Эти ошибки обычно приводят к появлению следующих сообщений об ошибках в Функциях Azure.

> `Exception: ModuleNotFoundError: No module named 'module_name'.`

Эта проблема возникает, когда приложению-функции Python не удается загрузить модуль Python. Основными причинами этой ошибки является одна из следующих проблем:

- [не удается найти пакет](#the-package-cant-be-found);
- [пакет не разрешается с помощью нужной библиотеки wheel в Linux](#the-package-isnt-resolved-with-proper-linux-wheel);
- [пакет несовместим с версией интерпретатора Python](#the-package-is-incompatible-with-the-python-interpreter-version);
- [пакет конфликтует с другими пакетами](#the-package-conflicts-with-other-packages);
- [пакет поддерживает только платформы Windows или macOS](#the-package-only-supports-windows-or-macos-platforms).

### <a name="view-project-files"></a>Просмотр файлов проекта

Чтобы узнать фактическую причину проблемы, необходимо получить файлы проекта Python, которые выполняются в вашем приложении-функции. Если у вас нет файлов проекта на локальном компьютере, их можно получить одним из следующих способов.

- Если приложение-функция имеет параметр приложения `WEBSITE_RUN_FROM_PACKAGE`, а его значение представляет собой URL-адрес, скачайте файл, скопировав и вставив этот URL-адрес в браузер.
- Если приложение-функция имеет параметр `WEBSITE_RUN_FROM_PACKAGE` и для него задано значение `1`, перейдите на страницу `https://<app-name>.scm.azurewebsites.net/api/vfs/data/SitePackages` и скачайте файл из последнего URL-адреса `href`.
- Если у приложения-функции нет указанного выше параметра приложения, перейдите на страницу `https://<app-name>.scm.azurewebsites.net/api/settings` и найдите URL-адрес в разделе `SCM_RUN_FROM_PACKAGE`. Скачайте файл, скопировав и вставив этот URL-адрес в адресную строку браузера.
- Если ни одно из этих действий не помогло, перейдите на страницу `https://<app-name>.scm.azurewebsites.net/DebugConsole` и откройте содержимое в разделе `/home/site/wwwroot`.

Дальше в этой статье приведены советы по устранению возможных причин этой ошибки путем проверки содержимого приложения-функции, определения основной причины и решения конкретной проблемы.

### <a name="diagnose-modulenotfounderror"></a>Диагностика ошибки ModuleNotFoundError

В этом разделе описаны возможные основные причины ошибок, связанных с модулем. После того как вы выясните, какая это может быть причина, можно приступать к устранению проблемы.

#### <a name="the-package-cant-be-found"></a>Не удается найти пакет

Перейдите на страницу `.python_packages/lib/python3.6/site-packages/<package-name>` или `.python_packages/lib/site-packages/<package-name>`. Если путь к файлу не существует, скорее всего, этот путь является основной причиной проблемы.

Ее может вызвать использование сторонних или устаревших средств при развертывании.

Способы решения этой проблемы см. в разделе [Включение возможности удаленной сборки](#enable-remote-build) или [Создание собственных зависимостей](#build-native-dependencies).

#### <a name="the-package-isnt-resolved-with-proper-linux-wheel"></a>Пакет не разрешается с помощью нужной библиотеки wheel в Linux

Перейдите на страницу `.python_packages/lib/python3.6/site-packages/<package-name>-<version>-dist-info` или `.python_packages/lib/site-packages/<package-name>-<version>-dist-info`. Откройте файл **wheel** в привычном текстовом редакторе и проверьте раздел **Tag:** . Если значение тега не содержит **linux**, в этом и может заключаться проблема.

Функции Python работают в Azure только в Linux: среда выполнения функций версии 2.x выполняется в Debian Stretch и среде выполнения версии 3.x в Debian Buster. Артефакт должен содержать правильные двоичные файлы Linux. Использование флага `--build local` в Core Tools, сторонних или устаревших средств может привести к использованию более старых двоичных файлов.

Способы решения этой проблемы см. в разделе [Включение возможности удаленной сборки](#enable-remote-build) или [Создание собственных зависимостей](#build-native-dependencies).

#### <a name="the-package-is-incompatible-with-the-python-interpreter-version"></a>Пакет несовместим с версией интерпретатора Python

Перейдите на страницу `.python_packages/lib/python3.6/site-packages/<package-name>-<version>-dist-info` или `.python_packages/lib/site-packages/<package-name>-<version>-dist-info`. Откройте файл МЕТАДАННЫХ в текстовом редакторе и проверьте раздел **Classifiers:** . Если раздел не содержит `Python :: 3`, `Python :: 3.6`, `Python :: 3.7` или `Python :: 3.8`, это означает, что версия пакета либо устарела, либо, скорее всего, пакет уже не обслуживается.

Версию приложения-функции Python можно проверить на [портале Azure](https://portal.azure.com). Перейдите к приложению-функции, выберите **Обозреватель ресурсов** и нажмите **Перейти**.

:::image type="content" source="media/recover-module-not-found/resource-explorer.png" alt-text="Откройте обозреватель ресурсов для приложения-функции на портале":::

После загрузки обозревателя выполните поиск параметра **LinuxFxVersion**, в котором отображается версия Python.

Для решения проблемы см. раздел [Обновление пакета до последней версии](#update-your-package-to-the-latest-version) или [Замена пакет на эквивалент](#replace-the-package-with-equivalents).

#### <a name="the-package-conflicts-with-other-packages"></a>Пакет конфликтует с другими пакетами

Если вы убедились, что пакет правильно разрешается с помощью нужной библиотеки wheel в Linux, причина может заключаться в конфликте с другими пакетами. Документация PyPi может прояснить, есть ли у конкретных пакетов несовместимые модули. Например, в [`azure 4.0.0`](https://pypi.org/project/azure/4.0.0/) существует следующая инструкция.

<pre>This package isn't compatible with azure-storage.
If you installed azure-storage, or if you installed azure 1.x/2.x and didn’t uninstall azure-storage,
you must uninstall azure-storage first.</pre>

Документацию по вашей версии пакета можно найти на странице `https://pypi.org/project/<package-name>/<package-version>`.

Для решения проблемы см. раздел [Обновление пакета до последней версии](#update-your-package-to-the-latest-version) или [Замена пакет на эквивалент](#replace-the-package-with-equivalents).

#### <a name="the-package-only-supports-windows-or-macos-platforms"></a>Пакет поддерживает только платформы Windows или macOS

Откройте `requirements.txt` в текстовом редакторе и проверьте пакет в `https://pypi.org/project/<package-name>`. Некоторые пакеты работают только на платформах Windows или macOS. Например, pywin32 работает только в Windows.

Ошибка `Module Not Found` может не возникать, если вы используете Windows или macOS для локальной разработки. Однако пакет не удается импортировать в Функции Azure, в которых используется Linux в среде выполнения. Проблема может появляться из-за использования `pip freeze` для экспорта виртуальной среды в файл requirements.txt с компьютера под управлением Windows или macOS во время инициализации проекта.

Для ее решения см. раздел [Замена пакета на эквивалент](#replace-the-package-with-equivalents) или [Обработка файла requirements.txt вручную](#handcraft-requirementstxt).

### <a name="mitigate-modulenotfounderror"></a>Устранение ошибки ModuleNotFoundError

Ниже приведены потенциальные способы устранения проблем, связанных с модулем. Используйте [приведенную выше схему диагностики](#diagnose-modulenotfounderror), чтобы определить, какие из них следует применить.

#### <a name="enable-remote-build"></a>Включение удаленной сборки

Убедитесь, что удаленная сборка включена. То, как это сделать, зависит от метода развертывания.

## <a name="visual-studio-code"></a>[Visual Studio Code](#tab/vscode)
Убедитесь, что установлена последняя версия [расширения Функций Azure для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions). Проверьте, существует ли `.vscode/settings.json` и содержит ли параметр `"azureFunctions.scmDoBuildDuringDeployment": true`. В противном случае создайте этот файл с включенным параметром `azureFunctions.scmDoBuildDuringDeployment` и повторно разверните проект.

## <a name="azure-functions-core-tools"></a>[Основные инструменты службы "Функции Azure"](#tab/coretools)

Убедитесь, что установлена последняя версия [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools/releases). Перейдите в папку проекта локальной функции и используйте `func azure functionapp publish <app-name>` для развертывания.

## <a name="manual-publishing"></a>[Ручная публикация](#tab/manual)

Если вы вручную публикуете пакет в конечной точке `https://<app-name>.scm.azurewebsites.net/api/zipdeploy`, убедитесь, что для параметров **SCM_DO_BUILD_DURING_DEPLOYMENT** и **ENABLE_ORYX_BUILD** установлено значение **true**. Дополнительные сведения см. в разделе [Как работать с параметрами приложения](functions-how-to-use-azure-function-app-settings.md#settings).

---

#### <a name="build-native-dependencies"></a>Создание собственных зависимостей

Убедитесь, что установлены последние версии **Docker** и [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools/releases). Перейдите в папку проекта локальной функции и используйте `func azure functionapp publish <app-name> --build-native-deps` для развертывания.

#### <a name="update-your-package-to-the-latest-version"></a>Обновление пакета до последней версии

Просмотрите последнюю версию пакета в `https://pypi.org/project/<package-name>` и проверьте раздел **Classifiers:** . Пакет должен быть `OS Independent` или совместим с `POSIX` или `POSIX :: Linux` в разделе **Операционная система**. Кроме того, раздел "Язык программирования" должен содержать `Python :: 3`, `Python :: 3.6`, `Python :: 3.7` или `Python :: 3.8`.

Если значения такие, можно обновить пакет до последней версии, изменив строку `<package-name>~=<latest-version>` в файле requirements.txt.

#### <a name="handcraft-requirementstxt"></a>Обработка файла requirements.txt вручную

Некоторые разработчики используют `pip freeze > requirements.txt`, чтобы создать список пакетов Python для своих сред разработки. Хотя эта удобная функция должна работать в большинстве случаев, в сценариях межплатформенного развертывания могут возникать проблемы, например когда разработка функций выполняется локально в Windows или macOS, а публикуются они в приложении-функции, работающем в Linux. В этом сценарии `pip freeze` может привести к непредвиденным зависимостям от операционной системы или зависимостям от локальной среды разработки. Эти зависимости могут нарушить работу приложения-функции Python в Linux.

Рекомендуется проверить операторы импорта в каждом файле .py в исходном коде проекта и записать только эти модули в файл requirements.txt. Это гарантирует, что разрешение пакетов будет правильно обработано в разных операционных системах.

#### <a name="replace-the-package-with-equivalents"></a>Замена пакета на эквивалент

Во-первых, мы посмотрим последнюю версию пакета в `https://pypi.org/project/<package-name>`. Обычно у этого пакета есть собственная страница в GitHub. Перейдите к разделу **Проблемы** на сайте GitHub и выполните поиск готового решения вашей проблемы. Если оно есть, обновите пакет до последней версии.

Иногда пакет может быть интегрирован в [стандартную библиотеку Python](https://docs.python.org/3/library/) (например, pathlib). Если он там есть, то пакет в файле requirements.txt необходимо удалить, поскольку в Функциях Azure (Python 3.6, Python 3.7 и Python 3.8) мы предлагаем определенный дистрибутив Python.

Однако, если у вас возникла проблема, которая не имеет готового решения, а времени немного, рекомендуется поискать аналогичный пакет для проекта. Как правило, сообщество Python предлагает широкий спектр похожих библиотек, которые можно использовать.

---

## <a name="troubleshoot-cannot-import-cygrpc"></a>Устранение неполадок не удается импортировать "цигрпк"

Этот раздел поможет устранить ошибки, связанные с "цигрпк", в приложении функции Python. Эти ошибки обычно приводят к появлению следующих сообщений об ошибках в Функциях Azure.

> `Cannot import name 'cygrpc' from 'grpc._cython'`

Эта проблема возникает, когда не удается запустить приложение-функцию Python с правильным интерпретатором Python. Основными причинами этой ошибки является одна из следующих проблем:

- [Интерпретатор Python не соответствует архитектуре ОС](#the-python-interpreter-mismatches-os-architecture)
- [Интерпретатор Python не поддерживается рабочей ролью Python в функциях Azure](#the-python-interpreter-is-not-supported-by-azure-functions-python-worker)

### <a name="diagnose-cygrpc-reference-error"></a>Диагностика ошибки ссылки "цигрпк"

#### <a name="the-python-interpreter-mismatches-os-architecture"></a>Интерпретатор Python не соответствует архитектуре ОС

Скорее всего, это вызвано тем, что в 64-разрядной операционной системе установлен интерпретатор Python 32-bit.

Если вы используете операционную систему x64, убедитесь, что интерпретатор Python 3,6, 3,7 или 3,8 также находится на 64-разрядной версии.

Чтобы проверить разрядность интерпретатора Python, выполните следующие команды:

В Windows в PowerShell: `py -c 'import platform; print(platform.architecture()[0])'`

В оболочке, похожей на UNIX: `python3 -c 'import platform; print(platform.architecture()[0])'`

В случае несоответствия между разрядностью интерпретатора Python и архитектурой операционной системы Скачайте правильный интерпретатор Python из [Python Software Foundation](https://python.org/downloads/release).

#### <a name="the-python-interpreter-is-not-supported-by-azure-functions-python-worker"></a>Интерпретатор Python не поддерживается рабочей ролью Python в функциях Azure

Рабочая роль Python для функций Azure поддерживает только Python 3,6, 3,7 и 3,8.
Убедитесь, что интерпретатор Python соответствует ожидаемой версии, `py --version` в Windows или `python3 --version` в системах, подобных Unix. Убедитесь, что возвращаемый результат — Python 3.6. x, Python 3.7. x или Python 3.8. x.

Если версия интерпретатора Python не соответствует нашему предположении, скачайте интерпретатор Python 3,6, 3,7 или 3,8 из [Python Software Foundation](https://python.org/downloads/release).

## <a name="next-steps"></a>Дальнейшие действия

Если вы не можете решить проблему, сообщите о ней команде функций:

> [!div class="nextstepaction"]
> [Сообщить о нерешенной проблеме](https://github.com/Azure/azure-functions-python-worker/issues)
