<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы включаете Microsoft Graph в приложение. Для этого приложения для вызова Microsoft Graph используется [SDK Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-objc) для Objective C.

## <a name="get-calendar-events-from-outlook"></a>Получить события календаря из Outlook

В этом разделе мы расширим класс, чтобы добавить функцию для получения событий пользователя за текущую неделю, и обновим для `GraphManager` `CalendarViewController` использования этой новой функции.

1. Откройте **GraphManager.h** и добавьте следующий код над `@interface` объявлением.

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSData* _Nullable data,
                                                   NSError* _Nullable error);
    ```

1. Добавьте в объявление следующий `@interface` код.

    ```objc
    - (void) getCalendarViewStartingAt: (NSString*) viewStart
                              endingAt: (NSString*) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion;
    ```

1. Откройте **GraphManager.m** и добавьте в класс следующую `GraphManager` функцию.

    ```objc
    - (void) getCalendarViewStartingAt: (NSString *) viewStart
                              endingAt: (NSString *) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion {
        // Set calendar view start and end parameters
        NSString* viewStartEndString =
        [NSString stringWithFormat:@"startDateTime=%@&endDateTime=%@",
         viewStart,
         viewEnd];

        // GET /me/calendarview
        NSString* eventsUrlString =
        [NSString stringWithFormat:@"%@/me/calendarview?%@&%@&%@&%@",
         MSGraphBaseURL,
         viewStartEndString,
         // Only return these fields in results
         @"$select=subject,organizer,start,end",
         // Sort results by start time
         @"$orderby=start/dateTime",
         // Request at most 25 results
         @"$top=25"];

        NSURL* eventsUrl = [[NSURL alloc] initWithString:eventsUrlString];
        NSMutableURLRequest* eventsRequest = [[NSMutableURLRequest alloc] initWithURL:eventsUrl];

        // Add the Prefer: outlook.timezone header to get start and end times
        // in user's time zone
        NSString* preferHeader =
        [NSString stringWithFormat:@"outlook.timezone=\"%@\"",
         self.graphTimeZone];
        [eventsRequest addValue:preferHeader forHTTPHeaderField:@"Prefer"];

        MSURLSessionDataTask* eventsDataTask =
        [[MSURLSessionDataTask alloc]
         initWithRequest:eventsRequest
         client:self.graphClient
         completion:^(NSData *data, NSURLResponse *response, NSError *error) {
             if (error) {
                 completion(nil, error);
                 return;
             }

             // TEMPORARY
             completion(data, nil);
         }];

        // Execute the request
        [eventsDataTask execute];
    }
    ```

    > [!NOTE]
    > Подумайте, что делает `getCalendarViewStartingAt` код.
    >
    > - Будет вызван URL-адрес `/v1.0/me/calendarview` .
    >   - Параметры `startDateTime` `endDateTime` и параметры запроса определяют начало и конец представления календаря.
    >   - Параметр запроса ограничивает поля, возвращаемые для каждого события, только теми, которые `select` будут фактически использовать представление.
    >   - Параметр `orderby` запроса сортировать результаты по времени начала.
    >   - Параметр `top` запроса запрашивает 25 результатов на страницу.
    >   - Заглавная запись заставляет Microsoft Graph возвращать время начала и окончания каждого события в `Prefer: outlook.timezone` часовом поясе пользователя.

1. Создайте новый класс **Cocoa Touch в** **проекте GraphTutorial** **с именем GraphToIana.** Выберите **NSObject в** **подклассе** поля.
1. Откройте **GraphToIana.h** и замените его содержимое следующим кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.h" id="GraphToIanaSnippet":::

1. Откройте **GraphToIana.m** и замените его содержимое на следующий код.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.m" id="GraphToIanaSnippet":::

    Это простой поиск идентификатора часового пояса IANA на основе имени часового пояса, возвращаемого Microsoft Graph.

1. Откройте **CalendarViewController.m** и замените все его содержимое следующим кодом.

    ```objc
    #import "CalendarViewController.h"
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import "GraphToIana.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>

    @interface CalendarViewController ()

    @property SpinnerViewController* spinner;

    @end

    @implementation CalendarViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        // Calculate the start and end of the current week
        NSString* timeZoneId = [GraphToIana
                                getIanaIdentifierFromGraphIdentifier:
                                [GraphManager.instance graphTimeZone]];

        NSDate* now = [NSDate date];
        NSCalendar* calendar = [NSCalendar calendarWithIdentifier:NSCalendarIdentifierGregorian];
        NSTimeZone* timeZone = [NSTimeZone timeZoneWithName:timeZoneId];
        [calendar setTimeZone:timeZone];

        NSDateComponents* startOfWeekComponents = [calendar
                                                   components:NSCalendarUnitCalendar |
                                                   NSCalendarUnitYearForWeekOfYear |
                                                   NSCalendarUnitWeekOfYear
                                                   fromDate:now];
        NSDate* startOfWeek = [startOfWeekComponents date];
        NSDate* endOfWeek = [calendar dateByAddingUnit:NSCalendarUnitDay
                                                 value:7
                                                toDate:startOfWeek
                                               options:0];

        // Convert start and end to ISO 8601 strings
        NSISO8601DateFormatter* isoFormatter = [[NSISO8601DateFormatter alloc] init];
        NSString* viewStart = [isoFormatter stringFromDate:startOfWeek];
        NSString* viewEnd = [isoFormatter stringFromDate:endOfWeek];

        [GraphManager.instance
         getCalendarViewStartingAt:viewStart
         endingAt:viewEnd
         withCompletionBlock:^(NSData * _Nullable data, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error) {
                     // Show the error
                     UIAlertController* alert = [UIAlertController
                                                 alertControllerWithTitle:@"Error getting events"
                                                 message:error.debugDescription
                                                 preferredStyle:UIAlertControllerStyleAlert];

                     UIAlertAction* okButton = [UIAlertAction
                                                actionWithTitle:@"OK"
                                                style:UIAlertActionStyleDefault
                                                handler:nil];

                     [alert addAction:okButton];
                     [self presentViewController:alert animated:true completion:nil];
                     return;
                 }

                 // TEMPORARY
                 self.calendarJSON.text = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
                 [self.calendarJSON sizeToFit];
             });
         }];
    }

    @end
    ```

1. Запустите приложение, во войти  и коснитесь элемента навигации "Календарь" в меню. В приложении должен отыменить дамп событий в JSON.

## <a name="display-the-results"></a>Отображение результатов

Теперь вы можете заменить дамп JSON на что-то, чтобы отобразить результаты в удобном для пользователя виде. В этом разделе вы измените функцию, чтобы она возвращала строго типные объекты, и измените для отображения событий представление `getCalendarViewStartingAt` `CalendarViewController` таблицы.

### <a name="update-getcalendarviewstartingat"></a>Обновление getCalendarViewStartingAt

1. Откройте **GraphManager.h.** Измените `GetCalendarViewCompletionBlock` определение типа на следующее.

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. Откройте **GraphManager.m**. Замените имеющуюся функцию `getCalendarViewStartingAt` указанным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetCalendarViewSnippet" highlight="42-61":::

### <a name="update-calendarviewcontroller"></a>Обновление CalendarViewController

1. Создайте новый файл **класса Cocoa Touch в** **проекте GraphTutorial** с именем `CalendarTableViewCell` . Выберите **UITableViewCell** в **подклассе** поля.
1. Откройте **CalendarTableViewCell.h** и замените его содержимое следующим кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. Откройте **CalendarTableViewCell.m** и замените его содержимое следующим кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. Создайте новый файл **класса Cocoa Touch в** **проекте GraphTutorial** с именем `CalendarTableViewController` . Выберите **UITableViewController** в **подклассе** поля.
1. Откройте **CalendarTableViewController.h** и замените его содержимое на следующий код.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.h" id="CalendarTableViewControllerSnippet":::

1. Откройте **CalendarTableViewController.m** и замените его содержимое следующим кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.m" id="CalendarTableViewControllerSnippet":::

1. Откройте **main.storyboard и** найдите **сцену календаря.** Удалите представление прокрутки из корневого представления.
1. С помощью **библиотеки добавьте** в **верхнюю** часть представления панели навигации.
1. Дважды щелкните **заголовок** на панели навигации и обновите его до `Calendar` .
1. С помощью **библиотеки добавьте** элемент кнопки **панели** в правую часть панели навигации.
1. Выберите новую кнопку панели, а затем — **инспектор атрибутов.** Измените **изображение** на **"плюс"**.
1. Добавьте представление **контейнера** из **библиотеки в** представление под панели навигации. Измейте представление контейнера, чтобы занять все оставшееся место в представлении.
1. Установите ограничения для панели навигации и представления контейнера следующим образом.

    - **Панель навигации**
        - Добавить ограничение: высота, значение: 44
        - Add constraint: Leading space to Safe Area, value: 0
        - Добавление ограничения: после пробела в безопасную область, значение: 0
        - Add constraint: Top space to Safe Area, value: 0
    - **Представление контейнера**
        - Add constraint: Leading space to Safe Area, value: 0
        - Добавление ограничения: после пробела в безопасную область, значение: 0
        - Добавление ограничения: верхнее место на панели навигации снизу, значение: 0
        - Добавление ограничения: нижнее пространство в безопасной области, значение: 0

1. Найдите второй контроллер представления, добавленный в storyboard при добавлении представления контейнера. Он подключается к сцене **календаря** с помощью встраиваемой segue. Выберите этот контроллер и используйте **identity Inspector,** чтобы изменить **класс** на **CalendarTableViewController.**
1. Удалите **представление** из **контроллера представления таблицы календаря.**
1. Добавьте представление **таблицы** из **библиотеки в** контроллер **представления таблицы календаря.**
1. Выберите представление таблицы, а затем выберите **инспектор атрибутов.** Установите **для прототипа ячеек** **1**.
1. Перетащите нижний край прототипа ячейки, чтобы предоставить вам большую область для работы.
1. Используйте **библиотеку,** чтобы добавить три **метки в** прототип ячейки.
1. Выберите прототип ячейки, а затем выберите **identity Inspector.** Измените **класс** **на CalendarTableViewCell.**
1. Выберите **"Attributes Inspector"** (Инспектор атрибутов) и за **установите для идентификатора (Identifier)** `EventCell` (Идентификатор).
1. Выбрав **EventCell,** выберите **Connections Inspector** и подключите его, а также метки, добавленные в ячейку `durationLabel` в `organizerLabel` `subjectLabel` storyboard.
1. Установите свойства и ограничения для трех меток следующим образом.

    - **Метка темы**
        - Add constraint: Leading space to Content View Leading Margin, value: 0
        - Добавление ограничения: в окн. место в поле "Точка с представлением содержимого", значение: 0
        - Add constraint: Top space to Content View Top Margin, value: 0
    - **Метка организатора**
        - Шрифт: System 12.0
        - Добавить ограничение: высота, значение: 15
        - Add constraint: Leading space to Content View Leading Margin, value: 0
        - Добавление ограничения: в окн. место в поле "Точка с представлением содержимого", значение: 0
        - Add constraint: Top space to Subject Label Bottom, value: Standard
    - **Метка длительности**
        - Шрифт: System 12.0
        - Цвет: темно-серый цвет
        - Добавить ограничение: высота, значение: 15
        - Add constraint: Leading space to Content View Leading Margin, value: 0
        - Добавление ограничения: в окн. место в поле "Точка с представлением содержимого", значение: 0
        - Add constraint: Top space to Organizer Label Bottom, value: Standard
        - Добавление ограничения: нижнее пространство для нижнего поля представления содержимого, значение: 0

1. Выберите **EventCell,** а затем выберите **инспектор размеров.** Включить **автоматически для** **высоты строки.**

    ![Снимок экрана: контроллеры представления таблиц календаря и календаря](images/calendar-table-storyboard.png)

1. Откройте **CalendarViewController.h** и удалите `calendarJSON` свойство.

1. Откройте **CalendarViewController.m и** замените его содержимое на следующий код.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewControllerSnippet":::

1. Запустите приложение, во войти и коснитесь вкладки **"Календарь".** Должен отлиться список событий.

    ![Снимок экрана с таблицей событий](./images/calendar-list.png)
