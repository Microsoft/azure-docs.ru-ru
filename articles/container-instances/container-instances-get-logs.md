---
title: Получение журналов экземпляров контейнера & событий
description: Узнайте, как получить журналы контейнеров и события в службе "экземпляры контейнеров Azure", чтобы помочь в устранении проблем с контейнерами.
ms.topic: article
ms.date: 12/30/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 20b6e1cfe6bb8f6ac721a401c3d0831d4f447edb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92746960"
---
# <a name="retrieve-container-logs-and-events-in-azure-container-instances"></a>Получение журналов и событий контейнеров в службе "Экземпляры контейнеров Azure"

При наличии неправильного поведения контейнера в службе "экземпляры контейнеров Azure" Начните с просмотра журналов с помощью команды [AZ Container Logs][az-container-logs]и потоковой передачи стандартного выхода и стандартной ошибки с помощью команды [AZ Container Attach][az-container-attach]. Кроме того, можно просматривать журналы и события для экземпляров контейнеров в портал Azure или передавать данные журналов и событий для групп контейнеров в [журналы Azure Monitor](container-instances-log-analytics.md).

## <a name="view-logs"></a>Просмотр журналов

Для просмотра журналов из кода приложения в контейнере можно использовать команду [az container logs][az-container-logs].

Ниже приведены выходные данные журнала из примера контейнера на основе задач в [строке установка командной строки в экземпляре контейнера](container-instances-start-command.md#azure-cli-example)после того, как был предоставлен недопустимый URL-адрес с помощью переопределения командной строки:

```azurecli
az container logs --resource-group myResourceGroup --name mycontainer
```

```output
Traceback (most recent call last):
  File "wordcount.py", line 11, in <module>
    urllib.request.urlretrieve (sys.argv[1], "foo.txt")
  File "/usr/local/lib/python3.6/urllib/request.py", line 248, in urlretrieve
    with contextlib.closing(urlopen(url, data)) as fp:
  File "/usr/local/lib/python3.6/urllib/request.py", line 223, in urlopen
    return opener.open(url, data, timeout)
  File "/usr/local/lib/python3.6/urllib/request.py", line 532, in open
    response = meth(req, response)
  File "/usr/local/lib/python3.6/urllib/request.py", line 642, in http_response
    'http', request, response, code, msg, hdrs)
  File "/usr/local/lib/python3.6/urllib/request.py", line 570, in error
    return self._call_chain(*args)
  File "/usr/local/lib/python3.6/urllib/request.py", line 504, in _call_chain
    result = func(*args)
  File "/usr/local/lib/python3.6/urllib/request.py", line 650, in http_error_default
    raise HTTPError(req.full_url, code, msg, hdrs, fp)
urllib.error.HTTPError: HTTP Error 404: Not Found
```

## <a name="attach-output-streams"></a>Присоединение потоков вывода

Команда [az container attach][az-container-attach] предоставляет диагностические сведения во время запуска контейнера. Она направляет потоки STDOUT и STDERR запущенного контейнера в локальную консоль.

Например, ниже приведены выходные данные контейнера, основанного на задачах, в разделе [Установка командной строки в экземпляре контейнера](container-instances-start-command.md#azure-cli-example)после того, как был указан допустимый URL-адрес большого текстового файла для обработки:

```azurecli
az container attach --resource-group myResourceGroup --name mycontainer
```

```output
Container 'mycontainer' is in state 'Unknown'...
Container 'mycontainer' is in state 'Waiting'...
Container 'mycontainer' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 19:42:39+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
Container 'mycontainer1' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 19:42:39+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
(count: 1) (last timestamp: 2019-03-21 19:42:52+00:00) Successfully pulled image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
(count: 1) (last timestamp: 2019-03-21 19:42:55+00:00) Created container
(count: 1) (last timestamp: 2019-03-21 19:42:55+00:00) Started container

Start streaming logs:
[('the', 22979),
 ('I', 20003),
 ('and', 18373),
 ('to', 15651),
 ('of', 15558),
 ('a', 12500),
 ('you', 11818),
 ('my', 10651),
 ('in', 9707),
 ('is', 8195)]
```

## <a name="get-diagnostic-events"></a>Получение диагностических событий

Если контейнер не удается развернуть, ознакомьтесь с диагностическими сведениями, предоставленными поставщиком ресурсов службы "экземпляры контейнеров Azure". Чтобы просмотреть события для контейнера, выполните команду [az container show][az-container-show]:

```azurecli-interactive
az container show --resource-group myResourceGroup --name mycontainer
```

Выходные данные содержат основные свойства контейнера, а также события развертывания (здесь показаны в усеченном виде):

```JSON
{
  "containers": [
    {
      "command": null,
      "environmentVariables": [],
      "image": "mcr.microsoft.com/azuredocs/aci-helloworld",
      ...
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:22+00:00",
            "lastTimestamp": "2019-03-21T19:46:22+00:00",
            "message": "pulling image \"mcr.microsoft.com/azuredocs/aci-helloworld\"",
            "name": "Pulling",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:28+00:00",
            "lastTimestamp": "2019-03-21T19:46:28+00:00",
            "message": "Successfully pulled image \"mcr.microsoft.com/azuredocs/aci-helloworld\"",
            "name": "Pulled",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:31+00:00",
            "lastTimestamp": "2019-03-21T19:46:31+00:00",
            "message": "Created container",
            "name": "Created",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:31+00:00",
            "lastTimestamp": "2019-03-21T19:46:31+00:00",
            "message": "Started container",
            "name": "Started",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "name": "mycontainer",
      "ports": [
        {
          "port": 80,
          "protocol": null
        }
      ],
      ...
    }
  ],
  ...
}
```
## <a name="next-steps"></a>Дальнейшие действия
Узнайте, как [устранить неполадки контейнера и развертывания](container-instances-troubleshooting.md) для службы "Экземпляры контейнеров Azure".

Сведения об отправке журналов и данных событий для групп контейнеров в [Azure Monitor журналов](container-instances-log-analytics.md).

<!-- LINKS - Internal -->
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
