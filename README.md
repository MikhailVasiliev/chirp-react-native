# DEPRECATED

## This project is no longer actively maintained.

See [Getting Started with React Native](https://facebook.github.io/react-native/docs/getting-started.html)

Under the Building Projects with Native Code tab.

See our [blog post](https://blog.chirp.io/chirp-react-native) for how to get
started with Chirp and React Native in your own projects.

----

## Getting Started

To get started right with Chirp and React Native you can use our example project.

You will need to sign up to the [Chirp Developer Hub](https://developers.chirp.io/sign-up),
and copy your Chirp app key and secret into `App.js`.

1. Clone the project

    `git clone https://github.com/chirp/chirp-react-native`

2. Install node_modules

    `cd chirp-react-native`

    `yarn install`

3. [iOS only] install dependencies

    `cd ios`

    `pod install`

4. Enter your application key and secret into `App.js`.

    `const key = 'YOUR_CHIRP_APPLICATION_KEY';`

    `const secret = 'YOUR_CHIRP_APPLICATION_SECRET';`

    `const config = 'YOUR_CHIRP_APPLICATION_CONFIG';`

5. Check that each project builds by opening in the `.xcworkspace` in Xcode,
and the `android` folder in Android Studio. This can solve some common set up issues.

6. Run the demo.

    `react-native run-ios`

    `react-native run-android`


----

## Usage

Follow the instructions below to get started with Chirp in your own project.

You will need to sign up to the [Chirp Developer Hub](https://developers.chirp.io/sign-up),
and copy your Chirp app key and secret into `App.js`.

### iOS

Open the xcode project in the `/ios` folder, and build first of all.
See [Troubleshooting](https://github.com/chirp/chirp-react-native/#troubleshooting) section.

Then follow `Install the SDK` steps at [Getting Started [iOS]](https://developers.chirp.io/docs/getting-started/ios/) to include the Chirp SDK into your project.

Copy [RCTChirpSDK.m](https://github.com/chirp/chirp-react-native/blob/master/ios/RCTChirpSDK.m) and [RCTChirpSDK.h](https://github.com/chirp/chirp-react-native/blob/master/ios/RCTChirpSDK.h) to your project.


### Android

Open the `/android` folder in Android Studio, and check the project builds.
See [Troubleshooting](https://github.com/chirp/chirp-react-native/#troubleshooting) section.

Then follow the `Install the SDK` steps at [Getting Started [Android]](https://developers.chirp.io/docs/getting-started/android/) to include the Chirp SDK into your project.

Copy [RCTChirpSDKModule.java](https://github.com/chirp/chirp-react-native/blob/master/android/app/src/main/java/com/chirpreactnative/RCTChirpSDKModule.java) and [RCTChirpSDKPackage.java](https://github.com/chirp/chirp-react-native/blob/master/android/app/src/main/java/com/chirpreactnative/RCTChirpSDKModule.java) to the project.

Import into your MainApplication.java

```java
import com.chirpsdk.rctchirpsdk.RCTChirpSDKPackage;
```

Add the ChirpSDK to the `createNativeModules` function

```java
@Override
  public List<NativeModule> createNativeModules(
        ReactApplicationContext reactContext) {
    List<NativeModule> modules = new ArrayList<>();

    modules.add(new RCTChirpSDKModule(reactContext));  // <---

    return modules;
}
```


### Application

Now the setup is complete, you can add the Chirp SDK to your React Native application.
You can use the `react-native-permissions` package to ensure that microphone permissions
have been granted.

```bash
yarn add react-native-permissions
```

In App.js

```javascript
import { NativeEventEmitter, NativeModules } from 'react-native';
import Permissions from 'react-native-permissions';

const ChirpSDK = NativeModules.ChirpSDK;
const ChirpSDKEmitter = new NativeEventEmitter(ChirpSDK);

export default class App extends Component<{}> {

  async componentDidMount() {
    const response = await Permissions.check('microphone')
    if (response !== 'authorized') {
      await Permissions.request('microphone')
    }

    this.onReceived = ChirpSDKEmitter.addListener(
      'onReceived',
      (event) => {
        if (event.data) {
          this.setState({ data: event.data });
        }
      }
    )
    this.onError = ChirpSDKEmitter.addListener(
      'onError', (event) => { console.warn(event.message) }
    )

    ChirpSDK.init(key, secret);
    ChirpSDK.setConfig(config);
    ChirpSDK.start();
    ChirpSDK.sendRandom();
  }

  componentWillUnmount() {
    this.onReceived.remove();
    this.onError.remove();
  }
}
```

## Reference


```javascript
// Initialise the SDK.
ChirpSDK.init(String key, String secret)

// Explicitly set the config string
ChirpSDK.setConfig(String config)

// Start the SDK
ChirpSDK.start()

// Stop the SDK
ChirpSDK.stop()

// Send an array of bytes to the speaker
ChirpSDK.send(Array data)

// Send a random array of bytes to the speaker
ChirpSDK.sendRandom()

// This event is called when the state of the SDK changes.
// The event contains the following body, where the state constants are accessible from the ChirpSDK interface.
// { "status": ChirpSDK.CHIRP_SDK_STATE_RUNNING }
ChirpSDKEmitter.addListener('onStateChanged', (event) => {})

// This event is called when the SDK begins sending data.
// The event contains the following body.
// { "data": [0, 1, 2, 3, 4] }
ChirpSDKEmitter.addListener('onSending', (event) => {})

// This event is called when the SDK has finished sending data.
// The event contains the following body.
// { "data": [0, 1, 2, 3, 4] }
ChirpSDKEmitter.addListener('onSent', (event) => {})

// This event is called when the SDK has started to receive data.
ChirpSDKEmitter.addListener('onReceiving', () => {})

// This event is called when the SDK has finished receiving data.
// The event contains the following body.
// { "data": [0, 1, 2, 3, 4] }
ChirpSDKEmitter.addListener('onReceived', (event) => {})

// This event is called if the SDK encounters an error.
// The event contains the following body.
// { "message": "An error has occurred" }
ChirpSDKEmitter.addListener('onError', (event) => {})

```

----

## TroubleShooting

React Native with native support doesn't work so well out of the box, so here
are some things that can go wrong.

### iOS

- Add react-native modules to header search paths. Open ios/<project>.xcodeproj.
Go to Project -> Build Settings -> Search Paths -> Header Search Paths
add the following with recursive set.

    `$(SRCROOT)/../node_modules/react-native/React`

- Create js bundle [No main.jsbundle found]

    `react-native bundle --entry-file ./index.js --platform ios --bundle-output ios/main.jsbundle`

- Fix third party modules [config.h file not found]

    `cd node_modules/react-native/third-party/glog-0.3.4`

    `../../scripts/ios-configure-glog.sh`

### Android

- Use java 8
[Could not initialize class com.android.sdklib.repository.AndroidSdkHandler]

    `brew tap caskroom/versions`
    `brew cask install java8`
    `export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home`

- Upgrade gradle to 2.3.3 by updating `build.gradle`
