---
author: baanders
description: файл include для учебников по Azure Digital Twins — предварительные требования для примера проекта
ms.service: digital-twins
ms.topic: include
ms.date: 1/20/2021
ms.author: baanders
ms.openlocfilehash: 56cda6158b7eeabaf0ce9e71decd0a9aa9f0419c
ms.sourcegitcommit: afb9e9d0b0c7e37166b9d1de6b71cd0e2fb9abf5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/14/2021
ms.locfileid: "103463913"
---
## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником требуется следующее. 

Если у вас еще нет подписки Azure, **создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)** , прежде чем начинать работу.

### <a name="get-required-resources"></a>Необходимые ресурсы

Для этого учебника необходимо **установить на компьютере разработки [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) как минимум версии 16.5**. Если у вас установлена более старая версия, можно открыть на компьютере приложение *Visual Studio Installer* и выполнить инструкции по обновлению установки.

Этот учебник создан на основе примера проекта, написанного на C#. Пример находится здесь: [Комплексные примеры для Azure Digital Twins](/samples/azure-samples/digital-twins-samples/digital-twins-samples) **Получите пример проекта** на компьютер. Для этого перейдите по соответствующей ссылке и нажмите кнопку *Browse code* (Просмотреть код) под заголовком. Вы перейдете в репозиторий GitHub для примеров, которые можно скачать как *ZIP*-файлы. Для этого нажмите кнопку *Код* и выберите элемент *Скачать ZIP-файл*.

:::image type="content" source="../articles/digital-twins/media/includes/download-repo-zip.png" alt-text="Представление репозитория примеров Digital Twins в GitHub. Выбрана кнопка &quot;Код&quot;, в результате чего отображается небольшое диалоговое окно с выделенной кнопкой &quot;Скачать ZIP-файл&quot;." lightbox="../articles/digital-twins/media/includes/download-repo-zip.png":::

На ваш компьютер будет скачана *ZIP*-папка с именем **digital-twins-samples-master.zip**. Распакуйте папку и извлеките файлы.

### <a name="prepare-an-azure-digital-twins-instance"></a>Подготовка экземпляра Azure Digital Twins

[!INCLUDE [Azure Digital Twins: instance prereq](digital-twins-prereq-instance.md)]
