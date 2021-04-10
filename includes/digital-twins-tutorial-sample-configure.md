---
author: baanders
description: Файл include для руководств Azure Digital Twins. Настройка примера проекта
ms.service: digital-twins
ms.topic: include
ms.date: 5/25/2020
ms.author: baanders
ms.openlocfilehash: 67a2799a93141ad84f458642d8499a58784cc19c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103463908"
---
## <a name="configure-the-sample-project"></a>Настройка примера проекта

Затем настройте пример клиентского приложения, который будет взаимодействовать с вашим экземпляром Azure Digital Twins.

На компьютере перейдите к файлу, скачанному ранее из [*полных примеров для Azure Digital Twins*](/samples/azure-samples/digital-twins-samples/digital-twins-samples), и при необходимости распакуйте его.

В папке перейдите во вложенную папку _AdtSampleApp_. Откройте _**AdtE2ESample.sln**_ в Visual Studio 2019. 

В Visual Studio выберите файл _SampleClientApp > **appsettings.json**_, чтобы открыть его в окне редактирования. Это будет предварительно заданный файл JSON с необходимыми переменными конфигурации для запуска проекта.

В файле измените `instanceUrl` на *URL-адрес hostName* экземпляра Azure Digital Twins (добавив **_https://_** перед *hostName*, как показано ниже).

```json
{
  "instanceUrl": "https://<your-Azure-Digital-Twins-instance-hostName>"
}
```

Сохраните файл и закройте его. 

Затем настройте файл *appsettings.json*, который будет скопирован в выходной каталог при создании *SampleClientApp*. Для этого щелкните правой кнопкой мыши файл *appsettings.json* и выберите **Свойства**. В окне инспектора **Свойства** найдите свойство *Копировать в выходной каталог*. При необходимости измените значение на **Копировать, если новее**.

:::image type="content" source="../articles/digital-twins/media/includes/copy-config.png" alt-text="Фрагмент окна Visual Studio: область Обозревателя решений с выделенным файлом appsettings.json и панель свойств со свойством &quot;Копировать во внешний каталог&quot;, для которого задано значение &quot;Копировать, если новее&quot;" border="false" lightbox="../articles/digital-twins/media/includes/copy-config.png":::

Не закрывайте проект _**AdtE2ESample**_ в Visual Studio, чтобы продолжить его использование в этом руководстве.

[!INCLUDE [Azure Digital Twins: local credentials prereq (outer)](digital-twins-local-credentials-outer.md)]
