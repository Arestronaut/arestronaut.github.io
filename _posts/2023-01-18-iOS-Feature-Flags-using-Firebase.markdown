---
layout: post
title:  iOS Feature Flags using Firebase
date:   2023-01-18 21:59:00 +0000
categories: jekyll update
tags:   Blog,iOS,Swift
---
Imagine these conversations with your PM over a couple of weeks:



PM: "Hey, we need to have this screen added to the onboarding flow"
You: "Alright, no problem. Here you have it"
PM (one week after the screen was added to the onboarding flow): "The requirements have changed and we need to hide the screen for now."
You: "Sure thing. There you go"
PM (Yet another week has past): …



You get the gist. Maybe you have already encountered this or a similar conversation with your PM. At first sight the case seems to be clear. Create the screen and add it to the flow, remove the screen from the flow, add it again, and so on. You would create a PR for this every time, wait until the build is deployed and let QA handle it from there. Overall this is a fine-ish way to handle the situation but there is an awful lot of overhead just for this one screen. Going through the full development cycle takes time, can be quite dreadful and of course error prone.Let me tell you this, there is a better way this situation could have been handled. It's called 



![Alt](https://c.tenor.com/ThI9D4QmkWMAAAAC/olympics-flag.gif)



What the heck is it
The principle is quite simple. You define a flag somewhere remotely (I'm going to use Firebase as an example), fetch it on app start and depending on the flag a feature is enabled or disabled.



The biggest advantage is that release is now independent from deployment. If anything goes sideways with the release, the feature can simply be disabled until the fixed is rolled out. With a little more tinkering Feature Flags build the foundation for A/B-Testing which is used throughout the tech industry.



Having this theoretical mumbo jumbo out of the way let's start implementing a bare bone solution.
Firebase setup
The minimum requirement for this first step is having a google account and I assume that nowadays everyone has one.



*If you already have a firebase project setup you can skip this step.*



After the project has been set up open the project's dashboard and go to 



![Alt](https://raw.githubusercontent.com/Arestronaut/arestronaut.github.io/main/assets/images/Firebase%20Feature%20Flags/firebase-console.jpg)



On the right side a popup should appear. This is the place where you will define your first flag. I'm going to name it 



As a 



![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/Firebase%20Feature%20Flags/firebase-new-remote-config.png?raw=true)



Now 



![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/Firebase%20Feature%20Flags/firebase-publish-changes.png?raw=true)



Firebase has this feature to prevent unwanted changes to be published. Just choose 



Project setup
Now to the juicy stuff. Connecting the app with firebase. Lucky for me firebase did a good job explaining how to set everything up which makes my life easier. But nevertheless I will point in the right direction and provide additional information.



I'm assuming now that you already have a project ready. If not, please do so before proceeding.



First go back to your 



As a quick side note: When adding the SDK to the project you are prompted to choose the package products. For this purpose you only need 



If everything is set up correctly, the 



You should be able to see that everything works correctly in the Analytics Dashboard.
Fetching the configuration



This part is going cover fetching the RemoteConfig from Firebase. Luckily the SDK provides a very straight forward way of fetching the data. The only thing I'm going to provide here is a wrapper around their functionality.



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



This wrapper is designed to work in concurrent environments therefore it's defined as an actor. Furthermore there shouldn't be multiple instances of this floating around that's why it's a singleton
RemoteConfig belongs to Firebase's SDK, it's what the wrapper is abstracting.
Here we create a new instance of 
`fetchAndActivate`



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



I must admit that I did a poor job of using the concurrent ability of the wrapper but it's supposed to be able to use in various contexts.
Reading remote values
If done correctly the app should fetch the remote configuration successfully. The final step is going to be consuming the values. As done before I'm going to define a nice abstraction that will make the usability much better. But first we need a way to access the remote configuration values.



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



For some nice error semantics, reading from the 
To understand why this needs to be 
And that's basically what is done here. Get the remote config value as a string, initialise the output value with that string and return it. Throw an error along the way if something fails.



Wow, that felt way more complicated than it actually is.



Now to the grand finale, a nice and handy way to access the feature flags. First we need a nice way of defining new features. As out desired type is going to be 



enum Features: String {
    case awesomeFeature = "AWESOME_FEATURE"
}



To coat things with some sugar, here is a nice and small property wrapper to access those Features.



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



That's it! With just a single line of code it can be determined if a feature is enabled or not.



@FeatureFlag(.awesomeFeature) var isAwesomeFeatureEnabled
![Alt](https://c.tenor.com/l_VVSBaEIPwAAAAC/finally.gif)



**Concluding remarks**
First of all, thanks for sticking till the end. There is not too much left to say but, as you might have noticed, this is not only a tutorial for enabling feature flags on iOS using Firebase. It's also somewhat explains the basics for 
Have a great one :)



