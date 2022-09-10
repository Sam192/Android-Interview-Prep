* **What is `Intent`?** - [Learn from here](https://blog.mindorks.com/what-are-intents-in-android)
  - An Intent is a messaging object which tells what kind of action to be performed..
  - Intents are used to move from one activity to another , it is the wiring b/w components.
  - we can start an Activity, Services and Deliver Broadcast using intents.

* **What is an Implicit `Intent`?** - [Learn from here](https://blog.mindorks.com/what-are-intents-in-android)
  - We just declared it and platform will find activity that can respond to it.
  - Here, you don’t need to specify the fully-qualified address. 
  - All you need to do is just specify the action that is to be performed by an Intent. 
  - By using the Implicit Intents you can communicate between various applications present in the mobile device
        
* **What is an Explicit `Intent`?** - [Learn from here](https://blog.mindorks.com/what-are-intents-in-android)
   - Explicit Intents are used to communicate with a particular component of the same application. 
   - For example, if you want to launch an Activity by clicking some button on the present Activity then you can specify the fully-qualified address of the desired Activity to launch that Activity

* **What is the function of an `IntentFilter`?** - [Learn from here](https://stackoverflow.com/questions/3321514/what-are-intent-filters-in-android#:~:text=100-,An%20intent%20filter%20is%20an%20expression%20in%20an%20app%27s%20manifest%20file%20that%20specifies%20the%20type%20of%20intents%20that%20the%20component%20would%20like%20to%20receive.,-When%20you%20create)
  - Implicit intent uses the intent filter to serve the user request.
  - The intent filter specifies the types of intents that an activity, service, or broadcast receiver can respond.
  - Intent filters are declared in the Android manifest file.
  - For instance, by declaring an intent filter for an activity, you make it possible for other apps to directly start your activity with a certain kind of intent.
  - Intent filter must contain < action >

* **What is a Sticky `Intent`?**
    - Sticky Intents allows communication between a function and a service. sendStickyBroadcast() performs a sendBroadcast(Intent) known as sticky, i.e. the Intent you are sending stays around after the broadcast is complete, so that others can quickly retrieve that data through the return value of registerReceiver(BroadcastReceiver, IntentFilter). For example, if you take an intent for ACTION_BATTERY_CHANGED to get battery change events: When you call registerReceiver() for that action — even with a null BroadcastReceiver — you get the Intent that was last Broadcast for that action. Hence, you can use this to find the state of the battery without necessarily registering for all future state changes in the battery.

* **Describe how broadcasts and intents work to be able to pass messages around your app?** - [Learn from here](https://stackoverflow.com/questions/7276537/using-a-broadcast-intent-broadcast-receiver-to-send-messages-from-a-service-to-a)

* **What is a `PendingIntent`?**
    - If you want someone to perform any Intent operation at future point of time on behalf of you, then we will use Pending Intent.
