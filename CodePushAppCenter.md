# Code Push App Center

## React Native Module for CodePush

Getting Started:

- npm install --save react-native-code-push

In order to integrate CodePush into your Android project, please perform the following steps:

### Plugin Installation and Configuration for React Native 0.60 version and above (Android)

1. In your `android/settings.gradle` file, make the following additions at the end of the file:

    ```gradle
      ...
      include ':app', ':react-native-code-push'
      project(':react-native-code-push').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-code-push/android/app')
    ```
    
2. In your `android/app/build.gradle` file, add the `codepush.gradle` file as an additional build task definition underneath `react.gradle`:

    ```gradle
    ...
    apply from: "../../node_modules/react-native/react.gradle"
    apply from: "../../node_modules/react-native-code-push/android/codepush.gradle"
    ...
    ```

3. Update the `MainApplication.java` file to use CodePush via the following changes:

    ```java
    ...
    // 1. Import the plugin class.
    import com.microsoft.codepush.react.CodePush;
    public class MainApplication extends Application implements ReactApplication {
        private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
            ...
            // 2. Override the getJSBundleFile method in order to let
            // the CodePush runtime determine where to get the JS
            // bundle location from on each app start
            @Override
            protected String getJSBundleFile() {
                return CodePush.getJSBundleFile();
            }
        };
    }
    // 2. (the one located in your-project/android/app/build.gradle), add these lines in:
    android.buildTypes.each { buildType ->
        // to prevent incorrect long value restoration from strings.xml we need to wrap it with double quotes
        // https://github.com/microsoft/cordova-plugin-code-push/issues/264
        buildType.resValue 'string', "CODE_PUSH_APK_BUILD_TIME", String.format("\"%d\"", System.currentTimeMillis())
    }
    ```

4. Add the Deployment key to `strings.xml`:

   To let the CodePush runtime know which deployment it should query for updates, open your app's `strings.xml` file and add a new string named `CodePushDeploymentKey`, whose value is the key of the deployment you want to configure this app against (like the key for the `Staging` deployment for the `FooBar` app). You can retrieve this value by running `appcenter codepush deployment list -a <ownerName>/<appName> -k` in the CodePush CLI (the `-k` flag is necessary since keys aren't displayed by default) and copying the value of the `Key` column which corresponds to the deployment you want to use (see below). Note that using the deployment's name (like Staging) will not work. The "friendly name" is intended only for authenticated management usage from the CLI, and not for public consumption within your app.

   ![Deployment list](https://cloud.githubusercontent.com/assets/116461/11601733/13011d5e-9a8a-11e5-9ce2-b100498ffb34.png)

   In order to effectively make use of the `Staging` and `Production` deployments that were created along with your CodePush app, refer to the [multi-deployment testing](../README.md#multi-deployment-testing) docs below before actually moving your app's usage of CodePush into production.

   Your `strings.xml` should looks like this:

   ```xml
    <resources>
        <string name="app_name">AppName</string>
        <string moduleConfig="true" name="CodePushDeploymentKey">DeploymentKey</string>
    </resources>
    ```

    *Note: If you need to dynamically use a different deployment, you can also override your deployment key in JS code using [Code-Push options](./api-js.md#CodePushOptions)*

Plugin Usage:

```js
//import liraries App.js
import React, { useEffect, useState } from 'react';
import { Modal, View, Text, ActivityIndicator } from 'react-native';
import codePush from "react-native-code-push";

let codePushOptions = { checkFrequency: codePush.CheckFrequency.MANUAL };

const App = () => {

  const [progress, setProgress] = useState(false)

  useEffect(() => {
    codePush.sync(
      {
        updateDialog: true,
        installMode: codePush.InstallMode.IMMEDIATE
      },
      codePushStatusDidChange,
      codePushDownloadDidProgress
    );
  }, [])


  function codePushStatusDidChange(syncStatus) {
    switch (syncStatus) {
      case codePush.SyncStatus.CHECKING_FOR_UPDATE:
        console.log("Checking for update.")
        break;
      case codePush.SyncStatus.DOWNLOADING_PACKAGE:
        console.log("Download packaging....")
        break;
      case codePush.SyncStatus.AWAITING_USER_ACTION:
        console.log("Awaiting user action....")
        break;
      case codePush.SyncStatus.INSTALLING_UPDATE:
        console.log("Installing update")
        setProgress(false)
        break;
      case codePush.SyncStatus.UP_TO_DATE:
        console.log("codepush status up to date")
        break;
      case codePush.SyncStatus.UPDATE_IGNORED:
        console.log("update cancel by user")
        setProgress(false)
        break;
      case codePush.SyncStatus.UPDATE_INSTALLED:
        console.log("Update installed and will be applied on restart.")
        setProgress(false)
        break;
      case codePush.SyncStatus.UNKNOWN_ERROR:
        console.log("An unknown error occurred")
        setProgress(false)
        break;
    }
  }

  function codePushDownloadDidProgress(progress) {
    setProgress(progress)
  }

  console.log("progress value++", progress)


  const showProgressView = () => {
    return (
      <Modal
        visible={true}
        transparent
      >
        <View style={{
          flex: 1,
          backgroundColor: 'rgba(0,0,0,0.8)',
          justifyContent: 'center',
          alignItems: 'center',
        }}>
          <View style={{
            backgroundColor: 'white',
            borderRadius: 8,
            padding: 16
          }}>
            <Text>In Progress.......</Text>

            <View
              style={{ alignItems: 'center' }}
            >
              <Text style={{marginTop: 16}}>{`${(Number(progress?.receivedBytes)/1048576).toFixed(2)}MB/${(Number(progress?.totalBytes)/1048576).toFixed(2)}`}</Text>
              <ActivityIndicator style={{ marginVertical: 8 }} color={'blue'} />
              <Text>{((Number(progress?.receivedBytes) / Number(progress?.totalBytes)) * 100).toFixed(0)}%</Text>
            </View>
          </View>
        </View>
      </Modal>
    )
  }

  return (
    <View style={{ flex: 1 }}>
      <Text>duong 1 </Text>
      {!!progress ? showProgressView() : null}
    </View>
  );
};

export default codePush(codePushOptions)(App);
```

## App Center
Account Management:
1. Login
  ```
    appcenter login
  ```
Install the App Center CLI:
1. Install the App Center CLI:
  ```
    npm install -g appcenter-cli
  ```
2. Release an app update:
  ```
    appcenter codepush release-react -a <ownerName>/MyApp
  ```
