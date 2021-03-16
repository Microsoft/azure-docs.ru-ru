---
title: 'ML Studio (классическая модель): миграция на Машинное обучение Azure — выполнение скрипта R'
description: Rebuild Studio (классическая модель). выполнение модулей R Script для запуска на Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: xiaoharper
ms.author: zhanxia
ms.date: 03/08/2021
ms.openlocfilehash: ac9ad296029451d624345d8b3bb365d881ba9a84
ms.sourcegitcommit: 18a91f7fe1432ee09efafd5bd29a181e038cee05
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103565289"
---
# <a name="migrate-execute-r-script-modules-in-studio-classic"></a>Миграция модулей выполнения скриптов R в Studio (классическая модель)

Из этой статьи вы узнаете, как перестроить модуль **выполнения сценария R** (классическая модель) в машинное обучение Azure.

Дополнительные сведения о миграции из Studio (классической) см. в [статье Обзор миграции](migrate-overview.md).

## <a name="execute-r-script"></a>Выполнение сценария R

Машинное обучение Azure конструктор теперь работает в Linux. Studio (классическая модель) работает в Windows. Из-за изменения платформы необходимо настроить **выполнение сценария R** во время миграции, в противном случае произойдет сбой конвейера.

Чтобы перенести модуль **выполнить сценарий R** из Studio (классическая модель), необходимо заменить `maml.mapInputPort` `maml.mapOutputPort` интерфейсы и стандартными функциями.

В следующей таблице приведена сводка изменений модуля R script.

|Функция|Студия (классическая)|Конструктор Машинного обучения Azure|
|---|---|---|
|Интерфейс скрипта|`maml.mapInputPort` и `maml.mapOutputPort`|Интерфейс функции|
|Платформа|Windows|Linux|
|Доступ в Интернет |Нет|Да|
|Память|14 ГБ|Зависит от расчетного номера SKU|

### <a name="how-to-update-the-r-script-interface"></a>Обновление интерфейса R Script

Ниже приведено содержимое примера модуля **выполнить сценарий R** в студии (классическая модель).
```r
# Map 1-based optional input ports to variables 
dataset1 <- maml.mapInputPort(1) # class: data.frame 
dataset2 <- maml.mapInputPort(2) # class: data.frame 

# Contents of optional Zip port are in ./src/ 
# source("src/yourfile.R"); 
# load("src/yourData.rdata"); 

# Sample operation 
data.set = rbind(dataset1, dataset2); 

 
# You'll see this output in the R Device port. 
# It'll have your stdout, stderr and PNG graphics device(s). 

plot(data.set); 

# Select data.frame to be sent to the output Dataset port 
maml.mapOutputPort("data.set"); 
```

Вот обновленное содержимое в конструкторе. Обратите внимание, что `maml.mapInputPort` и были `maml.mapOutputPort` заменены интерфейсом стандартной функции `azureml_main` . 
```r
azureml_main <- function(dataframe1, dataframe2){ 
    # Use the parameters dataframe1 and dataframe2 directly 
    dataset1 <- dataframe1 
    dataset2 <- dataframe2 

    # Contents of optional Zip port are in ./src/ 
    # source("src/yourfile.R"); 
    # load("src/yourData.rdata"); 

    # Sample operation 
    data.set = rbind(dataset1, dataset2); 


    # You'll see this output in the R Device port. 
    # It'll have your stdout, stderr and PNG graphics device(s). 
    plot(data.set); 

  # Return datasets as a Named List 

  return(list(dataset1=data.set)) 
} 
```
Дополнительные сведения см. в [справочнике по модулю выполнения скрипта R](../algorithm-module-reference/execute-r-script.md)в конструкторе.

### <a name="install-r-packages-from-the-internet"></a>Установка пакетов R из Интернета

Машинное обучение Azure конструктор позволяет устанавливать пакеты непосредственно из CRAN.

Это улучшение по сравнению с Studio (классической). Поскольку среда Studio (классическая) работает в изолированной среде без доступа к Интернету, для установки дополнительных пакетов пришлось передать скрипты в ZIP-пакете. 

Используйте следующий код для установки пакетов CRAN в модуль " **выполнение скрипта R** " конструктора:
```r
  if(!require(zoo)) { 
      install.packages("zoo",repos = "http://cran.us.r-project.org") 
  } 
  library(zoo) 
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как перенести модули выполнения скриптов R в Машинное обучение Azure.

См. Другие статьи в серии миграции Studio (классическая модель):

1. [Обзор миграции](migrate-overview.md).
1. [Перенос набора данных](migrate-register-dataset.md).
1. [Перестройте конвейер обучения Studio (классический)](migrate-rebuild-experiment.md).
1. [Перестройте веб-службу Studio (классическая)](migrate-rebuild-web-service.md).
1. [Интегрируйте веб-службу машинное обучение Azure с клиентскими приложениями](migrate-rebuild-integrate-with-client-app.md).
1. **Миграция модулей выполнения скриптов R**.