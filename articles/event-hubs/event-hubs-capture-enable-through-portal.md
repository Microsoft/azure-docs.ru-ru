---
title: 'Сбор событий потоковой передачи с помощью портала Azure: Центры событий'
description: В этой статье объясняется, как на портале Azure включить сбор событий потоковой передачи из Центров событий Azure.
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 8c89a6e56c2d0bc7b7ff09d5fd63737009239eb9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100653957"
---
# <a name="enable-capturing-of-events-streaming-through-azure-event-hubs"></a>Включение сбора событий, передающихся потоком из Центров событий Azure

[Функция сбора в Центрах событий][capture-overview] Azure позволяет автоматически доставлять данные потоковой передачи из Центров событий в выбранную учетную запись [Хранилища BLOB-объектов Azure](https://azure.microsoft.com/services/storage/blobs/) либо [Azure Data Lake Storage 1-го или 2-го поколения](https://azure.microsoft.com/services/data-lake-store/).

Запись можно настроить при создании концентратора событий с помощью [портала Azure](https://portal.azure.com). Данные можно записывать в контейнер [Хранилища BLOB-объектов Azure](https://azure.microsoft.com/services/storage/blobs/) либо учетную запись [Azure Data Lake Storage 1-го или 2-го поколения](https://azure.microsoft.com/services/data-lake-store/).

Подробнее см. в статье [Общие сведения о функции "Сбор" в Центрах событий Azure][capture-overview].

> [!IMPORTANT]
> Учетная запись целевой службы хранилища (Служба хранилища Azure или Azure Data Lake Storage) должна находиться в той же подписке, что и концентратор событий.

## <a name="capture-data-to-azure-storage"></a>Сбор данных в службу хранилища Azure

Запись можно включить при создании концентратора событий. Для этого на экране **Создание концентратора событий** на портале нажмите кнопку **Вкл**. Затем укажите учетную запись хранения и контейнер, выбрав **Служба хранилища Azure** в поле **поставщика записи**. Так как для функции "Сбор" в Центрах событий и хранилища используется проверка подлинности со взаимодействием между службами, указывать строку подключения к хранилищу не нужно. Средство выбора ресурсов автоматически выбирает универсальный код ресурса (URI) для вашей учетной записи хранения. При использовании Azure Resource Manager этот универсальный код ресурса необходимо явным образом указать как строку.

Продолжительность времени окна по умолчанию составляет 5 минут. Минимальное значение равно 1, а максимальное — 15. Диапазон **окна размера** составляет 10–500 МБ.

![Временное окно для сбора][1]

> [!NOTE]
> Вы можете включить или отключить создание пустых файлов за время окна сбора данных, когда события отсутствуют. 

## <a name="capture-data-to-azure-data-lake-storage-gen-2"></a>Сбор данных в Azure Data Lake Storage 2-го поколения 

1. Сведения о создании учетной записи хранения Azure см. в [этой статье](../storage/common/storage-account-create.md?tabs=azure-portal#create-a-storage-account). Установите для параметра **Иерархическое пространство имен** значение **Включено** на вкладке **Дополнительно**, чтобы определить учетную запись Azure Data Lake Storage 2-го поколения.
2. При создании концентратора событий: 

    1. Выберите значение **Включить** для параметра **Сбор**. 
    2. Выберите вариант **Служба хранилища Azure** в качестве поставщика сбора. Вариант **Azure Data Lake Store**, отображаемый для параметра **Поставщик записи**, используется для Azure Data Lake Storage 1-го поколения. Чтобы использовать Azure Data Lake Storage 2-го поколения, выберите вариант **Служба хранилища Azure**.
    2. Нажмите кнопку **Выбрать контейнер**. 

        ![Включение сбора в Data Lake Storage 2-го поколения](./media/event-hubs-capture-enable-through-portal/data-lake-storage-gen2.png)
3. Выберите учетную запись **Azure Data Lake Storage 2-го поколения** в списке. 

    ![Выбор Data Lake Storage 2-го поколения](./media/event-hubs-capture-enable-through-portal/select-data-lake-storage-gen2.png)
4. Выберите **контейнер** (файловая система в Data Lake Storage 2-го поколения).

    ![Выбор файловой системы в хранилище](./media/event-hubs-capture-enable-through-portal/select-file-system-data-lake-storage.png)
5. На странице **Создание Центра событий** нажмите кнопку **Создать**. 

    ![Выбор кнопки "Создать"](./media/event-hubs-capture-enable-through-portal/create-event-hub-data-lake-storage.png)

    > [!NOTE]
    > Контейнер, созданный в Azure Data Lake Storage 2-го поколения с помощью этого пользовательского интерфейса, отображается в разделе **Файловые системы** в **Обозревателе службы хранилища**. Точно так же файловая система, созданная в учетной записи Data Lake Storage 2-го поколения, отображается как контейнер в этом пользовательском интерфейсе. 


## <a name="capture-data-to-azure-data-lake-storage-gen-1"></a>Сбор данных в Azure Data Lake Storage 1-го поколения 

Для записи данных в Azure Data Lake Storage 1-го поколения создайте учетную запись Data Lake Storage 1-го поколения и концентратор событий.

### <a name="create-an-azure-data-lake-storage-gen-1-account-and-folders"></a>Создание учетной записи Azure Data Lake Storage 1-го поколения и папок

1. Создайте учетную запись Data Lake Storage, следуя инструкциям в статье [Начало работы с Azure Data Lake Storage 1-го поколения с помощью портала Azure](../data-lake-store/data-lake-store-get-started-portal.md).
2. Следуйте инструкциям из раздела [Назначение разрешений для Центров событий](../data-lake-store/data-lake-store-archive-eventhub-capture.md#assign-permissions-to-event-hubs), чтобы создать папку в учетной записи Data Lake Storage 1-го поколения, в которой нужно собирать данные из Центров событий. Затем назначьте разрешения для Центров событий, чтобы разрешить запись данных в вашу учетную запись Data Lake Storage 1-го поколения.  


### <a name="create-an-event-hub"></a>Создание концентратора событий

1. Концентратор событий должен находиться в той же подписке Azure, что и созданная учетная запись Azure Data Lake Storage 1-го поколения. Создайте концентратор событий, нажав кнопку **Вкл.** в разделе **Запись** на странице портала **Создание концентратора событий**. 
2. На странице портала **Создание концентратора событий** в поле **поставщика записи** выберите **Azure Data Lake Store**.
3. В поле **Выберите Store** рядом со списком **Data Lake Store** укажите учетную запись Data Lake Storage 1-го поколения, созданную ранее, а в поле **Путь к Data Lake** укажите путь к созданной папке данных.

    ![Выбор учетной записи Data Lake Storage][3]


## <a name="add-or-configure-capture-on-an-existing-event-hub"></a>Добавление или настройка функции записи для существующего концентратора событий

Функцию "Сбор" можно настроить в существующих концентраторах событий, расположенных в пространствах имен Центров событий. Чтобы включить запись в существующем концентраторе событий или изменить параметры записи, щелкните пространство имен. Появится экран обзора. Затем щелкните концентратор событий, для которого необходимо включить запись или изменить ее параметры. Наконец, на открывшейся странице слева щелкните параметр **записи** и измените параметры записи, как показано на следующих снимках экрана.

### <a name="azure-blob-storage"></a>хранилище BLOB-объектов Azure

![Настройка хранилища BLOB-объектов Azure][2]

### <a name="azure-data-lake-storage-gen-2"></a>Azure Data Lake Storage 2-го поколения

![Настройка Azure Data Lake Storage 2-го поколения](./media/event-hubs-capture-enable-through-portal/configure-data-lake-storage-gen2.png)

### <a name="azure-data-lake-storage-gen-1"></a>Azure Data Lake Storage 1-го поколения 

![Настройка Azure Data Lake Storage][4]

[1]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture1.png
[2]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture2.png
[3]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture3.png
[4]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture4.png

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о функции "Сбор" в Центрах событий см. в [этой статье][capture-overview].
- Функцию "Сбор" в Центрах событий можно также настроить с помощью шаблонов Azure Resource Manager. См. дополнительные сведения [о включении записи с помощью шаблона Azure Resource Manager](event-hubs-resource-manager-namespace-event-hub-enable-capture.md).
- [Узнайте, как создать подписку на Сетку событий Azure с пространством имен Центров событий в качестве источника](store-captured-data-data-warehouse.md)
- [Начало работы с Azure Data Lake Store с помощью портала Azure](../data-lake-store/data-lake-store-get-started-portal.md)

[capture-overview]: event-hubs-capture-overview.md
