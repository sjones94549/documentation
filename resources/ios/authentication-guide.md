#Layer Authentication

##Introduction
The Layer authentication architecture is designed to delegate the concerns of authentication and identity to an integrating partner via a simple, token based scheme. It requires that your backend application generate `Identity Tokens` on behalf of client applications. This token is simply a JSON Web Signature.

Included in the generation of the Layer `Identity Token` is your backend's identifier representing the user attempting to authenticate. 

```emphasis
This allows you to represent your users within the Layer service via your existing user identifiers. Participation in a Layer conversation is also represented by this same identifier.
``` 

This mechanism allows you to authenticate users within the Layer service without sharing credentials and greatly enhanced client security.

##Client Authentication Flow
The Layer `Identity Token` must be obtained via a call to your backend application and must include a nonce value that was obtained from the client SDK. The token must then be submitted to Layer via a public method on the [LYRClient](api/ios#lyrclient)` object.

There are libraries available in many popular languages for implementing JWS and generating `Idenity Tokens`. A few are listed below 

* [Node.js](https://github.com/brianloveswords/node-jws)
* [Go](https://github.com/dgrijalva/jwt-go)
* [Python](https://github.com/progrium/pyjwt/)
* [Ruby](https://github.com/progrium/ruby-jwt)


##Setup
Before your backend application can begin generating `Identity Tokens` and authenticating Layer applications, some setup must be performed. A `Provider ID` and `Key ID` must be retained by your back end application and used in the generation of the token. 

```emphasis 
**Provider ID** - The following `Provider ID` is specific to your account and should be kept private at all times.
```

%%C-PROVIDERID%%

```emphasis
**Key ID** - In order to acquire a `Key ID`, you must first generate an RSA cryptographic key pair and upload the public portion to Layer. **Layer can automatically generate the key pair on your behalf and upload the public portion to our serive. The private key will appear in a pop up.** Please copy and save the private key as it must be retained by your backend application and used to sign `Identity Tokens`.
```

%%C-KEYID%%

To manage your authentication keys please visit the [dashboard](/dashboard/account/auth).

##Generating Identity Tokens
Layer `Identity Tokens` are conceptually simple, consisting of a small pair of JSON dictionary structures (the JWS Header and Claim) and a cryptographic signature generated over them.

The Layer External Identity Token has the following structure for the Header and Claim:

```
// JWS Header
{
    "typ": "JWS", // String - Expresses a MIME Type of application/JWS
    "alg": "RS256" // String - Expresses the type of algorithm used to sign the token, must be RS256
    "cty": "layer-eit;v=1", // String - Express a Content Type of Layer External Identity Token, version 1
    "kid": "%%C-INLINE-KEYID%%" // String - Layer Key ID used to sign the token. This is your actual Key ID
}

// JWS Claim
{
    "iss": "%%C-INLINE-PROVIDERID%%", // String - The Layer Provider ID, this is your actual provider ID
    "prn": "APPLICATION USER ID", // String - Provider's internal ID for the authenticating user
    "iat": "TIME OF TOKEN ISSUANCE AS INTEGER", // Integer - Time of Token Issuance in RFC 3339 seconds
    "exp": "TIME OF TOKEN EXPIRATION AS INTEGER", // Integer - Arbitrary time of Token Expiration in RFC 3339 seconds
    "nce": "LAYER ISSUED NONCE" // The nonce obtained via the Layer client SDK.
}
```

If you are constructing Layer `Identity Token` without the aid of a 3rd party library, the tokens can be constructed in the following manner:

1. Base64 URL encode the `JWS Header`
2. Base64 URL encode the `JWS Claim`
3. Concatenate the encoded header and encoded claim with a ‘.’
4. Sign the concatenated string with the RSA (.pem) key
5. Concatenate both the unsigned string and the signed string with another ‘.’

The full structure of the Layer Identity Token looks like the following:

```console
(Base64URL JWS Header).(Base64URL JWS Claim).(Base64URL RSA Signature of ((Base64URL JWS Header).(Base64URL JWS Claim)))
```

##Identity Token Validation
We provide an [identity token validation tool](/dashboard/account/tools) in the Layer developer portal. To ensure you are generating identity tokens correctly, please validate your tokens. 

##Native Authentication Methods
The native methods your application must implement in order to authenticate the LayerClient are the following:

```objectivec
// Request an authentication nonce from Layer
[layerClient requestAuthenticationNonceWithCompletion:^(NSString *nonce, NSError *error) {
   NSLog(@"Authentication nonce %@", nonce);

   /*
    * Upon recipet of nonce, post to your backend and acquire a Layer identityToken  
    */

	// Submit the identity token to Layer for approval  
	[layerClient authenticateWithIdentityToken:@"generatedIdenityToken" completion:^(NSString *authenticatedUserID, NSError *error) {
	   NSLog(@"Authenticated as %@", authenticatedUserID);
	}];
}];
```

##Authentication Challenge
At certain times throughout the lifecycle of a Layer application, the Layer service may issue an authentication challenge to LayerKit. This challenge can occur for many reasons including expiration of tokens. Upon receiving a challenge, LayerKit effectively goes into an ‘offline state’ until you re-authenticate. Your application must implement the following LYRClientDelegate method in order to handle authentication challenges.

```objectivec
- (void)layerClient:(LYRClient *)client didReceiveAuthenticationChallengeWithNonce:(NSString *)nonce
{
	// Make a call to your backend server to obtain a Layer Identity Token including the nonce
}
```

The challenge delegate method supplies a nonce for you, so there is no need to request another one from from the SDK. Instead you should proceed with generating the `Identity Token` and then submit to LayerKit for validation via

```objectivec
// Authenticates a Layer application with an Identity Token. Upon completion of this
// method, the LayerKit is ready to start sending and receiving messages
[self.client authenticateWithIdentityToken:@"IDENTITY_TOKEN" completion:^(NSString *remoteUserID, NSError *error) {
    if (!error) {
       NSLog(@"Successful Auth with userID %@", remoteUserID);
    } else {
    	NSLog(@"Client did fail authentication with Error:%@", error);
    }
}];
```