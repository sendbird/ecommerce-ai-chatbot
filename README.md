
# [Sendbird](https://sendbird.com) Sendbird X ChatGPT E-Commerce AI Chatbot Demo

[![Platform](https://img.shields.io/badge/platform-iOS-orange.svg)](https://cocoapods.org/pods/SendBirdUIKit)
[![Languages](https://img.shields.io/badge/language-Swift-orange.svg)](https://github.com/sendbird/sendbird-uikit-ios)
[![Commercial License](https://img.shields.io/badge/license-Commercial-green.svg)](https://github.com/sendbird/sendbird-uikit-ios/blob/main/LICENSE.md)

This demo app showcases what AI Chatbots with Sendbird can do to enhance the customer experience of your service with more personalized and comprehensive customer support.
Utilizing OpenAI’s GPT3.5 and its Function Calling functionality, ***Sendbird helps you build a chatbot that can go extra miles: providing informative responses with the data source you feed to the bot, accommodating customer’s requests such as tracking and canceling their orders, and even recommending new products.*** Create your own next generation AI Chatbot by following the tutorial below.

![final_output](https://github.com/sendbird/ecommerce-ai-chatbot/assets/104121286/f191c66a-2642-4269-ab92-a3567da618aa)

## Prerequisites
1. [Sendbird Account](https://dashboard.sendbird.com/)
2. Application ID and ChatBot: Please refer to [Step1 ~ Step4](https://sendbird.com/developer/tutorials/create-an-ai-chatbot)

## How to open the demo app
1. Open Xcode Demo project
```shell
open Sample/QuickStart.xcodeproj
```
2. Set the `applicationId` and `botId` in [`AppDelegate.swift`](https://github.com/sendbird/ecommerce-ai-chatbot/blob/develop/Sample/QuickStart/AppDelegate.swift#L13-L23)
```swift
static let botId: String = <#botId: String#>
```

```swift
SendbirdUI.initialize(applicationId: <#applicationID: String#>)
```

## Table of Contents
1. [Use case: E-commerce](##use-case-e-commerce)
2. [How it works](##how-it-works)
3. [Demo app settings](##demo-app-settings)
4. [System Message and Function Calling](##system-message-and-function-calling)
5. [Welcome Message and Quick Replies](##welcome-message-and-quick-replies)
6. [UI Components](##ui-components)
7. [Limitations](##limitations)

## Use case: E-commerce
This demo app demonstrates the implementation of the AI Chatbot tailored for e-commerce. It includes functionalities such as ***retrieving the order list, showing order details, canceling orders, and providing recommendations.*** By leveraging ChatGPT’s new feature [Function Calling](https://openai.com/blog/function-calling-and-other-api-updates), the Chatbot now can make an API request to the 3rd party with a predefined Function Calling based on customer’s enquiry. Then it parses and presents the response in a conversational manner, enhancing overall customer experience.

## How it works
<img width="2556" alt="image" src="https://github.com/sendbird/sendbird-uikit-ios/assets/104121286/12a8cb5f-8127-41cb-9570-3c979f977ad4">

1. A customer sends a message containing a specific request on the client app to the Sendbird server.
2. The Sendbird server then delivers the message to Chat GPT.
3. Chat GPT then analyzes the message and determines that it needs to make a Function Calling request. In response, it delivers the relevant information to the Sendbird server.
4. The Sendbird server then sends back a Function Calling request to Chat GPT. The request contains both Function Calling data Chat GPT previously sent and the 3rd party API information provided by the client app. This configuration must be set on the client-side.
5. The 3rd party API returns a response in `data` to the Sendbird server.
6. The Sendbird server passes the `data` to Chat GPT.
7. Once received, Chat GPT analyzes the data and returns proper responses to the Sendbird server as `data`.
8. The Sendbird server passes Chat GPT’s answer to the client app.
9. The client app can process and display the answer with Sendbird Chat UIKit.

***Note***: Currently, calling a 3rd party function is an experimental feature, and some logics are handled on the client-side for convenience purposes. Due to this, the current version for iOS (3.7.0 beta) will see breaking changes in the future, especially for QuickReplyView and CardView. Also, the ad-hoc support from the server that goes into the demo may be discontinued at any time and will be replaced with a proper feature on Sendbird Dashboard in the future.

## Demo app settings
## Demo app settings
To run the demo app, you must specify `System prompt`, `Function Calls`.

### System Message
`System prompt` defines the Persona of the ChatBot, informing users of the role the ChatBot plays. For this E-Commerce AI ChatBot, It's designed to be an AI assistant that handles and manages customer orders.

You can find this setting under Chat > AI Chatbot > Manage bots > Edit > Bot settings > Parameter settings > System prompt.

- Input example
  

```
You are an AI assistant that handles and manages customer orders. You will be interacting with customers who have the orders.

1. Available 24/7 to assist customers with their order inquiries.
2. Customers may request to check the status of their orders or cancel them.
3. You have access to the customer's order list and the order details associated with it.
4. When a customer requests to cancel an order, you need to confirm the specific order number from their order list before proceeding.
5. Ensure confirmation for the cancellation to the customer once it has been processed successfully.
If a customer needs further assistance after order cancellation, be ready to provide it
```

### Function Calls
`Function Calls` allows you to define situations where the ChatBot needs to interface with external APIs. Within `Function Calls`, you need to enter definitions for the Function and Parameters to pass to GPT, and you can define the specs of the 3rd party API to obtain the actual data of the specified Function.

You can find this setting under Chat > AI Chatbot > Settings > Function Calls. 

- Example list of Function Calls
  

- Input example
  

In addition, you can enhance user experience by streamlining the communication with a Welcome Message, Quick Replies and Button. Using Quick Replies can improve the clarity of your customer’s intention as they are presented with a list of predefined options determined by you.

Mock API Server Information: [Link](https://documenter.getpostman.com/view/21816899/2s9Y5eMK9o)

## Welcome Message and Quick Replies
## Welcome Message and Suggested Replies


The `Welcome Message` is the first message displayed to users by the chatbot. Along with the `Welcome Message`, you can also set up `Suggested Replies` from the Dashboard.

You can find this setting under Chat > AI Chatbot > Manage bots > Edit > Bot settings > Default messages > Welcome message / Suggested replies.

- Input example
  


## UI Components
### CardView
The `data` in the response are displayed in a Card view. In the demo, information such as order items and their delivery status can be displayed in a card with an image, title, and description. Customization of the view can be done through `cardViewParamsCollectionBuilder` and `SBUCardViewParams`. The following codes show how to set the Card view of order status.

[SBUUserMessageCell.swift](https://github.com/sendbird/ecommerce-ai-chatbot/blob/develop/Sources/View/Channel/MessageCell/SBUUserMessageCell.swift#L159)
```swift
// MARK: Card List
if let cardListView = self.cardListView {
    self.contentVStackView.removeArrangedSubview(cardListView)
}

let functionResponse = json["function_calls"][0]

if functionResponse.type != .null {
    let statusCode = functionResponse["status_code"].intValue
    let functionName = functionResponse["name"].stringValue
    let response = functionResponse["response_text"]

    if statusCode == 200 {
        if functionName.contains("get_order_list") {
            customText = "Here is your order list"
            SBUGlobalCustomParams.cardViewParamsCollectionBuilder = { messageData in
                guard let json = try? JSON(parseJSON: messageData) else { return [] }

                return json.arrayValue.compactMap { order in
                    let deliveryStatus = order["status"].stringValue
                    var icon: String = ""

                    switch deliveryStatus {
                    case "delivered":
                        icon = "✅"
                    case "delivering":
                        icon = "🚚"
                    case "preparing":
                        icon = "⏳"
                    default:
                        break
                    }

                    let titleWithIcon = icon.isEmpty ? "Order #\(order["id"].stringValue)" : "\(icon) Order #\(order["id"].stringValue)"

                    return SBUCardViewParams(
                        imageURL: nil,
                        title: titleWithIcon,
                        subtitle: "Your Order \(deliveryStatus)",
                        description: "Items:" + ((order["items"].arrayObject as? [String])?.joined(separator: ", "))!,
                        link: nil
                    )
                }
            }
            if let items = try?SBUGlobalCustomParams.cardViewParamsCollectionBuilder?(response.rawString()!){
                self.addCardListView(with: items)
            }
        } else if functionName.contains("get_order_details") {
            customText = "Here is your order details"
            SBUGlobalCustomParams.cardViewParamsCollectionBuilder = { messageData in
                guard let json = try? JSON(parseJSON: messageData) else { return [] }

                // Convert the single order object into a SBUCardViewParams object
                let orderParams = SBUCardViewParams(
                    imageURL: nil,
                    title: "Order #\(json["id"].stringValue) by \(json["customer_name"].stringValue)",
                    subtitle: "- Status: \(json["status"].stringValue)\n- Estimated Delivery Date: \(json["estimatedDeliveryDate"].stringValue)",
                    description: "- Items: " + ((json["items"].arrayObject as? [String])?.joined(separator: ", "))! + "\n- Total Price: $\(json["purchasePrice"].intValue)",
                    link: nil
                )

                // Return the SBUCardViewParams object inside an array
                return [orderParams]
            }
            if let items = try?SBUGlobalCustomParams.cardViewParamsCollectionBuilder?(response.rawString()!){
                self.addCardListView(with: items)
            }
        } else if functionName.contains("get_recommendation") {
            customText = "Here are some recommended items for you"
            disableWebview = true
            SBUGlobalCustomParams.cardViewParamsCollectionBuilder = { messageData in
                guard let json = try? JSON(parseJSON: messageData) else { return [] }

                return json.arrayValue.compactMap { item in
                    return SBUCardViewParams(
                            imageURL: item["image"].stringValue,
                            title: item["name"].stringValue,
                            subtitle: "$\(item["price"].intValue)",
                            description: nil,
                            link: nil
                    )
                }
            }
            if let items = try?SBUGlobalCustomParams.cardViewParamsCollectionBuilder?(response.rawString()!){
                self.addCardListView(with: items)
            }
        }

    }
} else {
    self.cardListView = nil
}

```

### QuickReplyView
The following codes demonstrate how to set the view for Quick Replies. The values in `options` of `first_message_data.data` are used as Quick Replies.

[SBUUserMessageCell.swift](https://github.com/sendbird/ecommerce-ai-chatbot/blob/develop/Sources/View/Channel/MessageCell/SBUUserMessageCell.swift#L150)
```swift
// MARK: Quick Reply        
if let quickReplyView = self.quickReplyView {
    quickReplyView.removeFromSuperview()
    self.quickReplyView = nil
}

if let quickReplies = json["quick_replies"].arrayObject as? [String], !quickReplies.isEmpty {
    self.updateQuickReplyView(with: quickReplies)
}
```

## Limitations
`Tokens`
The maximum number of tokens allowed for `data` is 4027. Make sure that the settings information including the system message does not exceed the limit.
