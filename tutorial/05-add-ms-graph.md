<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f581a-101">В этом упражнении вы добавите Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="f581a-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="f581a-102">Для этого приложения вы будете использовать [пакет SDK Microsoft Graph для задания](https://github.com/microsoftgraph/msgraph-sdk-objc) с, чтобы совершать вызовы в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f581a-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="f581a-103">Получение событий календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="f581a-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="f581a-104">В этом разделе описывается расширение `GraphManager` класса для добавления функции, позволяющей получить события и обновление `CalendarViewController` , чтобы использовать эти новые функции.</span><span class="sxs-lookup"><span data-stu-id="f581a-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="f581a-105">Откройте **графманажер. h** и добавьте приведенный ниже код перед `@interface` объявлением.</span><span class="sxs-lookup"><span data-stu-id="f581a-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. <span data-ttu-id="f581a-106">Добавьте следующий код в `@interface` объявление.</span><span class="sxs-lookup"><span data-stu-id="f581a-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. <span data-ttu-id="f581a-107">Откройте **графманажер. m** и добавьте указанную ниже функцию в `GraphManager` класс.</span><span class="sxs-lookup"><span data-stu-id="f581a-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

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
    > <span data-ttu-id="f581a-108">Рассмотрите, какие действия `getEventsWithCompletionBlock` выполняет код.</span><span class="sxs-lookup"><span data-stu-id="f581a-108">Consider what the code in `getEventsWithCompletionBlock` is doing.</span></span>
    >
    > - <span data-ttu-id="f581a-109">URL-адрес, который будет вызываться — это `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="f581a-109">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="f581a-110">Параметр `select` запроса позволяет ограничить поля, возвращаемые для каждого события, только теми, которые будут использоваться приложением.</span><span class="sxs-lookup"><span data-stu-id="f581a-110">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="f581a-111">Параметр `orderby` запроса сортирует результаты по дате и времени создания, начиная с самого последнего элемента.</span><span class="sxs-lookup"><span data-stu-id="f581a-111">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="f581a-112">Откройте **календарвиевконтроллер. m** и замените все его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="f581a-112">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

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

1. <span data-ttu-id="f581a-113">Запустите приложение и выполните вход в систему, а затем нажмите элемент навигации в **календаре** в меню.</span><span class="sxs-lookup"><span data-stu-id="f581a-113">Run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="f581a-114">В приложении должен появиться дамп JSON событий.</span><span class="sxs-lookup"><span data-stu-id="f581a-114">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="f581a-115">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="f581a-115">Display the results</span></span>

<span data-ttu-id="f581a-116">Теперь вы можете заменить дамп JSON на какой-то способ отобразить результаты в удобном для пользователя виде.</span><span class="sxs-lookup"><span data-stu-id="f581a-116">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="f581a-117">В этом разделе будет изменена `getEventsWithCompletionBlock` функция, возвращающая строго типизированные объекты, и изменено `CalendarViewController` , чтобы использовать представление таблицы для отображения событий.</span><span class="sxs-lookup"><span data-stu-id="f581a-117">In this section, you will modify the `getEventsWithCompletionBlock` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getevents"></a><span data-ttu-id="f581a-118">Обновление событий</span><span class="sxs-lookup"><span data-stu-id="f581a-118">Update getEvents</span></span>

1. <span data-ttu-id="f581a-119">Откройте **графманажер. h**.</span><span class="sxs-lookup"><span data-stu-id="f581a-119">Open **GraphManager.h**.</span></span> <span data-ttu-id="f581a-120">Измените определение `GetEventsCompletionBlock` типа на следующее:</span><span class="sxs-lookup"><span data-stu-id="f581a-120">Change the `GetEventsCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="f581a-121">Откройте **графманажер. m**.</span><span class="sxs-lookup"><span data-stu-id="f581a-121">Open **GraphManager.m**.</span></span> <span data-ttu-id="f581a-122">Замените `completionBlock(data, nil);` строку в `getEventsWithCompletionBlock` функции указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="f581a-122">Replace the `completionBlock(data, nil);` line in the `getEventsWithCompletionBlock` function with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetEventsSnippet" highlight="24-43":::

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="f581a-123">Обновление Календарвиевконтроллер</span><span class="sxs-lookup"><span data-stu-id="f581a-123">Update CalendarViewController</span></span>

1. <span data-ttu-id="f581a-124">Создайте новый файл **класса Touch Cocoa** в проекте **графтуториал** с именем `CalendarTableViewCell`.</span><span class="sxs-lookup"><span data-stu-id="f581a-124">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="f581a-125">Выберите **уитаблевиевцелл** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="f581a-125">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="f581a-126">Откройте **календартаблевиевцелл. h** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="f581a-126">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="f581a-127">Откройте **календартаблевиевцелл. m** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="f581a-127">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="f581a-128">Откройте файл **Main. Storyboard** и перейдите к **сцене календаря**.</span><span class="sxs-lookup"><span data-stu-id="f581a-128">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="f581a-129">Выберите **представление** в **сцене календаря** и удалите его.</span><span class="sxs-lookup"><span data-stu-id="f581a-129">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![Снимок экрана с представлением в сцене календаря](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="f581a-131">Добавьте **представление таблицы** из **библиотеки** в **сцену в календаре**.</span><span class="sxs-lookup"><span data-stu-id="f581a-131">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="f581a-132">Выберите представление таблицы, а затем выберите **инспектор атрибутов**.</span><span class="sxs-lookup"><span data-stu-id="f581a-132">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="f581a-133">Задайте для **ячеек прототипа** значение **1**.</span><span class="sxs-lookup"><span data-stu-id="f581a-133">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="f581a-134">Используйте **библиотеку** , чтобы добавить три **метки** к ячейке прототипа.</span><span class="sxs-lookup"><span data-stu-id="f581a-134">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="f581a-135">Выберите ячейку прототип, а затем выберите **инспектор удостоверений**.</span><span class="sxs-lookup"><span data-stu-id="f581a-135">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="f581a-136">Замените **Class** на **календартаблевиевцелл**.</span><span class="sxs-lookup"><span data-stu-id="f581a-136">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="f581a-137">Выберите **инспектор атрибутов** и **идентификатор** набора `EventCell`.</span><span class="sxs-lookup"><span data-stu-id="f581a-137">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="f581a-138">Выбрав **евентцелл** , выберите **инспектор подключений** и подключение `durationLabel`, `organizerLabel`а `subjectLabel` также метки, добавленные в ячейку в раскадровке.</span><span class="sxs-lookup"><span data-stu-id="f581a-138">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="f581a-139">Задайте свойства и ограничения для этих трех меток, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="f581a-139">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="f581a-140">**Метка темы**</span><span class="sxs-lookup"><span data-stu-id="f581a-140">**Subject Label**</span></span>
        - <span data-ttu-id="f581a-141">Добавить ограничение: начальное пространство в представлении содержимого начальное поле, значение: 0</span><span class="sxs-lookup"><span data-stu-id="f581a-141">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="f581a-142">Add Constraint: замыкающий пробел для конечного поля представления содержимого, значение: 0</span><span class="sxs-lookup"><span data-stu-id="f581a-142">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="f581a-143">Добавить ограничение: Верхняя область для представления контента верхнее поле, значение: 0</span><span class="sxs-lookup"><span data-stu-id="f581a-143">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="f581a-144">**Метка "Организатор"**</span><span class="sxs-lookup"><span data-stu-id="f581a-144">**Organizer Label**</span></span>
        - <span data-ttu-id="f581a-145">Шрифт: System 12,0</span><span class="sxs-lookup"><span data-stu-id="f581a-145">Font: System 12.0</span></span>
        - <span data-ttu-id="f581a-146">Добавить ограничение: начальное пространство в представлении содержимого начальное поле, значение: 0</span><span class="sxs-lookup"><span data-stu-id="f581a-146">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="f581a-147">Add Constraint: замыкающий пробел для конечного поля представления содержимого, значение: 0</span><span class="sxs-lookup"><span data-stu-id="f581a-147">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="f581a-148">Добавление ограничения: Верхняя область для подписи темы внизу, значение: Standard</span><span class="sxs-lookup"><span data-stu-id="f581a-148">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="f581a-149">**Метка длительности**</span><span class="sxs-lookup"><span data-stu-id="f581a-149">**Duration Label**</span></span>
        - <span data-ttu-id="f581a-150">Шрифт: System 12,0</span><span class="sxs-lookup"><span data-stu-id="f581a-150">Font: System 12.0</span></span>
        - <span data-ttu-id="f581a-151">Цвет: темно-серый цвет</span><span class="sxs-lookup"><span data-stu-id="f581a-151">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="f581a-152">Добавить ограничение: начальное пространство в представлении содержимого начальное поле, значение: 0</span><span class="sxs-lookup"><span data-stu-id="f581a-152">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="f581a-153">Add Constraint: замыкающий пробел для конечного поля представления содержимого, значение: 0</span><span class="sxs-lookup"><span data-stu-id="f581a-153">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="f581a-154">Добавить ограничение: Верхняя область для метки организатора внизу, значение: Standard</span><span class="sxs-lookup"><span data-stu-id="f581a-154">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="f581a-155">Добавить ограничение: нижнее пространство для представления содержимого нижнее поле, значение: 8</span><span class="sxs-lookup"><span data-stu-id="f581a-155">Add constraint: Bottom space to Content View Bottom Margin, value: 8</span></span>

    ![Снимок экрана с макетом ячеек прототипа](./images/prototype-cell-layout.png)

1. <span data-ttu-id="f581a-157">Откройте **календарвиевконтроллер. h** и удалите `calendarJSON` свойство.</span><span class="sxs-lookup"><span data-stu-id="f581a-157">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>
1. <span data-ttu-id="f581a-158">Измените `@interface` объявление на следующее.</span><span class="sxs-lookup"><span data-stu-id="f581a-158">Change the `@interface` declaration to the following.</span></span>

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. <span data-ttu-id="f581a-159">Откройте **календарвиевконтроллер. m** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="f581a-159">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewSnippet":::

1. <span data-ttu-id="f581a-160">Запустите приложение и войдите в систему, а затем коснитесь вкладки **Календарь** . Вы должны увидеть список событий.</span><span class="sxs-lookup"><span data-stu-id="f581a-160">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Снимок экрана с таблицей событий](./images/calendar-list.png)
