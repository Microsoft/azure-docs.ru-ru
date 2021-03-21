---
title: Автоматизация сборок, тестов и развертываний задания Azure Stream Analytics с помощью средств CI/CD
description: В этой статье описывается, как использовать средства Azure Stream Analytics CI/CD для автоматической сборки, тестирования и развертывания Azure Stream Analytics проекта.
services: stream-analytics
author: su-jie
ms.author: sujie
ms.service: stream-analytics
ms.topic: how-to
ms.date: 09/10/2020
ms.openlocfilehash: 2dbb491e77f132daf7b432f27705eba9e3e3cd3c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102036968"
---
# <a name="automate-builds-tests-and-deployments-of-an-azure-stream-analytics-job-using-cicd-tools"></a>Автоматизация сборок, тестов и развертываний задания Azure Stream Analytics с помощью средств CI/CD

Пакет NPM CI/CD можно Azure Stream Analytics использовать для автоматической сборки, тестирования и развертывания Azure Stream Analytics Visual Studio Code или проектов Visual Studio. Проекты можно создавать с помощью средств разработки, а также экспортировать из существующих Stream Analyticsных заданий. В этой статье описывается, как использовать пакет NPM с любой системой CI/CD. Сведения о развертывании с Azure Pipelines см. в статье [Использование Azure DevOps для создания конвейера CI/CD для задания Stream Analytics](set-up-cicd-pipeline.md).

## <a name="installation"></a>Установка

