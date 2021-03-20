---
title: Синхронизация репозитория GitHub с Конфигурацией приложений
description: Используйте GitHub Actions для автоматического обновления экземпляра Конфигурации приложений при обновлении репозитория GitHub.
author: AlexandraKemperMS
ms.author: alkemper
ms.date: 05/28/2020
ms.topic: conceptual
ms.service: azure-app-configuration
ms.openlocfilehash: 1c290032f7a33079b560d3c4cc1fcb9526e70331
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98762146"
---
# <a name="sync-your-github-repository-to-app-configuration"></a>Синхронизация репозитория GitHub с Конфигурацией приложений

Команды, которые хотят продолжить применять существующие методы системы управления версиями, могут использовать GitHub Actions, чтобы автоматически синхронизировать свой репозиторий GitHub с хранилищем Конфигурации приложений. Это позволяет вносить изменения в файлы конфигурации обычным образом и в то же время использовать приведенные ниже преимущества Конфигурации приложений. <br>
&nbsp;&nbsp;&nbsp;&nbsp;• Централизованная настройка вне кода. <br>
&nbsp;&nbsp;&nbsp;&nbsp;• Обновление конфигурации без повторного развертывания всего приложения. <br>
&nbsp;&nbsp;&nbsp;&nbsp;• Интеграция с такими службами, как Служба приложений Azure и Функции Azure. 

