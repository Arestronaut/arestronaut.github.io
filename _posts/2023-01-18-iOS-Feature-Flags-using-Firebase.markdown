---
layout: post
title:  iOS Feature Flags using Firebase
date:   2023-01-18 21:59:00 +0000
categories: [Blog,iOS,Swift]
tags:   [Blog,iOS,Swift]
image: assets/images/geile_eier.jpg
description: ''
featured: false
hidden: false
---
Imagine these conversations with your PM over a couple of weeks:



> PM: "Hey, we need to have this screen added to the onboarding flow"
>
> You: "Alright, no problem. Here you have it"
>
> PM (one week after the screen was added to the onboarding flow): "The requirements have changed and we need to hide the screen for now."
>
> You: "Sure thing. There you go"
>
> PM (Yet another week has past): …



You get the gist. Maybe you have already encountered this or a similar conversation with your PM. At first sight the case seems to be clear. Create the screen and add it to the flow, remove the screen from the flow, add it again, and so on. You would create a PR for this every time, wait until the build is deployed and let QA handle it from there. Overall this is a fine-ish way to handle the situation but there is an awful lot of overhead just for this one screen. Going through the full development cycle takes time, can be quite dreadful and of course error prone.Let me tell you this, there is a better way this situation could have been handled. It's called **Feature Flags.**



