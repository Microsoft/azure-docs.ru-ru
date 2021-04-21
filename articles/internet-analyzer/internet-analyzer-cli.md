---
title: Создание теста Анализатора Интернета с помощью интерфейса командной строки | Документация Майкрософт
description: Из этой статьи вы узнаете, как создать тест Анализатора Интернета с помощью Azure CLI.
services: internet-analyzer
author: diego-perez-botero
ms.service: internet-analyzer
ms.topic: tutorial
ms.date: 10/16/2019
ms.author: mebeatty
ms.openlocfilehash: 46caae53ed81de335c2b9d5ddbd3fd7f89424fdd
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780421"
---
# <a name="create-an-internet-analyzer-test-using-cli-preview"></a>Создание теста Анализатора Интернета с помощью интерфейса командной строки (предварительная версия)

Есть два способа создания ресурса Анализатора Интернета — с помощью [портала Azure](internet-analyzer-create-test-portal.md) или с помощью интерфейса командной строки. В этом разделе показано, как создать ресурс Анализатора Интернета с помощью интерфейса командной строки. 


> [!IMPORTANT]
> Эта общедоступная предварительная версия предоставляется без соглашения об уровне обслуживания и не должна использоваться для производственных рабочих нагрузок. Некоторые функции могут не поддерживаться, иметь ограничения и быть доступными не во всех расположениях Azure. См. [дополнительные условия использования для предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="before-you-begin"></a>Перед началом

Общедоступная предварительная версия доступна для использования по всему миру. Однако на этапе этой версии хранилище данных предоставляется только в регионе *Западная часть США 2*.

## <a name="object-model"></a>Объектная модель
Интерфейс командной строки Анализатора Интернета предоставляет следующие типы ресурсов:
* **Тесты**. Тест сравнивает производительность пользователей двух конечных точек Интернета (A и B) с течением времени.
* **Профили**. Тесты создаются в профиле Анализатора Интернета. Профили позволяют группировать связанные тесты (один профиль может содержать один или несколько тестов).
* **Предварительно настроенные конечные точки**. Конечные точки с различными конфигурациями (регионами, технологиями ускорения и т. д.). В тестах можно использовать любую из этих предварительно настроенных конечных точек.
* **Системы показателей**. Система показателей предоставляет быстрые и осмысленные сводки результатов измерения. Дополнительные сведения см. в статье об [интерпретации системы показателей](internet-analyzer-scorecard.md).
* **Временные ряды**. Временной ряд показывает, как метрика изменяется со временем.

## <a name="profile-and-test-creation"></a>Создание профиля и теста
1. Получите доступ к предварительной версии Анализатора Интернета, следуя инструкциям, приведенным в разделе **об участии в предварительной версии** на странице [вопросов и ответов по Анализатору Интернета Azure](internet-analyzer-faq.md).
2. [Установка Azure CLI](/cli/azure/install-azure-cli).
3. Выполните команду `login`, чтобы запустить сеанс интерфейса командной строки:
    ```azurecli-interactive
    az login
    ```

    Если в CLI можно запустить браузер по умолчанию, откроется браузер со страницей входа.
    Если нет, самостоятельно откройте в браузере страницу https://aka.ms/devicelogin и введите код авторизации, отображаемый в терминале.

4. Выполните вход в браузере с помощью учетных данных.

5. Выберите идентификатор подписки, которому был предоставлен доступ к общедоступной предварительной версии Анализатора Интернета.

    После входа в систему появится список подписок, связанных с учетной записью Azure. В сведениях о подписке с `isDefault: true` указана текущая активная подписка, отображаемая после входа. Чтобы выбрать другую подписку, укажите идентификатор подписки с помощью команды [az account set](/cli/azure/account#az_account_set). См. дополнительные сведения о [выборе нужной подписки при использовании нескольких подписок Azure](/cli/azure/manage-azure-subscriptions-azure-cli).

    Вы также можете войти в автономном режиме, как описано в руководстве по [входу с помощью Azure CLI](/cli/azure/authenticate-azure-cli).

6. **[Необязательно]** . Создайте группу ресурсов Azure.
    ```azurecli-interactive
    az group create --location eastus --name "MyInternetAnalyzerResourceGroup"
    ```

7. Установите расширение Анализатора Интернета для интерфейса командной строки Azure.
     ```azurecli-interactive
    az extension add --name internet-analyzer
    ```

8. Создайте профиль Анализатора Интернета.
    ```azurecli-interactive
    az internet-analyzer profile create --location eastus --resource-group "MyInternetAnalyzerResourceGroup" --name "MyInternetAnalyzerProfile" --enabled-state Enabled
    ```

9. Выведите все предварительно настроенные конечные точки, доступные для созданного профиля.
    ```azurecli-interactive
    az internet-analyzer preconfigured-endpoint list --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile"
    ```

10. Создайте тест в созданном профиле Анализатора Интернета.
    ```azurecli-interactive
    az internet-analyzer test create --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --endpoint-a-name "contoso" --endpoint-a-endpoint "www.contoso.com/some/path/to/trans.gif" --endpoint-b-name "microsoft" --endpoint-b-endpoint "www.microsoft.com/another/path/to/trans.gif" --name "MyFirstInternetAnalyzerTest" --enabled-state Enabled
    ```

    Приведенная выше команда предполагает, что ресурсы `www.contoso.com` и `www.microsoft.com` содержат однопиксельное изображение ([trans.gif](https://fpc.msedge.net/apc/trans.gif)) в настраиваемом пути. Если путь к объекту не указан явно, то Анализатор Интернета по умолчанию будет использовать `/apc/trans.gif` в качестве пути к объекту, где в предварительно настроенных конечных точках размещается однопиксельное изображение. Обратите также внимание, что не нужно указывать схему (HTTPS/HTTP). Анализатор Интернета поддерживает только конечные точки HTTPS, поэтому предполагается использование протокола HTTPS.

11. Новый тест должен появиться в профиле Анализатора Интернета:
    ```azurecli-interactive
    az internet-analyzer test list --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile"
    ```

    Выходные данные примера:
    ````
    [
        {
            "description": null,
            "enabledState": "Enabled",
            "endpointA": {
            "endpoint": "www.contoso.com/some/path/to/1k.jpg",
            "name": "contoso"
            },
            "endpointB": {
            "endpoint": "www.microsoft.com/another/path/to/1k.jpg",
            "name": "microsoft"
            },
            "id": "/subscriptions/faa9ddd0-9137-4659-99b7-cdc55a953342/resourcegroups/MyInternetAnalyzerResourceGroup/providers/Microsoft.Network/networkexperimentprofiles/MyInternetAnalyzerProfile/experiments/MyFirstInternetAnalyzerTest",
            "location": null,
            "name": "MyFirstInternetAnalyzerTest",
            "resourceGroup": "MyInternetAnalyzerResourceGroup",
            "resourceState": "Enabled",
            "scriptFileUri": "https://fpc.msedge.net/client/v2/d8c6fc64238d464c882cee4a310898b2/ab.min.js",
            "status": "Created",
            "tags": null,
            "type": "Microsoft.Network/networkexperimentprofiles/experiments"
        }
    ]
    ````

12. Чтобы начать создание измерений, в веб-приложение нужно встроить файл JavaScript, на который указывает объект **scriptFileUri** теста. Специальные инструкции можно найти на странице [Embed the Internet Analyzer client](internet-analyzer-embed-client.md) (Внедрение клиента Анализатора Интернета).

13. Ход выполнения теста можно отслеживать по значению состояния:
    ```azurecli-interactive
    az internet-analyzer test show --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest"
    ```

14. Чтобы проверить полученные результаты теста, можно создать для него временные ряды или системы показателей:
    ```azurecli-interactive
    az internet-analyzer show-scorecard --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest" --aggregation-interval "Daily" --end-date-time-utc "2019-10-24T00:00:00"
    ```

    ```azurecli-interactive
    az internet-analyzer show-timeseries --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest" --aggregation-interval "Hourly" --start-date-time-utc "2019-10-23T00:00:00" --end-date-time-utc "2019-10-24T00:00:00" --timeseries-type MeasurementCounts
    ```


## <a name="next-steps"></a>Дальнейшие действия

* Полный список поддерживаемых команд и примеры использования см. в справочнике по [интерфейсу командной строки службы "Анализатор Интернета"](/cli/azure/ext/internet-analyzer/internet-analyzer).
* [Вопросы и ответы об Анализаторе Интернета](internet-analyzer-faq.md).
* Узнайте больше о внедрении [клиента Анализатора Интернета](internet-analyzer-embed-client.md) и создании [пользовательской конечной точки](internet-analyzer-custom-endpoint.md).