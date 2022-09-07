![image](https://user-images.githubusercontent.com/67669163/178596244-4595114d-5c5a-4d2f-bd9b-7a46b8443e83.png)


* **What is the difference between onCreate() and onStart()** - <br>
![IMG_20220908_020558](https://user-images.githubusercontent.com/67669163/188973741-a817f6d4-e7db-4e29-9f8a-03927b4fcf6b.jpg)



* **When only onDestroy is called for an activity without onPause() and onStop()?** - [Learn from here](https://www.youtube.com/watch?v=QSxcLnZ1-RU)
    - onPause() and onStop() will not be invoked if finish() is called from within the onCreate() method. This might occur, 
    - for example, if you detect an error during onCreate() and call finish() as a result. In such a case, though, any cleanup you expected to be done in onPause() and onStop() will not be executed

* **Why do we need to call setContentView() in onCreate() of Activity class?** - [Learn from here](https://www.youtube.com/watch?v=zeYK8JdMOi8)
    - As onCreate() of an Activity is called only once, this is the point where most initialization should go: calling setContentView(int) to inflate the activity's UI, using findViewById to programmatically interact with widgets in the UI, calling managedQuery(android.net.Uri, String[], String, String[], String) to retrieve cursors for data being displayed, etc.

    - It is inefficient to set the content in onResume() or onStart() (which are called multiple times) as the setContentView() is a heavy operation.

* **What is onSavedInstanceState() and onRestoreInstanceState() in activity?**
    - onSavedInstanceState() - This method is used to store data before pausing the activity.
    - onRestoreInstanceState() - This method is used to recover the saved state of an activity when the activity is recreated after destruction. So, the onRestoreInstanceState() receive the bundle that contains the instance state information.
