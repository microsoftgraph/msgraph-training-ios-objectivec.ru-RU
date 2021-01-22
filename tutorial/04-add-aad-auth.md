<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="aa717-101">В этом упражнении вы расширим приложение из предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="aa717-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="aa717-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="aa717-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="aa717-103">Для этого в приложение будет интегрирована библиотека проверки подлинности [Майкрософт (MSAL) для iOS.](https://github.com/AzureAD/microsoft-authentication-library-for-objc)</span><span class="sxs-lookup"><span data-stu-id="aa717-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="aa717-104">Создайте файл **списка свойств** в **проекте GraphTutorial** **с именем AuthSettings.plist.**</span><span class="sxs-lookup"><span data-stu-id="aa717-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="aa717-105">Добавьте следующие элементы в файл в **корневом словаре.**</span><span class="sxs-lookup"><span data-stu-id="aa717-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="aa717-106">Key</span><span class="sxs-lookup"><span data-stu-id="aa717-106">Key</span></span> | <span data-ttu-id="aa717-107">Тип</span><span class="sxs-lookup"><span data-stu-id="aa717-107">Type</span></span> | <span data-ttu-id="aa717-108">Значение</span><span class="sxs-lookup"><span data-stu-id="aa717-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="aa717-109">String</span><span class="sxs-lookup"><span data-stu-id="aa717-109">String</span></span> | <span data-ttu-id="aa717-110">ИД приложения на портале Azure</span><span class="sxs-lookup"><span data-stu-id="aa717-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="aa717-111">Массив</span><span class="sxs-lookup"><span data-stu-id="aa717-111">Array</span></span> | <span data-ttu-id="aa717-112">Три строковые значения: `User.Read` , `MailboxSettings.Read` и `Calendars.ReadWrite`</span><span class="sxs-lookup"><span data-stu-id="aa717-112">Three String values: `User.Read`, `MailboxSettings.Read`, and `Calendars.ReadWrite`</span></span> |

    ![Снимок экрана: файл AuthSettings.plist в Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="aa717-114">Если вы используете управление исходным кодом, например git, пришло бы время исключить файл **AuthSettings.plist** из системы управления исходным кодом, чтобы не допустить случайной утечки вашего ИД приложения.</span><span class="sxs-lookup"><span data-stu-id="aa717-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="aa717-115">Реализация входов</span><span class="sxs-lookup"><span data-stu-id="aa717-115">Implement sign-in</span></span>

<span data-ttu-id="aa717-116">В этом разделе мы настроим проект для MSAL, создадим класс диспетчера проверки подлинности и обновим приложение для входов и выходов.</span><span class="sxs-lookup"><span data-stu-id="aa717-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="aa717-117">Настройка проекта для MSAL</span><span class="sxs-lookup"><span data-stu-id="aa717-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="aa717-118">Добавьте новую группу ключей в возможности проекта.</span><span class="sxs-lookup"><span data-stu-id="aa717-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="aa717-119">Выберите проект **GraphTutorial,** а затем **& подписываю.**</span><span class="sxs-lookup"><span data-stu-id="aa717-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="aa717-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span><span class="sxs-lookup"><span data-stu-id="aa717-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="aa717-121">Добавьте группу ключей с этим `com.microsoft.adalcache` значением.</span><span class="sxs-lookup"><span data-stu-id="aa717-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="aa717-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span><span class="sxs-lookup"><span data-stu-id="aa717-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="aa717-123">Добавьте в элемент `<dict>` следующее:</span><span class="sxs-lookup"><span data-stu-id="aa717-123">Add the following inside the `<dict>` element.</span></span>

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
      </dict>
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. <span data-ttu-id="aa717-124">Откройте **файл AppDelegate.m** и добавьте в верхней части файла следующую выписку по импорту.</span><span class="sxs-lookup"><span data-stu-id="aa717-124">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="aa717-125">Добавьте к классу `AppDelegate` следующую функцию:</span><span class="sxs-lookup"><span data-stu-id="aa717-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="aa717-126">Создание диспетчера проверки подлинности</span><span class="sxs-lookup"><span data-stu-id="aa717-126">Create authentication manager</span></span>

1. <span data-ttu-id="aa717-127">Создайте новый класс **Cocoa Touch в** **проекте GraphTutorial** **с именем AuthenticationManager.**</span><span class="sxs-lookup"><span data-stu-id="aa717-127">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="aa717-128">Выберите **NSObject в** **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="aa717-128">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="aa717-129">Откройте **AuthenticationManager.h** и замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="aa717-129">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. <span data-ttu-id="aa717-130">Откройте **AuthenticationManager.m и** замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="aa717-130">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="aa717-131">Добавление и выход из нее</span><span class="sxs-lookup"><span data-stu-id="aa717-131">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="aa717-132">Откройте файл **SignInViewController.m** и замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="aa717-132">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. <span data-ttu-id="aa717-133">Откройте **WelcomeViewController.m** и добавьте следующий выписки в `import` верхней части файла.</span><span class="sxs-lookup"><span data-stu-id="aa717-133">Open **WelcomeViewController.m** and add the following `import` statement to the top of the file.</span></span>

    ```objc
    #import "AuthenticationManager.h"
    ```

1. <span data-ttu-id="aa717-134">Замените имеющуюся функцию `signOut` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="aa717-134">Replace the existing `signOut` function with the following.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. <span data-ttu-id="aa717-135">Сохраните изменения и перезапустите приложение в симуляторе.</span><span class="sxs-lookup"><span data-stu-id="aa717-135">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="aa717-136">При входе в приложение маркер доступа должен отображаться в окне выходных данных в Xcode.</span><span class="sxs-lookup"><span data-stu-id="aa717-136">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Снимок экрана: окно вывода в Xcode с маркером доступа](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="aa717-138">Получить сведения о пользователе</span><span class="sxs-lookup"><span data-stu-id="aa717-138">Get user details</span></span>

<span data-ttu-id="aa717-139">В этом разделе вы создадим дополнительный класс для удержания всех вызовов Microsoft Graph и обновим его, чтобы использовать этот новый класс для получения во `WelcomeViewController` входе пользователя.</span><span class="sxs-lookup"><span data-stu-id="aa717-139">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="aa717-140">Создайте новый класс **Cocoa Touch в** **проекте GraphTutorial** **с именем GraphManager.**</span><span class="sxs-lookup"><span data-stu-id="aa717-140">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="aa717-141">Выберите **NSObject в** **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="aa717-141">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="aa717-142">Откройте **GraphManager.h** и замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="aa717-142">Open **GraphManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import <MSGraphClientModels/MSCollection.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user,
                                         NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock) completion;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="aa717-143">Откройте **GraphManager.m и** замените его содержимое на следующий код.</span><span class="sxs-lookup"><span data-stu-id="aa717-143">Open **GraphManager.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "GraphManager.h"

    @interface GraphManager()

    @property MSHTTPClient* graphClient;

    @end

    @implementation GraphManager

    + (id) instance {
        static GraphManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {
            // Create the Graph client
            self.graphClient = [MSClientFactory
                                createHTTPClientWithAuthenticationProvider:AuthenticationManager.instance];
        }

        return self;
    }

    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock) completion {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me?%@",
                                 MSGraphBaseURL,
                                 @"$select=displayName,mail,mailboxSettings,userPrincipalName"];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completion(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completion(nil, graphError);
                } else {
                    completion(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. <span data-ttu-id="aa717-144">Откройте **WelcomeViewController.m** и добавьте следующие утверждения в `#import` верхней части файла.</span><span class="sxs-lookup"><span data-stu-id="aa717-144">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="aa717-145">Добавьте следующее свойство в объявление `WelcomeViewController` интерфейса.</span><span class="sxs-lookup"><span data-stu-id="aa717-145">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="aa717-146">Замените `viewDidLoad` существующий следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="aa717-146">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

<span data-ttu-id="aa717-147">Если вы сохраните изменения и перезапустите приложение, после того как вы войдите в пользовательский интерфейс, обновите его отображаемого имени и адрес электронной почты.</span><span class="sxs-lookup"><span data-stu-id="aa717-147">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
