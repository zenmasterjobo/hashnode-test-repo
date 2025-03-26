---
title: "Announcing Mobile Payments SDK GA and New Terminal API Features"
datePublished: Mon Feb 24 2025 07:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm8q3laot000h0ajp3g6ubnfa
slug: announcing-mobile-payments-sdk-ga-and-new-terminal-api-features
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743004021064/a8d17cb4-f6e2-4d75-8b6f-0b51267c2586.jpeg

---


When we opened our platform and provided developers with direct access to Square hardware, our goal was simple: to empower you to build custom, secure in-person payment solutions to meet the unique needs of sellers. Over the years, the Square Reader SDK and Terminal API have become flagship products, with hundreds of developers building integrations that enable thousands of sellers to use the Square Reader, Square Stand, and Square Terminal with their preferred third-party apps.

Today, we’re excited to introduce the next phase of innovation for these solutions. We’ve officially launched the Mobile Payments SDK – the successor to the Reader SDK – into general availability, along with new Terminal API features. These updates bring expanded market support, deeper integrations with the Square ecosystem, and enhanced capabilities to simplify your in-person checkout experiences and broaden your seller reach.

### Embed Payments Seamlessly with the Mobile Payments SDK - Now Generally Available

The Mobile Payments SDK lets you embed the Square payment flow into your iOS or Android mobile applications and integrate with various Square hardware options, like the Square Reader or Square Stand to process payments. The SDK makes it easy for your app to deliver a comprehensive in-person payment solution, fully integrated with Square’s commerce tools - all with minimal effort.

When we first launched in beta in June 2024, we introduced a powerful set of capabilities designed to supercharge your payment experience.

