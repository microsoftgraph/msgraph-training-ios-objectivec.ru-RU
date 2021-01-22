<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="343e9-101">Начните с создания нового проекта Objective-C.</span><span class="sxs-lookup"><span data-stu-id="343e9-101">Begin by creating a new Objective-C project.</span></span>

1. <span data-ttu-id="343e9-102">Откройте Xcode.</span><span class="sxs-lookup"><span data-stu-id="343e9-102">Open Xcode.</span></span> <span data-ttu-id="343e9-103">В меню **"Файл"** выберите **"Новый"** и **"Проект".**</span><span class="sxs-lookup"><span data-stu-id="343e9-103">On the **File** menu, select **New**, then **Project**.</span></span>
1. <span data-ttu-id="343e9-104">Выберите шаблон **приложения и** выберите **"Далее".**</span><span class="sxs-lookup"><span data-stu-id="343e9-104">Choose the **App** template and select **Next**.</span></span>

    ![Снимок экрана: диалоговое окно выбора шаблона Xcode](./images/xcode-select-template.png)

1. <span data-ttu-id="343e9-106">Установите для **имени продукта** и `GraphTutorial` для **языка** **objective-C**.</span><span class="sxs-lookup"><span data-stu-id="343e9-106">Set the **Product Name** to `GraphTutorial` and the **Language** to **Objective-C**.</span></span>
1. <span data-ttu-id="343e9-107">Заполните оставшиеся поля и выберите **"Далее".**</span><span class="sxs-lookup"><span data-stu-id="343e9-107">Fill in the remaining fields and select **Next**.</span></span>
1. <span data-ttu-id="343e9-108">Выберите расположение для проекта и выберите **"Создать".**</span><span class="sxs-lookup"><span data-stu-id="343e9-108">Choose a location for the project and select **Create**.</span></span>

## <a name="install-dependencies"></a><span data-ttu-id="343e9-109">Установка зависимостей</span><span class="sxs-lookup"><span data-stu-id="343e9-109">Install dependencies</span></span>

<span data-ttu-id="343e9-110">Прежде чем двигаться дальше, установите некоторые дополнительные зависимости, которые вы будете использовать позже.</span><span class="sxs-lookup"><span data-stu-id="343e9-110">Before moving on, install some additional dependencies that you will use later.</span></span>

