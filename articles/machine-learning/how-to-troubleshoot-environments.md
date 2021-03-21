---
title: Устранение неполадок с образами среды
titleSuffix: Azure Machine Learning
description: Узнайте, как устранять неполадки при сборке образов среды и установке пакетов.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: saachigopal
ms.author: sagopal
ms.date: 12/3/2020
ms.topic: troubleshooting
ms.custom: devx-track-python
ms.openlocfilehash: ec0c7d64f2145cdaf594cb903c072984f4d376a9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102519135"
---
# <a name="troubleshoot-environment-image-builds"></a>Устранение неполадок сборок образов среды

Узнайте, как устранять проблемы с сборками образов среды DOCKER и установками пакетов.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Попробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree).
* [Пакет SDK для Машинного обучения Azure](/python/api/overview/azure/ml/install).
* [Интерфейс командной строки Azure](/cli/azure/install-azure-cli).
* [Расширение CLI для Машинного обучения Azure](reference-azure-machine-learning-cli.md).
* Для локальной отладки вам необходима рабочая установка Docker в локальной системе.

## <a name="docker-image-build-failures"></a>Сбои сборки образа DOCKER
 
Для большинства сбоев сборки образа основная причина найдется в журнале сборки образа.
Найдите журнал сборки образа на портале Машинное обучение Azure (20 \_ образов \_ сборки \_log.txt) или в реестре контейнеров Azure запуск журналов выполнения задач.
 
Обычно проще воспроизвести ошибки локально. Проверьте тип ошибки и попробуйте выполнить одно из следующих `setuptools` действий.

- Локальная установка зависимости conda: `conda install suspicious-dependency==X.Y.Z` .
- Установите зависимость PIP локально: `pip install suspicious-dependency==X.Y.Z` .
- Попробуйте материализовать всю среду: `conda create -f conda-specification.yml` .

> [!IMPORTANT]
> Убедитесь, что платформа и интерпретатор в локальном кластере вычислений соответствуют параметрам в удаленном кластере вычислений. 

### <a name="timeout"></a>Время ожидания 
 
Ошибки времени ожидания могут быть вызваны следующими проблемами сети:

- Низкая пропускная способность Интернета
- Проблемы с сервером
- Большие зависимости, которые невозможно загрузить с заданными параметрами времени ожидания conda или PIP
 
В сообщениях, аналогичных приведенным ниже примерам, будет указано, что эта ошибка:
 
```
('Connection broken: OSError("(104, \'ECONNRESET\')")', OSError("(104, 'ECONNRESET')"))
```
```
ReadTimeoutError("HTTPSConnectionPool(host='****', port=443): Read timed out. (read timeout=15)",)
```

Если появляется сообщение об ошибке, попробуйте одно из следующих возможных решений.
 
- Попробуйте использовать другой источник, например зеркала, хранилище BLOB-объектов Azure или другие каналы Python, для зависимости.
- Обновите conda или PIP. Если вы используете пользовательский файл DOCKER, обновите параметры времени ожидания.
- Некоторые версии PIP имеют известные проблемы. Попробуйте добавить определенную версию PIP в зависимости среды.

### <a name="package-not-found"></a>Пакет не найден

Следующие ошибки наиболее распространены для сбоев сборки образа.

- Не удалось найти пакет Conda:

   ```
   ResolvePackageNotFound: 
   - not-existing-conda-package
   ```

- Не удалось найти указанный пакет PIP или версию:

   ```
   ERROR: Could not find a version that satisfies the requirement invalid-pip-package (from versions: none)
   ERROR: No matching distribution found for invalid-pip-package
   ```

- Неправильная вложенная зависимость PIP:

   ```
   ERROR: No matching distribution found for bad-package==0.0 (from good-package==1.0)
   ```

