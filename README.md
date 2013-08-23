#RMStore

A lightweight iOS framework for In-App Purchases.

RMStore adds [blocks](https://github.com/robotmedia/RMStore/blob/master/README.md#storekit-with-blocks) and [notifications](https://github.com/robotmedia/RMStore/README.md#notifications) to StoreKit, plus [receipt verification](https://github.com/robotmedia/RMStore/blob/master/README.md#receipt-verification) and purchase management. Purchasing a product is as simple as:

```objective-c
[[RMStore defaultStore] addPayment:productID success:^(SKPaymentTransaction *transaction) {
    NSLog(@"Purchased!", @"");
} failure:^(SKPaymentTransaction *transaction, NSError *error) {
    NSLog(@"Something went wrong", @"");
}];
```

##Add RMStore to your project

1. Add [`RMStore.h`](https://github.com/robotmedia/RMStore/blob/master/RMStore/RMStore.h) and [`RMStore.m`](https://github.com/robotmedia/RMStore/blob/master/RMStore/RMStore.m)
2. Link `StoreKit.framework`
3. Profit!

##StoreKit with blocks

RMStore adds blocks to all asynchronous StoreKit operations.

###Requesting products

```objective-c
[[RMStore defaultStore] requestProducts:[NSSet setWithArray:@[@"fabulousIdol", "rootBeer", @"rubberChicken"] success:^{
    NSLog(@"Products loaded", @"");
} failure:^(NSError *error) {
    NSLog(@"Something went wrong", @"");
}];
```

###Add payment

```objective-c
[[RMStore defaultStore] addPayment:@"waxLips" success:^(SKPaymentTransaction *transaction) {
    NSLog(@"Product purchased", @"");
} failure:^(SKPaymentTransaction *transaction, NSError *error) {
    NSLog(@"Something went wrong", @"");
}];
```

###Restore transactions

```objective-c
[[RMStore defaultStore] restoreTransactionsOnSuccess:^{
    NSLog(@"Transactions restored", @"");
} failure:^(NSError *error) {
    NSLog(@"Something went wrong", @"");
}];
```

##Notifications

RMStore sends notifications of StoreKit related events and extends `NSNotification` to provide relevant information. To receive them, implement the desired methods of the `RMStoreObserver` protocol and add the observer to `RMStore`.

###Adding and removing the observer

```objective-c
[[RMStore defaultStore] addStoreObserver:self];
...
[[RMStore defaultStore] removeStoreObserver:self];
```

###Products request notifications

```objective-c
- (void)storeProductsRequestFailed:(NSNotification*)notification
{
    NSError *error = notification.storeError;
}

- (void)storeProductsRequestFinished:(NSNotification*)notification { }
```

###Payment transaction notifications

```objective-c
- (void)storePaymentTransactionFailed:(NSNotification*)notification
{
    NSError *error = notification.storeError;
    NSString *productIdentifier = notification.productIdentifier;
    SKPaymentTransaction *transaction = notification.transaction; // Can be nil
}

- (void)storePaymentTransactionFinished:(NSNotification*)notification
{
    NSString *productIdentifier = notification.productIdentifier;
    SKPaymentTransaction *transaction = notification.transaction;
}
```

###Restore transactions notifications

```objective-c
- (void)storeRestoreTransactionsFailed:(NSNotification*)notification;
{
    NSError *error = notification.storeError;
}

- (void)storeRestoreTransactionsFinished:(NSNotification*)notification { }
```

##Receipt verification

While RMStore doesn't perform receipt verification by default, you can provide your own custom verification or use the app-side verification provided by the framework.

###App-side verification

RMStore provides optional app-side receipt verification via `RMStoreLocalReceiptVerificator`. To use it, add [RMStoreLocalReceiptVerificator.h](https://github.com/robotmedia/RMStore/blob/master/RMStore/RMStoreLocalReceiptVerificator.h) and [RMStoreLocalReceiptVerificator.m](https://github.com/robotmedia/RMStore/blob/master/RMStore/RMStoreLocalReceiptVerificator.m) to your project and set the verification delegate (`receiptVerificator`) at startup. For example:

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    _receiptVerificator = [[RMStoreLocalReceiptVerificator alloc] init]; // Keep a reference to the verificator as the below property is weak
    [RMStore defaultStore].receiptVerificator = _receiptVerificator;
    // Your code
    return YES;
}
```

###Custom verification

Apple strongly recommends to use your own server-side verification. To do this, implement the `RMStoreReceiptVerificator` protocol:

```objective-c
- (void)verifyReceiptOfTransaction:(SKPaymentTransaction*)transaction
                           success:(void (^)())successBlock
                           failure:(void (^)(NSError *error))failureBlock;
```

In most cases you will call a web service that performs the same logic than `RMStoreLocalReceiptVerificator`, but on your server. Call `successBlock` if the receipt passes verification, and `failureBlock` in any other case.

You will also need to set the `receiptVerificator` delegate at startup, as indicated above.

##Requirements

RMStore requires iOS 5.0 or above and ARC.

If you are using RMStore in your non-ARC project, you will need to set a `-fobjc-arc` compiler flag on all of the RMStore source files.

##Roadmap

RMStore is in early stage and currently supports consumables and non-consumables only. Future enhancements will include:

* Content download support
* Subcriptions support
* OS X support

If you are looking for something more mature, check out [CargoBay](https://github.com/mattt/CargoBay) or [MKStoreKit](https://github.com/MugunthKumar/MKStoreKit).

##License

 Copyright 2013 [Robot Media SL](http://www.robotmedia.net)
 
 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at
 
 http://www.apache.org/licenses/LICENSE-2.0
 
 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
