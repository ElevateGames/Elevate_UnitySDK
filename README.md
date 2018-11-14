# ElevateSDK Access Process For Unity3D

## 一、Unity3D Configuration

#### 1.Drag the following folder into the Unity3D scripts directory
 * `Elevate folder`
 * `LitJson folder`(If your project already contains LitJson dll or scripts, please ignore this step.)

#### 2.Click the toolbar `Elevate -> CreateConfig` to create a configuration file, which will be created under the `Resources` folder.
 * Modify "`App Id`" as your AppId(`Required`)
 * Click "`Apply`" to apply changes

#### 3.Add `game login` entry
```java
    ElevateManager.Instance.RegisterLoginRequest(Login);
	
    //Your game login entry is similar to the following method
    void Login(UserData user){...}
```

#### 4.Add `game logout` entry
```java
    ElevateManager.Instance.RegisterLogoutRequest(Logout);
	
    //Your game logout entry is similar to the following method
    void Logout(){...}
```

#### 5.Add `game exit callback`
```java
    ElevateManager.Instance.RegisterOnExitGame(OnExitGame);
	
    ////Your game logout entry is similar to the following method
    void OnExitGame(){
        ...
        Application.Quit();	
    }
```

#### 6.`SDK logout` entry
```java
    ElevateManager.Instance.LogoutSDK();
```

#### 7.`Pay` entry
```java
    ElevateManager.Instance.Pay("Product ID", "Product Number(default=1)");
```

#### 8.Set `Pay Callback`
```java
    ElevateManager.Instance.RegisterPayCallback(PaymentResult);
	
    //Your pay callback entry is similar to the following method
    void PaymentResult(PayResult result){...}
```

***When the in-game payment system function is turned off or disabled, the registration of the payment callback must be canceled.***

```java
	//E.g:`OnDestroy` or `OnDisable`(If you use OnDisable to cancel registration, you should re-register in `OnEnable`.)
	void OnDestroy(){
        ElevateManager.Instance.UnregisterPayCallback();
	}
```

#### 9. After the in-game payment system is initialized, call the following method to `continue the uncompleted order`

***This method must be called, otherwise the user's game property may be lost (the recharge has been paid but the corresponding game resources are not obtained)***

```java
    ElevateManager.Instance.ContinueUnfinishedOrders();
```

#### 10.`exit SDK` entry

```java
    ElevateManager.Instance.ExitSDK();
```

#### 11.Tool `toast` call entry
```java
    String message = "Toast";
    ElevateManager.Instance.Toast(message);
```

#### 12.Export the Unity project to the `Android` project
 * File -> `Build Setting`
 * Config Player Settings
    * Select the `Android` platform
    * Other Settings: Modify the package name to end with `.elegames`,e.g:`com.multiverse.sdkdemo.elegames`.
 * `Build Setting` select the `Android` platform
    * Configuration:
    	* Build System:`Gradle`
		* Export Project:`√`
 * Click on `Export` to export to Android Project.
 * If `Unable to list target platforms. Please make sure the android sdk path is correct.` problem occurs.
    * Make sure the sdk directory is correct.
    * Download [Android-tool-25](https://dl.google.com/android/repository/tools_r25.2.3-windows.zip)
    * After renaming the `tools folder` in the sdk directory, extract the `tools_r25.2.3-windows.zip` file and extract the tools folder to the sdk directory.

## 二、Android Studio Configuration

#### 1.Open the exported project file with `Android Studio`.
 * File -> Open -> Select the Android project exported in the previous step, select the `upper directory` of the libs directory, and click OK.
 * After opening, the window `Gradle Sync` will pop up, click OK, wait for the project to load.

#### 2.Switch "Android" in the upper left corner to "`Project`" directory structure and modify the following files.

##### **Modify the gradle tool version:**
 
 * gradle -> wrapper -> gradle-`wrapper.properties`  Change the gradle version to`gradle-4.6-all`.
    

##### **Add aar file:**
 
 * Add the `elevate-sdk.aar` file to the `libs` directory.
 

##### **Modify the build.gradle file:**
 
 * Open the `provided build.gradle` file and find the following:
	
```java

	android{
		defaultConfig{
			//Modify this field to be consistent with the build.gradle file in the project.
			applicationId 'com.multiverse.sdkdemo.elegames'
			...
		}
	}
```

   * Save the changes and use the file `replace` the `build.gradle` file in the project directory.
    
##### **Modify the AndroidManifest.xml file:**
   * Find the `src` -> AndroidManifest.xml file
    
   * Add the following `feature` and `permission` to the `<manifest/>` module.
```java
    <activity android:name="com.braintreepayments.api.BraintreeBrowserSwitchActivity"
    	android:launchMode="singleTask">
        
    	<intent-filter>
    	    <action android:name="android.intent.action.VIEW" />
    	    <data android:scheme="${applicationId}.braintree" />
    	    <category android:name="android.intent.category.DEFAULT" />
    	    <category android:name="android.intent.category.BROWSABLE" />
    	</intent-filter>
      
    </activity>
    
    <activity android:name="com.elevategames.api.WebActivity"
    	android:launchMode="standard"
    	android:screenOrientation="portrait"
    	android:theme="@style/Theme.AppCompat.DayNight.NoActionBar"/>
    
    <provider
    	android:name="android.support.v4.content.FileProvider"
    	android:authorities="${applicationId}.fileprovider"
    	android:exported="false"
    	android:grantUriPermissions="true">
    	<meta-data
    	    android:name="android.support.FILE_PROVIDER_PATHS"
    	    android:resource="@xml/file_paths" />
    </provider>
    
    <activity android:name="com.braintreepayments.api.threedsecure.ThreeDSecureWebViewActivity"
    	android:theme="@style/bt_three_d_secure_theme"
    	android:screenOrientation="portrait"/>
        
    <activity android:name="com.elevategames.pay.ui.DropInActivity"
    	android:theme="@style/bt_drop_in_activity_theme"
    	android:screenOrientation="portrait"
    	android:launchMode="standard"/>
        
    <activity android:name="com.elevategames.pay.ui.AddCardActivity"
    	android:theme="@style/bt_add_card_activity_theme"
    	android:screenOrientation="portrait"/>

    <service android:name="com.elevategames.api.WebSocketClient" android:enabled="true"/>

    <activity
    	android:name="com.alipay.sdk.app.H5PayActivity"
    	android:configChanges="orientation|keyboardHidden|navigation"
    	android:exported="false"
    	android:screenOrientation="behind" >
    </activity>
    <activity
    	android:name="com.alipay.sdk.auth.AuthActivity"
    	android:configChanges="orientation|keyboardHidden|navigation"
    	android:exported="false"
    	android:screenOrientation="behind" >
    </activity>
```

#### 3.After editing, click "`Sync Now`" in the upper right corner to wait for the synchronization to complete.
        After completion, click Build->Rebuild Project to generate apk (generated apk in the project name->build->outputs->apk directory), or click Run to test the project.