[Рабочий процесс](https://docs.github.com/en/actions/learn-github-actions/introduction-to-github-actions#the-components-of-github-actions) GitHub Actions определяет автоматизированный процесс в репозитории GitHub. Действие *синхронизации Конфигурации приложений Azure* активирует обновление экземпляра Конфигурации приложений при внесении изменений в исходный репозиторий. Оно использует файл в формате YAML (YML-файл), находящийся в папке `/.github/workflows/` репозитория, чтобы определить шаги и параметры. Вы можете активировать обновления конфигурации при отправке, просмотре или ветвлении файлов конфигурации приложения так же, как для кода приложения.

[Документация](https://docs.github.com/en/actions/learn-github-actions/introduction-to-github-actions) на сайте GitHub содержит подробные сведения о рабочих процессах и действиях GitHub. 

## <a name="enable-github-actions-in-your-repository"></a>Включение GitHub Actions в репозитории
Чтобы начать работу с этим действием GitHub, перейдите в репозиторий и перейдите на вкладку **действия** . Выберите **новый рабочий процесс**, а затем **Настройте рабочий процесс самостоятельно**. Наконец, выполните поиск по запросу "Azure App Configuration Sync" в marketplace.
> [!div class="mx-imgBorder"]
> ![Откройте вкладку "Action" (Действие).](media/find-github-action.png)

> [!div class="mx-imgBorder"]
> ![Выберите действие синхронизации Конфигурации приложений.](media/app-configuration-sync-action.png)

## <a name="sync-configuration-files-after-a-push"></a>Синхронизация файлов конфигурации после отправки
Это действие синхронизирует файлы Конфигурации приложений Azure при отправке изменения в `appsettings.json`. Когда разработчик отправляет изменение в `appsettings.json`, действие синхронизации Конфигурации приложений обновляет экземпляр Конфигурации приложений с помощью новых значений.

В первом разделе этого рабочего процесса указывается, что действие активируется *при* *отправке* , содержащемся `appsettings.json` в *главной* ветви. Во втором разделе перечислены задания, выполняемые после активации действия. Действие получает для изменения соответствующие файлы и обновляет экземпляр Конфигурации приложений, используя строку подключения, сохраненную в качестве секрета в репозитории.  Дополнительные сведения об использовании секретов в GitHub см. в [статье GitHub](https://docs.github.com/en/actions/reference/encrypted-secrets) о создании и использовании зашифрованных секретов.

```json
on: 
  push: 
    branches: 
      - 'main' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your                        
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }} 
          separator: ':' 
```

## <a name="use-strict-sync"></a>Использование строгой синхронизации
По умолчанию действие GitHub не включает строгий режим. Это означает, что при синхронизации в экземпляр Конфигурации приложения только будут добавляться пары "ключ-значение" из файла конфигурации (пары "ключ-значение" не будут удаляться). Включение строгого режима означает, что пары "ключ-значение", отсутствующие в файле конфигурации, будут удалены из экземпляра Конфигурации приложений, чтобы он соответствовал файлу конфигурации. Если вы выполняете синхронизацию из нескольких источников или используете Azure Key Vault с Конфигурацией приложений, вам потребуется применять разные префиксы или метки для строгой синхронизации, чтобы не удалить параметры конфигурации из других файлов (см. примеры ниже). 

```json
on: 
  push: 
    branches: 
      - 'main' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          label: 'Label' 
          prefix: 'Prefix:' 
          strict: true 
```
## <a name="sync-multiple-files-in-one-action"></a>Синхронизация нескольких файлов одним действием 

Если конфигурация хранится в нескольких файлах, можно использовать приведенный ниже шаблон для активации синхронизации при изменении любого из этих файлов. В этом шаблоне используется библиотека стандартная маска https://www.npmjs.com/package/glob . Обратите внимание, что если имя файла конфигурации содержит запятую, можно использовать обратную косую черту для экранирования запятой. 

```json
on:
  push:
    branches:
      - 'main'
    paths:
      - 'appsettings.json'
      - 'appsettings2.json'

jobs:
  syncconfig:
    runs-on: ubuntu-latest
    steps:
      # checkout done so that files in the repo can be read by the sync
      - uses: actions/checkout@v1
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: '{appsettings.json,appsettings2.json}'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
```

## <a name="sync-by-prefix-or-label"></a>Синхронизация по префиксу или метке
Если указать префиксы или метки в действии синхронизации, то будет синхронизироваться только набор с этими префиксами или метками. Это важно для строгой синхронизации нескольких файлов. В зависимости от способа настройки конфигурации, можно связать с каждым файлом префикс или метку, а затем синхронизировать их отдельно, чтобы ничего не перезаписать. Обычно префиксы используются для разных приложений или служб, а метки — для различных сред. 

Синхронизация по префиксу. 

```json
on:
  push:
    branches:
      - 'main'
    paths:
      - 'appsettings.json'

jobs:
  syncconfig:
    runs-on: ubuntu-latest
    steps:
      # checkout done so that files in the repo can be read by the sync
      - uses: actions/checkout@v1
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: 'appsettings.json'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
          prefix: 'Prefix::'
```

Синхронизация по метке. 

```json
on:
  push:
    branches:
      - 'main'
    paths:
      - 'appsettings.json'

jobs:
  syncconfig:
    runs-on: ubuntu-latest
    steps:
      # checkout done so that files in the repo can be read by the sync
      - uses: actions/checkout@v1
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: 'appsettings.json'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
          label: 'Label'

```

## <a name="use-a-dynamic-label-on-sync"></a>Использование динамической метки при синхронизации
Приведенное ниже действие вставляет динамическую метку при каждой синхронизации, гарантируя, что каждая операция синхронизации может быть однозначно идентифицирована. Это позволяет сопоставить изменения кода с изменениями конфигурации.

В первом разделе этого рабочего процесса указывается, что действие активируется *при* *отправке* , содержащемся `appsettings.json` в *главной* ветви. Во втором разделе выполняется задание, которое создает уникальную метку для обновления конфигурации на основе хэша фиксации. Затем это задание обновляет экземпляр Конфигурации приложений с помощью новых значений и уникальной метки этого обновления.

```json
on: 
  push: 
    branches: 
      - 'main' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # Creates a label based on the branch name and the first 8 characters          
      # of the commit hash 
      - id: determine_label 
        run: echo ::set-output name=LABEL::"${GITHUB_REF#refs/*/}/${GITHUB_SHA:0:8}" 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          label: ${{ steps.determine_label.outputs.LABEL }} 
```

## <a name="use-azure-key-vault-with-github-action"></a>Использование Azure Key Vault с действием GitHub
Разработчики, использующие Azure Key Vault с Конфигурацией приложений, должны использовать два отдельных файла (обычно это appSettings.json и secretreferences.json). Файл secretreferences.json будет содержать URL-адрес секрета хранилища ключей.

{ "mySecret": "{\"uri\":\"https://myKeyVault.vault.azure.net/secrets/mySecret"}" }

После этого можно настроить действие GitHub для строгой синхронизации на основе appSettings.json, за которой последует нестрогая синхронизация на основе secretreferences.json. Следующий пример активирует синхронизацию при обновлении любого из файлов.

```json
on:
  push:
    branches:
      - 'main'
    paths:
      - 'appsettings.json'
      - 'secretreferences.json'

jobs:
  syncconfig:
    runs-on: ubuntu-latest
    steps:
      # checkout done so that files in the repo can be read by the sync
      - uses: actions/checkout@v1
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: 'appsettings.json'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
          strict: true
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: 'secretreferences.json'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
          contentType: 'application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8'

```

## <a name="use-max-depth-to-limit-github-action"></a>Использование максимальной глубины для ограничения действия GitHub
Поведение по умолчанию для вложенных атрибутов JSON состоит в том, чтобы преобразовать в плоскую структуру весь объект.  Приведенный ниже код JSON определяет пару "ключ-значение".

| Клавиши | Значение |
| --- | --- |
| Object:Inner:InnerKey | InnerValue |

```json
{ "Object": 
    { "Inner":
        {
        "InnerKey": "InnerValue"
        }
    }
}
```

Если вложенным объектом должно быть значение, переданное в экземпляр Конфигурации, можно использовать значение *depth*, чтобы остановить преобразование в плоскую структуру на соответствующей глубине. 

```json
on: 
  push: 
    branches: 
      - 'main' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          depth: 2 
```

При значении depth, равном 2, приведенный выше пример возвращает следующую пару "ключ-значение":

| Клавиши | Значение |
| --- | --- |
| Object:Inner | {"InnerKey":"InnerValue"} |

## <a name="understand-action-inputs"></a>Анализ выходных данных действия
Входные параметры указывают данные, используемые действием во время выполнения.  В следующей таблице приведены входные параметры, принимаемые при синхронизации Конфигурации приложений, и ожидаемые значения каждого из них.  Дополнительные сведения о входных данных действий GitHub см. в [документации](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#inputs) на сайте GitHub.

> [!Note]
> Во входных идентификаторах учитывается регистр.


| Ввод имени | Обязательно? | Значение |
|----|----|----|
| configurationFile | Да | Относительный путь к файлу конфигурации в репозитории.  Стандартные маски поддерживаются и могут включать в себя несколько файлов. |
| format | Да | Формат файла конфигурации.  Допустимые форматы: JSON, YAML, properties. |
| connectionString | Да | Строка подключения для экземпляра Конфигурации приложений. Строка подключения должна храниться в качестве секрета в репозитории GitHub, а в рабочем процессе следует использовать только имя секрета. |
| separator | Да | Разделитель, используемый при сведении файла конфигурации к парам "ключ-значение".  Допустимые значения: . , ; : - _ __ / |
| prefix | Нет | Префикс, добавляемый в начало ключей. |
| label | Нет | Метка, используемая при настройке пар "ключ-значение". Если значение не указано, используется метка со значением NULL. |
| strict | Нет | Логическое значение, определяющее, включен ли строгий режим. Значение по умолчанию — false. |
| depth | Нет | Максимальная глубина при преобразовании в плоскую структуру файла конфигурации.  Значение глубины должно быть положительным числом.  Значение по умолчанию не задает максимальную глубину. |
| tags | Нет | Указывает набор тегов для пар "ключ-значение".  Ожидаемый формат представляет собой строковую форму объекта JSON следующего вида: { [propertyName: string]: string; }. Каждая пара имени и значения свойства становится тегом. |

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали о действии GitHub для синхронизации Конфигурации приложений и о том, как его использовать для автоматизации обновления экземпляра Конфигурации приложений. Чтобы узнать, как Конфигурация приложений Azure реагирует на изменения в парах "ключ-значение", перейдите к следующей [статье](./concept-app-configuration-event.md).
