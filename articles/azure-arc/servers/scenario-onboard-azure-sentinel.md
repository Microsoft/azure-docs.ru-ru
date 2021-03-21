---
title: Подключение сервера с поддержкой дуги Azure к Azure Sentinel
description: Узнайте, как добавить серверы с поддержкой ARC в Azure Sentinel и заблаговременно отслеживать их состояние безопасности.
ms.date: 11/16/2020
ms.topic: conceptual
ms.openlocfilehash: 2364ba72ac5b10ec4e1f433cc6d591c3ca389ecd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100584742"
---
# <a name="onboard-azure-arc-enabled-servers-to-azure-sentinel"></a>Подключение серверов с поддержкой дуги Azure к Azure Sentinel

Эта статья поможет вам подключить сервер Azure Arc к [Azure Sentinel](../../sentinel/overview.md) и начать сбор событий, связанных с безопасностью. Azure Sentinel предоставляет единое решение для обнаружения предупреждений, отображения угроз, упреждающего поиска и реагирования на угрозы для всего предприятия.

## <a name="prerequisites"></a>Предварительные условия

Прежде чем начать, убедитесь, что выполнены следующие требования:

- [Рабочая область Log Analytics](../../azure-monitor/logs/data-platform-logs.md). Дополнительные сведения о рабочих областях Log Analytics см. в статье [Designing your Azure Monitor Logs deployment](../../azure-monitor/logs/design-logs-deployment.md) (Планирование развертывания журналов Azure Monitor).

- [В вашей подписке включен](../../sentinel/quickstart-onboard.md)Sentinel Azure.

- Вы используете компьютер или сервер, подключенные к серверам с поддержкой Arc Azure.

## <a name="onboard-azure-arc-enabled-servers-to-azure-sentinel"></a>Подключение серверов с поддержкой дуги Azure к Azure Sentinel

В Azure Sentinel имеется ряд соединителей для решений Майкрософт, которые доступны для использования в режиме реального времени. На физических и виртуальных машинах можно установить агент Log Analytics, который собирает журналы и перенаправляет их в Azure Sentinel. Серверы с поддержкой Arc поддерживают развертывание агента Log Analytics с помощью следующих методов.

- Использование платформы расширений виртуальных машин.

    Эта функция на серверах с поддержкой Arc Azure позволяет развернуть расширение виртуальной машины агента Log Analytics на сервере Windows и (или) Linux, отличном от Azure. Для управления расширениями виртуальной машины можно использовать следующие методы на гибридных компьютерах или серверах, управляемых с помощью серверов с поддержкой ARC:

    - [портал Azure](manage-vm-extensions-portal.md).
    - [Интерфейс командной строки Azure](manage-vm-extensions-cli.md)
    - [Azure PowerShell](manage-vm-extensions-powershell.md)
    - [Шаблоны диспетчер ресурсов](manage-vm-extensions-template.md) Azure

- Использование политики Azure.

    При таком подходе вы используете политику Azure [Deploy log Analytics Agent на компьютерах Linux или Windows Azure Arc](../../governance/policy/samples/built-in-policies.md#monitoring) , чтобы выполнять аудит, если на сервере с поддержкой Arc установлен агент log Analytics. Если агент не установлен, он автоматически развертывает его с помощью задачи исправления. Кроме того, если планируется отслеживать компьютеры с Azure Monitor для виртуальных машин, используйте инициативу [Enable Azure Monitor для виртуальных машин](../../governance/policy/samples/built-in-initiatives.md#monitoring) для установки и настройки агента log Analytics.

Мы рекомендуем установить агент Log Analytics для Windows или Linux с помощью политики Azure.

После подключения серверов с поддержкой Arc ваши данные начинают потоковую передачу в Azure Sentinel и готовы начать работу с. Журналы можно просмотреть во [встроенных книгах](../../sentinel/quickstart-get-visibility.md). Затем вы можете приступить к [исследованию данных](../../sentinel/tutorial-investigate-cases.md), создав запросы в Log Analytics.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](../../sentinel/tutorial-detect-threats-built-in.md).