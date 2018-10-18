# AppCoins Unity Plugin

![Logo](Screenshots/logos.png)

This is the official Unity plugin for the AppCoins Protocol that allows you to integrate AppCoins In-App Billing into your Unity Android game.

## About AppCoins Unity Plugin
This plugin is developed to support the  [AppCoins](https://appcoins.io/) protocol.
 Integrating this plugin allows developers to integrate the AppCoins In-App Billing into their game.

## Pre requisites to run the project
To successfully run the project you need to do couple of things.
### Setup a Wallet
1. Download and install _AppCoins Wallet_ app (you can get it on [Aptoide](https://appcoins-wallet.en.aptoide.com/?store-name=asf-store) or [GooglePlay](https://play.google.com/store/apps/details?id=com.appcoins.wallet))

2. Open _AppCoins Wallet_ and create or restore a wallet

### Register on BDS
1. Go to the [BDS](https://blockchainds.com/) website and press the _Sign Up_ button. Press the _Developer_ button.

2. Fill in all the required details.

3. After this you'll have to read through and accept the _Terms of service_.

4. After this you'll have to read through and accept the _Certified Developer Distribution Agreement_.

5. Wait a few moments and you should've received a confirmation e-mail on the e-mail account you supplied. Open it and click the _Confirm Email_ link.

6. You'll be redirected to the _BDS portal_ and have your account confirmed. You can now login by pressing the _Login_ button, filling in your details and pressing the _Login_ button again.

7. Congratulations! You're now inside the BDS portal, ready to upload your first app. Before that let's get your _BDS Wallet Address_. On the left pane click _Wallet_ then click _Deposit AAPC_. This will load give you your _BDS Wallet Address_ store it as you will need it later.

8. Now on the same left pane click _Manage Apps_ and the _Add App_.

9. Fill in the _app package name_ (should be the same as the one used for Google Play). Press the _Next_ button.

10. Now you're asked if you want to start monetizing your app. Whatever the answer, make sure that you store the _Public Key_ that's given to you next to the _Wallet Address_. You'll need both these values as inputs when integrating the plugin on Unity.

### First App upload
Time to upload an APK of your app. For the upload to get through there are some mandatory steps that you need to follow.

- **Mandatory**:
It's mandatory that the APK does NOT feature the Google billing permission. To do this you need to edit the AndroidManifest.xml and remove the line:

```
<uses-permission android:name="com.android.vending.BILLING" />
```
- **Mandatory**:
The uploaded APK has to be signed (ideally with the same key used for Google Play). If you don't know how to do this check the section below **To make a signed build**

- **Mandatory**:
If your app has IAP you have to add the BDS billing permission to the manifest. Failing to do so will block the creation of IAP Products on the BDS Portal. To add the permission you need to edit the AndroidManifest.xml and add the line _uses-permission_ like so:

```
=Example AndroidManifest.xml=
<?xml version="1.0" encoding="utf-8"?>
<manifest
  xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.your.app"
  xmlns:tools="http://schemas.android.com/tools"
  android:installLocation="preferExternal">

  ...

  <uses-permission android:name="com.appcoins.BILLING" />

  ...

</manifest>
```

This app submission is mandatory and essential to enable the next step: Certification. Your app needs to be certified for the integration to work. You can't even test it before the certification process is done, as the service will act like if your app does not exist.

To certify the app you have two options.

- If you’re distributing this app on Google Play, with the same package name, we can send you a link to the contact e-mail associated to that app. Simply open the link on that e-mail and your app will be certified.
With this method your app is ready to be distributed on Aptoide with no action from the BDS team.

- If this package isn’t on Google Play, you can download a blank APK, sign it (with the same key you signed the app you uploaded) and upload it to BDS
This method will require the BDS team to review your request to assure the app is indeed yours and is not infringing any Intelectual Property. This process should last no longer than 5 working days

After this process is done you're finally ready to test the game. But while the certification is pending you can start to look at next section.
## Integrating the plugin into your game

1. Make sure you carefully went through all the pre-requisites.

2. Download the plugin package corresponding to your Unity version (The supported Unity versions are 2018, 2017 and 5.6). You can find them on the latest release [here](https://github.com/Aptoide/bds-unity-plugin/releases).  Open the package in your Unity project (double click the file or in Unity go to Assets -> Import Package -> Custom Package.... and find the file you just downloaded). Make sure to import the example folder.

3. Open the example scene.

4. Make sure you are on the Android platform. Go to File -> Build Settings, select Android and then switch platform.

![platform](Screenshots/platform.png)

5. On the Build Settings screen click "Player Settings...". Change the package name to: "com.appcoins.sample"

![packageName](Screenshots/packageName.png)

6. Build and run for Android.

7. If everything goes well you should see the message:
"OnInitialized: PASS" on the top of the phone screen. This means that you have everything correctly setup. You can try the consumable and non-consumable purchases to also see that they work.

  **NOTE:** this will only work on the main network (Ethereum)

![message](Screenshots/message.png)

8. Add these imports into your logic class
```
        using Appcoins.Purchasing;
        using UnityEngine.Purchasing;
```

9. Define product ids
```
        public static string kYourProductID = "yourSkuID";
        public static string kOtherProductID = "otherSkuID";
```
10. Define outlet for Purchaser class
```
        [SerializeField]
        private Purchaser _purchaser;
```
11. Define OnPurchaseSuccess to process the purchase result
```
        private void OnPurchaseSuccess(AppcoinsProduct product)
        {
            Debug.Log("On purchase success called with product: \n skuID: " + product.skuID + " \n type: " + product.productType);

            if (product.skuID == kYourProductID) {
                GiveYourProduct();
            } else if(product.skuID == kOtherProductID) {
                GiveOtherProduct();
            }
        }
```
12. Setup purchaser with defined product ids and correct product types
```
        void SetupPurchaser() {
            _purchaser.onInitializeSuccess.AddListener(OnInitializeSuccess);
            _purchaser.onPurchaseSuccess.AddListener(OnPurchaseSuccess);

            _purchaser.AddProduct(kYourProductID, ProductType.Consumable);
            _purchaser.AddProduct(kOtherProductID, ProductType.NonConsumable);

            //Only call initialize after adding all products!
            _purchaser.InitializePurchasing();
       }
```
13. Call the function you just created when you want to initialize the Purchasing system. We want it as soon as possible so we add it to the Start method of our logic class.

```
        void Start () {
          //... your code ...
          SetupPurchaser();
        }
```

14. Create an instance of the prefab _AppcoinsPurchasing_ located on the Prefabs folder.

15. Fill it in with the appropriate values for _Developer Wallet Address_ and _Developer BDS Public Key_ that you previously stored during the pre-requisites steps. (The default ones are working for the sample app)

16. Make sure to drag _AppcoinsPurchasing_ game object to the purchaser outlet your created in your logic object.

**NOTE:** If you want to easily debug the interactions with the BDS Purchasing system, you can attach a Unity.Text label to the Purchaser Status text outlet.

**NOTE:** If you want to get an IAP value in APPC you can easily do that by calling GetAAPCPriceStringForSKU(string skuID) from AppcoinsPurchasing object.

```
string priceStr = appcoinsPurchasing.GetAAPCPriceStringForSKU(someSkuID);

```

## Custom Payload Validation

The SDK allows you to have your own payload validation mechanism for extra safety. You can check a working sample of this mechanism on the provided sample. The Logic class acts as our custom validator and simply returns true.

**NOTE:** If you're reusing the Logic class from the sample and don't want custom payload validation you just have to:

- remove the _IPayloadValidator_ from the class definition
- remove the call to _SetupCustomValidator_ on _SetupPurchaser_ function
- remove the _IsValidPayload_ method from the Logic class.
- you can skip the rest of this section

If you want to implement custom validation for your purchases' payload you have to follow these steps:

1. Make sure your validator class implements our inteface  _IPayloadValidator_. (You can reuse the class you created to handle the purchase logic)

```
public class Logic : MonoBehaviour, IPayloadValidator {

}
```

2. Implement the mandatory method that handles the validation.

```
public bool IsValidPayload(string payload) {
  Debug.Log("Custom payload validation in place! Payload is " + payload);
  //Add your custom code here

  return true; //validates the purchase by default
}
```

3. In your purchase logic class, make sure you setup your validation class as a custom validator (in this example they're the same so we pass this as a reference). You can do this in the end of the SetupPurchaser method:

```
        void SetupPurchaser() {
          ...
          previous code
          ...

          //Setup custom validator with a reference to your class
          _appcoins.SetupCustomValidator(this);
        }

```

## Versioning
The priority of BDS and partners is to provide a good user experience to the user of Android apps and games.

BDS strongly recommends to use:
- Package ID: the same as in Google Play (e.g. “com.example.myapp”)
- Developer's signature: the same as in Google Play
- Vercode: higher than Google Play, by adding an extra digit - different than 0- in the left (e.g.: vercode Play=“573”, vercode BDS=“1573”)

To know more about this please check the [BDS FAQ](https://blockchainds.com/faq)

# Good practices you should follow

Disable purchase buttons if purchasing is not initialized yet.

Disable purchase buttons for already owned non-consumable buttons.

## To build the project

Go to the build menu (File -> Build Settings)
1. Check that the build system is set to "Gradle"
(if the import was done correctly this should've changed automatically).

![picture](Screenshots/buildSystemGradle.png)

Now click _Player Settings_.

On the _Player Settings_ window:

2. Click the other settings panel

3. Make sure you change the package name to your liking.

4. Make sure that you have min sdk version set to 21 (if the import was done correctly this should've changed automatically).

**Unity 2018.2b (and above)**

1. Connect the phone to your machine and click _Build and Run_

You should have your game running on the phone!

**Unity 2018.1.X and below (till Unity 5.6)**

1. Close the _Player Settings_ window

2. On the top bar click _AppCoins_

3. Click _Android Custom Build_

4. This popup will show up

![customBuild](Screenshots/customBuild.png)

5. The gradle path should be picked from the path to your Android Studio installation

6. Gradle and Dex heap size are just to be changed if you have issues building.

7. The adb path will be picked by you (assuming you have Android SDK installed)

8. Pick the scenes you want to include. The ones added to the build settings, as well as the scene you're currently added, will automatically be selected for you

9. When you click _Confirm_ a pop up will show up asking you to pick a folder to generate the _Android_ project to. Pick a folder of your liking preferably inside the project root (it can't be the project root itself).

  **NOTE:** The final build will be located here:
  FolderYouChoseToBuildTo/ProjectName/build/outputs/apk/
  in a subfolder called debug or release, depending on build settings)

10. When you pick the folder the build process will start. The normal build process will happen and then the custom build process will kick in opening a terminal window. Unity might seem to be not responding but worry not! This is normal because it's waiting for the terminal processes to finish.

11. If you ticked _Install build when done?_ make sure you have your phone connected to the computer and that you unlock it to allow ADB to install

12. If you ticked _Run build when done?_ make sure you have your phone connected to the computer and that you unlock it to allow ADB to run

13. The build process completed. You can run the app on your phone!

## To make a signed build
1. Go to Edit -> Project Settings -> Player

2. Open the Publishing Settings tab

If you already have a keystore:

3. Tick "Use Existing Keystore" and then click "Browse keystore" to fetch it.

4. You have to provide the keystore password to allow Unity to read the key aliases.

5. Pick the correct alias and provide its password as well

 ![picture](Screenshots/keyStore.png)

6. You're done!

If you don't have a key already:

7. Tick "Create new keystore..." and then click "Browse Keystore"

8. Now pick the path where the key will be created

9. Now pick a password and write it again to confirm

10. Click the alias dropdown and then chose "Create a new key"

![picture](Screenshots/pickAlias.png)

11. Fill in all the details and click "Create Key"

![picture](Screenshots/createAlias.png)

12. Now go back to the alias dropdown and pick the alias you just created

13. You're done!

You're DONE! Congrats!
