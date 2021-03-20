---
title: Использование пакета NuGet Azure Stream Analytics CI/CD
description: В этой статье описывается, как использовать пакет NuGet Azure Stream Analytics CI/CD для настройки процесса непрерывной интеграции и развертывания.
author: su-jie
ms.author: sujie
ms.service: stream-analytics
ms.topic: how-to
ms.date: 05/15/2019
ms.openlocfilehash: 0b4356c74b2e0c1494456d5d1082efd7b8953a15
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98693381"
---
# <a name="use-the-azure-stream-analytics-cicd-nuget-package-for-integration-and-development"></a>Использование пакета NuGet Azure Stream Analytics CI/CD для интеграции и разработки 
В этой статье описывается, как использовать пакет NuGet Azure Stream Analytics CI/CD для настройки процесса непрерывной интеграции и развертывания.

Чтобы получить поддержку MSBuild, используйте [инструменты Stream Analytics для Visual Studio](./stream-analytics-quick-create-vs.md) версии 2.3.0000.0 или выше.

Доступен пакет NuGet [Microsoft.Azure.Stream Analytics.CICD](https://www.nuget.org/packages/Microsoft.Azure.StreamAnalytics.CICD/). Он предоставляет средства MSBuild, Local Run и Deployment Tools, которые поддерживают процесс непрерывной интеграции и развертывания [Stream Analytics проектов Visual Studio](stream-analytics-vs-tools.md). 
> [!NOTE]
> Пакет NuGet может использоваться только с версией 2.3.0000.0 или более поздней версией средств Stream Analytics для Visual Studio. При наличии проектов, созданных в предыдущих версиях средств Visual Studio, откройте его с помощью версии 2.3.0000.0 или более поздней версии и сохраните. После этого будут включены новые возможности. 

Дополнительные сведения см. в статье [Использование инструментов Azure Stream Analytics для Visual Studio](./stream-analytics-quick-create-vs.md).

## <a name="msbuild"></a>MSBuild
Как и в стандартном интерфейсе Visual Studio MSBuild, для выполнения сборки проекта имеются две возможности. Можно щелкнуть правой кнопкой мыши проект и выбрать **Собрать**. Можно также использовать **MSBuild** в пакете NuGet из командной строки.
```
./build/msbuild /t:build [Your Project Full Path] /p:CompilerTaskAssemblyFile=Microsoft.WindowsAzure.StreamAnalytics.Common.CompileService.dll  /p:ASATargetsFilePath="[NuGet Package Local Path]\build\StreamAnalytics.targets"

```

При успешном выполнении сборки проекта Stream Analytics Visual Studio создаются два файла шаблона Azure Resource Manager в папке **bin/[Debug/Retail]/Deploy**: 

* файл шаблона Resource Manager;

   `[ProjectName].JobTemplate.json`

* файл параметров Resource Manager.
   
   `[ProjectName].JobTemplate.parameters.json`

Параметры по умолчанию в файле parameters.json на основе параметров в проекте Visual Studio. При необходимости развертывания в другой среде замените соответствующие параметры.

> [!NOTE]
> Для всех учетных данных значения по умолчанию имеют значение NULL. Их **необходимо** установить перед развертыванием в облаке.

```json
"Input_EntryStream_sharedAccessPolicyKey": {
      "value": null
    },
```
Узнайте, как [развернуть файл шаблона Resource Manager с помощью Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md). Узнайте, как [использовать объект в качестве параметра в шаблоне Resource Manager](/azure/architecture/guide/azure-resource-manager/advanced-templates/objects-as-parameters).

Чтобы в качестве приемника выходных данных использовать управляемое удостоверение для Azure Data Lake Storage 1-го поколения, предоставите доступ субъекту-службе с помощью PowerShell перед развертыванием в Azure. Дополнительные сведения см. в разделе о [развертывании ADLS 1-го поколения с управляемым удостоверением с помощью шаблона Resource Manager](stream-analytics-managed-identities-adls.md#resource-manager-template-deployment).


## <a name="command-line-tool"></a>Программа командной строки

### <a name="build-the-project"></a>Сборка проекта
В пакете NuGet есть служебная программа командной строки, которая называется **SA.exe**. Она поддерживает сборку проекта и локальное тестирование на произвольном компьютере, которое можно использовать в процессе непрерывной интеграции и непрерывной доставки. 

Файлы развертывания по умолчанию помещаются в текущем каталоге. С помощью параметра -OutputPath можно указать выходной путь.

```
./tools/SA.exe build -Project [Your Project Full Path] [-OutputPath <outputPath>] 
```

### <a name="test-the-script-locally"></a>Тестирование сценария локально

Если в проекте указаны локальные входные файлы Visual Studio, можно выполнить автоматическое тестирование сценария с помощью команды *localrun*. Выходной результат помещается в текущий каталог.
 
```
localrun -Project [ProjectFullPath]
```

### <a name="generate-a-job-definition-file-to-use-with-the-stream-analytics-powershell-api"></a>Создайте файл определения задания для использования с API PowerShell для Stream Analytics.

Команда *arm* принимает шаблон задания и файл параметров шаблона задания, созданный путем сборки в качестве входных данных. Затем объедините их в файл определения задания JSON, который может использоваться с API PowerShell для Stream Analytics.

```powershell
arm -JobTemplate <templateFilePath> -JobParameterFile <jobParameterFilePath> [-OutputFile <asaArmFilePath>]
```
Пример
```powershell
./tools/SA.exe arm -JobTemplate "ProjectA.JobTemplate.json" -JobParameterFile "ProjectA.JobTemplate.parameters.json" -OutputFile "JobDefinition.json" 
```



## <a name="next-steps"></a>Дальнейшие действия

* [Краткое руководство. Создание облачного задания Azure Stream Analytics в Visual Studio](stream-analytics-quick-create-vs.md)
* [Локальное тестирование запросов Stream Analytics с помощью Visual Studio](stream-analytics-vs-tools-local-run.md)
* [Изучение Azure Stream Analytics заданий с помощью Visual Studio](stream-analytics-vs-tools.md)