---
title: Интеграция с Git для Машинное обучение Azure
titleSuffix: Azure Machine Learning
description: Узнайте, как Машинное обучение Azure интегрируется с локальным репозиторием Git для хранения сведений о репозитории, ветви и текущей фиксации в процессе обучения.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jordane
author: jpe316
ms.date: 11/16/2020
ms.openlocfilehash: cc844cbd2518bb131f6902d1da3e7653951224b5
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102547855"
---
# <a name="git-integration-for-azure-machine-learning"></a>Интеграция с Git для Машинное обучение Azure

[Git](https://git-scm.com/) — это популярная система управления версиями, которая позволяет совместно использовать проекты и совместно работать над ними. 

Машинное обучение Azure полностью поддерживает репозитории Git для отслеживания работы. Вы можете клонировать репозитории непосредственно в общую файловую систему рабочей области, использовать Git на локальной рабочей станции или использовать Git из конвейера CI/CD.

При отправке задания в Машинное обучение Azure, если исходные файлы хранятся в локальном репозитории Git, сведения о репозитории будут отслеживаниться в рамках процесса обучения.

Поскольку Машинное обучение Azure отслеживает сведения из локального репозитория Git, он не привязан к какому-либо конкретному центральному репозиторию. Репозиторий можно клонировать из GitHub, GitLab, BitBucket, Azure DevOps или любой другой службы, совместимой с Git.

## <a name="clone-git-repositories-into-your-workspace-file-system"></a>Клонирование репозиториев Git в файловую систему рабочей области
Машинное обучение Azure предоставляет общую файловую систему для всех пользователей в рабочей области.
Чтобы клонировать репозиторий Git в этот файловый ресурс, рекомендуется создать вычислительный экземпляр & [Открыть терминал](how-to-access-terminal.md).
После открытия терминала Вы получите доступ к полному клиенту Git и можете клонировать и работать с Git с помощью интерфейса командной строки Git.

Рекомендуется клонировать репозиторий в каталог пользователей, чтобы другие пользователи не могли выполнять конфликты непосредственно в рабочей ветви.

Можно клонировать любой репозиторий Git, к которому можно выполнить аутентификацию (GitHub, Azure Repos, BitBucket и т. д.).

Дополнительные сведения о клонировании см. в разделе Руководство по [использованию Git CLI](https://guides.github.com/introduction/git-handbook/).

## <a name="authenticate-your-git-account-with-ssh"></a>Проверка подлинности учетной записи Git с помощью SSH
### <a name="generate-a-new-ssh-key"></a>Создать новый ключ SSH
1) [Откройте окно терминала](./how-to-access-terminal.md) на вкладке машинное обучение Azure Notebook (записная книжка).

2) Вставьте приведенный ниже текст, подставив его в адресе электронной почты.

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

При этом создается новый ключ SSH с использованием предоставленного адреса электронной почты в качестве метки.

```
> Generating public/private rsa key pair.
```

3) При появлении запроса "введите файл для сохранения ключа" нажмите клавишу ВВОД. Он принимает расположение файлов по умолчанию.

4) Убедитесь, что расположение по умолчанию — "/Хоме/азуреусер/.СШ", и нажмите клавишу ВВОД. В противном случае укажите расположение "/Хоме/азуреусер/.СШ".

> [!TIP]
> Убедитесь, что ключ SSH сохранен в "/Хоме/азуреусер/.СШ". Этот файл сохраняется в вычислительном экземпляре, доступный только владельцу вычислительного экземпляра

```
> Enter a file in which to save the key (/home/azureuser/.ssh/id_rsa): [Press enter]
```

5) В командной строке введите безопасную парольную фразу. Мы рекомендуем добавить парольную фразу в ключ SSH для повышения безопасности.

```
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```

### <a name="add-the-public-key-to-git-account"></a>Добавление открытого ключа в учетную запись Git
1) В окне терминала скопируйте содержимое файла открытого ключа. Если вы переименовали ключ, замените id_rsa. pub именем файла открытого ключа.

```bash
cat ~/.ssh/id_rsa.pub
```
> [!TIP]
> **Копирование и вставка в терминале**
> * Windows: `Ctrl-Insert` копирование и использование `Ctrl-Shift-v` или `Shift-Insert` для вставки.
> * Mac OS: `Cmd-c` для копирования и `Cmd-v` для вставки.
> * Браузеры FireFox и IE могут не поддерживать разрешения буфера обмена корректно.

2) Выберите и скопируйте выходные данные ключа в буфер обмена.

