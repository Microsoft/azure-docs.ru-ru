---
title: включить файл
description: включить файл
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/12/2019
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 93bfb515c46413400ecd47105de378c7b677cecd
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102564970"
---
## <a name="prepare-your-repository"></a>Подготовка репозитория

Чтобы получать автоматические сборки с сервера сборки Службы приложений Azure, убедитесь, что корень репозитория содержит нужные файлы проекта.

| Параметры выполнения | Файлы в корневом каталоге |
|-|-|
| ASP.NET (только для Windows) | _\*.SLN_-файлы, _\*.CSPROJ_-файлы или _default.aspx_ |
| ASP.NET Core | _\*.SLN_-файлы или _\*.CSPROJ_-файлы |
| PHP | _index.php_ |
| Ruby (только для Linux) | _Gemfile_ |
| Node.js | _server.js_, _app.js_ или _package.json_ со сценарием запуска |
| Python | _\*PY_-файлы, _requirements.txt_ или _runtime.txt_ |
| HTML | _default.htm_, _default.html_, _default.asp_, _index.htm_, _index.html_ или _iisstart.htm_ |
| веб-задания; | _\<job_name>/run.\<extension>_ в папке _App\_Data/jobs/continuous_ для непрерывных веб-заданий или в папке _App\_Data/jobs/triggered_ для активируемых веб-заданий. См. сведения в [документации по веб-заданиям Kudu](https://github.com/projectkudu/kudu/wiki/WebJobs). |
| Функции | Ознакомьтесь с разделом [Непрерывное развертывание для Функций Azure](../articles/azure-functions/functions-continuous-deployment.md#requirements-for-continuous-deployment). |

Чтобы настроить развертывание, добавьте в корень репозитория *DEPLOYMENT*-файл. См. сведения о [настройке развертываний](https://github.com/projectkudu/kudu/wiki/Customizing-deployments) и [настраиваемом скрипте развертывания](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script).

> [!NOTE]
> Если для разработки используется Visual Studio, позвольте [Visual Studio автоматически создать репозиторий](/azure/devops/repos/git/creatingrepo?tabs=visual-studio). Проект немедленно будет готов к развертыванию с помощью Git.
>

