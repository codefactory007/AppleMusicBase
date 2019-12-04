


## Overview

iOS Developer Challenge- 2020 is proudly presented by Blue Mango Global. 
 
The task is Apple Music API Integration. In short, you have to make Apple Music integration in native iOS app with list of playlist songs for user. In did select, the track or song have to be played via AVAudioPlayerNode() and not with MPMusicPlayerController.systemMusicPlayer or any other. 

Readmore:- https://www.bluemangoglobal.com/product-page/security-app

## Getting Started

You use a developer token to authenticate yourself as a trusted developer and member of the Apple Developer Program. A developer token is required in the header of every Apple Music API request. To create a developer token, first create a MusicKit signing key in your developer account, create a JSON Web Token (JWT) in the format Apple expects, and then sign it with the MusicKit signing key.  For more information about this process and how to create a developer token please see the following documentation:

* [Apple Music API Reference - Get Keys and Create Tokens](https://developer.apple.com/go/?id=apple-music-keys-and-tokens).

Once you have a developer token, you need to update the `AppleMusicManager.fetchDeveloperToken()` method in `AppleMusicManager.swift` to retrieve your valid developer token.

``` swift
func fetchDeveloperToken() -> String? {
    
    // MARK: ADAPT: YOU MUST IMPLEMENT THIS METHOD
    let developerAuthenticationToken: String? = nil
    return developerAuthenticationToken
}
```

Keep in mind, you should not hardcode the value of the developer token in your application.  This is so that if you need to generate a new developer token you are able to without having to submit a new version of your application to the App Store.

## Requesting Authorization

Before interacting with these APIs your application needs to request authorization from the user to interact with the device media library and with Apple Music.  

There are two different authorizations that iOS applications can request. Depending on your application's usecase, you may only need to request one of the above or request both of them.

### Media Library Authorization

If your application wants to access the items in the user's media library then you should request authorization using the [`MPMediaLibrary`](https://developer.apple.com/documentation/mediaplayer/mpmedialibrary) APIs.

To query your application's current [`MPMediaLibraryAuthorizationStatus`](https://developer.apple.com/documentation/mediaplayer/mpmedialibraryauthorizationstatus), you can call [`MPMediaLibrary.authorizationStatus()`](https://developer.apple.com/documentation/mediaplayer/mpmedialibrary/1621282-authorizationstatus).

``` swift
guard MPMediaLibrary.authorizationStatus() == .notDetermined else { return }
```

If the authorization status is `.notDetermined` then your application should request authorization by calling [`MPMediaLibrary.requestAuthorization(_:)`](https://developer.apple.com/documentation/mediaplayer/mpmedialibrary/1621276-requestauthorization).

``` swift
MPMediaLibrary.requestAuthorization { (_) in
    NotificationCenter.default.post(name: AuthorizationManager.cloudServiceDidUpdateNotification, object: nil)
}
```

### Cloud Service Authorization

 If your application wants to be able to playback items from the Apple Music catalog or add items to the user's iCloud Music Library then you should request authorization using the `SKCloudServiceController` APIs.

 To query your application's current [`SKCloudServiceAuthorizationStatus`](https://developer.apple.com/documentation/storekit/skcloudserviceauthorizationstatus), you can call [`SKCloudServiceController.authorizationStatus()`](https://developer.apple.com/documentation/storekit/skcloudservicecontroller/1620631-authorizationstatus).

``` swift
guard SKCloudServiceController.authorizationStatus() == .notDetermined else { return }
```

If the authorization status is `.notDetermined` then your application should request authorization by calling [`SKCloudServiceController.requestAuthorization(_:)`](https://developer.apple.com/documentation/storekit/skcloudservicecontroller/1620609-requestauthorization).

``` swift
SKCloudServiceController.requestAuthorization { [weak self] (authorizationStatus) in
    switch authorizationStatus {
    case .authorized:
        self?.requestCloudServiceCapabilities()
        self?.requestUserToken()
    default:
        break
    }
    
    NotificationCenter.default.post(name: AuthorizationManager.authorizationDidUpdateNotification, object: nil)
}
```

Once your application has the `.authorized` status, you can query the the device for more information about the capabilities associated with the device.  These capabilities are represented as [`SKCloudServiceCapability`](https://developer.apple.com/documentation/storekit/skcloudservicecapability) and can be queried by calling [`requestCapabilities(completionHandler:)`](https://developer.apple.com/documentation/storekit/skcloudservicecontroller/1620610-requestcapabilities) on an instance of [`SKCloudServiceController`](https://developer.apple.com/documentation/storekit/skcloudservicecontroller).

```swift
let controller = SKCloudServiceController()
controller.requestCapabilities(completionHandler: { (cloudServiceCapability, error) in
    guard error == nil else {
        // Handle Error accordingly, see SKError.h for error codes.
    }

    if cloudServiceCapabilities.contains(.addToCloudMusicLibrary) {
        // The application can add items to the iCloud Music Library.
    }

    if cloudServiceCapabilities.contains(.musicCatalogPlayback) {
        // The application can playback items from the Apple Music catalog.
    }

    if cloudServiceCapabilities.contains(.musicCatalogSubscriptionEligible) {
        // The iTunes Store account is currently elgible for and Apple Music Subscription trial.
    }
})
```

## Requesting a Music User Token

If your application makes calls to the Apple Music API for personalized requests that return user-specific data, your request will need to include a music user token.  To create a music user token you first need to have a valid developer token as discussed above in the "Getting Started" section.

Once you have a developer token,  you can use the native APIs availalbe on the `SKCloudServiceController` class as demonstrated below:

```swift
let completionHandler: (String?, Error?) -> Void = { [weak self] (token, error) in
    guard error == nil else {
        // Handle Error accordingly, see SKError.h for error codes.
    }

    guard let token = token else {
        print("Unexpected value from SKCloudServiceController for user token.")
        return
    }
    
    self?.userToken = token
}

if #available(iOS 11.0, *) {
    cloudServiceController.requestUserToken(forDeveloperToken: developerToken, completionHandler: completionHandler)
} else {
    cloudServiceController.requestPersonalizationToken(forClientToken: developerToken, withCompletionHandler: completionHandler)
}
```

Once you have a valid music user token, your application should cache it for future use in personalized requests for the Apple Music API.  For additional information about how the music user token is used in making requests, please see the following documentation:

* [Apple Music API Reference - Authenticate Requests](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/AppleMusicWebServicesReference/SetUpWebServices.html#//apple_ref/doc/uid/TP40017625-CH2-SW7).