+ [GitHub](https://docs.github.com/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

+ [GitLab](https://docs.gitlab.com/ee/ssh/#adding-an-ssh-key-to-your-gitlab-account)

+ [Azure DevOps](/azure/devops/repos/git/use-ssh-keys-to-authenticate#step-2--add-the-public-key-to-azure-devops-servicestfs)  Начните с **шага 2**.

+ [BitBucket](https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/#SetupanSSHkey-ssh2). Начните с **шага 4**.

### <a name="clone-the-git-repository-with-ssh"></a>Клонирование репозитория Git с помощью SSH

1) Скопируйте URL-адрес клонирования Git SSH из репозитория Git.

2) Вставьте URL-адрес в `git clone` приведенную ниже команду, чтобы использовать URL-адрес репозитория Git SSH. Это будет выглядеть примерно так:

```bash
git clone git@example.com:GitUser/azureml-example.git
Cloning into 'azureml-example'...
```

Вы увидите такой ответ:

```bash
The authenticity of host 'example.com (192.30.255.112)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.255.112' (RSA) to the list of known hosts.
```

SSH может отобразить отпечаток SSH сервера и попросить его проверить. Следует убедиться, что отображаемый отпечаток соответствует одному из отпечатков пальцев на странице открытых ключей SSH.

SSH отображает этот отпечаток при подключении к неизвестному узлу для защиты от [атак типа "злоумышленник в середине"](/previous-versions/windows/it-pro/windows-2000-server/cc959354(v=technet.10)). После принятия отпечатка узла SSH не будет выводить запрос повторно, пока не изменится отпечаток.

3) Когда вам будет предложено продолжить подключение, введите `yes` . Git будет клонировать репозиторий и настроить исходный удаленный доступ для подключения по протоколу SSH для будущих команд git.

## <a name="track-code-that-comes-from-git-repositories"></a>Трассировка кода, поступающих из репозиториев Git

При отправке обучающего запуска из пакета SDK для Python или Машинное обучение CLI файлы, необходимые для обучения модели, отправляются в рабочую область. Если `git` команда доступна в среде разработки, процесс отправки использует его для проверки того, хранятся ли файлы в репозитории Git. Если да, то сведения из репозитория Git также передаются в ходе обучающего запуска. Эти сведения хранятся в следующих свойствах для обучающего запуска:

| Свойство | Команда git, используемая для получения значения | Описание |
| ----- | ----- | ----- |
| `azureml.git.repository_uri` | `git ls-remote --get-url` | Универсальный код ресурса (URI), из которого был клонирован репозиторий. |
| `mlflow.source.git.repoURL` | `git ls-remote --get-url` | Универсальный код ресурса (URI), из которого был клонирован репозиторий. |
| `azureml.git.branch` | `git symbolic-ref --short HEAD` | Активная ветвь при отправке выполнения. |
| `mlflow.source.git.branch` | `git symbolic-ref --short HEAD` | Активная ветвь при отправке выполнения. |
| `azureml.git.commit` | `git rev-parse HEAD` | Хэш фиксации кода, отправленного для выполнения. |
| `mlflow.source.git.commit` | `git rev-parse HEAD` | Хэш фиксации кода, отправленного для выполнения. |
| `azureml.git.dirty` | `git status --porcelain .` | `True`, если ветвь или фиксация являются грязными; в противном случае — `false` . |

Эти сведения отправляются для запусков, использующих оценщик, конвейер машинного обучения или запуск скрипта.

Если файлы обучения не находятся в репозитории Git в среде разработки или `git` команда недоступна, сведения, связанные с Git, не будут отслеживаниться.

> [!TIP]
> Чтобы проверить, доступна ли команда git в среде разработки, откройте сеанс оболочки, командную строку, PowerShell или другой интерфейс командной строки и введите следующую команду:
>
> ```
> git --version
> ```
>
> Если этот параметр установлен и в пути, вы получаете ответ, аналогичный `git version 2.4.1` . Дополнительные сведения об установке Git в среде разработки см. на [веб-сайте Git](https://git-scm.com/).

## <a name="view-the-logged-information"></a>Просмотр записанных данных

Сведения о git хранятся в свойствах для обучающего запуска. Эти сведения можно просмотреть с помощью портал Azure, пакета SDK для Python и интерфейса командной строки. 

### <a name="azure-portal"></a>Портал Azure

1. На [портале студии](https://ml.azure.com)выберите рабочую область.
1. Выберите __эксперименты__ и выберите один из экспериментов.
1. Выберите один из запусков из столбца __номер запуска__ .
1. Выберите __выходные данные + журналы__, а затем разверните __журналы__ и записи __azureml__ . Выберите ссылку, которая начинается с __### \_ Azure__.

Записанные в журнал данные содержат текст, похожий на следующий код JSON:

```json
"properties": {
    "_azureml.ComputeTargetType": "batchai",
    "ContentSnapshotId": "5ca66406-cbac-4d7d-bc95-f5a51dd3e57e",
    "azureml.git.repository_uri": "git@github.com:azure/machinelearningnotebooks",
    "mlflow.source.git.repoURL": "git@github.com:azure/machinelearningnotebooks",
    "azureml.git.branch": "master",
    "mlflow.source.git.branch": "master",
    "azureml.git.commit": "4d2b93784676893f8e346d5f0b9fb894a9cf0742",
    "mlflow.source.git.commit": "4d2b93784676893f8e346d5f0b9fb894a9cf0742",
    "azureml.git.dirty": "True",
    "AzureML.DerivedImageName": "azureml/azureml_9d3568242c6bfef9631879915768deaf",
    "ProcessInfoFile": "azureml-logs/process_info.json",
    "ProcessStatusFile": "azureml-logs/process_status.json"
}
```

### <a name="python-sdk"></a>Пакет SDK для Python

После отправки обучающего запуска возвращается объект [запуска](/python/api/azureml-core/azureml.core.run%28class%29) . `properties`Атрибут этого объекта содержит записанные сведения о git. Например, следующий код получает хэш фиксации:

```python
run.properties['azureml.git.commit']
```

### <a name="cli"></a>CLI

`az ml run`Команду CLI можно использовать для получения свойств из запуска. Например, следующая команда возвращает свойства для последнего запуска в эксперименте с именем `train-on-amlcompute` :

```azurecli-interactive
az ml run list -e train-on-amlcompute --last 1 -w myworkspace -g myresourcegroup --query '[].properties'
```

Дополнительные сведения см. в справочной документации по [AZ ML по запуску](/cli/azure/ext/azure-cli-ml/ml/run) .

## <a name="next-steps"></a>Дальнейшие действия

* [Использование целевых объектов вычислений для обучения моделей](how-to-set-up-training-targets.md)