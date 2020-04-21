<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph. Для этого необходимо интегрировать [библиотеку проверки подлинности Microsoft (MSAL) для iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) в приложение.

1. Создайте новый файл со **списком свойств** в проекте **Графтуториал** с именем **ауссеттингс. plist**.
1. Добавьте следующие элементы в файл **корневого** словаря.

    | Key | Тип | Значение |
    |-----|------|-------|
    | `AppId` | String | Идентификатор приложения на портале Azure |
    | `GraphScopes` | Массив | Два строковых значения: `User.Read` и`Calendars.Read` |

    ![Снимок экрана: файл Ауссеттингс. plist в Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> Если вы используете систему управления версиями (например, Git), то теперь будет полезно исключить файл **ауссеттингс. plist** из системы управления версиями, чтобы избежать случайной утечки идентификатора приложения.

## <a name="implement-sign-in"></a>Реализация входа

В этом разделе описывается настройка проекта для MSAL, создание класса диспетчера проверки подлинности и обновление приложения для входа и выхода.

### <a name="configure-project-for-msal"></a>Настройка проекта для MSAL

1. Добавьте новую группу цепочки ключей в возможности проекта.
    1. Выберите проект **графтуториал** , а затем **подписывать & возможности**.
    1. Выберите пункт **+ возможность**, а затем дважды щелкните **общий доступ к цепочке ключей**.
    1. Добавьте группу цепочки ключей со значением `com.microsoft.adalcache`.

1. Нажмите кнопку **info. plist** и выберите **Открыть как**, а затем **Исходный код**.
1. Добавьте следующий `<dict>` элемент в элемент.

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

1. Откройте **аппделегате. m** и добавьте приведенный ниже оператор import в начало файла.

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. Добавьте к классу `AppDelegate` следующую функцию:

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>Создание диспетчера проверки подлинности

1. Создайте новый **класс Touch Cocoa** в проекте **Графтуториал** с именем **AuthenticationManager**. Выберите **NSObject** в **подклассе** поля.
1. Откройте **AuthenticationManager. h** и замените его содержимое приведенным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. Откройте **AuthenticationManager. m** и замените его содержимое приведенным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>Добавление входа и выхода

1. Откройте файл **сигнинвиевконтроллер. m** и замените его содержимое приведенным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. Откройте **велкомевиевконтроллер. m** и добавьте следующий `import` оператор в начало файла.

    ```objc
    #import "AuthenticationManager.h"
    ```

1. Замените имеющуюся функцию `signOut` указанным ниже кодом.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. Сохраните изменения и перезапустите приложение в симуляторе.

Если вы входите в приложение, в окне вывода в Xcode должен отображаться маркер доступа.

![Снимок экрана: окно вывода в Xcode, в котором отображается маркер доступа](./images/access-token-output.png)

## <a name="get-user-details"></a>Получение сведений о пользователе

В этом разделе описывается создание вспомогательного класса для хранения всех вызовов Microsoft Graph и обновление `WelcomeViewController` для использования этого нового класса для получения пользователя, выполнившего вход в систему.

1. Создайте новый **класс Touch Cocoa** в проекте **Графтуториал** с именем **графманажер**. Выберите **NSObject** в **подклассе** поля.
1. Откройте **графманажер. h** и замените его содержимое приведенным ниже кодом.

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import <MSGraphClientModels/MSCollection.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user, NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock)completionBlock;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. Откройте **графманажер. m** и замените его содержимое приведенным ниже кодом.

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

    - (void) getMeWithCompletionBlock:(GetMeCompletionBlock)completionBlock {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me", MSGraphBaseURL];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completionBlock(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completionBlock(nil, graphError);
                } else {
                    completionBlock(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. Откройте **велкомевиевконтроллер. m** и добавьте приведенные `#import` ниже операторы в начало файла.

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. Добавьте в объявление `WelcomeViewController` интерфейса следующее свойство:

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. Замените существующий `viewDidLoad` код на следующий.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

Если вы сохраните изменения и перезапустите приложение, после того как вы обновите пользовательский интерфейс, отобразите отображаемое имя и адрес электронной почты пользователя.
