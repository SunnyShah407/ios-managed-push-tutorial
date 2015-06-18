# iOS Managed Push Tutorial
Since iOS 3, Apple has given developers the ability to send push notifications to users, and by doing so, applications have become a far bigger part of our lives. 

Traditionally, one needed a third-party service or a custom app server to manage communications with Apple Push Notifications Service (APNS). This made it difficult to implement what should be an easy feature.

A few weeks ago, the team at Sinch brought us a game-changing feature. Not only does Sinch offer a full service voice, messaging, and PSTN service, but now it offers managed push built in. This means that we developers no longer have to worry about doing it ourselves. 

Now you can provide real-time notifications for the real-time features of your app in a rather short amount of time. 

Today we're going to be building the Sinch push notification app that's provided as an example when you download the Sinch SDK. You can download the starter project HERE, which includes all the features except for managed push. If you'd like to learn about any of the other features, or simply how to get started, you can check out the other tutorials on this site or browse through a doc, which includes a comprehensive beginner's guide for iOS.

Now, let’s begin. Open up the starter project and navigate over to **AppDelegate.m**. If you look for the initSinchClientWithUserId method, you will see the following code:

```objective-c
[_client setSupportActiveConnectionInBackground:YES];
```

This is the code currently powering our push notifications. As of iOS 7, Apple has allowed us to keep sockets open for a short amount of time once a user has exited an app, although this only applies to VoIP push notifications. Go ahead and try out the app as it is; you will find that you can receive a push notification for an incoming call, though if you were to leave your device idle for a short amount of time, this would no longer be the case. 

Go ahead and remove that line of code, as we're now going to be implementing managed push notifications, which will give us the ability to receive push notifications at all time. Replace this code with a call to enable managed push. 

```objective-c
[_client enableManagedPushNotifications];
```

Before we get into coding, there's some background work we need to take care of. On your Mac, find the application Keychain access and then find certificate assistant > Request a Certificate from a Certificate Authority.

![Getting Certificate](/images/1.png)

Now, follow the steps and make sure to save your certificate to somewhere accessible on your Mac.

![Saving Certificate](/images/2.png)

We're all done with Keychain access for the moment. Now, open up a web browser and visit [Apple’s developer portal](www.developer.apple.com). Once you've logged in, find Certificates, Identifiers & Profiles; from here, you're going to need to select 'App IDs' under the section 'Identifiers'.

Create a new app and under the 'App services' section and be sure to tick push notifications. Once this has been done, you will see that besides push notifications, we're given the "Configurable" status, which is exactly what we want.

Hit done and you will be presented with a list of your App IDs. Select the one you just created and press edit. From here, find the push notifications section

![Push Certificate](/images/3.png)

Today we will only be working in the development environment, so go ahead and create a certificate under Development SSL certificate.

Follow the steps and when prompted, upload the certificate we made at the start of the tutorial. Once the certificate has been generated, download, double-click, and then head back over to Keychain Access, where we're going to export a .p12 file for use with the Sinch dashboard. While we're here, it's probably a good idea to generate a Provisioning Profile for our new app; make sure that it's a development one and it's tied to the App ID we created earlier.

Find your certificate under the login section, right-click, and select export. You're now prompted to save the certificate, name it whatever you like, and save it. (Make sure it is in face in the .p12 format.) You can make a password for it, however, as this is a development environment, it's up to you. Now enter your login password and you're ready to head over to the Sinch dashboard to start configuring.

If you haven't already, make an account. Next, create a new app, select it, and, on the third tab, you should find the push settings. 

![Upload certificate](/images/sinch.png)

You can now drag and drop your .p12 file that we just created into Sinch.

It's that easy! Now we can get back to the good stuff: coding.

Once back in XCode, there are a few things we need to do before we can start writing code. Firstly, under build settings, you're going to want to set the provisioning profile to the one you created earlier. Next, set the bundle identifier to the one that you specified when creating your App ID.

Now it's really time to code :P

If you'd like, check out the code already in place and give the app a try. It's fully functional. 

