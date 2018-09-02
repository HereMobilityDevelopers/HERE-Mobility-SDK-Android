# HERE Mobility - Android SDK
### Version 1.1.17, September 2018
Copyright © 2018, HERE Technologies. All rights reserved. The software, information and all other materials contain in this documentation are confidential and proprietary information of HERE Technologies and are protected by applicable copyright legislation. The disclosure of information contained herein does not constitute any license or authorization to use or disclose information, ideas or concepts presented. No part of this documentation may be disclosed to any third party, copied, reproduced or stored on any type of media or used in any way by any party without the express prior, written consent of Here Technologies. This software, information and all other materials contain in this documentation are provided "as-is" and without warranties of any kind, either express or implied, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, satisfactory quality and non-infringement. HERE Technologies does not warrant that the content is error free and HERE Technologies does not warrant or make any representations regarding the quality, correctness, accuracy, or reliability of the content.
<div style="page-break-after: always;"></div>

## Table of contents

1. [INTRODUCTION](#introduction)
	1. [Mobility Demand](#mobility-demand)
	2. [Map Services](#map-services)
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
	9. [Authenticating App Users](#auth-users)
	10. [Using the HERE Sandbox Platform](#use-sandbox)
	11. [Update gms security provider (for Android API <= 19)](#security-provider)
4. [API REFERENCE](#api-reference)

<div style="page-break-after: always;"></div>

## 1. Introduction <a name="introduction"></a>
The HERE Mobility SDK helps you create apps with rich and dynamic mobility functionality. The SDK accesses the HERE Mobility Marketplace, a centralized platform for managing mobility data using intelligent algorithms.

The HERE Mobility SDK contains 2 packages:

* Mobility Demand
* Map Services  

You can use any combination of these packages. The only requirement is that you use the same version for all of them.

### 1.1. Mobility Demand<a name="mobility-demand"></a>
The Mobility Demand package allows your app’s users to request and manage passenger rides anywhere in the world, supplied through the HERE Mobility Marketplace. This includes requesting ride offers, booking and canceling rides, and updating a ride’s status.

### 1.2. Map Services <a name="map-services"></a>
The Map Services package provides comprehensive map capabilities, including: 

* Searching for locations by address or name 
* Dynamic rendering of map display 
* Point-to-point route calculation


## 2. Pre-Requisites <a name="prereqs"></a>

### 2.1. Operating System <a name="os"></a>
The HERE Mobility SDK version 1.1.17 supports Android version 4.0.4 (API level 15) or later.


## 3. Getting Started <a name="getting-started"></a>

### 3.1. Obtaining HERE Credentials for your App <a name="obtain-creds"></a>
To use the HERE Mobility SDK, you will need to receive an API Key and a Secret Key. To obtain these, contact us at mobility_developers@here.com. 

Please include the following information in your email:  

* Full name 
* Phone number
* Application name 
* Application id (as specified in your `build.gradle`)

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
	implementation "com.here.mobility.sdk:demand:1.1.17"
	implementation "com.here.mobility.sdk:map:1.1.17"
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
	
	HereMobilitySdk.init(this);
	if (HereMobilitySdk.isHereAgentProcess(this)){
		return;
	}
	
	// The rest of your code, if any, here
}
```

### 3.9. Authenticating App Users <a name="auth-users"></a>
In order to make Here SDK API calls on behalf of your users (e.g. book rides), you must first "prove" us that they indeed your users by signing their username with your HERE SDK Secret Key. The recommended procedure is to have your backend server do this when the user logs in, and send the signed "hash" to the app. Once the app has the hash, it should pass it to the SDK like so:

```java
HereMobilitySdk.setUserAuthInfo(
	HereSdkUserAuthInfo.create(userId, expirationInSeconds, signedHash));
```
The expiration parameter tells us when this authentication (in seconds, since Epoch) expires and should no longer be accepted by our servers.

Below is example Android code that uses the Secret Key to generate a signed hash:

```java
@NonNull
private static String signedHash(@NonNull String apiKey,     // Your API Key
                                 @NonNull String userId,     // The user's id in your app
                                 long expirationInSeconds,   // Expiration timestamp
                                 @NonNull String secretKey){ // Your Secret Key
	try{
		Mac mac = Mac.getInstance("HmacSHA256");
		Charset ascii = Charset.forName("US-ASCII");
		mac.init(new SecretKeySpec(secretKey.getBytes(ascii), mac.getAlgorithm()));
		
		String apiKey64 = Base64.encodeToString(apiKey.getBytes(ascii), Base64.NO_WRAP);
		String userId64 = Base64.encodeToString(userId.getBytes(ascii), Base64.NO_WRAP);

		String data = apiKey64 + "." + userId64 + "." + expirationInSeconds;
		byte[] rawHash = mac.doFinal(data.getBytes(ascii));
		
		// Encode into a hexadecimal string
		StringBuilder hash = new StringBuilder();
		for (int i = 0; i < rawHash.length; ++i){
			hash.append(String.format("%02x", rawHash[i]));
		}
		return hash.toString();
	} catch (NoSuchAlgorithmException | InvalidKeyException e){
		throw new IllegalStateException("HmacSHA256 and US-ASCII must be supported", e);
	}
}
```

***Note:*** It's important for us to be sure that each API call to the Mobility Demand API comes from a real user looking for rides because these calls are translated to actual taxis driving to pick up the users. Safeguard your Secret Key, and do not put it in the app, where it can be discovered by disassembling the app.

***Note for TechCrunch Hackathon developers:*** During the hackathon, and only then, for the purpose of saving time, you may put your Secret Key in the app, and perform the signing on the client.



### 3.10. Using the HERE Sandbox Platform <a name="use-sandbox"></a>
You can use the HERE Mobility Sandbox platform to develop and test your app’s functionality without calling the production platform. Requests to the sandbox environment are ephemeral (do not actually affect the real world). 

The HERE Mobility service directs your app’s calls to the sandbox or production environment according to the API key you provide. 

### 3.11. Update GMS security provider (for Android API <= 19) <a name="security-provider"></a>
Here mobility SDK uses  TLS 1.2.

If your app is using android API version <= 19 the TLS security provider might not be up-to-date, thus all network requests will fail.

Update the security provider using the following link: https://developer.android.com/training/articles/security-gms-provider

## 4. API Reference <a name="api-reference"></a>
For detailed information about HERE Mobility SDK functions, please refer to the [HERE Mobility API reference](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/). 
