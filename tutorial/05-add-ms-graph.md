<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1eafa-101">В этом упражнении вы добавите Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="1eafa-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="1eafa-102">Для этого приложения вы будете использовать [пакет SDK Microsoft Graph для задания](https://github.com/microsoftgraph/msgraph-sdk-objc) с, чтобы совершать вызовы в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1eafa-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="1eafa-103">Получение событий календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="1eafa-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="1eafa-104">В этом разделе описывается расширение `GraphManager` класса для добавления функции, позволяющей получить события и обновление `CalendarViewController` , чтобы использовать эти новые функции.</span><span class="sxs-lookup"><span data-stu-id="1eafa-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="1eafa-105">Откройте **графманажер. h** и добавьте приведенный ниже код перед `@interface` объявлением.</span><span class="sxs-lookup"><span data-stu-id="1eafa-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. <span data-ttu-id="1eafa-106">Добавьте следующий код в `@interface` объявление.</span><span class="sxs-lookup"><span data-stu-id="1eafa-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. <span data-ttu-id="1eafa-107">Откройте **графманажер. m** и добавьте указанную ниже функцию в `GraphManager` класс.</span><span class="sxs-lookup"><span data-stu-id="1eafa-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

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
    > <span data-ttu-id="1eafa-108">Рассмотрите, какие действия `getEventsWithCompletionBlock` выполняет код.</span><span class="sxs-lookup"><span data-stu-id="1eafa-108">Consider what the code in `getEventsWithCompletionBlock` is doing.</span></span>
    >
    > - <span data-ttu-id="1eafa-109">URL-адрес, который будет вызываться — это `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="1eafa-109">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="1eafa-110">Параметр `select` запроса позволяет ограничить поля, возвращаемые для каждого события, только теми, которые будут использоваться приложением.</span><span class="sxs-lookup"><span data-stu-id="1eafa-110">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="1eafa-111">Параметр `orderby` запроса сортирует результаты по дате и времени создания, начиная с самого последнего элемента.</span><span class="sxs-lookup"><span data-stu-id="1eafa-111">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="1eafa-112">Откройте **календарвиевконтроллер. m** и замените все его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="1eafa-112">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

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

<span data-ttu-id="1eafa-113">Теперь вы можете запустить приложение, войти и нажать в меню элемент Навигация по **календарю** .</span><span class="sxs-lookup"><span data-stu-id="1eafa-113">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="1eafa-114">В приложении должен появиться дамп JSON событий.</span><span class="sxs-lookup"><span data-stu-id="1eafa-114">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="1eafa-115">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="1eafa-115">Display the results</span></span>

<span data-ttu-id="1eafa-116">Теперь вы можете заменить дамп JSON на какой-то способ отобразить результаты в удобном для пользователя виде.</span><span class="sxs-lookup"><span data-stu-id="1eafa-116">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="1eafa-117">В этом разделе будет изменена `getEventsWithCompletionBlock` функция, возвращающая строго типизированные объекты, и изменено `CalendarViewController` , чтобы использовать представление таблицы для отображения событий.</span><span class="sxs-lookup"><span data-stu-id="1eafa-117">In this section, you will modify the `getEventsWithCompletionBlock` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getevents"></a><span data-ttu-id="1eafa-118">Обновление событий</span><span class="sxs-lookup"><span data-stu-id="1eafa-118">Update getEvents</span></span>

1. <span data-ttu-id="1eafa-119">Откройте **графманажер. h**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-119">Open **GraphManager.h**.</span></span> <span data-ttu-id="1eafa-120">Измените определение `GetEventsCompletionBlock` типа на следующее:</span><span class="sxs-lookup"><span data-stu-id="1eafa-120">Change the `GetEventsCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="1eafa-121">Откройте **графманажер. m**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-121">Open **GraphManager.m**.</span></span> <span data-ttu-id="1eafa-122">Замените `completionBlock(data, nil);` строку в `getEventsWithCompletionBlock` функции указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="1eafa-122">Replace the `completionBlock(data, nil);` line in the `getEventsWithCompletionBlock` function with the following code.</span></span>

    ```objc
    NSError* graphError;

    // Deserialize to an events collection
    MSCollection* eventsCollection = [[MSCollection alloc] initWithData:data error:&graphError];
    if (graphError) {
        completionBlock(nil, graphError);
        return;
    }

    // Create an array to return
    NSMutableArray* eventsArray = [[NSMutableArray alloc]
                                initWithCapacity:eventsCollection.value.count];

    for (id event in eventsCollection.value) {
        // Deserialize the event and add to the array
        MSGraphEvent* graphEvent = [[MSGraphEvent alloc] initWithDictionary:event];
        [eventsArray addObject:graphEvent];
    }

    completionBlock(eventsArray, nil);
    ```

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="1eafa-123">Обновление Календарвиевконтроллер</span><span class="sxs-lookup"><span data-stu-id="1eafa-123">Update CalendarViewController</span></span>

1. <span data-ttu-id="1eafa-124">Создайте новый файл **класса Touch Cocoa** в проекте **графтуториал** с именем `CalendarTableViewCell`.</span><span class="sxs-lookup"><span data-stu-id="1eafa-124">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="1eafa-125">Выберите **уитаблевиевцелл** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="1eafa-125">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="1eafa-126">Откройте **календартаблевиевцелл. h** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="1eafa-126">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <UIKit/UIKit.h>

    NS_ASSUME_NONNULL_BEGIN

    @interface CalendarTableViewCell : UITableViewCell

    @property (nonatomic) NSString* subject;
    @property (nonatomic) NSString* organizer;
    @property (nonatomic) NSString* duration;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="1eafa-127">Откройте **календартаблевиевцелл. m** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="1eafa-127">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "CalendarTableViewCell.h"

    @interface CalendarTableViewCell()

    @property (nonatomic) IBOutlet UILabel *subjectLabel;
    @property (nonatomic) IBOutlet UILabel *organizerLabel;
    @property (nonatomic) IBOutlet UILabel *durationLabel;

    @end

    @implementation CalendarTableViewCell

    - (void)awakeFromNib {
        [super awakeFromNib];
        // Initialization code
    }

    - (void)setSelected:(BOOL)selected animated:(BOOL)animated {
        [super setSelected:selected animated:animated];

        // Configure the view for the selected state
    }

    - (void) setSubject:(NSString *)subject {
        _subject = subject;
        self.subjectLabel.text = subject;
        [self.subjectLabel sizeToFit];
    }

    - (void) setOrganizer:(NSString *)organizer {
        _organizer = organizer;
        self.organizerLabel.text = organizer;
        [self.organizerLabel sizeToFit];
    }

    - (void) setDuration:(NSString *)duration {
        _duration = duration;
        self.durationLabel.text = duration;
        [self.durationLabel sizeToFit];
    }

    @end
    ```

1. <span data-ttu-id="1eafa-128">Откройте файл **Main. Storyboard** и перейдите к **сцене календаря**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-128">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="1eafa-129">Выберите **представление** в **сцене календаря** и удалите его.</span><span class="sxs-lookup"><span data-stu-id="1eafa-129">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![Снимок экрана с представлением в сцене календаря](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="1eafa-131">Добавьте **представление таблицы** из **библиотеки** в **сцену в календаре**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-131">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="1eafa-132">Выберите представление таблицы, а затем выберите **инспектор атрибутов**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-132">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="1eafa-133">Задайте для **ячеек прототипа** значение **1**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-133">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="1eafa-134">Используйте **библиотеку** , чтобы добавить три **метки** к ячейке прототипа.</span><span class="sxs-lookup"><span data-stu-id="1eafa-134">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="1eafa-135">Выберите ячейку прототип, а затем выберите **инспектор удостоверений**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-135">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="1eafa-136">Замените **Class** на **календартаблевиевцелл**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-136">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="1eafa-137">Выберите **инспектор атрибутов** и **идентификатор** набора `EventCell`.</span><span class="sxs-lookup"><span data-stu-id="1eafa-137">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="1eafa-138">В меню **Редактор** выберите команду **устранить проблемы с автоматическим макетом**, а затем выберите **добавить недостающие ограничения** **в все представления на контроллере приветствия**.</span><span class="sxs-lookup"><span data-stu-id="1eafa-138">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>
1. <span data-ttu-id="1eafa-139">Выбрав **евентцелл** , выберите **инспектор подключений** и подключение `durationLabel`, `organizerLabel`а `subjectLabel` также метки, добавленные в ячейку в раскадровке.</span><span class="sxs-lookup"><span data-stu-id="1eafa-139">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>

    ![Снимок экрана с макетом ячеек прототипа](./images/prototype-cell-layout.png)

1. <span data-ttu-id="1eafa-141">Откройте **календарвиевконтроллер. h** и удалите `calendarJSON` свойство.</span><span class="sxs-lookup"><span data-stu-id="1eafa-141">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>
1. <span data-ttu-id="1eafa-142">Измените `@interface` объявление на следующее.</span><span class="sxs-lookup"><span data-stu-id="1eafa-142">Change the `@interface` declaration to the following.</span></span>

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. <span data-ttu-id="1eafa-143">Откройте **календарвиевконтроллер. m** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="1eafa-143">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "WelcomeViewController.h"
    #import "SpinnerViewController.h"
    #import "AuthenticationManager.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>

    @interface WelcomeViewController ()

    @property SpinnerViewController* spinner;

    @end

    @implementation WelcomeViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];

        [GraphManager.instance
         getMeWithCompletionBlock:^(MSGraphUser * _Nullable user, NSError * _Nullable error) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [self.spinner stop];

                if (error) {
                    // Show the error
                    UIAlertController* alert = [UIAlertController
                                                alertControllerWithTitle:@"Error getting user profile"
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

                // Set display name
                self.userDisplayName.text = user.displayName ? : @"Mysterious Stranger";
                [self.userDisplayName sizeToFit];

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = user.mail ? : user.userPrincipalName;
                [self.userEmail sizeToFit];
            });
         }];
    }

    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }

    @end
    ```

1. <span data-ttu-id="1eafa-144">Запустите приложение и войдите в систему, а затем коснитесь вкладки **Календарь** . Вы должны увидеть список событий.</span><span class="sxs-lookup"><span data-stu-id="1eafa-144">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Снимок экрана с таблицей событий](./images/calendar-list.png)
