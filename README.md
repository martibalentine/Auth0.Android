# Auth0.Android

[![CI Status](http://img.shields.io/travis/auth0/Auth0.Android.svg?style=flat-square)](https://travis-ci.org/auth0/Auth0.Android)
[![Coverage Status](https://img.shields.io/codecov/c/github/auth0/Auth0.Android/master.svg?style=flat-square)](https://codecov.io/github/auth0/Auth0.Android)

Android java toolkit for Auth0 API

## Requirements

Android API version 15 or newer

## Installation

###Gradle

Auth0.android is available through [Gradle](https://gradle.org/). To install it, simply add the following line to your `build.gradle` file:

```gradle
dependencies {
    compile "com.auth0.android:auth0:1.0.0-beta.2"
}
```

### Permissions 

Open your app's `AndroidManifest.xml` file and add the following permission.

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Usage

First create an instance of `Auth0` with your client information

```java
Auth0 account = new Auth0("{YOUR_CLIENT_ID}", "{YOUR_DOMAIN}");
```

### Authentication API

The client provides methods to authenticate the user against Auth0 server.
 
 Create a new instance by passing the account:
 
 ```java
AuthenticationAPIClient authentication = new AuthenticationAPIClient(account);
 ```

#### Login with database connection

```java
authentication
    .login("info@auth0.com", "a secret password", "my-database-connection")
    .start(new BaseCallback<Credentials>() {
        @Override
        public void onSuccess(Credentials payload) {
            //Logged in!
        }
    
        @Override
        public void onFailure(AuthenticationException error) {
            //Error!
        }
    });
```

#### Passwordless Login

Step 1: Request the code

```java
authentication
    .passwordlessWithEmail("info@auth0.com", PasswordlessType.CODE)
    .start(new BaseCallback<Credentials>() {
        @Override
        public void onSuccess(Void payload) {
            //Code sent!
        }
    
        @Override
        public void onFailure(AuthenticationException error) {
            //Error!
        }
    });
```

Step 2: Input the code

```java
authentication
    .loginWithEmail("info@auth0.com", "a secret password", "my-passwordless-connection")
    .start(new BaseCallback<Credentials>() {
        @Override
        public void onSuccess(Credentials payload) {
            //Logged in!
        }
    
        @Override
        public void onFailure(AuthenticationException error) {
            //Error!
        }
    });
```


#### Sign Up with database connection

```java
authentication
    .signUp("info@auth0.com", "a secret password", "my-database-connection")
    .start(new BaseCallback<Credentials>() {
        @Override
        public void onSuccess(Credentials payload) {
            //Signed Up & Logged in!
        }
    
        @Override
        public void onFailure(AuthenticationException error) {
            //Error!
        }
    });
```


#### Get user information

```java
authentication
   .tokenInfo("user id_token")
   .start(new BaseCallback<Credentials>() {
       @Override
       public void onSuccess(UserProfile payload) {
           //Got the profile!
       }
   
       @Override
       public void onFailure(AuthenticationException error) {
           //Error!
       }
   });
```


### Management API (Users)

The client provides methods to link and unlink users account.

Create a new instance by passing the account:

```java
Auth0 account = new Auth0("clientId", "domain");
UsersAPIClient apiClient = new UsersAPIClient(account);
```


#### Link users

```java
apiClient
    .link("primary user_id", "primary id_token", "secondary id_token")
    .start(new BaseCallback<List<UserIdentity>>() {
        @Override
        public void onSuccess(List<UserIdentity> payload) {
            //Got the updated identities! Accounts linked.
        }

        @Override
        public void onFailure(Auth0Exception error) {
            //Error!
        }
    });
```

#### Unlink users

```java
apiClient
    .unlink("primary user_id", "primary id_token", "secondary user_id", "secondary provider")
    .start(new BaseCallback<List<UserIdentity>>() {
        @Override
        public void onSuccess(List<UserIdentity> payload) {
            //Got the updated identities! Accounts linked.
        }

        @Override
        public void onFailure(Auth0Exception error) {
            //Error!
        }
    });
```

### Update User Metadata

```java
Map<String, Object> metadata = new HashMap<>();
metadata.put("name", Arrays.asList("My", "Name", "Is"));
metadata.put("phoneNumber", "1234567890");

apiClient
    .updateMetadata("user_id", "id_token", metadata)
    .start(new BaseCallback<UserProfile, ManagementException>() {
        @Override
        public void onSuccess(UserProfile payload) {
            //Metadata updated
        }
    
        @Override
        public void onFailure(ManagementException error) {
            //Error!
        }
    });
```

### Web-based Auth

First go to [Auth0 Dashboard](https://manage.auth0.com/#/applications) and go to your application's settings. Make sure you have in *Allowed Callback URLs* a URL with the following format:

```
https://{YOUR_AUTH0_DOMAIN}/android/{YOUR_APP_PACKAGE_NAME}/callback
```

#### Configuration: Using the WebView

Open your app's `AndroidManifest.xml` file and add the following permissions.

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

> The `ACCESS_NETWORK_STATE` permission it's not mandatory, but used by the WebAuthActivity to check for internet availability before executing a request.

Also register the WebAuthActivity inside the `application` tag like


```xml
    <application android:theme="@style/AppTheme">

        <!-- ... -->
        
        <activity
            android:name="com.auth0.android.provider.WebAuthActivity"
            android:theme="@style/MyAppTheme"/>
            
        <!-- ... -->

    </application>
```

Finally, define a constant like `WEB_REQ_CODE` that holds the request code (an `int`), that will be sent back with the intent once the authentication is finished in the webview.


#### Configuration: Using the Browser

Open your app's `AndroidManifest.xml` file and add the following permission.

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Also register the intent filters inside your activity's tag, so you can receive the call in your activity. Note that you will have to specify the callback url inside the `data` tag.

```xml
    <application android:theme="@style/AppTheme">

        <!-- ... -->
        
        <activity
            android:name="com.mycompany.MainActivity"
            android:theme="@style/MyAppTheme">
            
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data
                    android:host="{YOUR_AUTH0_DOMAIN}"
                    android:pathPrefix="/android/{YOUR_APP_PACKAGE_NAME}/callback"
                    android:scheme="https" />
            </intent-filter>
            
        </activity>
            
        <!-- ... -->

    </application>
```

In your `Activity` class define a constant like `WEB_REQ_CODE` that holds the request code (an `int`), that will be sent back with the intent once the auth is finished in the browser/webview. To capture the response, override the `OnActivityResult` and the `onNewIntent` methods and call `WebAuthProvider.resume()` with the received parameters:

```java
public class MyActivity extends Activity {

    private static final int WEB_REQ_CODE = 110;

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        switch (requestCode) {
            case WEB_REQ_CODE:
                lockView.showProgress(false);
                WebAuthProvider.resume(requestCode, resultCode, data);
                break;
            default:
                super.onActivityResult(requestCode, resultCode, data);
        }
    }
    
    @Override
    protected void onNewIntent(Intent intent) {
        if (WebAuthProvider.resume(intent)) {
            return;
        }
        super.onNewIntent(intent);
    }
}

```

#### Authenticate with any Auth0 connection

```java
WebAuthProvider.init(account)
                .withConnection("twitter")
                .start(MainActivity.this, authCallback, WEB_REQ_CODE);
```

#### Use Code grant with PKCE

Before you can use `Code Grant`, make sure to go to your application's [dashboard](https://manage.auth0.com/#/applications) and check in the Settings section that the `Token Endpoint Authentication Method` is set to `NONE`.

```java
WebAuthProvider.init(account)
                .useCodeGrant(true)
                .start(MainActivity.this, authCallback, WEB_REQ_CODE);
```

#### Use browser instead of WebView

```java
WebAuthProvider.init(account)
                .useBrowser(true)
                .start(MainActivity.this, authCallback, WEB_REQ_CODE);
```
> Make sure to check the callback url and manifest configuration explained above.


#### Specify scope

```java
WebAuthProvider.init(account)
                .withScope("user openid")
                .start(MainActivity.this, authCallback, WEB_REQ_CODE);
```

#### Authenticate with Auth0 hosted login page

```java
WebAuthProvider.init(account)
                .useBrowser(true)
                .start(MainActivity.this, authCallback, WEB_REQ_CODE);
```


## FAQ 

A bug _may_ appear when trying to `build` the project, regarding an `invalid package` on the `okio` dependency. This can be fixed by adding in the module `build.gradle` file the following statement:

```gradle
android {
    //...
    lintOptions {
       warning 'InvalidPackage'
    }
}
```

ref: https://github.com/square/okio/issues/58#issuecomment-72672263

## What is Auth0?

Auth0 helps you to:

* Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, amont others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**.
* Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
* Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
* Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
* Analytics of how, when and where users are logging in.
* Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a free Auth0 Account

1. Go to [Auth0](https://auth0.com) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Author

[Auth0](auth0.com)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE.txt) file for more info.
