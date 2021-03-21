---
title: Создание кластеров Apache Hadoop с помощью шаблонов Azure HDInsight
description: Узнайте о создании кластеров для HDInsight с помощью шаблонов Resource Manager.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-azurecli
ms.date: 04/07/2020
ms.openlocfilehash: 43f9736ce902d4c195d6b07cfdf7011ba9ca0c2c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101702748"
---
# <a name="create-apache-hadoop-clusters-in-hdinsight-by-using-resource-manager-templates"></a>Создание кластеров Apache Hadoop в HDInsight с помощью шаблонов Resource Manager

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Из этой статьи вы узнаете несколько способов создания кластеров Azure HDInsight с помощью [шаблонов Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md). Чтобы узнать о других средствах и функциях создания кластера, щелкните селектор вкладок в верхней части этой страницы. См. также [методы создания кластера](hdinsight-hadoop-provision-linux-clusters.md#cluster-setup-methods).

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="resource-manager-templates"></a>Шаблоны Resource Manager

Шаблон диспетчер ресурсов позволяет легко создавать следующие ресурсы для приложения в одной координированной операции:

* кластеры HDInsight и зависимые ресурсы (например, учетная запись хранения по умолчанию);
* другие ресурсы (например, база данных SQL Azure для использования [Apache Sqoop](https://sqoop.apache.org/)).

В шаблоне определяются ресурсы, необходимые для приложения. Можно также указать параметры развертывания в качестве входных значений для различных сред. Шаблон состоит из кода JSON и выражений, на основе которых можно создавать значения для развертывания.

Примеры шаблонов HDInsight можно найти в статье [шаблоны](https://azure.microsoft.com/resources/templates/?term=hdinsight)быстрого запуска Azure. Используйте кроссплатформенный редактор [Visual Studio Code](https://code.visualstudio.com/#alt-downloads) с [расширением Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools) или текстовый редактор, чтобы сохранить шаблон в файл на своей рабочей станции.

Дополнительные сведения о шаблонах Resource Manager см. в перечисленных ниже статьях и примерах:

* [Шаблоны диспетчера ресурсов Azure](../azure-resource-manager/templates/template-syntax.md)
* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../azure-resource-manager/templates/deploy-powershell.md)
* Справочник по шаблону [Microsoft.HDInsight/clusters](/azure/templates/microsoft.hdinsight/allversions)
* [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Hdinsight&pageNumber=1&sort=Popular)

## <a name="generate-templates"></a>Создание шаблонов

Resource Manager позволяет экспортировать шаблон Resource Manager из имеющихся ресурсов в подписке, используя различные инструменты. Используя созданный шаблон, можно изучить синтаксис шаблонов или при необходимости автоматизировать повторное развертывание решения. Дополнительные сведения см. в разделе [Экспорт шаблонов](../azure-resource-manager/templates/export-template-portal.md).

## <a name="deploy-using-the-portal"></a>Развертывание с помощью портала

Шаблон Resource Manager можно развернуть с помощью портала Azure. Дополнительные сведения см. в разделе [Развертывание ресурсов с помощью настраиваемого шаблона](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template).

## <a name="deploy-using-powershell"></a>Развертывание с помощью PowerShell

Шаблон Resource Manager можно развернуть с помощью Azure PowerShell. Дополнительные сведения см. в статьях [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) и [Развертывание частного шаблона Resource Manager с использованием токена SAS и Azure PowerShell](../azure-resource-manager/templates/secure-template-with-sas-token.md).

## <a name="deploy-using-azure-cli"></a>Развертывание с помощью Azure CLI

Шаблон Resource Manager можно развернуть с помощью Azure CLI. Дополнительные сведения см. в статьях [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../azure-resource-manager/templates/deploy-cli.md) и [Развертывание частного шаблона Resource Manager с использованием токена SAS и Azure CLI](../azure-resource-manager/templates/secure-template-with-sas-token.md).

## <a name="deploy-using-the-rest-api"></a>Развертывание с помощью REST API

Шаблон Resource Manager можно развернуть с помощью REST API. Дополнительные сведения см. в статье [Развертывание ресурсов с использованием шаблонов и REST API Resource Manager](../azure-resource-manager/templates/deploy-rest.md).

## <a name="deploy-with-visual-studio"></a>Развертывание с помощью Visual Studio

 С помощью Visual Studio можно создать проект группы ресурсов и развернуть его в Azure, используя пользовательский интерфейс. Выберите тип ресурсов, добавляемых в проект. Эти ресурсы автоматически добавляются в шаблон Resource Manager. Проект также предоставляет сценарий PowerShell для развертывания шаблона.

Обзорные сведения об использовании групп ресурсов в Visual Studio см. в статье [Создание и развертывание групп ресурсов Azure с помощью Visual Studio](../azure-resource-manager/templates/create-visual-studio-deployment-project.md).

## <a name="troubleshoot"></a>Диагностика

Если при создании кластеров HDInsight возникли проблемы, см. раздел [Создание кластеров](hdinsight-hadoop-customize-cluster-linux.md#access-control).

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы ознакомились с несколькими способами создания кластера HDInsight. Дополнительные сведения см. в следующих статьях:

* Дополнительные шаблоны, связанные с HDInsight, см. в статье шаблоны быстрого запуска [Azure](https://azure.microsoft.com/resources/templates/?term=hdinsight).
* Пример развертывания ресурсов с помощью клиентской библиотеки .NET см. в статье [Развертывание виртуальной машины Azure с помощью C# и шаблона Resource Manager](/previous-versions/azure/virtual-machines/windows/csharp-template?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
* Подробный пример развертывания приложения см. в статье [Предсказуемые подготовка и развертывание микрослужб в Azure](../app-service/deploy-complex-application-predictably.md).
* Инструкции по развертыванию своего решения в различных средах см. в статье [Среды разработки и тестирования в Microsoft Azure](../devtest-labs/devtest-lab-overview.md).
* Дополнительную информацию о разделах в шаблоне Azure Resource Manager см. в статье [Создание шаблонов диспетчера ресурсов Azure](../azure-resource-manager/templates/template-syntax.md).
* Список функций, которые можно использовать в шаблоне Azure Resource Manager, см. в статье [Функции шаблонов диспетчера ресурсов Azure](../azure-resource-manager/templates/template-functions.md).