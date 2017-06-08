---
layout: "post"
title: "Simplified Fragment Management"
date: "2017-06-08 10:03"
categories: [android]
tags: [android, fragment, fragmentmanager]
---
Handling fragments(lifecycle) in android app was complicated from the beginning.
Nowadays, a lot of internal bugs is fixed, but still there is a general negativity
towards fragment usage.

Many developers are avoiding fragments and returning to old fashion apps with the large
number of the activities. Just because it's "complicated", it
does not mean that we have to stop using it. Fragments are (at the moment) part of
our apps and we have to deal with it the best we can.

Ok then, we have to help ourself to handle the fragments as easier as possible.
By working on many apps, I've noticed few constant problem-patterns when it comes to fragments:

1. Getting instance of Fragment (for whatever reason) from the existing stack of transactions, regardless of the position in the stack.

   One way of getting the last added fragment might be this one:

   ```java
   //fetching all fragments
   List<Fragment> fragments = fragmentManager.getFragments();

   //getting last added fragment
   Fragment currentFragment = fragments.get(fragments.size() - 1);
   ```

   Although this approach makes sense, it is possible that some fragments are removed in the meantime and this list will contain `null` (instead of trimmed size).

   If for example we have `three` fragments on stack and we just tapped back button, we might get `fragment.size() == 3` but
   actual number of fragments is `two` and `fragments.get(fragments.size() - 1)` will return `null`.

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

2. Handling onBackPressed navigation and application title update

   Typical android application shows `Toolbar` on top of the screen. If the `Toolbar` is part of the `Activity` layout
   we need to update its `title` according to currently active fragment.

   For example:

   `HomeActivity.java`

   ```java
   @Inject FragmentManager supportFragmentManager;
   @BindView(R.id.toolbar) Toolbar toolbar;

   public void onCreate(Bundle savedInstanceState){
     super.onCreate(savedInstanceState);

     setSupportActionBar(toolbar);
     supportFragmentManager.beginTransaction()
                .add(R.id.activity_home_fragment_container, LoginFragment.newInstance())
                .addToBackStack(null)
                .commit();
   }

   public void setTitle(@StringRes int titleId){
     getSupportActionBar().setTitle(getString(titleId));
   }
   ```

   `LoginFragment.java`

   ```java
   @Override
    public void onStart() {
        super.onStart();
        ((HomeActivity)getActivity()).setTitle(R.string.login);
    }
   ```

   And this actually works. Except you might get `NullPointerException` sometimes or `java.lang.IllegalStateException: Fragment LoginFragment not attached to Activity` and honestly I've stopped looking for reasons why. If you move `getActivity().setTitle(R.string.login);` to some other location in the fragment, `setTitle` might get called too late and we're triggering bad user experience.

   On the other hand, when you're popping the fragment from the stack, `setTitle()` might not get called at all, and you end up with `title` in the `Toolbar` that does not match the content.

3. Reseting stack triggers lifecycle of each fragment along the way

   At some point, we have to return to the starting point of the app. So we'll be popping the fragments from the stack
   until stack gets empty or until there is only one fragment shown. Not so obvious problem that occurs is that
   each popped fragment gets re-instantiated with call to all lifecycle methods, and causing execution of logic we put in
   each fragment. This is definitely unwanted behavior. We just want to remove them so that we can start from the beginning.

4. Communication between fragments and the hosting Activity



<script src="https://gist.github.com/bajicdusko/683766ab74b93519be27df0ae6e0793f.js"></script>
