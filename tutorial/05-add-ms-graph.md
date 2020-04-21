<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы добавите Microsoft Graph в приложение. Для этого приложения вы будете использовать [пакет SDK Microsoft Graph для задания](https://github.com/microsoftgraph/msgraph-sdk-objc) с, чтобы совершать вызовы в Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Получение событий календаря из Outlook

В этом разделе описывается расширение `GraphManager` класса для добавления функции, позволяющей получить события и обновление `CalendarViewController` , чтобы использовать эти новые функции.

1. Откройте **графманажер. h** и добавьте приведенный ниже код перед `@interface` объявлением.

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. Добавьте следующий код в `@interface` объявление.

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. Откройте **графманажер. m** и добавьте указанную ниже функцию в `GraphManager` класс.

    ```objc
    - (void) getEventsWithCompletionBlock:(GetEventsCompletionBlock)completionBlock {
        // GET /me/events?$select='subject,organizer,start,end'$orderby=createdDateTime DESC
        NSString* eventsUrlString =
        [NSString stringWithFormat:@"%@/me/events?%@&%@",
         MSGraphBaseURL,
         // Only return these fields in results
         @"$select=subject,organizer,start,end",
         // Sort results by when they were created, newest first
         @"$orderby=createdDateTime+DESC"];

        NSURL* eventsUrl = [[NSURL alloc] initWithString:eventsUrlString];
        NSMutableURLRequest* eventsRequest = [[NSMutableURLRequest alloc] initWithURL:eventsUrl];

        MSURLSessionDataTask* eventsDataTask =
        [[MSURLSessionDataTask alloc]
         initWithRequest:eventsRequest
         client:self.graphClient
         completion:^(NSData *data, NSURLResponse *response, NSError *error) {
             if (error) {
                 completionBlock(nil, error);
                 return;
             }

             // TEMPORARY
             completionBlock(data, nil);
         }];

        // Execute the request
        [eventsDataTask execute];
    }
    ```

    > [!NOTE]
    > Рассмотрите, какие действия `getEventsWithCompletionBlock` выполняет код.
    >
    > - URL-адрес, который будет вызываться — это `/v1.0/me/events`.
    > - Параметр `select` запроса позволяет ограничить поля, возвращаемые для каждого события, только теми, которые будут использоваться приложением.
    > - Параметр `orderby` запроса сортирует результаты по дате и времени создания, начиная с самого последнего элемента.

1. Откройте **календарвиевконтроллер. m** и замените все его содержимое приведенным ниже кодом.

    ```objc
    #import "CalendarViewController.h"
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
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

        [GraphManager.instance
         getEventsWithCompletionBlock:^(NSData * _Nullable data, NSError * _Nullable error) {
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

1. Запустите приложение и выполните вход в систему, а затем нажмите элемент навигации в **календаре** в меню. В приложении должен появиться дамп JSON событий.

## <a name="display-the-results"></a>Отображение результатов

Теперь вы можете заменить дамп JSON на какой-то способ отобразить результаты в удобном для пользователя виде. В этом разделе будет изменена `getEventsWithCompletionBlock` функция, возвращающая строго типизированные объекты, и изменено `CalendarViewController` , чтобы использовать представление таблицы для отображения событий.

### <a name="update-getevents"></a>Обновление событий

1. Откройте **графманажер. h**. Измените определение `GetEventsCompletionBlock` типа на следующее:

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. Откройте **графманажер. m**. Замените `completionBlock(data, nil);` строку в `getEventsWithCompletionBlock` функции указанным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetEventsSnippet" highlight="24-43":::

### <a name="update-calendarviewcontroller"></a>Обновление Календарвиевконтроллер

1. Создайте новый файл **класса Touch Cocoa** в проекте **графтуториал** с именем `CalendarTableViewCell`. Выберите **уитаблевиевцелл** в **подклассе** поля.
1. Откройте **календартаблевиевцелл. h** и замените его содержимое приведенным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. Откройте **календартаблевиевцелл. m** и замените его содержимое приведенным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. Откройте файл **Main. Storyboard** и перейдите к **сцене календаря**. Выберите **представление** в **сцене календаря** и удалите его.

    ![Снимок экрана с представлением в сцене календаря](./images/view-in-calendar-scene.png)

1. Добавьте **представление таблицы** из **библиотеки** в **сцену в календаре**.
1. Выберите представление таблицы, а затем выберите **инспектор атрибутов**. Задайте для **ячеек прототипа** значение **1**.
1. Используйте **библиотеку** , чтобы добавить три **метки** к ячейке прототипа.
1. Выберите ячейку прототип, а затем выберите **инспектор удостоверений**. Замените **Class** на **календартаблевиевцелл**.
1. Выберите **инспектор атрибутов** и **идентификатор** набора `EventCell`.
1. Выбрав **евентцелл** , выберите **инспектор подключений** и подключение `durationLabel`, `organizerLabel`а `subjectLabel` также метки, добавленные в ячейку в раскадровке.
1. Задайте свойства и ограничения для этих трех меток, как показано ниже.

    - **Метка темы**
        - Добавить ограничение: начальное пространство в представлении содержимого начальное поле, значение: 0
        - Add Constraint: замыкающий пробел для конечного поля представления содержимого, значение: 0
        - Добавить ограничение: Верхняя область для представления контента верхнее поле, значение: 0
    - **Метка "Организатор"**
        - Шрифт: System 12,0
        - Добавить ограничение: начальное пространство в представлении содержимого начальное поле, значение: 0
        - Add Constraint: замыкающий пробел для конечного поля представления содержимого, значение: 0
        - Добавление ограничения: Верхняя область для подписи темы внизу, значение: Standard
    - **Метка длительности**
        - Шрифт: System 12,0
        - Цвет: темно-серый цвет
        - Добавить ограничение: начальное пространство в представлении содержимого начальное поле, значение: 0
        - Add Constraint: замыкающий пробел для конечного поля представления содержимого, значение: 0
        - Добавить ограничение: Верхняя область для метки организатора внизу, значение: Standard
        - Добавить ограничение: нижнее пространство для представления содержимого нижнее поле, значение: 8

    ![Снимок экрана с макетом ячеек прототипа](./images/prototype-cell-layout.png)

1. Откройте **календарвиевконтроллер. h** и удалите `calendarJSON` свойство.
1. Измените `@interface` объявление на следующее.

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. Откройте **календарвиевконтроллер. m** и замените его содержимое приведенным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewSnippet":::

1. Запустите приложение и войдите в систему, а затем коснитесь вкладки **Календарь** . Вы должны увидеть список событий.

    ![Снимок экрана с таблицей событий](./images/calendar-list.png)
