---
layout: post
title: This In-App Purchase Has Aready Been Bought
date: '2017-03-22T15:14:00.000-08:00'
author: David Green
tags:
  - WikiText
modified_time: '2017-03-22T15:14:00.000-08:00'
comments: true
---

If you're developing iOS apps with in-app purchases, chances are you've seen this dreaded dialog and wondered why `SKPaymentQueue` isn't calling your `SKPaymentTransactionObserver`.

<img alt="In-App Purchase Dialog: This In-App Purchase has already been bought" src="/images/blog/2017-03/ios_inapp_already_bought.png" class="border center" />

"This In-App Purchase has already been bought.  It will be restored for free."

This scenario can easily occur when an error occurs before calling `SKPaymentQueue.default().finishTransaction(transaction)`.

Apple has designed the payment queue in such a way as to enable apps to fulfill a purchase before completing - with `finishTransaction` being the indication that fulfillment is complete.  In the case that a failure occurs during fulfillment, the app can recover when `SKPaymentQueue` calls the app's observer again.

This is all good and fine - except [many](https://forum.unity3d.com/threads/closed-restorepurchase-on-ios-not-return-processpurchase-callback.392000/) [developers](http://stackoverflow.com/questions/37941466/catch-the-in-app-purchase-has-already-been-bought-event) [have](http://stackoverflow.com/questions/34001868/ios-this-in-app-purchase-has-already-been-bought-pop-up) encountered a situation where this process breaks down.  Their `SKPaymentTransactionObserver` is never called again, and the user is presented with the dreaded "already bought" dialog and no apparent way to fix the problem.

Like many problems in software development, it turns out that the problem is quite simple and completely non-obvious.  `SKPaymentQueue` does indeed call a transaction observer - just not the one in your app.  A library (in my case [Firebase analytics](https://firebase.google.com/docs/analytics/)) is adding its own `SKPaymentTransactionObserver` before your app adds its own observer.  As a result, `SKPaymentQueue` is calling that observer first, and by the time your observer is added, as far as the payment queue is concerned it has already delivered the notification and there's nothing left to do.

To fix the problem, ensure that your app registers its own observer before any third party libraries get a chance to do the same:

        class AppDelegate: UIApplicationDelegate {

            func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

                ...

                // ORDER DEPENDENCY: before analytics
                SKPaymentQueue.default().add(createPaymentTransactionObserver())

                // ORDER DEPENDENCY: do this after store service setup!
                FIRApp.configure()

                return true
            }
            ...
        }

That's all!
