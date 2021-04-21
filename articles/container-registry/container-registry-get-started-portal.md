---
title: Краткое руководство. Создание реестра на портале
description: Быстрый способ изучить создание частного реестра контейнеров Azure с помощью портала Azure.
ms.date: 08/04/2020
ms.topic: quickstart
ms.custom:
- mvc
- mode-portal
ms.openlocfilehash: 4618a83a43d9c27f44432d5d346fdb053cee7054
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773591"
---
# <a name="quickstart-create-an-azure-container-registry-using-the-azure-portal"></a>Краткое руководство. Создание реестра контейнеров Azure с помощью портала Azure

Реестр контейнеров Azure — это частный реестр Docker в Azure, где можно хранить частные образы и связанные артефакты контейнеров Docker и управлять ими. В этом кратком руководстве вы создадите реестр контейнеров с помощью портала Azure. Затем используйте команды Docker, чтобы отправить образ контейнера в реестр, после чего извлеките образ из контейнера и запустите его.

Чтобы войти в реестр для работы с образами контейнеров, в этом кратком руководстве требуется использовать Azure CLI (рекомендуется версия 2.0.55 или более поздняя). Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli].

Также необходим локально установленный модуль Docker. Docker содержит пакеты, которые позволяют быстро настроить Docker в любой системе [Mac][docker-mac], [Windows][docker-windows] или [Linux][docker-linux].

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу https://portal.azure.com.

## <a name="create-a-container-registry"></a>Создание реестра контейнеров

Последовательно выберите **Создать ресурс** > **Контейнеры** > **Реестр контейнеров**.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-01.png" alt-text="Переход к реестру контейнеров на портале":::

На вкладке **Основные сведения** введите значения в поле **Группа ресурсов** и **Имя реестра**. Имя реестра должно быть уникальным в пределах Azure и содержать от 5 до 50 буквенно-цифровых символов. В целях этого руководства создайте группу ресурсов в регионе `West US` в расположении с именем `myResourceGroup`. Для **SKU** задайте значение "Базовый".

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-03.png" alt-text="Создание реестра контейнеров на портале":::

Примите значения по умолчанию для остальных параметров. Щелкните **Просмотр и создание**. Проверив параметры, нажмите кнопку **Создать**.

В этом кратком руководстве описано, как создать реестр ценовой категории *Базовый*. Это оптимальный (недорогой) вариант для разработчиков, которые знакомятся с Реестром контейнеров Azure. Дополнительные сведения об уровнях служб (SKU) см. в статье [Уровни служб реестра контейнеров][container-registry-skus].

Когда появится сообщение **Развертывание прошло успешно**, выберите реестр контейнеров на портале. 

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-05.png" alt-text="Страница обзора реестра контейнеров на портале":::

Запишите имя реестра и значение **сервера входа**. Эти значения используются в следующих шагах при отправке и извлечении образов с помощью Docker.

## <a name="log-in-to-registry"></a>Вход в раздел реестра

Перед отправкой и извлечением образов контейнеров необходимо войти в экземпляр реестра. [Войдите в Azure CLI][get-started-with-azure-cli] на локальном компьютере, а затем выполните команду [az acr login][az-acr-login]. При входе с помощью Azure CLI укажите только имя реестра. Не используйте имя сервера для входа, которое содержит суффикс домена, например `azurecr.io`.

```azurecli
az acr login --name <registry-name>
```

Пример

```azurecli
az acr login --name mycontainerregistry
```

По завершении команда возвращает `Login Succeeded`. 

[!INCLUDE [container-registry-quickstart-docker-push](../../includes/container-registry-quickstart-docker-push.md)]

## <a name="list-container-images"></a>Список образов контейнеров

Чтобы вывести список образов в вашем реестре, перейдите в реестр на портале и выберите **Репозитории**, а затем выберите репозиторий **hello-world**, созданный с помощью `docker push`.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-09.png" alt-text="Список образов контейнеров на портале":::

Выбрав репозиторий **hello-world**, вы увидите образ с тегом `v1` в разделе **Теги**.

[!INCLUDE [container-registry-quickstart-docker-pull](../../includes/container-registry-quickstart-docker-pull.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, перейдите к группе ресурсов **myResourceGroup** на портале. После загрузки группы ресурсов щелкните пункт **Удалить группу ресурсов**, чтобы удалить группу ресурсов, реестр контейнеров и сохраненные образы контейнеров.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-08.png" alt-text="Удаление группы ресурсов на портале":::


## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы создали Реестр контейнеров Azure с использованием портала Azure, отправили образ контейнера, а затем извлекли этот образ оттуда и запустили его. Чтобы продолжить работу с Реестром контейнеров Azure, перейдите к следующим руководствам.

> [!div class="nextstepaction"]
> [Руководства по использованию Реестра контейнеров Azure][container-registry-tutorial-prepare-registry]

> [!div class="nextstepaction"]
> [Руководство. Создание и развертывание образов контейнера в облаке с помощью службы "Задачи Реестра контейнеров Azure"][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[container-registry-tutorial-prepare-registry]: container-registry-tutorial-prepare-registry.md
[container-registry-skus]: container-registry-skus.md
[azure-cli]: /cli/azure/install-azure-cli
[get-started-with-azure-cli]: /cli/azure/get-started-with-azure-cli
[az-acr-login]: /cli/azure/acr#az_acr_login
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
