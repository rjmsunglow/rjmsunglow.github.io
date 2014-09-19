---
layout: post
title: "Reactive Cocoa -- RAC的一些概念"
date: 2014-09-19 09:14
comments: true
categories: 
---
ReactiveCocoa是Github开源的一款cocoa FRP 框架.
	
Native app有很大一部分的时间是在等待事件发生，然后响应事件，比如等待网络请求完成，等待用户的操作，等待某些状态值的改变等等，等这些事件发生后，再做进一步处理。 但是这些等待和响应，并没有一个统一的处理方式。Delegate, Notification, Block, KVO, 常常会不知道该用哪个最合适。有时需要chain或者compose某几个事件，就需要多个状态变量，而状态变量一多，复杂度也就上来了。为了解决这些问题，Github的工程师们开发了ReactiveCocoa。
- - -
####Signal and Subscriber
这是RAC最核心的内容，这里我想用插头和插座来描述，插座是Signal，插头是Subscriber。想象某个遥远的星球，他们的电像某种物质一样被集中存储，且很珍贵。插座负责去获取电，插头负责使用电，而且一个插座可以插任意数量的插头。当一个插座(Signal)没有插头(Subscriber)时什么也不干，也就是处于冷(Cold)的状态，只有插了插头时才会去获取，这个时候就处于热(Hot)的状态。

Signal获取到数据后，会调用Subscriber的sendNext, sendComplete, sendError方法来传送数据给Subscriber，Subscriber自然也有方法来获取传过来的数据，如：[signal subscribeNext:error:completed]。这样只要没有sendComplete和sendError，新的值就会通过sendNext源源不断地传送过来.
```swift
[RACObserve(self, username) subscribeNext: ^(NSString *newName){ 
    NSLog(@"newName:%@", newName); 
}]; 
```
RACObserve使用了KVO来监听property的变化，只要username被自己或外部改变，block就会被执行。但不是所有的property都可以被RACObserve，该property必须支持KVO，比如NSURLCache的currentDiskUsage就不能被RACObserve。

Signal是很灵活的，它可以被修改(map)，过滤(filter)，叠加(combine)，串联(chain)，这有助于应对更加复杂的情况:
```swift
RAC(self.logInButton, enabled) = [RACSignal 
        combineLatest:@[ 
            self.usernameTextField.rac_textSignal, 
            self.passwordTextField.rac_textSignal, 
            RACObserve(LoginManager.sharedManager, loggingIn), 
            RACObserve(self, loggedIn) 
        ] reduce:^(NSString *username, NSString *password, NSNumber *loggingIn, NSNumber *loggedIn) { 
            return @(username.length > 0 && password.length > 0 && !loggingIn.boolValue && !loggedIn.boolValue); 
        }]; 
 ```
 左边的RAC(...)，它的作用是将self.logInButton.enabled属性与右边的signal的sendNext值绑定。也就是如果右边的reduce的返回值为NO，那么enabled就为NO。右边的combineLatest是获取这4个signal的next值。其中可以看到self.usernameTextField.rac_textSignal这么个东东，rac_textSignal是RAC为UITextField添加的category，只要usernameTextField的值有变化，这个值就会被返回(sendNext)。combineLatest需要每个signal至少都有过一次sendNext。reduce的作用是根据接收到的值，再返回一个新的值，这里是@(YES)和@(NO)，必须是object。
- - -
####冷信号(Cold)和热信号(Hot)
冷信号默认什么也不干.
```swift
RACSignal *signal = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) { 
    NSLog(@"triggered"); 
    [subscriber sendNext:@"foobar"]; 
    [subscriber sendCompleted]; 
    return nil; 
}]; 
```
我们创建了一个Signal，但因为没有被subscribe，所以什么也不会发生。加了下面这段代码后，signal就处于Hot的状态了，block里的代码就会被执行。
```swift
[signal subscribeCompleted:^{ 
    NSLog(@"subscription %u", subscriptions); 
}]; 
```
- - -
####Side Effect
如果有多个subscriber，那么signal就会又一次被触发，控制台里会输出两次triggered。这或许是你想要的，或许不是。如果要避免这种情况的发生，可以使用 replay 方法，它的作用是保证signal只被触发一次，然后把sendNext的value存起来，下次再有新的subscriber时，直接发送缓存的数据。
























