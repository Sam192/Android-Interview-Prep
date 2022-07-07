# Android-Interview-Prep

## Contents

* [Core Android](#core-android)
* [Android Libraries](#android-libraries)
* [Android Architecture](#android-architecture)
* [Android Design Problem](#android-design-problem)


### Core Android
* **What is android?**
  -  Android is an S/W stack for Mobile devices, that includes an Operating System, middle-ware and key applications.
  -  By Using JNI (Java Native Interface) we can communicate with C/C++ libraries.
  -  The latest version of Android is Android 12 (API level 31) and the initial stable release date is October 4, 2021
![image](https://user-images.githubusercontent.com/67669163/177872071-57864e6d-5024-4187-9a00-9e9de741a4f9.png)


* **Why android?**
  -  ![image](https://user-images.githubusercontent.com/67669163/177873250-84377899-19c2-4e2b-b1e6-dfb5792bad19.png)
  
  
* ### **Tell all the Android application components.**
  -  ![image](https://user-images.githubusercontent.com/67669163/177873746-271e6f40-d367-4684-8cc9-e9b4db56dbc0.png)
  * **Activities**
     -  An activity represents a single screen with a user interface,in-short Activity performs actions on the screen. For example, an email application might have one activity that shows a list of new emails, another activity to compose an email, and another activity for reading emails. If an application has more than one activity, then one of them should be marked as the activity that is presented when the application is launched.
     -  An activity is implemented as a subclass of Activity class as follows 
       **public class MainActivity extends Activity { }**
  
  * **Services**
     - A service is a component that runs in the background to perform long-running operations. For example, a service might play music in the background while the user is in a different application, or it might fetch data over the network without blocking user interaction with an activity.
     - A service is implemented as a subclass of Service class as follows 
      **public class MyService extends Service { }**
      
  * **Broadcast Receivers**
      - Broadcast Receivers simply respond to broadcast messages from other applications or from the system. For example, applications can also initiate broadcasts to let other applications know that some data has been downloaded to the device and is available for them to use, so this is broadcast receiver who will intercept this communication and will initiate appropriate action.
      - A broadcast receiver is implemented as a subclass of BroadcastReceiver class and each message is broadcaster as an Intent object.
      - **public class MyReceiver  extends  BroadcastReceiver { 
        public void onReceive(context,intent){ }  }**
        
  * **Content Providers**
      -  A content provider component supplies data from one application to others on request. Such requests are handled by the methods of the ContentResolver class. The data may be stored in the file system, the database or somewhere else entirely.
      -  A content provider is implemented as a subclass of ContentProvider class and must implement a standard set of APIs that enable other applications to perform transactions.
      - **public class MyContentProvider extends  ContentProvider {  public void onCreate(){ }  }**
  * **Additional Components**
      -  There are additional components which will be used in the construction of above mentioned entities, their logic, and wiring between them. These components are-
      
          1	
          **Fragments**

          Represents a portion of user interface in an Activity.

          2	
          **Views**

          UI elements that are drawn on-screen including buttons, lists forms etc.

          3	
          **Layouts**

          View hierarchies that control screen format and appearance of the views.

          4	
          **Intents**

          Messages wiring components together.

          5	
          **Resources**

          External elements, such as strings, constants and drawable pictures.

          6	
          **Manifest**

          Configuration file for the application.
          

      