- <span data-ttu-id="343e9-111">[Microsoft Authentication Library (MSAL) для iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) для проверки подлинности в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="343e9-111">[Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) for authenticating to with Azure AD.</span></span>
- <span data-ttu-id="343e9-112">[Microsoft Graph SDK для Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) для вызовов Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="343e9-112">[Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="343e9-113">[Microsoft Graph Models SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) for strongly-typed objects representing Microsoft Graph resources like users or events.</span><span class="sxs-lookup"><span data-stu-id="343e9-113">[Microsoft Graph Models SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) for strongly-typed objects representing Microsoft Graph resources like users or events.</span></span>

1. <span data-ttu-id="343e9-114">Выйти из Xcode.</span><span class="sxs-lookup"><span data-stu-id="343e9-114">Quit Xcode.</span></span>
1. <span data-ttu-id="343e9-115">Откройте терминал и измените каталог на расположение проекта **GraphTutorial.**</span><span class="sxs-lookup"><span data-stu-id="343e9-115">Open Terminal and change the directory to the location of your **GraphTutorial** project.</span></span>
1. <span data-ttu-id="343e9-116">Чтобы создать Podfile, запустите следующую команду.</span><span class="sxs-lookup"><span data-stu-id="343e9-116">Run the following command to create a Podfile.</span></span>

    ```Shell
    pod init
    ```

1. <span data-ttu-id="343e9-117">Откройте Podfile и добавьте следующие строки сразу после `use_frameworks!` строки.</span><span class="sxs-lookup"><span data-stu-id="343e9-117">Open the Podfile and add the following lines just after the `use_frameworks!` line.</span></span>

    ```Ruby
    pod 'MSAL', '~> 1.1.13'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. <span data-ttu-id="343e9-118">Сохраните Podfile и запустите следующую команду, чтобы установить зависимости.</span><span class="sxs-lookup"><span data-stu-id="343e9-118">Save the Podfile, then run the following command to install the dependencies.</span></span>

    ```Shell
    pod install
    ```

1. <span data-ttu-id="343e9-119">После выполнения команды откройте созданное **пространство GraphTutorial.xcworkspace** в Xcode.</span><span class="sxs-lookup"><span data-stu-id="343e9-119">Once the command completes, open the newly created **GraphTutorial.xcworkspace** in Xcode.</span></span>

## <a name="design-the-app"></a><span data-ttu-id="343e9-120">Проектирование приложения</span><span class="sxs-lookup"><span data-stu-id="343e9-120">Design the app</span></span>

<span data-ttu-id="343e9-121">В этом разделе вы создадим представления для приложения: страницу для входов, навигатор панели вкладок, страницу приветствия и страницу календаря.</span><span class="sxs-lookup"><span data-stu-id="343e9-121">In this section you will create the views for the app: a sign in page, a tab bar navigator, a welcome page, and a calendar page.</span></span> <span data-ttu-id="343e9-122">Вы также создадим наложение индикатора активности.</span><span class="sxs-lookup"><span data-stu-id="343e9-122">You'll also create an activity indicator overlay.</span></span>

### <a name="create-sign-in-page"></a><span data-ttu-id="343e9-123">Страница "Создание страницы для входов"</span><span class="sxs-lookup"><span data-stu-id="343e9-123">Create sign in page</span></span>

1. <span data-ttu-id="343e9-124">**Развертка папки GraphTutorial** в Xcode, а затем выберите **файл ViewController.m.**</span><span class="sxs-lookup"><span data-stu-id="343e9-124">Expand the **GraphTutorial** folder in Xcode, then select the **ViewController.m** file.</span></span>
1. <span data-ttu-id="343e9-125">В **инспекторе файлов** **измените** имя файла на `SignInViewController.m` .</span><span class="sxs-lookup"><span data-stu-id="343e9-125">In the **File Inspector**, change the **Name** of the file to `SignInViewController.m`.</span></span>

    ![Снимок экрана: инспектор файлов](./images/rename-file.png)

1. <span data-ttu-id="343e9-127">Откройте **SignInViewController.m** и замените его содержимое следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="343e9-127">Open **SignInViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "SignInViewController.h"

    @interface SignInViewController ()

    @end

    @implementation SignInViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.
    }

    - (IBAction)signIn {
        [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
    }
    @end
    ```

1. <span data-ttu-id="343e9-128">Выберите файл **ViewController.h.**</span><span class="sxs-lookup"><span data-stu-id="343e9-128">Select the **ViewController.h** file.</span></span>
1. <span data-ttu-id="343e9-129">В **инспекторе файлов** **измените** имя файла на `SignInViewController.h` .</span><span class="sxs-lookup"><span data-stu-id="343e9-129">In the **File Inspector**, change the **Name** of the file to `SignInViewController.h`.</span></span>
1. <span data-ttu-id="343e9-130">Откройте **SignInViewController.h** и измените все экземпляры `ViewController` на `SignInViewController` .</span><span class="sxs-lookup"><span data-stu-id="343e9-130">Open **SignInViewController.h** and change all instances of `ViewController` to `SignInViewController`.</span></span>

1. <span data-ttu-id="343e9-131">Откройте файл **Main.storyboard.**</span><span class="sxs-lookup"><span data-stu-id="343e9-131">Open the **Main.storyboard** file.</span></span>
1. <span data-ttu-id="343e9-132">Раз **развернуть сцену контроллера представления,** а затем выберите **контроллер представления.**</span><span class="sxs-lookup"><span data-stu-id="343e9-132">Expand **View Controller Scene**, then select **View Controller**.</span></span>

    ![Снимок экрана: Xcode с выбранным контроллером просмотра](./images/storyboard-select-view-controller.png)

1. <span data-ttu-id="343e9-134">Выберите identity **Inspector,** а затем измените **выпадающий** класс на **SignInViewController.**</span><span class="sxs-lookup"><span data-stu-id="343e9-134">Select the **Identity Inspector**, then change the **Class** dropdown to **SignInViewController**.</span></span>

    ![Снимок экрана инспектора удостоверений](./images/change-class.png)

1. <span data-ttu-id="343e9-136">Выберите **библиотеку,** а затем перетащите **кнопку** на контроллер **просмотра для входов.**</span><span class="sxs-lookup"><span data-stu-id="343e9-136">Select the **Library**, then drag a **Button** onto the **Sign In View Controller**.</span></span>

    ![Снимок экрана: библиотека в Xcode](./images/add-button-to-view.png)

1. <span data-ttu-id="343e9-138">После нажатия кнопки выберите **"Attributes Inspector"** (Инспектор атрибутов) и измените **заголовок** кнопки на `Sign In` .</span><span class="sxs-lookup"><span data-stu-id="343e9-138">With the button selected, select the **Attributes Inspector** and change the **Title** of the button to `Sign In`.</span></span>

    ![Снимок экрана с полем "Заголовок" в инспекторе атрибутов в Xcode](./images/set-button-title.png)

1. <span data-ttu-id="343e9-140">Нажатие кнопки "Выровнять" в нижней части storyboard. </span><span class="sxs-lookup"><span data-stu-id="343e9-140">With the button selected, select the **Align** button at the bottom of the storyboard.</span></span> <span data-ttu-id="343e9-141">Выберите и **горизонтально** в  контейнере, и по вертикали в ограничениях контейнера, оставьте их значения 0, а затем выберите **"Добавить 2 ограничения".**</span><span class="sxs-lookup"><span data-stu-id="343e9-141">Select both the **Horizontally in container** and **Vertically in container** constraints, leave their values as 0, then select **Add 2 constraints**.</span></span>

    ![Снимок экрана с настройками ограничений выравнивания в Xcode](./images/add-alignment-constraints.png)

1. <span data-ttu-id="343e9-143">Выберите контроллер **просмотра для входов,** а затем **выберите инспектор подключений.**</span><span class="sxs-lookup"><span data-stu-id="343e9-143">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="343e9-144">В **области "Received Actions"** перетащите незаполненный круг рядом с **кнопкой signIn.**</span><span class="sxs-lookup"><span data-stu-id="343e9-144">Under **Received Actions**, drag the unfilled circle next to **signIn** onto the button.</span></span> <span data-ttu-id="343e9-145">Выберите **"Сенсорный экран" во** всплывающее меню.</span><span class="sxs-lookup"><span data-stu-id="343e9-145">Select **Touch Up Inside** on the pop-up menu.</span></span>

    ![Снимок экрана: перетаскивание действия signIn на кнопку в Xcode](./images/connect-sign-in-button.png)

### <a name="create-tab-bar"></a><span data-ttu-id="343e9-147">Создание панели вкладок</span><span class="sxs-lookup"><span data-stu-id="343e9-147">Create tab bar</span></span>

1. <span data-ttu-id="343e9-148">Выберите **библиотеку,** а затем перетащите контроллер **панели вкладок** в storyboard.</span><span class="sxs-lookup"><span data-stu-id="343e9-148">Select the **Library**, then drag a **Tab Bar Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="343e9-149">Выберите контроллер **просмотра для входов,** а затем **выберите инспектор подключений.**</span><span class="sxs-lookup"><span data-stu-id="343e9-149">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="343e9-150">В **области "Триггеры"** перетащите незаполненный круг рядом с вручную на контроллер панели вкладок **в** storyboard. </span><span class="sxs-lookup"><span data-stu-id="343e9-150">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Tab Bar Controller** on the storyboard.</span></span> <span data-ttu-id="343e9-151">Выберите **"Представить" модально** во всплывающее меню.</span><span class="sxs-lookup"><span data-stu-id="343e9-151">Select **Present Modally** in the pop-up menu.</span></span>

    ![Снимок экрана: перетаскивание вручную в новый контроллер панели вкладок в Xcode](./images/add-segue.png)

1. <span data-ttu-id="343e9-153">Выберите только что добавленную сегу, а затем — **инспектор атрибутов.**</span><span class="sxs-lookup"><span data-stu-id="343e9-153">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="343e9-154">Установите для **поля "Идентификатор"** и `userSignedIn` **"Презентация"** полноэкранный **режим.**</span><span class="sxs-lookup"><span data-stu-id="343e9-154">Set the **Identifier** field to `userSignedIn`, and set **Presentation** to **Full Screen**.</span></span>

    ![Снимок экрана с полем идентификатора в инспекторе атрибутов в Xcode](./images/set-segue-identifier.png)

1. <span data-ttu-id="343e9-156">Выберите сцену **"Элемент 1"** и выберите **"Connections Inspector" (Инспектор подключений).**</span><span class="sxs-lookup"><span data-stu-id="343e9-156">Select the **Item 1 Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="343e9-157">В **области "Триггеры"** перетащите незаполненный круг рядом с вручную на контроллер просмотра для входов **в** storyboard. </span><span class="sxs-lookup"><span data-stu-id="343e9-157">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Sign In View Controller** on the storyboard.</span></span> <span data-ttu-id="343e9-158">Выберите **"Представить" модально** во всплывающее меню.</span><span class="sxs-lookup"><span data-stu-id="343e9-158">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="343e9-159">Выберите только что добавленную сегу, а затем — **инспектор атрибутов.**</span><span class="sxs-lookup"><span data-stu-id="343e9-159">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="343e9-160">Установите для **поля "Идентификатор"** и `userSignedOut` **"Презентация"** полноэкранный **режим.**</span><span class="sxs-lookup"><span data-stu-id="343e9-160">Set the **Identifier** field to `userSignedOut`, and set **Presentation** to **Full Screen**.</span></span>

### <a name="create-welcome-page"></a><span data-ttu-id="343e9-161">Создание страницы приветствия</span><span class="sxs-lookup"><span data-stu-id="343e9-161">Create welcome page</span></span>

1. <span data-ttu-id="343e9-162">Выберите файл **Assets.xcassets.**</span><span class="sxs-lookup"><span data-stu-id="343e9-162">Select the **Assets.xcassets** file.</span></span>
1. <span data-ttu-id="343e9-163">В меню **"Редактор"** выберите **"Добавить новый актив"** и **"Набор изображений".**</span><span class="sxs-lookup"><span data-stu-id="343e9-163">On the **Editor** menu, select **Add New Asset**, then **Image Set**.</span></span>
1. <span data-ttu-id="343e9-164">Выберите новый ресурс **Image** и используйте **attribute Inspector,** чтобы установить его **имя.** `DefaultUserPhoto`</span><span class="sxs-lookup"><span data-stu-id="343e9-164">Select the new **Image** asset and use the **Attribute Inspector** to set its **Name** to `DefaultUserPhoto`.</span></span>
1. <span data-ttu-id="343e9-165">Добавьте любое изображение, которое вы хотите выступать в качестве фотографии профиля пользователя по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="343e9-165">Add any image you like to serve as a default user profile photo.</span></span>

    ![Снимок экрана: представление ресурсов "Набор изображений" в Xcode](./images/add-default-user-photo.png)

1. <span data-ttu-id="343e9-167">Создайте файл **класса Cocoa Touch в** папке **GraphTutorial** с именем `WelcomeViewController` .</span><span class="sxs-lookup"><span data-stu-id="343e9-167">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `WelcomeViewController`.</span></span> <span data-ttu-id="343e9-168">Выберите **UIViewController** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="343e9-168">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="343e9-169">Откройте **WelcomeViewController.h** и добавьте в объявление следующий `@interface` код.</span><span class="sxs-lookup"><span data-stu-id="343e9-169">Open **WelcomeViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    @property (nonatomic) IBOutlet UIImageView *userProfilePhoto;
    @property (nonatomic) IBOutlet UILabel *userDisplayName;
    @property (nonatomic) IBOutlet UILabel *userEmail;
    ```

1. <span data-ttu-id="343e9-170">Откройте **WelcomeViewController.m** и замените его содержимое следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="343e9-170">Open **WelcomeViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "WelcomeViewController.h"

    @interface WelcomeViewController ()

    @end

    @implementation WelcomeViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        // TEMPORARY
        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];
        self.userDisplayName.text = @"Default User";
        [self.userDisplayName sizeToFit];
        self.userEmail.text = @"default@contoso.com";
        [self.userEmail sizeToFit];
    }

    - (IBAction)signOut {
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }

    @end
    ```

1. <span data-ttu-id="343e9-171">Откройте **main.storyboard.**</span><span class="sxs-lookup"><span data-stu-id="343e9-171">Open **Main.storyboard**.</span></span> <span data-ttu-id="343e9-172">Выберите сцену **"Элемент 1"** и выберите инспектор **удостоверений.**</span><span class="sxs-lookup"><span data-stu-id="343e9-172">Select the **Item 1 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="343e9-173">Измените **значение класса** на **WelcomeViewController.**</span><span class="sxs-lookup"><span data-stu-id="343e9-173">Change the **Class** value to **WelcomeViewController**.</span></span>
1. <span data-ttu-id="343e9-174">С помощью **библиотеки добавьте** следующие элементы в сцену **"Элемент 1".**</span><span class="sxs-lookup"><span data-stu-id="343e9-174">Using the **Library**, add the following items to the **Item 1 Scene**.</span></span>

    - <span data-ttu-id="343e9-175">Одно **представление изображения**</span><span class="sxs-lookup"><span data-stu-id="343e9-175">One **Image View**</span></span>
    - <span data-ttu-id="343e9-176">Две **метки**</span><span class="sxs-lookup"><span data-stu-id="343e9-176">Two **Labels**</span></span>
    - <span data-ttu-id="343e9-177">Одна **кнопка**</span><span class="sxs-lookup"><span data-stu-id="343e9-177">One **Button**</span></span>

1. <span data-ttu-id="343e9-178">Используя **Connections Inspector,** сделайте следующие подключения.</span><span class="sxs-lookup"><span data-stu-id="343e9-178">Using the **Connections Inspector**, make the following connections.</span></span>

    - <span data-ttu-id="343e9-179">**Привяжете выход userDisplayName** к первой метке.</span><span class="sxs-lookup"><span data-stu-id="343e9-179">Link the **userDisplayName** outlet to the first label.</span></span>
    - <span data-ttu-id="343e9-180">**Привяжете выход userEmail** к второй метке.</span><span class="sxs-lookup"><span data-stu-id="343e9-180">Link the **userEmail** outlet to the second label.</span></span>
    - <span data-ttu-id="343e9-181">**Привяжете выход userProfilePhoto** к представлению изображения.</span><span class="sxs-lookup"><span data-stu-id="343e9-181">Link the **userProfilePhoto** outlet to the image view.</span></span>
    - <span data-ttu-id="343e9-182">Связать **полученное действие signOut** с кнопкой **"Касание вверх" внутри.**</span><span class="sxs-lookup"><span data-stu-id="343e9-182">Link the **signOut** received action to the button's **Touch Up Inside**.</span></span>

1. <span data-ttu-id="343e9-183">Выберите представление изображения, а затем выберите **инспектор размеров.**</span><span class="sxs-lookup"><span data-stu-id="343e9-183">Select the image view, then select the **Size Inspector**.</span></span>
1. <span data-ttu-id="343e9-184">Установите для **ширины** **и высоты** 196.</span><span class="sxs-lookup"><span data-stu-id="343e9-184">Set the **Width** and **Height** to 196.</span></span>
1. <span data-ttu-id="343e9-185">Используйте **кнопку** "Выровнять", чтобы добавить ограничение по горизонтали **в контейнере** со значением 0.</span><span class="sxs-lookup"><span data-stu-id="343e9-185">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="343e9-186">Используйте **кнопку "Добавить новые ограничения"** (рядом с кнопкой **"Выравнивание"),** чтобы добавить следующие ограничения:</span><span class="sxs-lookup"><span data-stu-id="343e9-186">Use the **Add New Constraints** button (next to the **Align** button) to add the following constraints:</span></span>

    - <span data-ttu-id="343e9-187">Выравнивание вверху: безопасная область, значение: 0</span><span class="sxs-lookup"><span data-stu-id="343e9-187">Align Top to: Safe Area, value: 0</span></span>
    - <span data-ttu-id="343e9-188">Bottom Space to: User Display Name, value: Standard</span><span class="sxs-lookup"><span data-stu-id="343e9-188">Bottom Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="343e9-189">Высота, значение: 196</span><span class="sxs-lookup"><span data-stu-id="343e9-189">Height, value: 196</span></span>
    - <span data-ttu-id="343e9-190">Ширина, значение: 196</span><span class="sxs-lookup"><span data-stu-id="343e9-190">Width, value: 196</span></span>

    ![Снимок экрана с новыми настройками ограничений в Xcode](./images/add-new-constraints.png)

1. <span data-ttu-id="343e9-192">Выберите первую метку, а затем  с помощью кнопки **"Выравнивание"** добавьте ограничение по горизонтали в контейнере со значением 0.</span><span class="sxs-lookup"><span data-stu-id="343e9-192">Select the first label, then use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="343e9-193">Используйте **кнопку "Добавить новые ограничения",** чтобы добавить следующие ограничения:</span><span class="sxs-lookup"><span data-stu-id="343e9-193">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="343e9-194">Top Space to: User Profile Photo, value: Standard</span><span class="sxs-lookup"><span data-stu-id="343e9-194">Top Space to: User Profile Photo, value: Standard</span></span>
    - <span data-ttu-id="343e9-195">Bottom Space to: User Email, value: Standard</span><span class="sxs-lookup"><span data-stu-id="343e9-195">Bottom Space to: User Email, value: Standard</span></span>

1. <span data-ttu-id="343e9-196">Выберите вторую метку, а затем **— инспектор атрибутов.**</span><span class="sxs-lookup"><span data-stu-id="343e9-196">Select the second label, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="343e9-197">Измените **цвет на** **темно-серый и** измените **шрифт** на **System 12.0.**</span><span class="sxs-lookup"><span data-stu-id="343e9-197">Change the **Color** to **Dark Gray Color**, and change the **Font** to **System 12.0**.</span></span>
1. <span data-ttu-id="343e9-198">Используйте **кнопку** "Выровнять", чтобы добавить ограничение по горизонтали **в контейнере** со значением 0.</span><span class="sxs-lookup"><span data-stu-id="343e9-198">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="343e9-199">Используйте **кнопку "Добавить новые ограничения",** чтобы добавить следующие ограничения:</span><span class="sxs-lookup"><span data-stu-id="343e9-199">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="343e9-200">Top Space to: User Display Name, value: Standard</span><span class="sxs-lookup"><span data-stu-id="343e9-200">Top Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="343e9-201">Bottom Space to: Sign Out, value: 14</span><span class="sxs-lookup"><span data-stu-id="343e9-201">Bottom Space to: Sign Out, value: 14</span></span>

1. <span data-ttu-id="343e9-202">Выберите кнопку, а затем — **инспектор атрибутов.**</span><span class="sxs-lookup"><span data-stu-id="343e9-202">Select the button, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="343e9-203">Измените **заголовок** на `Sign Out` .</span><span class="sxs-lookup"><span data-stu-id="343e9-203">Change the **Title** to `Sign Out`.</span></span>
1. <span data-ttu-id="343e9-204">Используйте **кнопку** "Выровнять", чтобы добавить ограничение по горизонтали **в контейнере** со значением 0.</span><span class="sxs-lookup"><span data-stu-id="343e9-204">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="343e9-205">Используйте **кнопку "Добавить новые ограничения",** чтобы добавить следующие ограничения:</span><span class="sxs-lookup"><span data-stu-id="343e9-205">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="343e9-206">Top Space to: User Email, value: 14</span><span class="sxs-lookup"><span data-stu-id="343e9-206">Top Space to: User Email, value: 14</span></span>

1. <span data-ttu-id="343e9-207">Выберите элемент панели вкладок в нижней части сцены, а затем выберите **инспектор атрибутов.**</span><span class="sxs-lookup"><span data-stu-id="343e9-207">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="343e9-208">Измените **заголовок** на `Me` .</span><span class="sxs-lookup"><span data-stu-id="343e9-208">Change the **Title** to `Me`.</span></span>
1. <span data-ttu-id="343e9-209">В меню **"Редактор"** выберите **"Устранение** проблем с автоматическим макетом", а затем выберите "Добавить отсутствующие **ограничения"** под всеми представлениями в **контроллере приветствия.**</span><span class="sxs-lookup"><span data-stu-id="343e9-209">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>

<span data-ttu-id="343e9-210">После этого сцена приветствия должна выглядеть примерно так:</span><span class="sxs-lookup"><span data-stu-id="343e9-210">The welcome scene should look similar to this once you're done.</span></span>

![Снимок экрана макета сцены приветствия](./images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a><span data-ttu-id="343e9-212">Страница "Создание календаря"</span><span class="sxs-lookup"><span data-stu-id="343e9-212">Create calendar page</span></span>

1. <span data-ttu-id="343e9-213">Создайте файл **класса Cocoa Touch в** папке **GraphTutorial** с именем `CalendarViewController` .</span><span class="sxs-lookup"><span data-stu-id="343e9-213">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `CalendarViewController`.</span></span> <span data-ttu-id="343e9-214">Выберите **UIViewController** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="343e9-214">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="343e9-215">Откройте **CalendarViewController.h** и добавьте в объявление следующий `@interface` код.</span><span class="sxs-lookup"><span data-stu-id="343e9-215">Open **CalendarViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    @property (nonatomic) IBOutlet UITextView *calendarJSON;
    ```

1. <span data-ttu-id="343e9-216">Откройте **CalendarViewController.m и** замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="343e9-216">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"

    @interface CalendarViewController ()

    @end

    @implementation CalendarViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        // TEMPORARY
        self.calendarJSON.text = @"Calendar";
        [self.calendarJSON sizeToFit];
    }

    @end
    ```

1. <span data-ttu-id="343e9-217">Откройте **main.storyboard.**</span><span class="sxs-lookup"><span data-stu-id="343e9-217">Open **Main.storyboard**.</span></span> <span data-ttu-id="343e9-218">Выберите сцену **"Элемент 2"** и выберите инспектор **удостоверений.**</span><span class="sxs-lookup"><span data-stu-id="343e9-218">Select the **Item 2 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="343e9-219">Измените **значение класса** на **CalendarViewController.**</span><span class="sxs-lookup"><span data-stu-id="343e9-219">Change the **Class** value to **CalendarViewController**.</span></span>
1. <span data-ttu-id="343e9-220">С помощью **библиотеки добавьте** представление **текста в** сцену **"Элемент 2".**</span><span class="sxs-lookup"><span data-stu-id="343e9-220">Using the **Library**, add a **Text View** to the **Item 2 Scene**.</span></span>
1. <span data-ttu-id="343e9-221">Выберите только что добавленное текстовое представление.</span><span class="sxs-lookup"><span data-stu-id="343e9-221">Select the text view you just added.</span></span> <span data-ttu-id="343e9-222">В **редакторе выберите** **"Встраить"** и прокрутите **представление.**</span><span class="sxs-lookup"><span data-stu-id="343e9-222">On the **Editor**, choose **Embed In**, then **Scroll View**.</span></span>
1. <span data-ttu-id="343e9-223">С помощью **Connections Inspector** подключите выход **calendarJSON** к представлению текста.</span><span class="sxs-lookup"><span data-stu-id="343e9-223">Using the **Connections Inspector**, connect the **calendarJSON** outlet to the text view.</span></span>
1. <span data-ttu-id="343e9-224">Выберите элемент панели вкладок в нижней части сцены, а затем выберите **инспектор атрибутов.**</span><span class="sxs-lookup"><span data-stu-id="343e9-224">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="343e9-225">Измените **заголовок** на `Calendar` .</span><span class="sxs-lookup"><span data-stu-id="343e9-225">Change the **Title** to `Calendar`.</span></span>
1. <span data-ttu-id="343e9-226">В меню **"Редактор"** выберите **"Устранение** проблем с автоматическим макетом", а затем выберите "Добавить отсутствующие **ограничения"** под всеми представлениями в **контроллере приветствия.**</span><span class="sxs-lookup"><span data-stu-id="343e9-226">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>

<span data-ttu-id="343e9-227">После этого сцена календаря должна выглядеть примерно так:</span><span class="sxs-lookup"><span data-stu-id="343e9-227">The calendar scene should look similar to this once you're done.</span></span>

![Снимок экрана с макетом сцены "Календарь"](./images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a><span data-ttu-id="343e9-229">Создание индикатора активности</span><span class="sxs-lookup"><span data-stu-id="343e9-229">Create activity indicator</span></span>

1. <span data-ttu-id="343e9-230">Создайте файл **класса Cocoa Touch в** папке **GraphTutorial** с именем `SpinnerViewController` .</span><span class="sxs-lookup"><span data-stu-id="343e9-230">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `SpinnerViewController`.</span></span> <span data-ttu-id="343e9-231">Выберите **UIViewController** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="343e9-231">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="343e9-232">Откройте **SpinnerViewController.h** и добавьте следующий код в `@interface` объявление.</span><span class="sxs-lookup"><span data-stu-id="343e9-232">Open **SpinnerViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    - (void) startWithContainer:(UIViewController*) container;
    - (void) stop;
    ```

1. <span data-ttu-id="343e9-233">Откройте **SpinnerViewController.m** и замените его содержимое следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="343e9-233">Open **SpinnerViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SpinnerViewController.m" id="SpinnerViewSnippet":::

## <a name="test-the-app"></a><span data-ttu-id="343e9-234">Тестирование приложения</span><span class="sxs-lookup"><span data-stu-id="343e9-234">Test the app</span></span>

<span data-ttu-id="343e9-235">Сохраните изменения и запустите приложение.</span><span class="sxs-lookup"><span data-stu-id="343e9-235">Save your changes and launch the app.</span></span> <span data-ttu-id="343e9-236">Вы должны иметь возможность перемещаться между  экранами  с помощью кнопок "Вход" и "Выход" и панели вкладок.</span><span class="sxs-lookup"><span data-stu-id="343e9-236">You should be able to move between the screens using the **Sign In** and **Sign Out** buttons and the tab bar.</span></span>

![Снимки экрана приложения](./images/app-screens.png)
