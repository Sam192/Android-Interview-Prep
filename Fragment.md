* **What is `Fragment` and its lifecycle.** - [Learn from here](https://blog.mindorks.com/android-fragments-and-its-lifecycle)
    - Represents a portion of the user interface in an Activity
    - A Fragment is a piece of an activity which enable more modular activity design. It will not be wrong if we say, a fragment is a kind of sub-activity.
        Following are important points about fragment −
    - A fragment has its own layout and its own behaviour with its own life cycle callbacks.
    - You can add or remove fragments in an activity while the activity is running.
    - You can combine multiple fragments in a single activity to build a multi-pane UI.
    - A fragment can be used in multiple activities.
    - Fragment life cycle is closely related to the life cycle of its host activity which means when the activity is paused, all the fragments available in the activity will also be stopped.
    - A fragment can implement a behaviour that has no user interface component.
    - Fragments were added to the Android API in Honeycomb version of Android which API version 11.<br>
    
    ![image](https://user-images.githubusercontent.com/67669163/178601480-50fd4413-e698-4970-a3f2-ed5106e9a004.png)
    ![image](https://user-images.githubusercontent.com/67669163/178601186-9d821aaa-d6be-4ec1-bb59-9972a2de6a60.png)


* **What is the difference between a `Fragment` and an `Activity`? Explain the relationship between the two.** - [Learn from here](https://stackoverflow.com/questions/10478233/why-fragments-and-when-to-use-fragments-instead-of-activities#:~:text=Activities%20are%20the,via%20the%20parent.)
    - Activities are the full screen components in the app with the toolbar, everything else are preferably Fragments.

* **When should you use a Fragment rather than an Activity?**
    - When you have some UI components to be used across various activities
    - When multiple view can be displayed side by side just like viewPager

* **What is the difference between FragmentPagerAdapter vs FragmentStatePagerAdapter?**
    - FragmentPagerAdapter: Each fragment visited by the user will be stored in the memory but the view will be destroyed. When the page is revisited, then the view will be created not the instance of the fragment.
    - FragmentStatePagerAdapter: Here, the fragment instance will be destroyed when it is not visible to the user, except the saved state of the fragment.

* **What is the difference between adding/replacing fragment in backstack?** - [Learn from here](https://stackoverflow.com/questions/24466302/basic-difference-between-add-and-replace-method-of-fragment/24466345#:~:text=The%20important%20difference%20is%3A)
    - replace removes the existing fragment and adds a new fragment..
    - but add retains the existing fragments and adds a new fragment that means existing fragment will be active and they wont be in 'paused' state hence when a back button is pressed onCreateView() is not called for the existing fragment(the fragment which was there before new fragment was added).

* **Why is it recommended to use only the default constructor to create a `Fragment`?** - [Learn from here](https://www.youtube.com/watch?v=9EdvcycKP9A)
    - whenever the Android framework decides to recreate our fragment for example in case of orientation changes Android calls the no argument constructor of our    fragment the reason behind is that Android framework has no idea what constructor we have created.
    
    - the recommended way for creating the new instance of the fragment here is said arguments method is very important we must notice that we are passing the bundle inside it so when we create the instance for the first time using the new instance method the android framework can extract the bundle and store it so when in case of the orientation changes the android framework recreates the new fragment using the no argument constructor and can attach the bundle to the fragment as it has stored the bundle earlier and later again we can access that data in our on create method by using gatearguments like this so it means that when the system restores a fragment it will automatically restore our bundle and we will be able to restore the state of the fragment to the same state the fragment was initialized

* **How would you communicate between two Fragments?** - [Learn from here](https://blog.mindorks.com/how-to-communicate-between-fragments#:~:text=ENROLL%20NOW-,Fragment%20Communication,-The%20communication%20between)
    - The communication between fragments should not be done directly. There are two ways of doing so. 
    - With the help of ViewModel
    - With the help of Interface
    - To have a sharing of data between Fragments, either you can use a shared ViewModel that is shared between all the Fragments or you can make an Interface and then use this interface to communicate between fragments.

* **What is retained `Fragment`?**
    - By default, Fragments are destroyed and recreated along with their parent Activity’s when a configuration change occurs. Calling setRetainInstance(true) allows us to bypass this destroy-and-recreate cycle, signaling the system to retain the current instance of the fragment when the activity is recreated.

* **What is the purpose of `addToBackStack()` while commiting fragment transaction?**
    - By calling addToBackStack(), the replace transaction is saved to the back stack so the user can reverse the transaction and bring back the previous fragment by pressing the Back button. For more [Learn from here](https://stackoverflow.com/questions/22984950/what-is-the-meaning-of-addtobackstack-with-null-parameter)