![1145 PersonalTraining Williamsburg 1842 01 NP SIMP horizontal crop 01](//images.ctfassets.net/1wryd5vd9xez/5JRb07OevsNybxkluh9kLb/337a00d803fb5c29b8563a0e32f58401/1145_PersonalTraining_Williamsburg_1842_01_NP_SIMP_horizontal_crop_01.png)

### Build a reliable payments experience

Support **offline payments** and ensure sellers can continue processing transactions and never miss a sale, even if their devices lose network connection or they experience an outage. Plus, our integrated **Reader Management API** keeps you informed about the health and status of connected Square Readers, such as their battery level, software version, serial numbers, and more, enabling you to address issues quickly and minimize downtime.

### Streamline payments and orders with Square integrations

Integrate with the [Orders API](https://developer.squareup.com/docs/orders-api/what-it-does) and [Payments API](https://developer.squareup.com/docs/payments-refunds) to connect with other parts of the Square ecosystem, unlocking a more unified orders and payments experience. Display itemized line items (such as discounts or taxes) within the checkout flow and on localized receipts to ensure buyer transparency, send orders directly to the Square Point of Sale and Square Kitchen Display System for seamless fulfillment, provide greater control over payments with delayed capture, and more.

### Test with ease and confidence

Test your integration in the Square sandbox by simulating transactions with a virtual reader device - no physical card reader needed - ensuring your integration is fully ready before going live.

After months of beta testing, we’ve taken your feedback and refined the Mobile Payments SDK to make it even more remarkable. The result? Brand new features and expanded availability, designed to help you reach new markets, offer greater payment flexibility, and streamline development. Here’s what’s new:

### Scale across the U.S., Canada, and Australia

As your business grows, expanding into new markets can be essential for unlocking revenue opportunities and reaching more customers. The Mobile Payments SDK enables you to extend your global reach and build for sellers in leading markets like the U.S., Canada, and Australia. Payment methods such as Interac in Canada and EFTPOS in Australia are supported by the SDK as well.

<blockquote>
  DH Pace has 50 locations nationwide and uses the Mobile Payments SDK to enable their custom mobile app to accept payments in the field for residential door installation and maintenance services.

"The Mobile Payments SDK comes with pre-built payments screens, so we didn’t have to develop our own. They were great accelerators for our integration - you just trigger a payment and it pops up a standard payment form. The branding on them is limited, and if we did need to customize it, all the options are there. This just got us to market a lot faster."

– Miles Rush, Mobile Development Manager at DH Pace
</blockquote>

### Enable flexible payments with Tap to Pay on iPhone and Android (Beta)

We’ve made it simpler, more convenient, and cost-effective for sellers to accept payments via the Mobile Payments SDK with Tap to Pay on iPhone and Tap to Pay on Android support. Now, sellers using your Mobile Payments SDK integration can run your app on compatible iPhone or Android phones and accept contactless payments directly on these devices – no external hardware needed. Buyers can hold a contactless card or a mobile wallet over the seller’s device to complete the transaction.

![image2](//images.ctfassets.net/1wryd5vd9xez/1WvYilZBpHzxNVYN1kjSVo/454068ffc8eb69073a68f636e86bee3e/image2.png)

### Accelerate development with Flutter and React Native plugins

Integrating with the Mobile Payments SDK is now simpler and more efficient with our newly built [Flutter](https://github.com/square/mobile-payments-sdk-flutter) and [React Native](https://github.com/square/mobile-payments-sdk-react-native) plugins. You can now use a single codebase to seamlessly deploy your integration across both iOS and Android, saving time and resources.

Many developers have already started building powerful custom solutions with the Mobile Payments SDK to process payments at a kiosk, on the move, or over-the-counter:

* **Lumina:** Built a self-service kiosk app that enables them to use Square Reader for accepting kiosk payments.
* **DH Pace:** Built a custom mobile app that allows them to use Square Reader to accept field service payments.
* **TicketSocket:** Ticketing software that uses Square Stand and Square Reader to accept ticket payments on the move or over-the-counter. 

## How to start a payment

Taking a payment with Mobile Payments SDK is simple and straightforward. Follow our build guides for [Android](https://developer.squareup.com/docs/mobile-payments-sdk/android) and [iOS](https://developer.squareup.com/docs/mobile-payments-sdk/ios) to initialize and authorize the SDK, then call `startPayment(...)` with a set of parameters to leverage the out-of-box payment UI or customize for your use case. 

To include **Tap to Pay on iPhone** as a payment option in the default payment prompt, add `.tapToPay` or `.all` in your `PromptParameters` and listen for delegate callbacks as the SDK progresses through the payment flow.

```swift
let paymentParameters = PaymentParameters(
    idempotencyKey: idempotencyKey,
    amountMoney: Money(amount: 100, currency: .USD)
)

let promptParameters = PromptParameters(
    mode: .default,
    additionalMethods: .tapToPay
)

mobilePaymentsSDK.paymentManager.startPayment(
    paymentParameters,
    promptParameters: promptParameters,
    from: viewHolder.controller,
    delegate: viewModel
)
```

## Start Building

The Mobile Payments SDK is generally available to developers and partners serving sellers in the U.S., Canada, and Australia. 

To start building, check out our [developer documentation](https://developer.squareup.com/docs/mobile-payments-sdk), [iOS technical reference](https://developer.squareup.com/docs/sdk/mobile-payments/ios), [Android technical reference](https://developer.squareup.com/docs/sdk/mobile-payments/android), and [payments pricing](https://developer.squareup.com/docs/payments-pricing). You can also view our [iOS Sample App](https://github.com/square/mobile-payments-sdk-ios) and [Android Sample App](https://github.com/square/mobile-payments-sdk-android).

**Using the Reader SDK?** [Migrate](https://developer.squareup.com/docs/mobile-payments-sdk/migrate) to the Mobile Payments SDK by **December 31, 2025**. The Reader SDK is officially deprecated and will no longer receive new releases, and support for new operating systems is not guaranteed. After the deadline, the Reader SDK will be retired.

## Expand Globally with Terminal API Features - Now in More Markets

The Terminal API allows you to connect apps built on your preferred mobile or web platform with the Square Terminal to accept in-person payments. Responding to valuable feedback we’ve gathered from developers, we’re expanding Terminal API capabilities to more global markets and introducing new region-specific features. 

We’re excited to roll out several highly requested capabilities, including Orders API integration across Canada, Australia, and the United Kingdom, and multi-brand QR code payments in Japan. 

![ph220117L NappilyNaturals 0647 03 horizontal crop 01 (1)](//images.ctfassets.net/1wryd5vd9xez/4OnxZxJCFRoy9ZEcLC1BzK/ddbe09d35b65136d16676652687b4170/ph220117L_NappilyNaturals_0647_03_horizontal_crop_01__1_.png)

### Create cohesive orders and payments experiences across markets

You can now integrate the Terminal API with the [Orders API](https://developer.squareup.com/docs/orders-api/what-it-does) to extend your global reach beyond the U.S. and deliver a more unified orders and payments experience for sellers in key markets like Canada, Australia, and the United Kingdom. With this  integration, you can:

### Accept a payment against an order

Associate an Order ID with a transaction made on the Square Terminal, enabling you to send the order and payment directly to the Square Point of Sale and Square Kitchen Display System for fulfillment. Plus, easily retrieve all order details related to the payment to ensure accuracy in the order process and reporting.

### Itemize order details

Sync Square orders data with Terminals and display itemized line items (such as discounts or taxes) on the devices and on localized email or text message receipts, ensuring buyer transparency at checkout. Receipts can be customized based on the seller's location and currency settings.

<div style="display: flex; gap: 1rem;">
  <img src="//images.ctfassets.net/1wryd5vd9xez/1st6i11RscCmSCZ1JmQuSi/539ad17cfbe0d86f58c2423326792005/image4.png" alt="image4" style="width: 34%;"/>
  <img src="//images.ctfassets.net/1wryd5vd9xez/4Z04HnCOe0g8lgfHdKT0EM/4830785869c2a686f6c9b64b887e4b83/image3.png" alt="image3" style="width: 64%;"/>
</div>

<blockquote>
KAIKAKU uses the Terminal API and Orders API to build a custom food-ordering kiosk, enabling them to launch the UK’s first fully robotic kitchen in London.

“Square has been amazing to work with, pushing out features that we’ve been desiring for months. It’s rare to find a company so willing to work with customers. This has massively streamlined operations, and has made the Square Terminal a game changer for financial analysis.”

– Gian-Luca Fenocchi, Lead Software Engineer at KAIKAKU
</blockquote>

### Integrating the Orders API

Start with an Order in an OPEN state from the Orders API. Then simply include the `order_id` value when creating a Terminal Checkout, and make sure the checkout’s `amount` matches the Order total.  If the buyer successfully completes the payment for this Terminal Checkout, both the Terminal Checkout and Order will move to a successful `COMPLETED` state.

To make the most of our itemized Order features, make sure to set `show_itemized_cart` to `true` and `skip_receipt_screen` to `false`. 

```bash
curl https://connect.squareup.com/v2/terminals/checkouts \
  -X POST \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'Content-Type: application/json' \
  -d '{
    "idempotency_key": "5f2e1b9c-86a6-4310-88d2-c0b49a84df66",
    "checkout": {
        "amount_money": {
            "amount": 500,
            "currency": "GBP"
        },
        "device_options": {
            "device_id": "R5WNWB5BKNG9R",
            "show_itemized_cart": true,
            "skip_receipt_screen": false
        },
        "order_id": "7piQ5cvmkIYaUuv8XLULddDhefUZY"
    }
  }'
```

### Streamline payments with multi-brand QR code payments in Japan

With the rise of QR code payments in Japan, Japanese sellers need a simplified way to accept multiple payment methods. Integrating various payment options into your app can be complex, but the Terminal API simplifies this. Now, you can enable Japanese sellers to display a single QR code on Square Terminals to accept **seven major payment methods**: RakutenPay, AliPay+, WeChat Pay, PayPay, dBarai, Merpay, and au Pay.

![image1](//images.ctfassets.net/1wryd5vd9xez/4vh9Rjwn6MAxLXvvqHULP2/d852367396ae5a2ddf04ca55c89f99f9/image1.gif)

### Adding multi-brand QR code payments

To start, simply create a Terminal Checkout with `payment_type` set to `QR_CODE`. The Square Terminal will display a multi-brand QR code that supports all QR payment methods enabled at the target location.

```bash
curl https://connect.squareup.com/v2/terminals/checkouts \
  -X POST \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'Content-Type: application/json' \
  -d '{
    "idempotency_key": "5f2e1b9c-86a6-4310-88d2-c0b49a84df66",
    "checkout": {
      "payment_type": "QR_CODE",
      "amount_money": {
        "amount": 100,
        "currency": "JPY"
      },
      "device_options": {
        "device_id": "R5WNWB5BKNG9R"
      }
    }
  }'
```

## Start Building

The Terminal API orders integration is generally available in the U.S., Canada, Australia, and United Kingdom, while multi-brand QR code payments is available in Japan.

To start building, check out our [developer documentation](https://developer.squareup.com/docs/terminal-api/square-terminal-payments), [technical reference](https://developer.squareup.com/reference/square/terminal-api), and [payments pricing](https://developer.squareup.com/docs/payments-pricing).

As always, please share your feedback in our [community Discord channel](https://discord.com/invite/squaredev) or [Square Developer Forums](https://developer.squareup.com/forums). If you want to keep up to date with the rest of our content, be sure to follow this [blog](https://developer.squareup.com/blog) and our [X](https://twitter.com/SquareDev) account.