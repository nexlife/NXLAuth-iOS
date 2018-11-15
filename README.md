# NXLAuth-iOS

![NXLAuth for iOS](https://rawgit.com/openid/AppAuth-iOS/master/appauth_lockup.svg)
[![Build Status](https://travis-ci.org/openid/AppAuth-iOS.svg?branch=master)](https://travis-ci.org/openid/AppAuth-iOS)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

NXLAuth for iOS is a client SDK for communicating with 
[OAuth 2.0](https://tools.ietf.org/html/rfc6749) and 
[OpenID Connect](http://openid.net/specs/openid-connect-core-1_0.html) providers. 
It strives to
directly map the requests and responses of those specifications, while following
the idiomatic style of the implementation language. In addition to mapping the
raw protocol flows, convenience methods are available to assist with common
tasks like performing an action with fresh tokens.

It follows the best practices set out in 
[RFC 8252 - OAuth 2.0 for Native Apps](https://tools.ietf.org/html/rfc8252)
including using `SFAuthenticationSession` and `SFSafariViewController` on iOS
for the auth request. `UIWebView` and `WKWebView` are explicitly *not*
supported due to the security and usability reasons explained in
[Section 8.12 of RFC 8252](https://tools.ietf.org/html/rfc8252#section-8.12).

It also supports the [PKCE](https://tools.ietf.org/html/rfc7636) extension to
OAuth which was created to secure authorization codes in public clients when
custom URI scheme redirects are used. The library is friendly to other
extensions (standard or otherwise) with the ability to handle additional params
in all protocol requests and responses.

## Specification

### iOS

#### Supported Versions

NXLAuth supports iOS 7 and above.

iOS 9+ uses the in-app browser tab pattern
(via `SFSafariViewController`), and falls back to the system browser (mobile
Safari) on earlier versions.

#### Authorization Server Requirements

Both Custom URI Schemes (all supported versions of iOS) and Universal Links
(iOS 9+) can be used with the library.

In general, AppAuth can work with any Authorization Server (AS) that supports
native apps as documented in [RFC 8252](https://tools.ietf.org/html/rfc8252),
either through custom URI scheme redirects, or universal links.
AS's that assume all clients are web-based or require clients to maintain
confidentiality of the client secrets may not work well.

## Try

Want to try out NXLAuth? Just [CLICK HERE](https://github.com/nexlife/NXLAuth-iOS-example) to try out our Demo App

Follow the instructions in [the Demo app README.md](https://github.com/nexlife/NXLAuth-iOS-example) to configure
with your own OAuth client (you need to update 3 configuration points with your
client info to try the demo).


## Setup & Download Framework

### Setup podfile

1. With [CocoaPods](https://guides.cocoapods.org/using/getting-started.html),
   add the following line to your `Podfile`:

    `pod 'AppAuth', :git => 'https://github.com/nexlife/AppAuth-iOS.git'`

   Then run 
   
    `pod install`.


### Download NXLAuth Framework

2. Download the NXLAuth Framework file 👉 [HERE](https://github.com/nexlife/NXLAuth-iOS/archive/master.zip) 👈
   
   - Unzip and move the framework into your project.
   
     <img src="/images/drag_framework.gif" width="100%" height="100%" />
   
   
   - Add NXLAuth.framework into Embedded Binaries.
   
     <img src="/images/link_binary.gif" width="100%" height="100%" />

3. Create a NXLAuthConfig.plist in your project.
   add the following line to your NXLAuthConfig.plist.

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>NXLAuthConfig</key>
        <array>
            <dict>
                <key>Issuer</key>
                <string>YOUR ISSUER</string>
                <key>ClientID</key>
                <string>YOUR CLIENT ID</string>
                <key>RedirectURI</key>
                <string>YOUR REDIRECT URL</string>
            </dict>
        </array>
    </dict>
    </plist>
    ```

### Static Library

You can also use AppAuth as a static library. This requires linking the library
and your project and including the headers.  Suggested configuration:

1. Create an Xcode Workspace.
2. Add `AppAuth.xcodeproj` to your Workspace.
3. Include libAppAuth as a linked library for your target (in the "General ->
Linked Framework and Libraries" section of your target).
4. Add `AppAuth-iOS/Source` to your search paths of your target ("Build Settings ->
"Header Search Paths").

## Auth Flow

AppAuth supports both manual interaction with the Authorization Server
where you need to perform your own token exchanges, as well as convenience
methods that perform some of this logic for you. This example uses the
convenience method which returns either an `OIDAuthState` object, or an error.

`OIDAuthState` is a class that keeps track of the authorization and token
requests and responses, and provides a convenience method to call an API with
fresh tokens. This is the only object that you need to serialize to retain the
authorization state of the session.

### Configuration

Create a NXLAuthConfig.plist in your project.
add the following line to your NXLAuthConfig.plist.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>NXLAuthConfig</key>
	<array>
		<dict>
			<key>Issuer</key>
			<string>YOUR ISSUER</string>
			<key>ClientID</key>
			<string>YOUR CLIENT ID</string>
			<key>RedirectURI</key>
			<string>YOUR REDIRECT URL</string>
		</dict>
	</array>
</dict>
</plist>
```

### Authorizing – iOS

First you need to have a property in your AppDelegate to hold the session, in
order to continue the authorization flow from the redirect.

```objc
// property of the app's AppDelegate
@property(nonatomic, strong, nullable)
    id<OIDExternalUserAgentSession> currentAuthorizationFlow;
```

And your main class, a property to store the auth state:

```objc
// property of the containing class
@property(nonatomic, strong, nullable) OIDAuthState *authState;
```

Then, initiate the authorization request. By using the 
`authStateByPresentingAuthorizationRequest` convenience method, the token
exchange will be performed automatically, and everything will be protected with
PKCE (if the server supports it). AppAuth also allows you to perform these
requests manually. See the `authNoCodeExchange` method in the included Example
app for a demonstration.

```objc
// builds authentication request
OIDAuthorizationRequest *request =
    [[OIDAuthorizationRequest alloc] initWithConfiguration:configuration
                                                  clientId:kClientID
                                                    scopes:@[OIDScopeOpenID,
                                                             OIDScopeProfile]
                                               redirectURL:KRedirectURI
                                              responseType:OIDResponseTypeCode
                                      additionalParameters:nil];

// performs authentication request
AppDelegate *appDelegate =
    (AppDelegate *)[UIApplication sharedApplication].delegate;
appDelegate.currentAuthorizationFlow =
    [OIDAuthState authStateByPresentingAuthorizationRequest:request
        presentingViewController:self
                        callback:^(OIDAuthState *_Nullable authState,
                                   NSError *_Nullable error) {
  if (authState) {
    NSLog(@"Got authorization tokens. Access token: %@",
          authState.lastTokenResponse.accessToken);
    [self setAuthState:authState];
  } else {
    NSLog(@"Authorization error: %@", [error localizedDescription]);
    [self setAuthState:nil];
  }
}];
```

*Handling the Redirect*

The authorization response URL is returned to the app via the iOS openURL
app delegate method, so you need to pipe this through to the current
authorization session (created in the previous session).

```objc
- (BOOL)application:(UIApplication *)app
            openURL:(NSURL *)url
            options:(NSDictionary<NSString *, id> *)options {
  // Sends the URL to the current authorization flow (if any) which will
  // process it if it relates to an authorization response.
  if ([_currentAuthorizationFlow resumeExternalUserAgentFlowWithURL:url]) {
    _currentAuthorizationFlow = nil;
    return YES;
  }

  // Your additional URL handling (if any) goes here.

  return NO;
}
```

### Making API Calls

NXLAuth gives you the raw token information, if you need it. However we
recommend that users of the `OIDAuthState` convenience wrapper use the provided
`performActionWithFreshTokens:` method to perform their API calls to avoid
needing to worry about token freshness.

```objc
[_authState performActionWithFreshTokens:^(NSString *_Nonnull accessToken,
                                           NSString *_Nonnull idToken,
                                           NSError *_Nullable error) {
  if (error) {
    NSLog(@"Error fetching fresh tokens: %@", [error localizedDescription]);
    return;
  }

  // perform your API request using the tokens
}];
```

## API Documentation

Browse the [API documentation](https://nexlife.github.io/NXLAuth-iOS/doc/html/index.html).

## Included Samples

Sample apps that explore core NXLAuth features are available for iOS, follow the instructions in [Examples repo](https://github.com/nexlife/NXLAuth-iOS-example) to get started.
