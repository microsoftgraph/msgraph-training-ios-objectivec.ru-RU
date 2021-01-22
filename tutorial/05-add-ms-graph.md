<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="71b85-101">В этом упражнении вы включаете Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="71b85-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="71b85-102">Для этого приложения для вызова Microsoft Graph используется [SDK Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-objc) для Objective C.</span><span class="sxs-lookup"><span data-stu-id="71b85-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="71b85-103">Получить события календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="71b85-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="71b85-104">В этом разделе мы расширим класс, чтобы добавить функцию для получения событий пользователя за текущую неделю, и обновим для `GraphManager` `CalendarViewController` использования этой новой функции.</span><span class="sxs-lookup"><span data-stu-id="71b85-104">In this section you will extend the `GraphManager` class to add a function to get the user's events for the current week and update `CalendarViewController` to use this new function.</span></span>

1. <span data-ttu-id="71b85-105">Откройте **GraphManager.h** и добавьте следующий код над `@interface` объявлением.</span><span class="sxs-lookup"><span data-stu-id="71b85-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSData* _Nullable data,
                                                   NSError* _Nullable error);
    ```

1. <span data-ttu-id="71b85-106">Добавьте в объявление следующий `@interface` код.</span><span class="sxs-lookup"><span data-stu-id="71b85-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getCalendarViewStartingAt: (NSString*) viewStart
                              endingAt: (NSString*) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion;
    ```

1. <span data-ttu-id="71b85-107">Откройте **GraphManager.m** и добавьте в класс следующую `GraphManager` функцию.</span><span class="sxs-lookup"><span data-stu-id="71b85-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

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
    > <span data-ttu-id="71b85-108">Подумайте, что делает `getCalendarViewStartingAt` код.</span><span class="sxs-lookup"><span data-stu-id="71b85-108">Consider what the code in `getCalendarViewStartingAt` is doing.</span></span>
    >
    > - <span data-ttu-id="71b85-109">Будет вызван URL-адрес `/v1.0/me/calendarview` .</span><span class="sxs-lookup"><span data-stu-id="71b85-109">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
    >   - <span data-ttu-id="71b85-110">Параметры `startDateTime` `endDateTime` и параметры запроса определяют начало и конец представления календаря.</span><span class="sxs-lookup"><span data-stu-id="71b85-110">The `startDateTime` and `endDateTime` query parameters define the start and end of the calendar view.</span></span>
    >   - <span data-ttu-id="71b85-111">Параметр запроса ограничивает поля, возвращаемые для каждого события, только теми, которые `select` будут фактически использовать представление.</span><span class="sxs-lookup"><span data-stu-id="71b85-111">The `select` query parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="71b85-112">Параметр `orderby` запроса сортировать результаты по времени начала.</span><span class="sxs-lookup"><span data-stu-id="71b85-112">The `orderby` query parameter sorts the results by start time.</span></span>
    >   - <span data-ttu-id="71b85-113">Параметр `top` запроса запрашивает 25 результатов на страницу.</span><span class="sxs-lookup"><span data-stu-id="71b85-113">The `top` query parameter requests 25 results per page.</span></span>
    >   - <span data-ttu-id="71b85-114">Заглавная запись заставляет Microsoft Graph возвращать время начала и окончания каждого события в `Prefer: outlook.timezone` часовом поясе пользователя.</span><span class="sxs-lookup"><span data-stu-id="71b85-114">the `Prefer: outlook.timezone` header causes the Microsoft Graph to return the start and end times of each event in the user's time zone.</span></span>

1. <span data-ttu-id="71b85-115">Создайте новый класс **Cocoa Touch в** **проекте GraphTutorial** **с именем GraphToIana.**</span><span class="sxs-lookup"><span data-stu-id="71b85-115">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphToIana**.</span></span> <span data-ttu-id="71b85-116">Выберите **NSObject в** **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="71b85-116">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="71b85-117">Откройте **GraphToIana.h** и замените его содержимое следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="71b85-117">Open **GraphToIana.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.h" id="GraphToIanaSnippet":::

1. <span data-ttu-id="71b85-118">Откройте **GraphToIana.m** и замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="71b85-118">Open **GraphToIana.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.m" id="GraphToIanaSnippet":::

    <span data-ttu-id="71b85-119">Это простой поиск идентификатора часового пояса IANA на основе имени часового пояса, возвращаемого Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="71b85-119">This does a simple lookup to find an IANA time zone identifier based on the time zone name returned by Microsoft Graph.</span></span>

1. <span data-ttu-id="71b85-120">Откройте **CalendarViewController.m** и замените все его содержимое следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="71b85-120">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

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