We're currently managing the Sinch client from our AppDelegate, so we're going to get started in **AppDelegate.m**. Make a property of type SINManagedPush and name it Push.

```objective-c
@property (nonatomic, readwrite, strong) id<SINManagedPush> push;
```
We've now got a property that's accessible globally within our class—exactly what we want. If you check out our docs, you will see that it's best to init the push client as early as possible in the app's lifecycle. Because of this, we’ll go ahead and add this code to the top of the method didFinishLaunchingWithOptions.

```objective-c
self.push = [Sinch managedPushWithAPSEnvironment:SINAPSEnvironmentAutomatic];
  self.push.delegate = self;
  [self.push setDesiredPushTypeAutomatically];
```

Here you can see we're setting the APS Environment to automatic, our delegate, and then the desired push type to automatic. When automatic is available, use it!



If you run the code now, you'll see an issue with what we just implemented because our app delegate doesn't conform to the SINManagedPush delegate. Go ahead and make it.

```objective-c
@interface AppDelegate () <SINClientDelegate, SINCallClientDelegate, SINManagedPushDelegate>
```

Once again, we've got more errors. Not to worry, though, as they're just letting us know we've got to implement didReceiveIncomingPushWithPayload.

```objective-c
- (void)managedPush:(id<SINManagedPush>)unused
    didReceiveIncomingPushWithPayload:(NSDictionary *)payload
                              forType:(NSString *)pushType {

}
```

Here's our go-to place if we're to receive a push notification. As you can see, there isn't any functionality here. Instead of implementing it all here, we're going to create a separate method with some other logic to handle the push notification.

```objective-c
- (void)handleRemoteNotification:(NSDictionary *)userInfo {
  if (!_client) {
    NSString *userId = [[NSUserDefaults standardUserDefaults] objectForKey:@"userId"];
    if (userId) {
      [self initSinchClientWithUserId:userId];
    }
  }
  [self.client relayRemotePushNotification:userInfo];
}
```

Here we pass in a dictionary, userInfo from Sinch, which will give us the data we need. We then check if the client is active; it more than likely won't be if a push notification has been sent. In the login process, we store our userId in NSUserDefaults, then if the client isn't active, we're able to init our client here.

We then call another method, relayRemotePushNotification, once we know the client is active. Go back and call this method from didReceiveIncomingPushWithPayload. The didReceiveIncomingPushWithPayload method is built into the Sinch SDK and provides a link between the Sinch SDK and iOS push notifications. Go ahead and implement the didReceiveRemoteNotification method.

```objective-c
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    NSLog(@"Remote notification");
  [self.push application:application didReceiveRemoteNotification:userInfo];
}
```
This method calls our push property and from that passes in our userInfo dictionary. There's one method already in place that you should take note of, as it's used to setup push notifications. Before it was handling incoming push notifications from our active background connecting, however now it will be receiving them as Apple push notifications.

```objective-c
- (SINLocalNotification *)client:(id<SINClient>)client localNotificationForIncomingCall:(id<SINCall>)call {
  SINLocalNotification *notification = [[SINLocalNotification alloc] init];
  notification.alertAction = @"Answer";
  notification.alertBody = [NSString stringWithFormat:@"Incoming call from %@", [call remoteUserId]];
  return notification;
}
```

This method creates a SINLocalNotification, sets some basic properties, and then returns the notification so it can be displayed.

There's two types of push notifications: VoIP and traditional. VoIP notifications are available as of iOS 8 and is managed by adding the framework PushKit, which has already been done for us in the starter project. If we were to receive traditional push notifications, which might be the case if we add instant messaging to our application, you'd need to add these two additional methods. 

```objective-c
- (void)application:(UIApplication *)application
    didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
        [self.push application:application didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}
- (void)application:(UIApplication *)application 
    didReceiveRemoteNotification:(NSDictionary *)userInfo {
        [self.push application:application didReceiveRemoteNotification:userInfo];
}
```

It's good to have these methods implemented, as in versions prior to iOS 8, we aren't able to use VoIP push notifications. 
