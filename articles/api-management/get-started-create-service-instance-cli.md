---
title: Краткое руководство. Создание экземпляра службы "Управление API Azure" с помощью CLI
description: Создание экземпляра службы "Управление API Azure" с помощью Azure CLI
author: dlepow
ms.service: api-management
ms.topic: quickstart
ms.custom: ''
ms.date: 09/10/2020
ms.author: apimpm
ms.openlocfilehash: 72d1faac02a21f23d46eb992af1d501bca89e71f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101688082"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-by-using-the-azure-cli"></a>Краткое руководство. Создание экземпляра службы "Управление API Azure" с помощью Azure CLI

Служба управления API Azure помогает организациям публиковать API-интерфейсы для внешних пользователей, партнеров и собственных разработчиков, раскрывая таким образом потенциал своих данных и услуг. Служба управления API предоставляет базовые возможности для успешного выполнения программы API за счет привлечения разработчиков, бизнес-аналитики, анализа, безопасности и защищенности. Служба "Управление API" позволяет создавать современные шлюзы API для существующих серверных служб, размещенных в любом месте, и управлять ими. Дополнительные сведения см. в статье [Общие сведения о Службе контейнеров Azure](api-management-key-concepts.md).

В этом кратком руководстве описаны шаги по созданию экземпляра службы "Управление API" с помощью команд [az apim](/cli/azure/apim) в Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.11.1 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Экземпляры службы "Управление API Azure", как и все ресурсы Azure, должны быть развернуты в группе ресурсов. Группы ресурсов позволяют организовать соответствующие ресурсы Azure и управлять ими.

Для начала создайте группу ресурсов с именем *myResourceGroup* в регионе "центральная часть США" с помощью следующей команды [az group create](/cli/azure/group#az-group-create):

```azurecli-interactive
az group create --name myResourceGroup --location centralus
```

## <a name="create-a-new-service"></a>Создание службы

Теперь, когда у вас есть группа ресурсов, можно создать экземпляр службы "Управление API". Создайте его с помощью команды [az apim create](/cli/azure/apim#az-apim-create), а также укажите имя службы и сведения об издателе. Имя пользователя должно быть уникальным в пределах Azure. 

В следующем примере для имени службы используется *myapim*. Замените его уникальным значением. Также обновите имя организации издателя API и адрес электронной почты для получения уведомлений. 

```azurecli-interactive
az apim create --name myapim --resource-group myResourceGroup \
  --publisher-name Contoso --publisher-email admin@contoso.com \
  --no-wait
```

По умолчанию команда создает экземпляр на уровне разработчика, экономичный вариант для оценки службы "Управление API Azure". Этот уровень не предназначен для использования в рабочей среде. Дополнительные сведения о масштабировании категорий службы управления API см. в статье о [повышении категории и масштабировании](upgrade-and-scale.md). 

> [!TIP]
> Обычно создание и активация службы "Управление API" на этом уровне занимает от 30 до 40 минут. Предыдущая команда использует параметр `--no-wait`, чтобы результаты возвращались сразу же после создания службы.

Проверьте состояние развертывания, выполнив команду [az apim show](/cli/azure/apim#az-apim-show):

```azurecli-interactive
az apim show --name myapim --resource-group myResourceGroup --output table
```

Изначально выходные данные аналогичны приведенным ниже с состоянием `Activating`:

```console
NAME         RESOURCE GROUP    LOCATION    GATEWAY ADDR    PUBLIC IP    PRIVATE IP    STATUS      TIER       UNITS
-----------  ----------------  ----------  --------------  -----------  ------------  ----------  ---------  -------
myapim       myResourceGroup   Central US                                             Activating  Developer  1
```

После активации состояние изменится на `Online`, а экземпляр службы получит адрес шлюза и общедоступный IP-адрес. Сейчас эти адреса не предоставляют никакого содержимого. Например, такое:

```console
NAME         RESOURCE GROUP    LOCATION    GATEWAY ADDR                       PUBLIC IP     PRIVATE IP    STATUS    TIER       UNITS
-----------  ----------------  ----------  ---------------------------------  ------------  ------------  --------  ---------  -------
myapim       myResourceGroup   Central US  https://myapim.azure-api.net       203.0.113.1                 Online    Developer  1
```

Когда экземпляр службы "Управление API" находится в сети, его можно использовать. Изучите сведения из учебника, чтобы [импортировать и опубликовать первый API](import-and-publish.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Ненужные группу ресурсов и экземпляр службы "Управление API" можно удалить с помощью команды [az group delete](/cli/azure/group#az-group-delete).

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Импорт и публикация первого API](import-and-publish.md)
