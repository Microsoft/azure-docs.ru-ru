---
title: Использование приложений для рендеринга
description: Сведения об использовании приложений для рендеринга в пакетной службе Azure. В этой статье описано, как запустить каждое приложение для рендеринга.
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: ace3fe7aee6b9ffc7226448b455bcfea1f931458
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85964878"
---
# <a name="rendering-applications"></a>Приложения для рендеринга

Чтобы использовать приложения для рендеринга, в пакетной службе создаются задания и задачи. Задача имеет свойство командной строки, в котором указывается командная строка и параметры для выполнения.  Задачи для задания проще всего создать с помощью шаблонов обозревателя пакетной службы, как описано в [этой статье](./batch-rendering-using.md#using-batch-explorer).  Вы можете просматривать эти шаблоны и создавать на их основе новые версии по мере необходимости.

В этой статье описано, как запустить каждое приложение для рендеринга.

## <a name="rendering-with-autodesk-3ds-max"></a>Рендеринг в Autodesk 3ds Max

### <a name="renderer-support"></a>Поддержка рендеринга

Помимо средств для рендеринга, встроенных в 3ds Max, в файле сцены 3ds Max вы можете указать еще несколько таких средств, включенных в образы виртуальных машин для рендеринга:

* Autodesk Arnold.
* V-Ray от Chaos Group

### <a name="task-command-line"></a>Командная строка задачи

Вызовите приложение `3dsmaxcmdio.exe` для выполнения рендеринга в узле пула в командной строке.  Это приложение размещается в том пути, откуда выполняется задача. Приложение `3dsmaxcmdio.exe` поддерживает те же параметры, что и приложение `3dsmaxcmd.exe`, как описано в [справочной документации по 3ds Max](https://help.autodesk.com/view/3DSMAX/2018/ENU/) (в разделе, посвященному рендерингу из командной строки).

Пример:

```
3dsmaxcmdio.exe -v:5 -rfw:0 -start:{0} -end:{0} -bitmapPath:"%AZ_BATCH_JOB_PREP_WORKING_DIR%\sceneassets\images" -outputName:dragon.jpg -w:1280 -h:720 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scenes\dragon.max"
```

Примечания.

* Уделите внимание тому, чтобы все файлы ресурсов могли быть обнаружены.  Убедитесь, что все пути указаны правильно и предоставляются в относительной форме, используя окно **отслеживания ресурсов**, или примените параметр `-bitmapPath` в командной строке.
* Проверьте, существуют ли проблемы с рендерингом (например, невозможность найти ресурсы), проверив файл `stdout.txt`, который 3ds Max создает при выполнении задачи.

### <a name="batch-explorer-templates"></a>Шаблоны Batch Explorer

Шаблоны пула и задания вы можете найти в **коллекции** Batch Explorer.  Исходные файлы шаблона доступны в [репозитории данных Batch Explorer на сайте GitHub](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/3dsmax).

## <a name="rendering-with-autodesk-maya"></a>Рендеринг в Autodesk Maya

### <a name="renderer-support"></a>Поддержка рендеринга

Помимо средств для рендеринга, встроенных в Maya, в файле сцены 3ds Max вы можете указать еще несколько таких средств, включенных в образы виртуальных машин для рендеринга:

* Autodesk Arnold.
* V-Ray от Chaos Group

### <a name="task-command-line"></a>Командная строка задачи

В командной строке задачи используется средство для рендеринга `renderer.exe`. Средство для рендеринга для командной строки описан в [справке по Maya](https://help.autodesk.com/view/MAYAUL/2018/ENU/?guid=GUID-EB558BC0-5C2B-439C-9B00-F97BCB9688E4).

В следующем примере задача подготовки задания используется для копирования файлов сцены и ресурсов в рабочий каталог задания подготовки, а в выходной папке сохраняется отображенное изображение, созданное в результате рендеринга кадра 10.

```
render -renderer sw -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

Для рендеринга в V-Ray файл сцены Maya обычно указывает V-Ray в качестве средства для рендеринга.  Это можно указать и в командной строке:

```
render -renderer vray -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

Для рендеринга в Arnold файл сцены Maya обычно указывает Arnold в качестве средства для рендеринга.  Это можно указать и в командной строке:

```
render -renderer arnold -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

### <a name="batch-explorer-templates"></a>Шаблоны Batch Explorer

Шаблоны пула и задания вы можете найти в **коллекции** Batch Explorer.  Исходные файлы шаблона доступны в [репозитории данных Batch Explorer на сайте GitHub](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/maya).

## <a name="next-steps"></a>Дальнейшие действия

Используйте шаблоны пула и задания из [репозитория данных в GitHub](https://github.com/Azure/BatchExplorer-data/tree/master/ncj) с помощью Batch Explorer.  При желании вы можете создать новые шаблоны или изменить любой из стандартных шаблонов.
