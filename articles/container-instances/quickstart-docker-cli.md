---
title: Краткое руководство. Развертывание контейнера Docker в экземпляр контейнеров — Docker CLI
description: В этом кратком руководстве описано, как быстро развернуть контейнерное веб-приложение, выполняющееся в изолированном экземпляре контейнера Azure с помощью Docker CLI
ms.topic: quickstart
ms.date: 09/14/2020
ms.custom: devx-track-js
ms.openlocfilehash: 9904fb8c2142816196a1939a16445318bdb245d0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91262320"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-the-docker-cli"></a>Краткое руководство. Развертывание экземпляра контейнера в Azure с помощью Docker CLI

Служба "Экземпляры контейнеров Azure" позволяет легко и быстро запускать бессерверные контейнеры Docker в Azure. Выполняйте развертывание в экземпляр контейнера по запросу, если разрабатываете облачные приложения и хотите легко переходить с локальной разработки на облачное развертывание.

В этом кратком руководстве вы используете собственные команды Docker CLI, чтобы развернуть контейнер Docker и сделать его приложение доступным в Экземплярах контейнеров. Эта возможность предоставляется благодаря [интеграции Docker и Azure](https://docs.docker.com/engine/context/aci-integration/). Через несколько секунд после выполнения команды `docker run` вы можете перейти к приложению, которое выполняется в контейнере:

:::image type="content" source="media/quickstart-docker-cli/view-application-running-in-an-azure-container-instance.png" alt-text="Приложение, развернутое с помощью службы &quot;Экземпляры контейнеров Azure&quot; (просмотр в браузере)":::

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись][azure-account], прежде чем начинать работу.

Для целей этого краткого руководства вам понадобится Docker Desktop версии 2.3.0.5 или более поздней для [Windows](https://desktop.docker.com/win/edge/Docker%20Desktop%20Installer.exe) или [macOS](https://desktop.docker.com/mac/edge/Docker.dmg). Или же установите [интерфейс командной строки интеграции Docker ACI для Linux](https://docs.docker.com/engine/context/aci-integration/#install-the-docker-aci-integration-cli-on-linux). 

> [!IMPORTANT]
> Поддерживаются не все функции Экземпляров контейнеров Azure. Предоставьте отзыв об интеграции Docker-Azure, создав вопрос в репозитории GitHub для [aci-integration-beta](https://github.com/docker/aci-integration-beta).

[!INCLUDE [container-instances-create-docker-context](../../includes/container-instances-create-docker-context.md)]

## <a name="create-a-container"></a>Создание контейнера

Создав контекст Docker, можно приступить к созданию контейнера в Azure. В этом кратком руководстве используется общедоступный образ `mcr.microsoft.com/azuredocs/aci-helloworld`. Этот образ содержит небольшое веб-приложение, написанное на Node.js, которое обслуживает статические HTML-страницы.

Сначала перейдите в контекст ACI. В этом контексте выполняются все последующие команды Docker.

```
docker context use myacicontext
```

Выполните команду `docker run`, чтобы создать экземпляр контейнера Azure с портом 80, доступным в Интернете.

```
docker run -p 80:80 mcr.microsoft.com/azuredocs/aci-helloworld
```

Пример выходных данных при успешном развертывании:

```
[+] Running 2/2
 ⠿ hungry-kirch            Created                                                                               5.1s
 ⠿ single--container--aci  Done                                                                                 11.3s
hungry-kirch
```

Запустите `docker ps`, чтобы получить сведения о выполняющемся контейнере и общедоступном IP-адресе:

```
docker ps
```


В примере выходных данных показан общедоступный IP-адрес, в данном случае *52.230.225.232*:

```
CONTAINER ID        IMAGE                                        COMMAND             STATUS              PORTS
hungry-kirch        mcr.microsoft.com/azuredocs/aci-helloworld                       Running             52.230.225.232:80->80/tcp
```

 Затем перейдите по IP-адресу в браузере. Если появится примерно такая веб-страница, поздравляем! Вы успешно развернули приложение, выполняющееся в контейнере Docker в Azure.

:::image type="content" source="media/quickstart-docker-cli/view-application-running-in-an-azure-container-instance.png" alt-text="Приложение, развернутое с помощью службы &quot;Экземпляры контейнеров Azure&quot; (просмотр в браузере)":::

## <a name="pull-the-container-logs"></a>Извлечение журналов контейнера

При необходимости устранения неполадок с контейнером или запущенным в нем приложением (или просто чтобы просмотреть выходные данные) начните с просмотра журналов экземпляра контейнера.

Например, выполните команду `docker logs`, чтобы просмотреть журналы контейнера *hungry-kirch* в контексте ACI:

```azurecli-interactive
docker logs hungry-kirch
```

В выходных данных будут содержаться журналы для контейнера, а также должны отобразиться запросы HTTP GET, созданные при просмотре приложения в браузере.

```output
listening on port 80
::ffff:10.240.255.55 - - [07/Jul/2020:17:43:53 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [07/Jul/2020:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [07/Jul/2020:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
```


## <a name="clean-up-resources"></a>Очистка ресурсов

Завершив работу с контейнером, запустите `docker rm`, чтобы удалить его. Эта команда останавливает и удаляет экземпляр контейнера Azure.

```
docker rm hungry-kirch
```


## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали экземпляр контейнера Azure из общедоступного образа, используя интеграцию между Docker и Azure. Дополнительные сведения о сценариях интеграции см. в [документации по Docker](https://docs.docker.com/engine/context/aci-integration/). 

Кроме того, вы можете использовать [расширение Docker для Visual Studio Code](https://aka.ms/VSCodeDocker), чтобы получить доступ к интегрированному интерфейсу для разработки, запуска контейнеров, образов и контекстов, а также управления ими.

Чтобы использовать средства Azure для создания экземпляров контейнеров и управления ими, ознакомьтесь с другими краткими руководствами об использовании [Azure CLI](container-instances-quickstart.md), [Azure PowerShell](container-instances-quickstart-powershell.md), [портала Azure](container-instances-quickstart-portal.md) и [шаблонов Azure Resource Manager](container-instances-quickstart-template.md).

Если вы хотите использовать Docker Compose, чтобы определить и запустить многоконтейнерное приложение локально, а затем переключиться на Экземпляры контейнеров Azure, перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Учебник по Docker Compose](./tutorial-docker-compose.md)

<!-- LINKS - External -->

[azure-account]: https://azure.microsoft.com/free/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