Убедитесь, что пакет существует в указанных источниках. Чтобы проверить зависимости PIP, используйте [Поиск PIP](https://pip.pypa.io/en/stable/reference/pip_search/) .

- `pip search azureml-core`

Для зависимостей conda используйте [Поиск conda](https://docs.conda.io/projects/conda/en/latest/commands/search.html):

- `conda search conda-forge::numpy`

Для получения дополнительных параметров попробуйте:
- `pip search -h`
- `conda search -h`

#### <a name="installer-notes"></a>Примечания к установщику

Убедитесь, что для указанной платформы и версии интерпретатора Python существует необходимое распределение.

Для зависимостей PIP перейдите к, `https://pypi.org/project/[PROJECT NAME]/[VERSION]/#files` чтобы узнать, доступна ли требуемая версия. Чтобы просмотреть пример, перейдите по адресу https://pypi.org/project/azureml-core/1.11.0/#files .

Для зависимостей conda Проверьте пакет в репозитории каналов.
Для каналов, поддерживаемых Anaconda, Inc., просмотрите [страницу пакеты Anaconda](https://repo.anaconda.com/pkgs/).

### <a name="pip-package-update"></a>Обновление пакета PIP

Во время установки или обновления пакета PIP может потребоваться обновить уже установленный пакет для удовлетворения новых требований.
Удаление может завершиться сбоем по различным причинам, связанным с версией PIP, или способом установки зависимости.
Наиболее распространенным сценарием является то, что зависимость, установленная conda, не может быть удалена с помощью PIP.
Для этого сценария рассмотрите возможность удаления зависимости с помощью `conda remove mypackage` .

```
  Attempting uninstall: mypackage
    Found existing installation: mypackage X.Y.Z
ERROR: Cannot uninstall 'mypackage'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.
``` 
### <a name="installer-issues"></a>Проблемы с установщиком

В некоторых версиях установщика возникают проблемы в процессе поиска пакетов, которые могут привести к сбою сборки.

Если вы используете пользовательский базовый образ или Dockerfile, рекомендуется использовать conda версии 4.5.4 или более поздней.

Для установки зависимостей PIP требуется пакет PIP. Если в среде не указана версия, будет использоваться последняя версия.
Рекомендуется использовать известную версию PIP, чтобы избежать временных проблем или критических изменений, которые могут возникнуть в последней версии средства.

Рекомендуется закреплять версию PIP в своей среде, если отображается следующее сообщение:

   ```
   Warning: you have pip-installed dependencies in your environment file, but you do not list pip itself as one of your conda dependencies. Conda may not use the correct pip to install your packages, and they may end up in the wrong place. Please add an explicit pip dependency. I'm adding one for you, but still nagging you.
   ```

Ошибка подпроцесса PIP:
   ```
   ERROR: THESE PACKAGES DO NOT MATCH THE HASHES FROM THE REQUIREMENTS FILE. If you have updated the package versions, update the hashes as well. Otherwise, examine the package contents carefully; someone may have tampered with them.
   ```

Установка PIP может задержаться в бесконечном цикле, если в зависимостях есть неразрешимые конфликты. Если вы работаете локально, понизить версию PIP до < 20,3. В среде conda, созданной из файла YAML, эту ошибку можно увидеть только в том случае, если conda-подделка является каналом с наивысшим приоритетом. Чтобы устранить эту ошибку, явно укажите PIP < 20,3 (! = 20,3 или = 20.2.4 ПИН-код для другой версии) в качестве зависимости conda в файле спецификации conda.

## <a name="service-side-failures"></a>Сбои на стороне службы

Чтобы устранить возможные сбои на стороне службы, см. следующие сценарии.

### <a name="youre-unable-to-pull-an-image-from-a-container-registry-or-the-address-couldnt-be-resolved-for-a-container-registry"></a>Не удается извлечь образ из реестра контейнеров, или не удалось разрешить адрес для реестра контейнеров.

Возможные проблемы:
- Возможно, путь к реестру контейнеров неправильно разрешается. Убедитесь, что в именах образов используются двойные косые черты и направление косой черты на узлах Linux и Windows является правильным.
- Если в реестре контейнеров, расположенном за виртуальной сетью, используется частная конечная точка в [неподдерживаемом регионе](../private-link/private-link-overview.md#availability), настройте реестр контейнеров с помощью конечной точки службы (общедоступного доступа) на портале и повторите попытку.
- После размещения реестра контейнеров для виртуальной сети запустите [шаблон Azure Resource Manager](./how-to-network-security-overview.md) , чтобы Рабочая область могла взаимодействовать с экземпляром реестра контейнеров.

### <a name="you-get-a-401-error-from-a-workspace-container-registry"></a>Вы получаете ошибку 401 из реестра контейнеров рабочих областей.

Повторно синхронизируйте ключи хранилища с помощью [WS.sync_keys ()](/python/api/azureml-core/azureml.core.workspace.workspace#sync-keys--).

### <a name="the-environment-keeps-throwing-a-waiting-for-other-conda-operations-to-finish-error"></a>Среда продолжает выдачу "ожидания завершения других операций conda..." план

Когда сборка образа выполняется, conda блокируется клиентом SDK. В случае сбоя или неправильного завершения процесса пользователем conda остается в заблокированном состоянии. Чтобы устранить эту проблему, удалите файл блокировки вручную. 

### <a name="your-custom-docker-image-isnt-in-the-registry"></a>Пользовательский образ DOCKER отсутствует в реестре

Проверьте, используется ли [правильный тег](./how-to-use-environments.md#create-an-environment) `user_managed_dependencies = True` . `Environment.python.user_managed_dependencies = True` отключает conda и использует установленные пользователем пакеты.

### <a name="you-get-one-of-the-following-common-virtual-network-issues"></a>Вы получаете одну из следующих распространенных проблем виртуальной сети.

- Убедитесь, что учетная запись хранения, кластер вычислений и реестр контейнеров находятся в одной подсети виртуальной сети.
- Если реестр контейнеров находится за виртуальной сетью, он не может быть напрямую использован для создания образов. Для создания образов необходимо использовать кластер вычислений.
- Может потребоваться разместить хранилище за виртуальной сетью, если:
    - Используйте заданное или закрытое колесо.
    - См. ошибки службы 403 (несанкционированные).
    - Не удается получить сведения об образе из реестра контейнеров Azure.

### <a name="the-image-build-fails-when-youre-trying-to-access-network-protected-storage"></a>Сборка образа завершается сбоем, если вы пытаетесь получить доступ к защищенному сетевому хранилищу

- Задачи реестра контейнеров Azure не работают за виртуальной сетью. Если у пользователя есть реестр контейнеров, который находится за виртуальной сетью, для построения образа необходимо использовать этот кластер.
- Хранилище должно находиться позади виртуальной сети, чтобы извлечь из него зависимости.

### <a name="you-cant-run-experiments-when-storage-has-network-security-enabled"></a>Вы не можете запускать эксперименты, если для хранилища включена безопасность сети

Если вы используете образы DOCKER по умолчанию и включаете управляемые пользователем зависимости, используйте [теги службы](./how-to-network-security-overview.md) Микрософтконтаинеррегистри и Азурефронтдур. фирстпарти для разрешенных реестра контейнеров Azure и его зависимостей.

 Дополнительные сведения см. в разделе [Включение виртуальных сетей](./how-to-network-security-overview.md).

### <a name="you-need-to-create-an-icm"></a>Необходимо создать ICM

При создании или назначении ICM для хранилище метаданных включите запрос в службу поддержки CSS, чтобы мы могли лучше понять эту ошибку.

## <a name="next-steps"></a>Дальнейшие действия

- [Обучение модели машинного обучения для категоризации цветов](how-to-train-scikit-learn.md)
- [Обучение модели машинного обучения с помощью пользовательского образа DOCKER](how-to-train-with-custom-image.md)
