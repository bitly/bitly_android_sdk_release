# Bitly Android SDK

![Releases](https://img.shields.io/github/release/bitly/bitly_android_sdk.svg) ![Bintray](https://img.shields.io/bintray/v/mobilebitly/maven/bitly.svg)  [![Build Status](https://travis-ci.com/bitly/bitly_android_sdk.svg?token=GQk2M5gzMUKUJCESKF18&branch=master)](https://travis-ci.com/bitly/bitly_android_sdk)

## Getting Started
Before integrating the SDK you will need to configure your mobile app in Bitly. To do this go to Brand Manager -> Mobile Apps. When creating the Android app you will need to provide the `sha256_cert_fingerprints` for your app. You can determine the value using this command
                                                                                                                                                                                                                                   
```
keytool -list -v -keystore my-release-key.keystore
```

Once the app is created the app ID required later will be available on the app detail panel.  Copy it for later in the setup.

To create the Digital Asset Links JSON file, which would be located at https://yourdomain.com/.well-known/assetlinks.json, you must got to Branded Short Domains -> yourdomain.com -> Mobile Behavior and associate the newly created Android App with your domain. After saving the changes you can validate the file exists. 

## Install the SDK
1. Add the following to your `build.gradle` file

  ```
  implementation 'com.bitly:bitlysdk:1.0.x'
  ```
2. You may need to also add GitHub as a repository

  ```
  repositories {
     maven {
        url "https://maven.pkg.github.com/bitly/bitly_android_sdk_release"
        credentials{
            username = project.findProperty("gpr.user") ?: System.getenv("GH_USERNAME")
            password = project.findProperty("gpr.key") ?: System.getenv("GH_TOKEN")
        }
    }
  }
  ```  

## Configure the App Links

1. Create an Activity to handle the App Links
2. Configure the Activity in your AndroidManifest.xml to receive the App Links

    ```xml
    <activity android:name=”MainActivity”>
        <intent-filter android:autoVerify="true">
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="http" android:host="yourdomain.com" />
            <data android:scheme="https" android:host="yourdomain.com" />
        </intent-filter>
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="yourscheme" android:host="*" />
        </intent-filter>
    </activity>
    ```

    >You cannot combine your launch activity and your App Links activity. These must be two separate activities.    

3. Multiple `<intent-filter>` can be created to support multiple domains

## Configure the SDK for App Links

1. If your application doesn't already have a custom Application class, create it and configure it in the AndroidManifest.xml
2. Import the SDK in your Application instance

    ```java
    import com.bitly.Bitly;
    ```
3. Initialize the SDK on Application creation

    ```java
    @Override
    public void onCreate() {
        super.onCreate();

        Bitly.initialize(this, "YOUR_APP_ID", Arrays.asList("yourdomain.com","yourotherdomain.com"), Arrays.asList("yourscheme"), new Bitly.Callback() {
            @Override
            public void onResponse(Response response) {
                // response provides a Response object which contains the full URL information
                // response includes a status code
                // Your custom logic goes here...
            }

            @Override
            public void onError(Error error) {
                // error provides any errors in retrieving information about the URL
                // Your custom logic goes here...
            }
        });
    }
    ```
    
    > You can also provide the access token for shortening in the same initialize method

4. Import the SDK in your Activity for handling App Links

    ```java
    import com.bitly.Bitly;
    ```

5. Handle the first intent

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        //Handle the intent creating the activity
        Bitly.handleIntent(getIntent());
    }
    ```   

6. Handle any additional intents after activity creation

    ```java
    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        Bitly.handleIntent(intent);
    }
    ```

    >If the activity is specified as having an `android:launchMode` that only allows for the creation of a single instance ("singleTop", etc) then handling intents in `protected void onNewIntent(Intent intent)` is necessary

7. Retry on error

    ```java
    Bitly.retryError(error)
    ```

    >If there are any issues in handling an App Link the operation can be retried using the following line of code in your response handler from step 2

## Configure the SDK for Shortening

1. Configure an OAUTH token for your app http://bitly.com/a/oauth_apps
2. If your application doesn't already have a custom Application class, create it and configure it in the AndroidManifest.xml
3. Import the SDK in your Application instance

    ```java
    import com.bitly.Bitly;
    ```
4. Initialize the SDK on Application creation

    ```java
    @Override
    public void onCreate() {
        super.onCreate();

        Bitly.initialize(this, "YOUR_APP_ACCESS_TOKEN");
    }
    ```
    
    > You can combine this initialization with the App Link initialization

5. You can shorten from anywhere in your application

    ```java
    Bitly.shorten("http://theurlyouwishtoshorten.com", new Bitly.Callback() {
        @Override
        public void onResponse(Response response) {
            // response provides a Response object which contains the shortened Bitlink
            // response includes a status code
            // Your custom logic goes here...
        }

        @Override
        public void onError(Error error) {
            // error provides any errors in shortening the link
            // Your custom logic goes here...
        }
    });
    ```
