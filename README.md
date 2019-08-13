# HERE Mobility - Android SDK
### Version 2.0.75, August 2019

## Table of contents

1. [INTRODUCTION](#introduction)
	1. [Mobility Demand](#mobility-demand)
	2. [Map Services](#map-services)
	3. [Sample apps](#sample-apps)
2. [PRE-REQUISITES](#prereqs)
	1. [Operating System](#os)
3. [GETTING STARTED](#getting-started)
	1. [Obtaining HERE Credentials for Your App](#obtain-creds)
	2. [Integrating Google Play Services into Your App](#google-play-services)
	3. [Adding the HERE Mobility SDK as a Dependency](#add-dependency)
	4. [Enabling Multidex](#multidex)
	5. [Setting the Java Language Version](#java-version)
	6. [Configuring Your `AndroidManifest.xml` File](#android-xml)
	7. [Creating an `Application` Class](#application-class)
	8. [Initializing the HERE Mobility SDK](#init-sdk)
	9. [Authenticating your app](#auth-app)
	10. [Using the HERE Sandbox Platform](#use-sandbox)
	11. [Update gms security provider (for Android API <= 19)](#security-provider)
4. [3rd Party dependencies](#3rd-Party-Packages)
5. [TUTORIAL](#tutorial)
6. [API REFERENCE](#api-reference)

<div style="page-break-after: always;"></div>

## 1. Introduction <a name="introduction"></a>
The HERE Mobility SDK helps you create apps with rich and dynamic mobility functionality. The SDK accesses the HERE Mobility Marketplace, a centralized platform for managing mobility data using intelligent algorithms.

The HERE Mobility SDK contains 2 packages:

* Mobility Demand
* Map Services  

You can use any combination of these packages. The only requirement is that you use the same version for all of them.

### 1.1. Mobility Demand<a name="mobility-demand"></a>
The Mobility Demand package allows your app to request and manage passenger rides anywhere in the world, supplied with the HERE Mobility Marketplace. This includes requesting ride offers, booking and canceling rides, and updating a ride’s status.

### 1.2. Map Services <a name="map-services"></a>
The Map Services package provides comprehensive map capabilities, including: 

* Searching for locations by address or name 
* Dynamic rendering of map display 
* Point-to-point route calculation

### 1.3. Sample Apps <a name="sample-apps"></a>
Try out our sample apps:

[Android](https://github.com/HereMobilityDevelopers/Here-Mobility-SDK-Android-SampleApp/)

[React Native](https://github.com/HereMobilityDevelopers/Here-Mobility-SDK-React-Native-SampleApp)

## 2. Pre-Requisites <a name="prereqs"></a>

### 2.1. Operating System <a name="os"></a>
The HERE Mobility SDK version 2.0.75 supports Android version 4.1 (API level 16) or later.

Also the SDK requires [androidx](https://developer.android.com/jetpack/androidx) components.

## 3. Getting Started <a name="getting-started"></a>

### 3.1. Obtaining HERE Credentials for your App <a name="obtain-creds"></a>
To use the HERE Mobility SDK, you'll need App ID key and App secret key values. To get new app keys, sign up first to [Here Mobility Developer Zone](https://developer.mobility.here.com) and then register your app using the app bundle id/package name.

### 3.2. Integrating Google Play Services into Your App <a name="google-play-services"></a>
[Set up Google Play Services](https://developers.google.com/android/guides/setup) in your app and add the play-services-location module as a dependency.

### 3.3. Adding the HERE Mobility SDK as a Dependency <a name="add-dependency"></a>
In your project’s root `build.gradle`, add the following to your repositories section:

```groovy
repositories{ 
	...
	maven{
		url "https://mobility.bintray.com/sdk"
	} 
}
```
In your app module’s build.gradle, add the following lines to your dependencies section (only include the modules you want):

```groovy
dependencies{
	...
	implementation "com.here.mobility.sdk:demand:2.0.75"
	implementation "com.here.mobility.sdk:map:2.0.75"
}
```

### 3.4. Enabling Multidex <a name="multidex"></a>
To use the HERE Mobility SDK, you will need to [enable multidex for your app](https://developer.android.com/studio/build/multidex.html).

### 3.5. Setting the Java Language Version<a name="java-version"></a>
If you’re using Java 7, add the following to the Android section of your app module’s build.gradle:

```groovy
android{ 
	...
	compileOptions{ 
		sourceCompatibility JavaVersion.VERSION_1_7 
		targetCompatibility JavaVersion.VERSION_1_8 
	}
}
```
If you’re using Java 8, add the following to the Android section of your app module’s build.gradle:

```groovy
android{ 
	...
	compileOptions{ 
		sourceCompatibility JavaVersion.VERSION_1_8 
		targetCompatibility JavaVersion.VERSION_1_8 
	}
}
```

### 3.6. Configuring Your `AndroidManifest.xml` File<a name="android-xml"></a>
#### HERE Credentials
Add the following lines to the `<application>` section of your `AndroidManifest.xml` file (inserting your actual API key in place of `YOUR_API_KEY`):

```xml
<meta-data android:name="com.here.mobility.sdk.API_KEY" android:value="YOUR_API_KEY"/> 
```
Alternately you may set the  API key in code. see 3.8. below

#### Internet and Location Access Permissions
To allow your app to access location information and to enable HERE Mobility SDK operation, add the following permissions to your AndroidManifest.xml file: 

```xml
<uses-permission android:name="android.permission.INTERNET"/> 
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/> 
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```
**Note**: If your `targetSdkVersion` is 23 or higher, you must also [ask the user for location permissions at runtime](https://developer.android.com/training/permissions/requesting.html).

### 3.7. Creating an `Application` Class <a name="application-class"></a>
If you already have an `Application` class, you can skip this step. Otherwise, refer to [android documentation](https://developer.android.com/reference/android/app/Application.html) for instructions. 

### 3.8. Initializing the HERE Mobility SDK <a name="init-sdk"></a>
In your `Application` class’ `onCreate()` method, add Sdk.init(this), as follows:

```java
public void onCreate(){
	super.onCreate();
	
	MobilitySdk.init(this);
	if (MobilitySdk.getInstance().isHereAgentProcess()){
		return;
	}
	
	// The rest of your code, if any, here
}
```

If you wish to set your  API key in code instead of in manifest, you should call
```java
    MobilitySdk.init(this, YOUR_API_KEY);
```
Instead.

### 3.9.1 Authenticating your app <a name="auth-app"></a>
In order to make Here SDK API calls, we must validate your app by signing your App Key with the Secret Key provided to you as part of your app registration process. 
You may use the Hash app level authentication to get access for : requestRideOffers, getVerticalsCoverage requests only.
The recommended procedure is:
1. Hold your secret key in the backend server of your app.
2. Generate the hash in your app's backend.
3. Pass the hash to the client.

Then pass it to the SDK following these steps:

```java
MobilitySdk.getInstance().authenticateApplication(
	ApplicationAuthInfo.create(hash, currentTimeSec));
```
The currentTimeSec parameter tells us when this authentication hash was created (in seconds, since Epoch) .

Below is example Android code that uses the Secret Key to generate a signed hash:

```java
@NonNull
private static String signedHash(@NonNull String apiKey,     // Your API Key
                                    long currentTimeSec,     // The creation time of the Application auth info in seconds since epoch
                                 @NonNull String secretKey){ // Your Secret Key
	try{
		Mac mac = Mac.getInstance("HmacSHA384");
		Charset ascii = Charset.forName("US-ASCII");
		mac.init(new SecretKeySpec(secretKey.getBytes(ascii), mac.getAlgorithm()));
		
		String apiKey64 = Base64.encodeToString(apiKey.getBytes(ascii), Base64.NO_WRAP);
       String data = apiKey64 + "." + currentTimeSec;
       byte[] rawHash = mac.doFinal(data.getBytes(ascii));
		
		// Encode into a hexadecimal string
		StringBuilder hash = new StringBuilder();
		for (int i = 0; i < rawHash.length; ++i){
			hash.append(String.format("%02x", rawHash[i]));
		}
		return hash.toString();
	} catch (NoSuchAlgorithmException | InvalidKeyException e){
		throw new IllegalStateException("HmacSHA384 and US-ASCII must be supported", e);
	}
}
```
_**Note:**_ For reference on how to generate a signed hash (given the secret key) please check [Android sampleApp project](https://github.com/HereMobilityDevelopers/Here-Mobility-SDK-Android-SampleApp) (function `appLogin` in [AuthUtils](https://github.com/HereMobilityDevelopers/Here-Mobility-SDK-Android-SampleApp/blob/master/app/src/main/java/com/here/mobility/sdk/sampleapp/util/AuthUtils.java)).


### 3.9.2 User verfication 

We require a second level of authentication for booking a ride ,receiving ride updates, cancellation etc.. (all rides related demand API calls) ,verifying that the phone number or email provided by your users is valid.
Access to the Maps module API doesn’t require phoneor email verification
Phone number or email verification is done in 3 steps:

#### 3.9.2.1 Check if user is verified
```java
	boolean isUserVerified = MobilitySdk.getInstance().isUserVerified();
```

#### 3.9.2.2 Receive verification code by phone number or email
Phone verification:
```java
	ResponseFuture<Void> futureVerification = 
			MobilitySdk.getInstance().sendVerificationSms(userPhoneNumber);
	futureVerification.registerListener(phoneVerificationResponseListener);
```
Email verification:
```java
ResponseFuture<Void> futureVerification = 
MobilitySdk.getInstance().sendVerificationEmail(userEmail);
futureVerification.registerListener(emailVerificationResponseListener);
```

#### 3.9.2.3 Verify pin
Verify phone number pin:
```java
	ResponseFuture<Void> verifyUserPhoneFuture = 
		MobilitySdk.getInstance().verifyUserPhoneNumber(phone, code);
        verifyUserPhoneFuture.registerListener(verifyPhoneFutureResponseListener);
```
Verify email pin:
```java
ResponseFuture<Void> verifyUserEmailFuture = 
MobilitySdk.getInstance().verifyUserEmail(email, code);
verifyUserEmailFuture.registerListener(verifyEmailFutureResponseListener);
```


### 3.10. Using the HERE Sandbox Platform <a name="use-sandbox"></a>
You can use the HERE Mobility Sandbox platform to develop and test your app’s functionality without calling the production platform. Requests to the sandbox environment are ephemeral (do not actually affect the real world). 

The HERE Mobility service directs your app’s calls to the sandbox or production environment according to the API key you provide. 

### 3.11. Update GMS security provider (for Android API <= 19) <a name="security-provider"></a>
Here mobility SDK uses  TLS 1.2.

If your app is using android API version <= 19 the TLS security provider might not be up-to-date, thus all network requests will fail.

Update the security provider using the following link: https://developer.android.com/training/articles/security-gms-provider

## 4. 3rd Party Packages <a name="3rd-Party-Packages"></a>
* [gRPC](https://github.com/grpc/grpc)
* [Tangram ES](https://github.com/tangrams/tangram-es)

## 5. TUTORIAL <a name="tutorial"></a>
For Tutorial [HERE Mobility Tutorial](tutorial.md). 

## 6. API Reference <a name="api-reference"></a>
For detailed information about HERE Mobility SDK functions, please refer to the [HERE Mobility API reference](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/). 
