<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы расширим приложение из предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph. Для этого в приложение будет интегрирована библиотека проверки подлинности [Майкрософт (MSAL) для iOS.](https://github.com/AzureAD/microsoft-authentication-library-for-objc)

1. Создайте файл **списка свойств** в **проекте GraphTutorial** **с именем AuthSettings.plist.**
1. Добавьте следующие элементы в файл в **корневом словаре.**

    | Key | Тип | Значение |
    |-----|------|-------|
    | `AppId` | String | ИД приложения на портале Azure |
    | `GraphScopes` | Массив | Три строковые значения: `User.Read` , `MailboxSettings.Read` и `Calendars.ReadWrite` |

    ![Снимок экрана: файл AuthSettings.plist в Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> Если вы используете управление исходным кодом, например git, пришло бы время исключить файл **AuthSettings.plist** из системы управления исходным кодом, чтобы не допустить случайной утечки вашего ИД приложения.

## <a name="implement-sign-in"></a>Реализация входов

В этом разделе мы настроим проект для MSAL, создадим класс диспетчера проверки подлинности и обновим приложение для входов и выходов.

### <a name="configure-project-for-msal"></a>Настройка проекта для MSAL

1. Добавьте новую группу ключей в возможности проекта.
    1. Выберите проект **GraphTutorial,** а затем **& подписываю.**
    1. Select **+ Capability**, then double-click **Keychain Sharing**.
    1. Добавьте группу ключей с этим `com.microsoft.adalcache` значением.

1. Control click **Info.plist** and select **Open As**, then **Source Code**.
1. Добавьте в элемент `<dict>` следующее:

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

1. Откройте **файл AppDelegate.m** и добавьте в верхней части файла следующую выписку по импорту.

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. Добавьте к классу `AppDelegate` следующую функцию:

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>Создание диспетчера проверки подлинности

1. Создайте новый класс **Cocoa Touch в** **проекте GraphTutorial** **с именем AuthenticationManager.** Выберите **NSObject в** **подклассе** поля.
1. Откройте **AuthenticationManager.h** и замените его содержимое на следующий код.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. Откройте **AuthenticationManager.m и** замените его содержимое на следующий код.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>Добавление и выход из нее

1. Откройте файл **SignInViewController.m** и замените его содержимое на следующий код.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. Откройте **WelcomeViewController.m** и добавьте следующий выписки в `import` верхней части файла.

    ```objc
    #import "AuthenticationManager.h"
    ```

1. Замените имеющуюся функцию `signOut` указанным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. Сохраните изменения и перезапустите приложение в симуляторе.

При входе в приложение маркер доступа должен отображаться в окне выходных данных в Xcode.

![Снимок экрана: окно вывода в Xcode с маркером доступа](./images/access-token-output.png)

## <a name="get-user-details"></a>Получить сведения о пользователе

В этом разделе вы создадим дополнительный класс для удержания всех вызовов Microsoft Graph и обновим его, чтобы использовать этот новый класс для получения во `WelcomeViewController` входе пользователя.

1. Создайте новый класс **Cocoa Touch в** **проекте GraphTutorial** **с именем GraphManager.** Выберите **NSObject в** **подклассе** поля.
1. Откройте **GraphManager.h** и замените его содержимое на следующий код.

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

1. Откройте **GraphManager.m и** замените его содержимое на следующий код.

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

1. Откройте **WelcomeViewController.m** и добавьте следующие утверждения в `#import` верхней части файла.

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. Добавьте следующее свойство в объявление `WelcomeViewController` интерфейса.

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. Замените `viewDidLoad` существующий следующим кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

Если вы сохраните изменения и перезапустите приложение, после того как вы войдите в пользовательский интерфейс, обновите его отображаемого имени и адрес электронной почты.