![Alt](https://c.tenor.com/ThI9D4QmkWMAAAAC/olympics-flag.gif)


## What the heck is it
The principle is quite simple. You define a flag somewhere remotely (I'm going to use Firebase as an example), fetch it on app start and depending on the flag a feature is enabled or disabled.



The biggest advantage is that release is now independent from deployment. If anything goes sideways with the release, the feature can simply be disabled until the fixed is rolled out. With a little more tinkering Feature Flags build the foundation for A/B-Testing which is used throughout the tech industry.



Having this theoretical mumbo jumbo out of the way let's start implementing a bare bone solution.

## Firebase setup
The minimum requirement for this first step is having a google account and I assume that nowadays everyone has one.



*If you already have a firebase project setup you can skip this step.*Go to the [Firebase console](https://console.firebase.google.com/) and create a new project. Name it however you want.



After the project has been set up open the project's dashboard and go to *Build->Remote Config*. Now create a new configuration.



![Alt](https://raw.githubusercontent.com/Arestronaut/arestronaut.github.io/main/assets/images/Firebase%20Feature%20Flags/firebase-console.jpg)


On the right side a popup should appear. This is the place where you will define your first flag. I'm going to name it `AWESOME_FEATURE`. It basically doesn't matter how you name it. Be aware that you are going to use that name later on in the code base but of course you can always come back to check.



As a *data type* you want to choose `Boolean` as the feature should either be activate or disabled. Optionally you can choose a *description* but the name should be expressive enough to give a hint of what is going on. The default value should be set to `true`.



![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/Firebase%20Feature%20Flags/firebase-new-remote-config.png?raw=true)


Now `Save` the changes and you should be good to go. On the dashboard you should see something like this:



![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/Firebase%20Feature%20Flags/firebase-publish-changes.png?raw=true)


Firebase has this feature to prevent unwanted changes to be published. Just choose `Publish changes` and the web part is (almost) done.



## Project setup
Now to the juicy stuff. Connecting the app with firebase. Lucky for me firebase did a good job explaining how to set everything up which makes my life easier. But nevertheless I will point in the right direction and provide additional information.



I'm assuming now that you already have a project ready. If not, please do so before proceeding.



First go back to your [firebase console](https://console.firebase.google.com/). On the Project Overview page in the top center you can add a new app to your project. Choose *iOS* and follow the steps. It's pretty straight forward, I promise.



> As a quick side note: When adding the SDK to the project you are prompted to choose the package products. For this purpose you only need **FirebaseAnalytics** and **FirebaseRemoteConfig**.



If everything is set up correctly, the `GoogleService-Info.plist` added and the SDK fetched. And of course most importantly `FirebaseApp.configure()` is called from `AppDelegate.swift`.



You should be able to see that everything works correctly in the Analytics Dashboard.

## Fetching the configuration


This part is going cover fetching the RemoteConfig from Firebase. Luckily the SDK provides a very straight forward way of fetching the data. The only thing I'm going to provide here is a wrapper around their functionality.



```
swift
import Firebase
import FirebaseRemoteConfig

// 1
actor RemoteConfigService {

    enum FetchError: Error {
        case fetchFailed
        case underlyingError(Error)
    }

    static var shared: RemoteConfigService { .init() }
    
    // 2
    private let remoteConfig: RemoteConfig

    private init() {
    	// 3
        remoteConfig = Firebase.RemoteConfig.remoteConfig()

        let settings = RemoteConfigSettings()
        // NOTE: This should only be applied for testing purposes
        settings.minimumFetchInterval = 0.0

        remoteConfig.configSettings = settings
    }

	// 4
    @discardableResult
    func fetchAndActivate() async throws -> RemoteConfigFetchAndActivateStatus {
        try await withCheckedThrowingContinuation { continuation in
            remoteConfig.fetchAndActivate { status, error in
                switch status {
                case .successFetchedFromRemote, .successUsingPreFetchedData:
                    continuation.resume(with: .success(status))
                case .error where error != nil:
                    continuation.resume(with: .failure(FetchError.underlyingError(error!)))
                default:
                    continuation.resume(with: .failure(FetchError.fetchFailed))
                }
            }
        }
    }
}
```


1. This wrapper is designed to work in concurrent environments therefore it's defined as an actor. Furthermore there shouldn't be multiple instances of this floating around that's why it's a singleton
2. RemoteConfig belongs to Firebase's SDK, it's what the wrapper is abstracting.
3. Here we create a new instance of `RemoteConfig`. With `RemoteConfigSettings` it's possible to some configuration like providing default values and as in this case setting the `minimumFetchInterval` to zero. This should only be done when testing. Per default the `minimumFetchInterval` is 12 hours which is also recommended by Firebase. The reason being is that there is an hourly limit on how many requests can be sent which should not be met when running in production
4. `fetchAndActivate` is the main API that needs to be called. As it is not designed in an async/await manner I used `withCheckedThrowingContinuation`. It doesn't really matter when the remote configuration is fetched during the app's lifecycle though it should happen after the Firebase is configured and before it's used by the client. For simplicity I did it in `SceneDelegate`


```
swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        fetchRemoteConfig()

        guard let windowScene = (scene as? UIWindowScene) else { return }
        window = UIWindow(windowScene: windowScene)
        let viewController = ViewController()
        window?.rootViewController = viewController
        window?.makeKeyAndVisible()
    }

    private func fetchRemoteConfig() {
        Task {
            do {
                try await RemoteConfigService.shared.fetchAndActivate()
            } catch {
                print("[RemoteConfig] FetchAndActivate failed with error: \(error.localizedDescription)")
            }
        }
    }
}
```


I must admit that I did a poor job of using the concurrent ability of the wrapper but it's supposed to be able to use in various contexts.

## Reading remote values
If done correctly the app should fetch the remote configuration successfully. The final step is going to be consuming the values. As done before I'm going to define a nice abstraction that will make the usability much better. But first we need a way to access the remote configuration values.



```
swift
// RemoteConfiService.swift

// 1
enum ValueError: Error {
    case notFound
	case notCastable
}

// ....

// 2
nonisolated func value<Value: LosslessStringConvertible>(for key: String) throws -> Value {
	// 3
	guard let value = remoteConfig[key].stringValue else { throw ValueError.notFound }
	guard let typedValue = Value(value) else { throw ValueError.notCastable }
	return typedValue
}
```


1. For some nice error semantics, reading from the `RemoteConfig` can also fail. Both cases are handled.
2. To understand why this needs to be `nonisolated` [here](https://www.avanderlee.com/swift/actors/) is an amazing post to learn everything about actors. In general remote config values can be anything, Boolean, Integer, String, custom data types, etc. The access method therefore needs to be generic as well. But why is the generic value constraint to be `LosslessStringConvertible`? Firebase SDK only offers to unwrap the remote config value to a string-, bool-, data-, json- and number value. More or less all values can be represented as `String` so it made sense to first get the untyped value as a String and later type cast it to the desired type. To make sure this type cast can work `Value` needs to be initializable from a string.
3. And that's basically what is done here. Get the remote config value as a string, initialise the output value with that string and return it. Throw an error along the way if something fails.


Wow, that felt way more complicated than it actually is.



Now to the grand finale, a nice and handy way to access the feature flags. First we need a nice way of defining new features. As out desired type is going to be `Bool` it's sufficient to only have a place to collect the keys. To do that, create an enum `Features`



```
swift
enum Features: String {
    case awesomeFeature = "AWESOME_FEATURE"
}
```


To coat things with some sugar, here is a nice and small property wrapper to access those Features.



```
swift
@propertyWrapper struct FeatureFlag {
    private let feature: Features

    var wrappedValue: Bool? {
        do {
            let flag: Bool = try RemoteConfigService.shared.value(for: feature.rawValue)
            return flag
        } catch {
            print("[FeatureFlag] \(error.localizedDescription)")
            return nil
        }
    }

    init(_ feature: Features) {
        self.feature = feature
    }
}
```


That's it! With just a single line of code it can be determined if a feature is enabled or not.



```
swift
@FeatureFlag(.awesomeFeature) var isAwesomeFeatureEnabled
```
![Alt](https://c.tenor.com/l_VVSBaEIPwAAAAC/finally.gif)


## **Concluding remarks**
First of all, thanks for sticking till the end. There is not too much left to say but, as you might have noticed, this is not only a tutorial for enabling feature flags on iOS using Firebase. It's also somewhat explains the basics for `RemoteConfig`which can be a very mighty tool, not only for enabling/disabling features. If there are any questions left, feel free to reach out.

Have a great one :)