Вы можете [скачать пакет](https://www.npmjs.com/package/azure-streamanalytics-cicd) напрямую или установить его [глобально](https://docs.npmjs.com/downloading-and-installing-packages-globally) с помощью `npm install -g azure-streamanalytics-cicd` команды. Рекомендуется использовать команду, которая также может использоваться в задаче "Скрипт" PowerShell или Azure CLI конвейера сборки в **Azure pipelines**.

## <a name="build-the-project"></a>Сборка проекта

Пакет **ASA-streamanalytics-cicd** NPM предоставляет средства для создания шаблонов Azure Resource Manager Stream Analytics [Visual Studio Code проектов](./quick-create-visual-studio-code.md) или [проектов Visual Studio](stream-analytics-quick-create-vs.md). Пакет NPM можно также использовать в Windows, macOS и Linux без установки Visual Studio Code или Visual Studio.

После установки пакета используйте следующую команду, чтобы выполнить сборку проектов Stream Analytics.

```powershell
azure-streamanalytics-cicd build -project <projectFullPath> [-outputPath <outputPath>]
```

Команда *Build* проверяет синтаксис ключевого слова и выводит шаблон Azure Resource Manager.

| Параметр | Описание |
|---|---|
| `-project` | Абсолютный путь **asaproj.jsв** файле для проекта Visual Studio Code или **[имя проекта]. Асапрож** для проекта Visual Studio. |
| `-outputPath` | Путь к выходной папке для шаблонов Azure Resource Manager. Если он не указан, шаблоны будут помещены в текущий каталог. |

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd build -project "/Users/username/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd build -project "/Users/username/projects/samplejob/samplejob.asaproj"
```

--- 

При успешном построении проекта Stream Analytics в выходной папке создаются два следующих файла:

* Файл шаблона Azure Resource Manager

   `[ProjectName].JobTemplate.json`

* Файл параметров Azure Resource Manager

   `[ProjectName].JobTemplate.parameters.json`

Параметры по умолчанию в parameters.jsдля файла зависят от параметров в проекте Visual Studio Code или Visual Studio. При необходимости развертывания в другой среде замените соответствующие параметры.

Значения по умолчанию для всех учетных данных имеют **значение NULL**. Перед развертыванием в Azure необходимо задать значения.

```json
"Input_EntryStream_sharedAccessPolicyKey": {
      "value": null
    },
```

Чтобы использовать управляемое удостоверение для Azure Data Lake Store Gen1 в качестве приемника выходных данных, перед развертыванием в Azure необходимо предоставить доступ к субъекту-службе с помощью PowerShell. Дополнительные сведения см. в разделе о [развертывании ADLS 1-го поколения с управляемым удостоверением с помощью шаблона Resource Manager](stream-analytics-managed-identities-adls.md#resource-manager-template-deployment).

## <a name="local-run"></a>Локальный запуск

Если проект содержит указанные локальные входные файлы, можно запустить скрипт Stream Analytics локально с помощью `localrun` команды.

```powershell
azure-streamanalytics-cicd localrun -project <projectFullPath> [-outputPath <outputPath>] [-customCodeZipFilePath <zipFilePath>]
```

| Параметр | Описание |
|---|---|
| `-project` | Путь **asaproj.jsв** файле для проекта Visual Studio Code или **[имя проекта]. Асапрож** для проекта Visual Studio. |
| `-outputPath` | Путь к выходной папке. Если он не указан, выходные файлы результатов будут помещены в текущий каталог. |
| `-customCodeZipFilePath` | Путь к ZIP-файлу для пользовательского кода C#, например UDF или десериализатор, если они используются. Упакуйте библиотеки DLL в ZIP-файл и укажите этот путь. |

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd localrun -project "/Users/roger/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd localrun -project "/Users/roger/projects/samplejob/samplejob.asaproj"
```

---

> [!Note] 
> Функция UDF JavaScript работает только в Windows.

## <a name="automated-test"></a>Автоматический тест

Пакет NPM CI/CD можно использовать для настройки и запуска автоматических тестов для скрипта Stream Analytics.

### <a name="add-a-test-case"></a>Добавить тестовый случай

Тестовые случаи описаны в файле конфигурации теста. Чтобы приступить к работе, используйте `addtestcase` команду, чтобы добавить шаблон тестового случая в файл конфигурации теста. Если тестовый файл конфигурации не существует, он создается по умолчанию.

```powershell
azure-streamanalytics-cicd addtestcase -project <projectFullPath> [-testConfigPath <testConfigFileFullPath>]
```

| Параметр | Описание |
|---|---|
| `-project` | Путь **asaproj.jsв** файле для проекта Visual Studio Code или **[имя проекта]. Асапрож** для проекта Visual Studio. |
| `-testConfigPath` | Путь к файлу конфигурации теста. Если он не указан, будет выполнен поиск файла в **\тест** в текущем каталоге **asaproj.jsв** файле с именем файла по умолчанию **testConfig.json**. Новый файл будет создан, если он не существовал. |

> [!NOTE]
> `Script`Значение в созданной **testConfig.jsв** файле предназначено только для предоставления контекста; Он не используется в логике тестирования. 

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd addtestcase -project "/Users/roger/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd addtestcase -project "/Users/roger/projects/samplejob/samplejob.asaproj"
```
---

Если файл конфигурации теста пуст, в файл записывается следующее содержимое. В противном случае в массив **TestCases** добавляется тестовый случай. Необходимые входные конфигурации автоматически заполняются в соответствии с входными файлами конфигурации, если они существуют. В противном случае настраиваются значения по умолчанию. Перед выполнением теста необходимо указать **путь к файлу** для каждого входного и ожидаемого выходных данных. Конфигурацию можно изменить вручную.

Если вы хотите, чтобы проверка теста проигнорировала определенные выходные данные, задайте для **требуемого** поля ожидаемые выходные данные **значение false**.

```json
{
  "Script": "",
  "TestCases": [
    {
      "Name": "Case 1",
      "Inputs": [
        {
          "InputAlias": [Input alias string],
          "Type": "Data Stream",
          "Format": "JSON",
          "FilePath": [Required],
          "ScriptType": "InputMock"
        }
      ],
      "ExpectedOutputs": [
        {
          "OutputAlias": [Output alias string],
          "FilePath": [Required],
          "Required": true
        }
      ]
    }
  ]
}
```

> [!NOTE]
> Сейчас единственным допустимым значением для `ScriptType` элемента является `InputMock` , которое также является значением по умолчанию. Если задано любое другое значение, оно игнорируется и используется значение по умолчанию ( `InputMock` ). 

### <a name="run-a-unit-test"></a>Выполнение модульного теста

Чтобы запустить несколько тестовых случаев для проекта, можно использовать следующую команду. Сводка результатов теста создается в выходной папке. Процесс завершается с кодом **0** для всех пройденных тестов. **-1** для исключения; **-2** для тестов с ошибками.

```powershell
azure-streamanalytics-cicd test -project <projectFullPath> [-testConfigPath <testConfigFileFullPath>] [-outputPath <outputPath>] [-customCodeZipFilePath <zipFilePath>]
```

| Параметр | Описание |
|---|---|
| `-project` | Путь **asaproj.jsв** файле для проекта Visual Studio Code или **[имя проекта]. Асапрож** для проекта Visual Studio. |
| `-testConfigPath` | Путь к файлу конфигурации теста. Если он не указан, будет выполнен поиск файла в **\тест** в текущем каталоге **asaproj.jsв** файле с именем файла по умолчанию **testConfig.json**.
| `-outputPath` | Путь к выходной папке результатов теста. Если он не указан, выходные файлы результатов будут помещены в текущий каталог. |
| `-customCodeZipFilePath` | Путь к ZIP-файлу для пользовательского кода, например UDF или десериализатор, если они используются. |

По завершении всех тестов в выходной папке создается сводка результатов теста в формате JSON. Файл сводки называется **testResultSummary.jsв**.

```json
{
  "Total": (integer) total_number_of_test_cases,
  "Passed": (integer) number_of_passed_test_cases,
  "Failed": (integer) number_of_failed_test_cases,
  "Script": (string) absolute_path_to_asaql_file,
  "Results": [ (array) detailed_results_of_test_cases
    {
      "Name": (string) name_of_test_case,
      "Status": (integer) 0(passed)_or_1(failed),
      "Time": (string) time_span_of_running_test_case,
      "OutputMatched": [ (array) records_of_actual_outputs_equal_to_expected_outputs
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputNotEqual": [ (array) records_of_actual_outputs_not_equal_to_expected_outputs
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputMissing": [ (array) records_of_actual_outputs_missing
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": ""
        }
      ],
      "OutputUnexpected": [ (array) records_of_actual_outputs_unexpected
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": "",
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputUnrequired": [ (array) records_of_actual_outputs_unrequired_to_be_checked
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ]
    }
  ],
  "Time": (string) time_span_of_running_all_test_cases,
}
```

## <a name="deploy-to-azure"></a>Развернуть в Azure

Вы можете использовать шаблон Azure Resource Manager и файлы параметров, созданные в процессе сборки, для [развертывания задания в Azure](../azure-resource-manager/templates/template-tutorial-use-parameter-file.md?tabs=azure-powershell#deploy-template).

## <a name="next-steps"></a>Следующие шаги

* [Непрерывная интеграция и непрерывное развертывание для Azure Stream Analytics](cicd-overview.md)
* [Настройка конвейера CI/CD для задания Stream Analytics с помощью Azure Pipelines](set-up-cicd-pipeline.md)
