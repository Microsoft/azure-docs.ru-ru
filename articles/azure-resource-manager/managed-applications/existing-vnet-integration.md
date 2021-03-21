---
title: Развертывание в существующей виртуальной сети
description: Описывает, как разрешить пользователям управляемого приложения выбирать существующую виртуальную сеть. Виртуальная сеть может находиться за пределами управляемого приложения.
author: tfitzmac
ms.topic: conceptual
ms.date: 05/11/2020
ms.author: tomfitz
ms.openlocfilehash: fa5e59b96aada06c2dd486094d9be6a52c79e43e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84260687"
---
# <a name="use-existing-virtual-network-with-azure-managed-applications"></a>Использование существующей виртуальной сети с управляемыми приложениями Azure

В этой статье показано, как определить управляемое приложение Azure, которое интегрируется с существующей виртуальной сетью в подписке потребителя. Управляемое приложение позволяет потребителю решить, следует ли создать новую виртуальную сеть или использовать существующую. Существующая виртуальная сеть может находиться за пределами управляемой группы ресурсов.

## <a name="main-template"></a>Основной шаблон

Сначала рассмотрим **mainTemplate.js** файла. Ниже показан весь шаблон для развертывания виртуальной машины и связанных с ней ресурсов. Позже вы узнаете о том, какие части шаблона связаны с использованием существующей виртуальной сети.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/mainTemplate.json":::

Обратите внимание, что виртуальная сеть [условно развернута](../templates/conditional-resource-deployment.md). Потребитель передает значение параметра, которое указывает, следует ли создать новую или использовать существующую виртуальную сеть. Если потребитель выбирает новую виртуальную сеть, ресурс развертывается. В противном случае ресурс будет пропущен во время развертывания.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/mainTemplate.json" range="111-132" highlight="2":::

Переменная для идентификатора виртуальной сети имеет два свойства. Одно свойство возвращает идентификатор ресурса при развертывании новой виртуальной сети. Другое свойство возвращает идентификатор ресурса при использовании существующей виртуальной сети. Идентификатор ресурса для существующей виртуальной сети включает имя группы ресурсов, содержащей виртуальную сеть.

Идентификатор подсети создается на основе значения идентификатора виртуальной сети. Он использует значение, совпадающее с выбором получателей.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/mainTemplate.json" range="98-109" highlight="6-10":::

Для сетевого интерфейса задается переменная с ИДЕНТИФИКАТОРом подсети.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/mainTemplate.json" range="142-163" highlight="16":::

## <a name="ui-definition"></a>Определение пользовательского интерфейса

Теперь давайте взглянем на **createUiDefinition.js** файла. Файл целиком:

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/createUiDefinition.json":::

Файл содержит элемент виртуальной сети.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/createUiDefinition.json" range="53-81":::

Этот элемент позволяет потребителю выбрать новую или существующую виртуальную сеть.

:::image type="content" source="./media/existing-vnet-integration/new-or-existing-vnet.png" alt-text="Новая или Существующая виртуальная сеть":::

В выходных данных включается значение, указывающее, выбрал ли потребитель новую или существующую виртуальную сеть. Также существует управляемое значение идентификатора.

> [!NOTE]
> Выходное значение для управляемого удостоверения должно называться **managedIdentity**.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/createUiDefinition.json" range="136-148" highlight="6,12":::

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о создании файла определения пользовательского интерфейса см. в статье [CreateUiDefinition.jsо создании управляемого приложения Azure](create-uidefinition-overview.md).
