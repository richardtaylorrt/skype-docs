# Platform considerations
## iOS considerations

### Concurrency

All of the SDK’s interfaces must be used only from the application main thread (main run loop). Notifications are delivered in the same thread as well.  As a result, no synchronization around the SDK’s interfaces is required.  The SDK, however, may create threads for internal purposes.

### Supported OS and hardware versions

The SDK works on iOS 8 and above, and on any iOS hardware model running these OS versions.
Like all current iOS apps, it must be built using XCode7.
The SDK is provided as an Objective C framework. It can be used transparently from Swift code by using a standard interoperability layer.

### API patterns

#### Error handling

Certain methods may fail. They follow the standard Objective C convention: the return value, which is either a BOOL or object pointer, shows the success (YES or non-nil) or failure (NO or nil).  The last parameter is an optional pointer to NSError * where the error instance is stored. This plays well with Swift where errors are translated into Swift exceptions. Neither Objective C nor C++ exceptions are thrown in case of I/O errors.
SFBError.h header declares the Skype for Business specific structure of userInfo.

#### Properties and KVO

The properties of many objects change dynamically due to network events or other external activity. Those changes are propagated by using the standard KVO mechanism provided by NSObject.

#### Collections

Collection properties are NSArray’s of some type. They also implement KVO and notifications are sent whenever objects are added or removed from that collection. 

>**Note**: To listen to these notifications you add an observer to the parent object and not to the NSArray property itself.

Some collections are “lazy”. They don’t hold fully initialized contents all the time. Instead when you get an item from such a collection, it transparently gets created (if not cached). You should avoid keeping references to collections’ items unless a corresponding UI view is visible.

#### Conditional methods

Some methods may be unavailable at certain times and their availability is changed dynamically. Those methods have a BOOL property named like can<DoSomething> logically attached to them. This property is KVO-enabled. You can observe it to monitor a method’s availability. You can always call a method directly without looking at corresponding property, but if so, you should be prepared to receive an error if the method is disabled.

### Audio configuration

DevicesManager audio interfaces are reduced on iOS. When headphones or a headset is connected, microphone and speaker are implicitly rerouted. The only configurable option is the speaker endpoint, which can be switched between loudspeaker (which is the phone’s own loudspeaker no matter what devices are connected) and non-loudspeaker (which is an external device or the phone’s internal speaker).

### Process lifecycle

TBD.

### Permissions and security


You must request access to the microphone and camera before calling any audio and video functions.

 
## Android considerations

### Development environment
The Skype for Business App SDK samples are intended to be 

### Android versions 

The SDK is currently supported on following versions.

||
|:-----|:-----|
|MIN_SDK_VERSION|14 (ICE_CREAM_SANDWICH)|
|TARGET_SDK_VERSION|23 (M)|

The Public Preview App SDK supports armeabi-v7a processors only. You cannot use the SDK on devices with an x86 CPU architecture. You should run any apps in development on a physical Android device. For the Public Preview, emulators are not supported.
 
### API patterns

#### Error handling

Methods provided in the SDK follow the standard Java conventions: return result where specified or throw checked exception of type **SFBException**. The **SFBException** class provides the underlying error code that caused the exception. A complete list of error codes will be published  (see **ErrorCode** enum). 

Developers are expected to handle these exceptions and provide the necessary localized error strings where appropriate.

#### Observable properties

Some of the interfaces provided expose properties. These properties are observable. Clients can register for callbacks when the property changes. They can do so by providing an implementation of the **Observable.OnPropertyChanged** class, as shown below. 
For example, Conversation State is exposed as a property and represented by STATE\_PROPERTY_ID in the Conversation interface.

```java
// Client implements a property change handler for the conversation object. E.g. 
class ConversationPropertyChangeHandler extends Observable.OnPropertyChangedCallback 
{
   public void onPropertyChanged(Observable observable, int propId) {
       switch(propId) {
	// Handle property change
	case Conversation.STATE_PROPERTY_ID:
	// Get the new state	
	conversationState = conversation.getState()
	break;
	}	
}

// Join the meeting and register for events
Conversation conversation = conversationsManager.getOrCreateConversationByUri (URI);
conversation.addOnPropertyChangedCallback (conversationPropertyChangeHandlerInstance);
```


#### Observable collections

Similar to properties, the collections provided are observable. These collections implement the **ObservableList** interface. Clients can register for callbacks when the collections changes. The can do so by providing an implementation of **ObservableList.OnListChangedCallback** class.

For example, the collection of conversations provided by the **ConversationsManager** is Observable.

#### Conditional methods

Some of the operations or actions provided by the interfaces may not be available at all times. For these operations, we expose corresponding can<doSomeThing> methods.   For example, the ChatService exposes a **canSendMessage(**) method that should be called before calling the **sendMessage()** method.

These methods are Observable properties so that clients can listen for state changes, using the technique described previously.  For example, the property ID for **canSendMessage()** is  **ChatService.CAN_SEND_MESSAGE_PROPERTY_ID**.

#### Data binding

The Android platform does not provide data binding support. This support was recently announced and it is provided as a [Beta library]( https://developer.android.com/tools/data-binding/guide.html). 

By providing Observable properties and collections, using very similar interfaces, we hope to make this transition to data binding interfaces seamless with minimal changes when official platform support is available.


### Permissions and security

Add the following permissions to the AndroidManifest of your Android Studio project:

```java
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission
        android:name="android.permission.WRITE_EXTERNAL_STORAGE"
        tools:node="replace" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_MULTICAST_STATE" />
    <uses-permission android:name="android.permission.AUTHENTICATE_ACCOUNTS" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.MANAGE_ACCOUNTS" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.CALL_PHONE" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.WRITE_CONTACTS" />
    <uses-permission android:name="android.permission.WRITE_SETTINGS" />
    <uses-permission android:name="android.permission.READ_SYNC_STATS" />
    <uses-permission android:name="android.permission.READ_SYNC_SETTINGS" />
    <uses-permission android:name="android.permission.WRITE_SYNC_SETTINGS" />
    <uses-permission android:name="android.permission.BROADCAST_STICKY" />
    <uses-permission android:name="android.permission.READ_LOGS" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />

```
