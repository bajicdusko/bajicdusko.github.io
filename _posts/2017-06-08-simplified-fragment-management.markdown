---
layout: "post"
title: "Simplified Fragment Management"
date: "2017-06-08 10:03"
categories: [android]
tags: [android, fragment, fragmentmanager]
twitter_excerpt: "EasyFragmentManager is basically a wrapper around FragmentManager and exposes 'add', 'replace', 'popUp', 'onBackPressed' and 'getCurrentFragment' methods. EasyFragmentManager assumes that..."
---
Handling fragments(lifecycle) in android app was complicated from the beginning.
Nowadays, a lot of internal bugs is fixed, but still there is a general negativity
towards fragment usage.

Many developers are avoiding fragments and returning to old fashion apps with the large
number of the activities. Just because it's "complicated", it
does not mean that we have to stop using it. Fragments are (at the moment) part of
our apps and we have to deal with it the best we can.

These issues might feel a little bit basic, however, by working on many android apps, I've noticed few constant problem-patterns (listed-below) when it comes to fragments. If you'd just like to jump to implementation, [click here](#solution).


1. __Getting instance of Fragment__ (for whatever reason) from the existing stack of transactions, regardless of the position in the stack.

   One way of getting the last added fragment might be this one:

   ```java
   //fetching all fragments
   List<Fragment> fragments = fragmentManager.getFragments();

   //getting last added fragment
   Fragment currentFragment = fragments.get(fragments.size() - 1);
   ```

   Although this approach makes sense, it is possible that some fragments are removed in the meantime and this list will contain `null` (instead of trimmed size).

   If, for example, we have `three` fragments on stack and we just tapped back button, we might get `fragment.size() == 3` but
   actual number of fragments is `two` and third position `fragments.get(fragments.size() - 1)` will return `null`.

   And I cannot point the obvious more than it is, but __this is the stack__ and still it can happen with any other index in the list. Since we have a `List<>` interface exposed, then we can call `fragments.get(0)` and wow, it returns `null`, `fragments.size()` value will be incorrect and no matter what index we use, we won't get correct fragment instance.

   Then we might say, ok we can use this approach instead:

   ```java
   //fetching all fragments
   List<Fragment> fragments = fragmentManager.getFragments();

   //getting the actual number of fragments from the fragmentManager
   int fragmentsCount = fragmentManager.getBackStackEntryCount();

   //getting last added fragment
   Fragment currentFragment = fragments.get(fragmentsCount - 1);
   ```

   And great, now we have the exact number of fragments on the stack, however, `fragments` list
   still can contain `null` values on any position and we'll end up with wrong fragment or with `java.lang.IndexOutOfBoundsException` in the worst case. Additional work is needed anyway and it seams its a lot
   of work just to get instance of fragment we have created in the first place.

2. __Handling onBackPressed navigation and application title update__

   Typical android application shows `Toolbar` on top of the screen. If the `Toolbar` is part of the `Activity` layout
   we need to update its `title` according to currently active fragment.

   For example:

   `HomeActivity.java`

   ```java
   @Inject FragmentManager supportFragmentManager;
   @BindView(R.id.toolbar) Toolbar toolbar;

   public void onCreate(Bundle savedInstanceState){
     super.onCreate(savedInstanceState);
     ...
     setSupportActionBar(toolbar);
     supportFragmentManager.beginTransaction()
                .add(R.id.activity_home_fragment_container, HomeFragment.newInstance())
                .addToBackStack(null)
                .commit();
   }

   public void setTitle(@StringRes int titleId){
     getSupportActionBar().setTitle(getString(titleId));
   }
   ```

   `HomeFragment.java`

   ```java
   @Override
   public void onStart() {
      super.onStart();
      ((HomeActivity)getActivity()).setTitle(R.string.home);
   }
   ```

   And this actually works. Except you might get `NullPointerException` sometimes or `java.lang.IllegalStateException: Fragment LoginFragment not attached to Activity` and honestly, I've stopped looking for reasons why. If you move `getActivity().setTitle(R.string.login);` to some other location in the fragment, `setTitle` might get called too late and we're triggering bad user experience.

   On the other hand, when you're popping the fragment from the stack, you have to call `setTitle()` method of the `Fragment` which is popping up and at some point `setTitle()` method might not get called at all, and you end up with `title` in the `Toolbar` that does not match the content.

   It does not have to be the title, you might want to change navigation options for different fragments, to show back arrow instead of hamburger icon etc, and setting these options will depend of some method call inside the `Fragment`.

3. __Reseting stack__ triggers lifecycle of each fragment along the way

   At some point, we have to return to the starting point of the app. So we'll be popping the fragments from the stack
   until stack gets empty or until there is only one fragment shown. Not so obvious problem that occurs is that
   each popped fragment gets re-instantiated with call to all lifecycle methods, and causing execution of logic we put in
   each fragment. This is definitely unwanted behavior. We just want to remove them so that we can start from the beginning.

4. __Communication__ between fragments and the hosting Activity should not be implemented by casting `getActivity()` method and I like to think that no one implements it like that. Instead, communication between activity and the fragments should be implemented via interface which `Activity` implements, I know, obvious.

<a name="solution">Simple solution</a> I came up with have four main components:

- `EasyFragmentManager` which is basically a wrapper around `FragmentManager` and exposes the methods for `add`, `replace`, `popUp`, `popUpAll` and `getCurrentFragment` operations. `EasyFragmentManager` assumes that each `Fragment` implements `IFragment` interface.

- `IFragment` interface is forcing each `Fragment` to have its own unique `name` or `tag` under which is added to transaction, `dispose()` method (which is very handy when implementing `RxJava2`), and yes, method to `setTitle()` of the `Toolbar/ActionBar`.

- `FragmentTagStack` to handle stack easily and retrieve fragments. It does not contain fragment instances, but its `tags` as `String` values instead and have only basic methods for `pop`, `popUpAll`, `peek`, `push` and `getActiveTag` which returns tag of currently shown `Fragment` on the screen.

- `FragmentChannel` is the most basic interface for callback implementation between the `Fragment` and hosting environment, which can be `Activity` or parent `Fragment`.

And here is the sequence diagram that simplifies the explanation, that shows basic interaction with adding, replacing and removing fragments.

![Sequence Diagram]({{ site.url }}/images/posts/2017/2017-06-08-fragmentdiagram.svg)

It is obvious that implementation is really simple. These are the preconditions you must fulfill.

1. Activity initialize `EasyFragmentManager`.
  - Override `onSaveInstanceState` and `onRestoreInstanceState` methods and forward the state to `EasyFragmentManager`.
  - Override `onBackPressed` method forward the call to `EasyFragmentManager`.
  - Implement `FragmentChannel`.
2. Each `Fragment` implements `IFragment` interface. _Tip: Usually, having `abstract BaseFragment` class which implements `IFragment` and initializes `FragmentChannel` is less error prone, then implementing these methods for all Fragments individually._

Example implementation can be found in this __[Gist](https://gist.github.com/bajicdusko/683766ab74b93519be27df0ae6e0793f)__. as well as implementations of described components below.
Once you establish hierarchy as below, you won't have to think about it much. Only component that you'll update constantly is `FragmentChannel` interface since you'll be implementing all `Fragment` -> `Activity` calls through it. `showLogin()` function in example below is one such case.

`EasyFragmentManager.kt`

```kotlin
class EasyFragmentManager(private val fragmentManager: FragmentManager, private val fragmentContainerId: Int) {

    private val KEY_TAGS = "key_tags"
    private var fragmentTagStack: FragmentTagStack = FragmentTagStack()

    constructor(fragmentManager: FragmentManager, fragmentContainer: FrameLayout) : this(fragmentManager, fragmentContainer.id)

    fun addFragment(fragment: IFragment) {
        fragmentTagStack.push(fragment.getFragmentName())
        fragmentManager.beginTransaction()
                .add(fragmentContainerId, fragment as Fragment, fragment.getFragmentName())
                .addToBackStack(fragment.getFragmentName())
                .commit()
    }

    fun replaceFragment(fragment: IFragment) {
        fragmentTagStack.push(fragment.getFragmentName())
        fragmentManager.beginTransaction()
                .replace(fragmentContainerId, fragment as Fragment, fragment.getFragmentName())
                .addToBackStack(fragment.getFragmentName())
                .commit()
    }

    /**
     * When [Activity.onBackPressed] is called it is a good practice to override it and to call this method.
     * This method won't allow removal of last fragment on the stack.
     *
     * @return TRUE if fragment is poppedUp aka method is consumed, or FALSE if there is only one Fragment
     * on the stack.
     */
    fun onBackPressed(): Boolean {
        if (fragmentManager.backStackEntryCount > 1) {
            popUp()
            var currentFragment = getCurrentFragment()
            currentFragment?.setTitle()
            return true
        } else {
            return false
        }
    }

    fun popUp() {
        fragmentManager.popBackStackImmediate()
        fragmentTagStack.pop()
    }

    fun popUpAll() {
        fragmentManager.popBackStack(null, FragmentManager.POP_BACK_STACK_INCLUSIVE)
        fragmentTagStack.popUpAll()
    }

    fun getCurrentFragment(): IFragment? = fragmentManager.findFragmentByTag(fragmentTagStack.activeTag) as IFragment?

    fun dispose() = getCurrentFragment()?.dispose()

    fun onSaveInstanceState(state: Bundle?) {
        state?.putParcelable(KEY_TAGS, Parcels.wrap(fragmentTagStack))
    }

    fun onRestoreInstanceState(savedInstanceState: Bundle?) {
        if (savedInstanceState != null) {
            fragmentTagStack = Parcels.unwrap(savedInstanceState.getParcelable(KEY_TAGS))
        } else {
            fragmentTagStack = FragmentTagStack()
        }
    }
}
```

`FragmentTagStack.kt`

```kotlin
@Parcel(Parcel.Serialization.BEAN)
class FragmentTagStack {

    @Transient private var showLogs: Boolean = false

    var tags: MutableList<String>
        private set

    var activeTag: String? = null
        private set

    constructor(){
        tags = LinkedList()
    }

    @ParcelConstructor
    constructor(tags: MutableList<String>, activeTag: String) {
        this.tags = tags
        this.activeTag = activeTag
    }

    /**
     * Enabling or disabling fragment stack logs. Logs are disabled by default.

     * @param showLogs
     */
    fun setShowLogs(showLogs: Boolean) {
        this.showLogs = showLogs
    }

    /**
     * Pushing new tag to stack and setting is as [FragmentTagStack.activeTag]

     * @param tag
     */
    fun push(tag: String) {
        tags.add(tag)
        activeTag = tag
        logStack()
    }

    /**
     * Popping tag from the stack and setting the tag below as new [FragmentTagStack.activeTag]
     *
     *
     * Example:
     *
     *
     * Stack [3, 2, 1, 0] (3 is last added tag). [FragmentTagStack.activeTag] have value 3
     *
     *
     * Now we're popping the last added tag
     *
     *
     * Stack [2, 1, 0] [FragmentTagStack.activeTag] have value 2

     * @return Popped up value. From the example above, returned value will be 3. In case of empty stack, null will be returned.
     */
    fun pop(): String? {
        val tag = peek()
        if (!TextUtils.isEmpty(tag)) {
            tags.removeAt(tags.size - 1)
        }

        activeTag = peek()
        logStack()
        return tag
    }

    /**
     * Peeking to the top of the stack, without data modification.

     * @return Value on top of the stack. If stack is empty, returned value will be null.
     */
    fun peek(): String? {
        if (tags.size > 0) {
            return tags[tags.size - 1]
        } else {
            return null
        }
    }

    private fun logStack() {
        if (showLogs) {
            val stringBuilder = StringBuilder()
            stringBuilder.append("Active tag: $activeTag\nStack:\n")
            for (i in tags.size - 1 downTo 0) {
                stringBuilder.append("[ ${tags[i]}]\n")
            }
            Log.d(TAG, stringBuilder.toString())
        }
    }

    /**
     * Clears the stack.
     */
    fun popUpAll() {
        tags.clear()
    }

    companion object {

        @Transient private val TAG = "FragmentTagStack"
    }
}
```

`IFragment.kt`

```kotlin

interface IFragment {
    fun getFragmentName(): String

    fun setTitle(): Unit?

    fun dispose()
}
```

`FragmentChannel.kt`

```kotlin

interface FragmentChannel{
  fun setTitle(title: String): Unit?
  fun showLogin()
}
```

`BaseFragment.kt`

```kotlin

abstract class BaseFragment : Fragment(), IFragment{

  var fragmentChannel: FragmentChannel? = null

  override fun onAttach(context: Context){
    super.onAttach(context)
    if(context is FragmentChannel){
      fragmentChannel = context
    }
  }

  override fun onCreate(savedInstanceState: Bundle?){
    super.onCreate(savedInstanceState)
    if(parentFragment != null && parentFragment is FragmentChannel){
      fragmentChannel = parentFragment
    }
  }
}
```

`HomeFragment.kt`

```kotlin

class HomeFragment : BaseFragment() {

  private val FRAGMENT_NAME = "Home"

  companion object{
    fun newInstance() = HomeFragment()
  }

  override fun getFragmentName(): String = FRAGMENT_NAME

  override fun setTitle() = fragmentChannel?.setTitle(R.string.home)

  fun onLoginButtonClick(){
    fragmentChannel?.showLogin()
  }

  override fun dispose(){
    TODO("Call presenter dispose method")
  }
}
```

`HomeActivity.kt`

```kotlin
class HomeActivity : AppCompatActivity(), FragmentChannel{

  @BindView(R.layout.activity_home_container)
  lateinit var flContainer: FrameLayout

  val easyFragmentManager by lazy {
    EasyFragmentManager(getSupportFragmentManager(), flContainer)
  }

  override fun onCreate(savedInstanceState: Bundle?){
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_home)
    ButterKnife.bind(this)
    easyFragmentManager.add(HomeFragment.newInstance())
  }

  override fun onSaveInstanceState(outState: Bundle?){
    super.onSaveInstanceState(outState)
    easyFragmentManager.onSaveInstanceState(outState)
  }

  override fun onRestoreInstanceState(savedInstanceState: Bundle?){
    super.onRestoreInstanceState(savedInstanceState)
    easyFragmentManager.onRestoreInstanceState(savedInstanceState)
  }

  override fun setTitle(titleId: Int) {
    supportActionBar?.title = getString(titleId)
  }

  override fun showLogin(){
    easyFragmentManager.replace(LoginFragment.newInstance())
  }

  override fun onBackPressed(){
    if(!easyFragmentManager.onBackPressed){
      finish();
    }
  }
}
```

And last but not least on latest Google IO (2017), guys from Google found it necessary to remind everyone about the basics as well.

<iframe width="560" height="315" src="https://www.youtube.com/embed/eUG3VWnXFtg" frameborder="0" allowfullscreen></iframe>
