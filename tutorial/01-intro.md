<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="e6caf-101">В этом руководстве рассказывается, как создать собственное приложение, которое использует API Microsoft Graph для получения сведений о календаре для пользователя.</span><span class="sxs-lookup"><span data-stu-id="e6caf-101">This tutorial teaches you how to build an React Native app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="e6caf-102">Если вы предпочитаете просто скачать заполненный учебник, вы можете скачать или клонировать [репозиторий GitHub](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span><span class="sxs-lookup"><span data-stu-id="e6caf-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="e6caf-103">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="e6caf-103">Prerequisites</span></span>

<span data-ttu-id="e6caf-104">Прежде чем приступить к работе с этим руководством, на вашем компьютере для разработки должны быть установлены следующие компоненты.</span><span class="sxs-lookup"><span data-stu-id="e6caf-104">Before you start this tutorial, you should have the following installed on your development machine.</span></span>

- [<span data-ttu-id="e6caf-105">Xcode</span><span class="sxs-lookup"><span data-stu-id="e6caf-105">Xcode</span></span>](https://developer.apple.com/xcode/)
- [<span data-ttu-id="e6caf-106">CocoaPods</span><span class="sxs-lookup"><span data-stu-id="e6caf-106">CocoaPods</span></span>](https://cocoapods.org)

> [!NOTE]
> <span data-ttu-id="e6caf-107">Это руководство было написано с помощью Xcode версии 10,3 и CocoaPods версии 1.7.5.</span><span class="sxs-lookup"><span data-stu-id="e6caf-107">This tutorial was written using Xcode version 10.3 and CocoaPods version 1.7.5.</span></span> <span data-ttu-id="e6caf-108">Действия, описанные в этом руководстве, могут работать с другими версиями, но не тестировались.</span><span class="sxs-lookup"><span data-stu-id="e6caf-108">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="e6caf-109">Отзывы</span><span class="sxs-lookup"><span data-stu-id="e6caf-109">Feedback</span></span>

<span data-ttu-id="e6caf-110">Сообщите о нем в [репозиторий GitHub](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span><span class="sxs-lookup"><span data-stu-id="e6caf-110">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span></span>