1. <span data-ttu-id="71b85-121">Запустите приложение, во войти  и коснитесь элемента навигации "Календарь" в меню.</span><span class="sxs-lookup"><span data-stu-id="71b85-121">Run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="71b85-122">В приложении должен отыменить дамп событий в JSON.</span><span class="sxs-lookup"><span data-stu-id="71b85-122">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="71b85-123">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="71b85-123">Display the results</span></span>

<span data-ttu-id="71b85-124">Теперь вы можете заменить дамп JSON на что-то, чтобы отобразить результаты в удобном для пользователя виде.</span><span class="sxs-lookup"><span data-stu-id="71b85-124">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="71b85-125">В этом разделе вы измените функцию, чтобы она возвращала строго типные объекты, и измените для отображения событий представление `getCalendarViewStartingAt` `CalendarViewController` таблицы.</span><span class="sxs-lookup"><span data-stu-id="71b85-125">In this section, you will modify the `getCalendarViewStartingAt` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getcalendarviewstartingat"></a><span data-ttu-id="71b85-126">Обновление getCalendarViewStartingAt</span><span class="sxs-lookup"><span data-stu-id="71b85-126">Update getCalendarViewStartingAt</span></span>

1. <span data-ttu-id="71b85-127">Откройте **GraphManager.h.**</span><span class="sxs-lookup"><span data-stu-id="71b85-127">Open **GraphManager.h**.</span></span> <span data-ttu-id="71b85-128">Измените `GetCalendarViewCompletionBlock` определение типа на следующее.</span><span class="sxs-lookup"><span data-stu-id="71b85-128">Change the `GetCalendarViewCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="71b85-129">Откройте **GraphManager.m**.</span><span class="sxs-lookup"><span data-stu-id="71b85-129">Open **GraphManager.m**.</span></span> <span data-ttu-id="71b85-130">Замените имеющуюся функцию `getCalendarViewStartingAt` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="71b85-130">Replace the existing `getCalendarViewStartingAt` function with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetCalendarViewSnippet" highlight="42-61":::

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="71b85-131">Обновление CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="71b85-131">Update CalendarViewController</span></span>

1. <span data-ttu-id="71b85-132">Создайте новый файл **класса Cocoa Touch в** **проекте GraphTutorial** с именем `CalendarTableViewCell` .</span><span class="sxs-lookup"><span data-stu-id="71b85-132">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="71b85-133">Выберите **UITableViewCell** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="71b85-133">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="71b85-134">Откройте **CalendarTableViewCell.h** и замените его содержимое следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="71b85-134">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="71b85-135">Откройте **CalendarTableViewCell.m** и замените его содержимое следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="71b85-135">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="71b85-136">Создайте новый файл **класса Cocoa Touch в** **проекте GraphTutorial** с именем `CalendarTableViewController` .</span><span class="sxs-lookup"><span data-stu-id="71b85-136">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewController`.</span></span> <span data-ttu-id="71b85-137">Выберите **UITableViewController** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="71b85-137">Choose **UITableViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="71b85-138">Откройте **CalendarTableViewController.h** и замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="71b85-138">Open **CalendarTableViewController.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.h" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="71b85-139">Откройте **CalendarTableViewController.m** и замените его содержимое следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="71b85-139">Open **CalendarTableViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.m" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="71b85-140">Откройте **main.storyboard и** найдите **сцену календаря.**</span><span class="sxs-lookup"><span data-stu-id="71b85-140">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="71b85-141">Удалите представление прокрутки из корневого представления.</span><span class="sxs-lookup"><span data-stu-id="71b85-141">Delete the scroll view from the root view.</span></span>
1. <span data-ttu-id="71b85-142">С помощью **библиотеки добавьте** в **верхнюю** часть представления панели навигации.</span><span class="sxs-lookup"><span data-stu-id="71b85-142">Using the **Library**, add a **Navigation Bar** to the top of the view.</span></span>
1. <span data-ttu-id="71b85-143">Дважды щелкните **заголовок** на панели навигации и обновите его до `Calendar` .</span><span class="sxs-lookup"><span data-stu-id="71b85-143">Double-click the **Title** in the navigation bar and update it to `Calendar`.</span></span>
1. <span data-ttu-id="71b85-144">С помощью **библиотеки добавьте** элемент кнопки **панели** в правую часть панели навигации.</span><span class="sxs-lookup"><span data-stu-id="71b85-144">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="71b85-145">Выберите новую кнопку панели, а затем — **инспектор атрибутов.**</span><span class="sxs-lookup"><span data-stu-id="71b85-145">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="71b85-146">Измените **изображение** на **"плюс"**.</span><span class="sxs-lookup"><span data-stu-id="71b85-146">Change **Image** to **plus**.</span></span>
1. <span data-ttu-id="71b85-147">Добавьте представление **контейнера** из **библиотеки в** представление под панели навигации.</span><span class="sxs-lookup"><span data-stu-id="71b85-147">Add a **Container View** from the **Library** to the view under the navigation bar.</span></span> <span data-ttu-id="71b85-148">Измейте представление контейнера, чтобы занять все оставшееся место в представлении.</span><span class="sxs-lookup"><span data-stu-id="71b85-148">Resize the container view to take all of the remaining space in the view.</span></span>
1. <span data-ttu-id="71b85-149">Установите ограничения для панели навигации и представления контейнера следующим образом.</span><span class="sxs-lookup"><span data-stu-id="71b85-149">Set constraints on the navigation bar and container view as follows.</span></span>

    - <span data-ttu-id="71b85-150">**Панель навигации**</span><span class="sxs-lookup"><span data-stu-id="71b85-150">**Navigation Bar**</span></span>
        - <span data-ttu-id="71b85-151">Добавить ограничение: высота, значение: 44</span><span class="sxs-lookup"><span data-stu-id="71b85-151">Add constraint: Height, value: 44</span></span>
        - <span data-ttu-id="71b85-152">Add constraint: Leading space to Safe Area, value: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-152">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="71b85-153">Добавление ограничения: после пробела в безопасную область, значение: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-153">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="71b85-154">Add constraint: Top space to Safe Area, value: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-154">Add constraint: Top space to Safe Area, value: 0</span></span>
    - <span data-ttu-id="71b85-155">**Представление контейнера**</span><span class="sxs-lookup"><span data-stu-id="71b85-155">**Container View**</span></span>
        - <span data-ttu-id="71b85-156">Add constraint: Leading space to Safe Area, value: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-156">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="71b85-157">Добавление ограничения: после пробела в безопасную область, значение: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-157">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="71b85-158">Добавление ограничения: верхнее место на панели навигации снизу, значение: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-158">Add constraint: Top space to Navigation Bar Bottom, value: 0</span></span>
        - <span data-ttu-id="71b85-159">Добавление ограничения: нижнее пространство в безопасной области, значение: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-159">Add constraint: Bottom space to Safe Area, value: 0</span></span>

