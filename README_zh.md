# ElevateSDK Access Process For Unity3D

## 一、Unity3D配置

#### 1.将以下文件夹拖入到Unity3D脚本目录下
 * `Elevate文件夹`
 * `LitJson文件夹`(如果你的项目中已经包含LitJson的dll或者脚本，此步骤请忽略)

#### 2.点击工具栏`Elevate -> CreateConfig`创建配置文件，文件将创建在`Resources`文件夹下
 * 修改"`App Id`"为你的AppId(`必填`)
 * 点击"`Apply`"应用修改

#### 3.添加`账户登录`入口
```java
    ElevateManager.Instance.RegisterLoginRequest(Login);
	
    //你的游戏登录入口类似如下方法
    void Login(UserData user){...}
```

#### 4.添加`账户登出`入口
```java
    ElevateManager.Instance.RegisterLogoutRequest(Logout);
	
    //你的游戏登出入口类似如下方法
    void Logout(){...}
```

#### 5.添加`游戏退出回调`
```java
    ElevateManager.Instance.RegisterOnExitGame(OnExitGame);
	
    ////你的游戏退出回调类似如下方法
    void OnExitGame(){
        ...
        Application.Quit();	
    }
```

#### 6.`SDK登出`入口
```java
    ElevateManager.Instance.LogoutSDK();
```

#### 7.`支付`入口
```java
    ElevateManager.Instance.Pay("商品ID", "商品数量(默认为1)");
```

#### 8.注册`支付回调`
```java
    ElevateManager.Instance.RegisterPayCallback(PaymentResult);
	
    //你的支付回调入口类似如下方法
    void PaymentResult(PayResult result){...}
```

***当游戏内支付系统功能被关闭或禁用时，必须取消支付回调的注册***

```java
	//例如OnDestroy或OnDisable(若使用OnDisable取消注册，应在OnEnable中重新注册)
	void OnDestroy(){
        ElevateManager.Instance.UnregisterPayCallback();
	}
```

#### 9.游戏内支付系统初始化完成后，调用以下方法`继续未完成订单`

***此方法必须被调用，否则可能会造成用户游戏财产损失(充值已支付却未获得相应的游戏资源)***

```java
    ElevateManager.Instance.ContinueUnfinishedOrders();
```

#### 10.`退出SDK`调用入口

```java
    ElevateManager.Instance.ExitSDK();
```

#### 11.工具`toast`调用入口
```java
    String message = "Toast";
    ElevateManager.Instance.Toast(message);
```

#### 12.Unity项目导出为`Android`工程
 * File -> `Build Setting`
 * 配置Player Settings
    * 选择`Android`平台
    * Other Settings:修改包名以`.elegames`结尾即可，例如:`com.multiverse.sdkdemo.elegames`
 * `Build Setting`选择Android平台
    * 配置:
    	* Build System:`Gradle`
		* Export Project:`√`
 * 点击`Export`导出为Android工程
 * 如果出现`Unable to list target platforms. Please make sure the android sdk path is correct.`问题
    * 确保sdk目录正确
    * 下载[安卓工具包-25](https://dl.google.com/android/repository/tools_r25.2.3-windows.zip)
    * 将sdk目录下的`tools文件夹`重命名后，将下载的`tools_r25.2.3-windows.zip`文件解压提取tools文件夹至sdk目录下
	
## 二、Android Studio配置

#### 1.使用`Android Studio`打开导出的工程文件
 * File -> Open -> 选择上一步导出的Android工程，选中libs目录的`上一级目录`，点击OK
 * 打开后将弹出窗口`Gradle Sync`，点击OK，等待工程加载完成

#### 2.将左上角“Android”切换为“`Project`”目录结构并修改以下文件

##### **修改gradle工具版本：**
 
 * gradle -> wrapper -> gradle-`wrapper.properties` 将gradle版本改为`gradle-4.6-all`
    

##### **添加aar文件：**
 
 * 将`elevate-sdk.aar`文件添加到`libs`目录下
 

##### **修改build.gradle文件：**
 
 * 打开`提供的build.gradle`文件，找到：
	
```java

	android{
		defaultConfig{
			//修改该字段与项目中build.gradle文件中的一致
			applicationId 'com.multiverse.sdkdemo.elegames'
			...
		}
	}
```

   * 保存修改并使用该文件`替换`项目目录下的`build.gradle`文件
    
##### **修改AndroidManifest.xml文件：**
   * 找到`src` -> AndroidManifest.xml文件
    
   * 在`<manifest/>`模块中添加以下`feature`和`permission`
```java

	<uses-feature android:name="android.software.leanback"
		android:required="false"/>
		
	<uses-permission android:name="android.permission.INTERNET"/>        
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
	<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>        
 ```

   * 在`<application>`标签中添加 `persistent="true" tools:replase="icon, theme"`
    
   * 在`<application/>`模块中的`<activity>`标签中修改android:name=`"com.sdkdemo.elegames.UnityPlayerActivity"`为android:name=`"com.elevategames.api.MainActivity"`
   
   * 在`<application/>`模块中的`<activity>`标签中删除`android:launchMode="singleTask"`
   
   * 在`<mainifest/>`模块中添加以下activity和service
    
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

#### 3.修改完毕后点击右上角"`Sync Now`"等待同步完成
        完成后可点击Build->Rebuild Project生成apk(生成的apk在项目名->build->outputs->apk目录下)，或者点击运行进行项目测试。