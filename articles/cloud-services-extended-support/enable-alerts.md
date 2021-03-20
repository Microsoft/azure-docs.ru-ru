---
title: Включение мониторинга в облачных службах (Расширенная поддержка) с помощью портал Azure
description: Включение мониторинга для экземпляров облачных служб (Расширенная поддержка) с помощью портал Azure
ms.topic: how-to
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: 4591253e1a305632b7f41afe692f297d71366382
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98744855"
---
# <a name="enable-monitoring-for-cloud-services-extended-support-using-the-azure-portal"></a>Включение мониторинга для облачных служб (Расширенная поддержка) с помощью портал Azure

В этой статье объясняется, как включить оповещения для существующих развертываний облачной службы (Расширенная поддержка). 

## <a name="add-monitoring-rules"></a>Добавление правил мониторинга
1. Войдите на [портал Azure](https://portal.azure.com). 
2. Выберите Развертывание облачной службы (Расширенная поддержка), для которого необходимо включить оповещения. 
3. Выберите колонку **оповещения** . 

    :::image type="content" source="media/enable-alerts-1.png" alt-text="На рисунке показано, как выбрать вкладку оповещения в портал Azure.":::

4. Выберите **новый значок оповещения** .
     :::image type="content" source="media/enable-alerts-2.png" alt-text="На рисунке показано, как выбрать параметр Добавить новое оповещение.":::

5. Введите требуемые условия и требуемые действия в зависимости от метрик, которые вы хотите отслеживать. Правила можно определить на основе отдельных метрик или журнала действий. 

     :::image type="content" source="media/enable-alerts-3.png" alt-text="На рисунке показано, куда добавлять условия в оповещения.":::

     :::image type="content" source="media/enable-alerts-4.png" alt-text="На рисунке показана настройка логики сигнала.":::

     :::image type="content" source="media/enable-alerts-5.png" alt-text="На рисунке показано, как настроить логику группы действий.":::

6. После завершения настройки оповещений сохраните изменения и на основе настроенных метрик вы начнете заполнять колонку **оповещения** с течением времени.

## <a name="next-steps"></a>Дальнейшие действия 
- Ознакомьтесь с [часто задаваемыми вопросами об Облачных службах (расширенная поддержка)](faq.md).
- Разверните Облачную службу (расширенная поддержка) с помощью [портала Azure](deploy-portal.md), [PowerShell](deploy-powershell.md), [шаблона](deploy-template.md) или [Visual Studio](deploy-visual-studio.md).