1. <span data-ttu-id="71b85-160">Найдите второй контроллер представления, добавленный в storyboard при добавлении представления контейнера.</span><span class="sxs-lookup"><span data-stu-id="71b85-160">Locate the second view controller added to the storyboard when you added the container view.</span></span> <span data-ttu-id="71b85-161">Он подключается к сцене **календаря** с помощью встраиваемой segue.</span><span class="sxs-lookup"><span data-stu-id="71b85-161">It is connected to the **Calendar Scene** by an embed segue.</span></span> <span data-ttu-id="71b85-162">Выберите этот контроллер и используйте **identity Inspector,** чтобы изменить **класс** на **CalendarTableViewController.**</span><span class="sxs-lookup"><span data-stu-id="71b85-162">Select this controller and use the **Identity Inspector** to change **Class** to **CalendarTableViewController**.</span></span>
1. <span data-ttu-id="71b85-163">Удалите **представление** из **контроллера представления таблицы календаря.**</span><span class="sxs-lookup"><span data-stu-id="71b85-163">Delete the **View** from the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="71b85-164">Добавьте представление **таблицы** из **библиотеки в** контроллер **представления таблицы календаря.**</span><span class="sxs-lookup"><span data-stu-id="71b85-164">Add a **Table View** from the **Library** to the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="71b85-165">Выберите представление таблицы, а затем выберите **инспектор атрибутов.**</span><span class="sxs-lookup"><span data-stu-id="71b85-165">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="71b85-166">Установите **для прототипа ячеек** **1**.</span><span class="sxs-lookup"><span data-stu-id="71b85-166">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="71b85-167">Перетащите нижний край прототипа ячейки, чтобы предоставить вам большую область для работы.</span><span class="sxs-lookup"><span data-stu-id="71b85-167">Drag the bottom edge of the prototype cell to give you a larger area to work with.</span></span>
1. <span data-ttu-id="71b85-168">Используйте **библиотеку,** чтобы добавить три **метки в** прототип ячейки.</span><span class="sxs-lookup"><span data-stu-id="71b85-168">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="71b85-169">Выберите прототип ячейки, а затем выберите **identity Inspector.**</span><span class="sxs-lookup"><span data-stu-id="71b85-169">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="71b85-170">Измените **класс** **на CalendarTableViewCell.**</span><span class="sxs-lookup"><span data-stu-id="71b85-170">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="71b85-171">Выберите **"Attributes Inspector"** (Инспектор атрибутов) и за **установите для идентификатора (Identifier)** `EventCell` (Идентификатор).</span><span class="sxs-lookup"><span data-stu-id="71b85-171">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="71b85-172">Выбрав **EventCell,** выберите **Connections Inspector** и подключите его, а также метки, добавленные в ячейку `durationLabel` в `organizerLabel` `subjectLabel` storyboard.</span><span class="sxs-lookup"><span data-stu-id="71b85-172">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="71b85-173">Установите свойства и ограничения для трех меток следующим образом.</span><span class="sxs-lookup"><span data-stu-id="71b85-173">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="71b85-174">**Метка темы**</span><span class="sxs-lookup"><span data-stu-id="71b85-174">**Subject Label**</span></span>
        - <span data-ttu-id="71b85-175">Add constraint: Leading space to Content View Leading Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-175">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="71b85-176">Добавление ограничения: в окн. место в поле "Точка с представлением содержимого", значение: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-176">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="71b85-177">Add constraint: Top space to Content View Top Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-177">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="71b85-178">**Метка организатора**</span><span class="sxs-lookup"><span data-stu-id="71b85-178">**Organizer Label**</span></span>
        - <span data-ttu-id="71b85-179">Шрифт: System 12.0</span><span class="sxs-lookup"><span data-stu-id="71b85-179">Font: System 12.0</span></span>
        - <span data-ttu-id="71b85-180">Добавить ограничение: высота, значение: 15</span><span class="sxs-lookup"><span data-stu-id="71b85-180">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="71b85-181">Add constraint: Leading space to Content View Leading Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-181">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="71b85-182">Добавление ограничения: в окн. место в поле "Точка с представлением содержимого", значение: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-182">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="71b85-183">Add constraint: Top space to Subject Label Bottom, value: Standard</span><span class="sxs-lookup"><span data-stu-id="71b85-183">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="71b85-184">**Метка длительности**</span><span class="sxs-lookup"><span data-stu-id="71b85-184">**Duration Label**</span></span>
        - <span data-ttu-id="71b85-185">Шрифт: System 12.0</span><span class="sxs-lookup"><span data-stu-id="71b85-185">Font: System 12.0</span></span>
        - <span data-ttu-id="71b85-186">Цвет: темно-серый цвет</span><span class="sxs-lookup"><span data-stu-id="71b85-186">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="71b85-187">Добавить ограничение: высота, значение: 15</span><span class="sxs-lookup"><span data-stu-id="71b85-187">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="71b85-188">Add constraint: Leading space to Content View Leading Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-188">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="71b85-189">Добавление ограничения: в окн. место в поле "Точка с представлением содержимого", значение: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-189">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="71b85-190">Add constraint: Top space to Organizer Label Bottom, value: Standard</span><span class="sxs-lookup"><span data-stu-id="71b85-190">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="71b85-191">Добавление ограничения: нижнее пространство для нижнего поля представления содержимого, значение: 0</span><span class="sxs-lookup"><span data-stu-id="71b85-191">Add constraint: Bottom space to Content View Bottom Margin, value: 0</span></span>

1. <span data-ttu-id="71b85-192">Выберите **EventCell,** а затем выберите **инспектор размеров.**</span><span class="sxs-lookup"><span data-stu-id="71b85-192">Select the **EventCell**, then select the **Size Inspector**.</span></span> <span data-ttu-id="71b85-193">Включить **автоматически для** **высоты строки.**</span><span class="sxs-lookup"><span data-stu-id="71b85-193">Enable **Automatic** for **Row Height**.</span></span>

    ![Снимок экрана: контроллеры представления таблиц календаря и календаря](images/calendar-table-storyboard.png)

1. <span data-ttu-id="71b85-195">Откройте **CalendarViewController.h** и удалите `calendarJSON` свойство.</span><span class="sxs-lookup"><span data-stu-id="71b85-195">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>

1. <span data-ttu-id="71b85-196">Откройте **CalendarViewController.m и** замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="71b85-196">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewControllerSnippet":::

1. <span data-ttu-id="71b85-197">Запустите приложение, во войти и коснитесь вкладки **"Календарь".** Должен отлиться список событий.</span><span class="sxs-lookup"><span data-stu-id="71b85-197">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Снимок экрана с таблицей событий](./images/calendar-list.png)
