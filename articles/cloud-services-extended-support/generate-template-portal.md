---
title: Создание шаблона ARM для облачных служб (Расширенная поддержка) с помощью портал Azure
description: Создание и скачивание шаблона ARM и файла параметров для облачных служб (Расширенная поддержка) с помощью портал Azure
ms.topic: how-to
ms.service: cloud-services-extended-support
author: surbhijain
ms.author: surbhijain
ms.reviewer: gachandw
ms.date: 03/07/2021
ms.custom: ''
ms.openlocfilehash: 9d40bbd7e08d8d3869166827a22f3f08536532bb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104590710"
---
# <a name="generate-arm-template-for-cloud-services-extended-support-using-the-azure-portal"></a>Создание шаблона ARM для облачных служб (Расширенная поддержка) с помощью портал Azure

В этой статье объясняется, как скачать шаблон ARM и файл параметров из [портал Azure](https://portal.azure.com) после развертывания облачной службы (Расширенная поддержка). Шаблон ARM и файл параметров можно использовать в будущих развертываниях для обновления или обновления облачной службы (Расширенная поддержка).

## <a name="get-arm-template-via-portal"></a>Получение шаблона ARM через портал

  1. Перейдите к группе ресурсов и выберите развертывания.
  :::image type="content" source="media/generate-template-portal-1.png" alt-text="На рисунке показано, как выбрать развертывания в группе ресурсов на портал Azure.":::
  
  2. Выберите облачную службу (расширенную поддержку) и щелкните шаблон.
  :::image type="content" source="media/generate-template-portal-2.png" alt-text="На рисунке показан выбор шаблона в разделе облачная служба (Расширенная поддержка) на портал Azure.":::
  
  3. Скачайте шаблон и файлы параметров. Их можно использовать для будущих развертываний с помощью PowerShell.
  :::image type="content" source="media/generate-template-portal-3.png" alt-text="На рисунке показана загрузка файла шаблона на портал Azure.":::
  
## <a name="next-steps"></a>Дальнейшие действия 
- Ознакомьтесь с [часто задаваемыми вопросами об Облачных службах (расширенная поддержка)](faq.md).
- Развертывание облачной службы (Расширенная поддержка) с помощью [портал Azure](deploy-portal.md)
  
