---
layout: "post"
title: "RecyclerView with MVP"
date: "2017-07-21 13:00"
categories: [android]
tags: [android, recyclerview, mvp, presenter, view]
description: "Implementing RecyclerView in android, with MVP approach can be tricky and complicated. This is the Part 1 of RecyclerView with MVP implementation."
---
MVC/MVP/MVVM, we're all talking about it. It's modern, it's cool, but besides that it's useful. We'll see if the MVP is dead, now that [Google announced its own archtecture components](https://developer.android.com/topic/libraries/architecture/index.html), however, that's something to deal with later.
Currently MVP with Clean Architecture is doing a great job for us. And most importantly it is so easy to implement and understand.
Sometimes, implementation starts to be chaotic and/or confusing and RecyclerView is such example.

Usually, when implementing `RecyclerView` we've used to send list of some objects to its adapter, to create some ViewHolders and bind the data to it. If we're good citizens, we'll at least move `ViewHolder` logic to its own class, instead of referencing it inside `onBindViewHolder` method.

```kotlin

class TestAdapter(val context: Context) : RecyclerView.Adapter<TestViewHolder>(){

    private var testItemsList = mutableListOf<TestItem>()

    fun onDataChange(testItems: List<TestItem>){
      testItems.forEach {
        if(!testItemsList.contains(it)){
          testItemsList.add(it)
        }
      }

      notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): TestViewHolder =
      TestViewHolder(LayoutInflater.fromContext(context).inflate(R.layout.test_list_item, parent, false))

    override fun onBindViewHolder(holder: TestViewHolder?, position: Int) {
      var testItem = testItemsList[position]
      holder?.title.text = testItem.title
      holder?.description.text = testItem.description
    }

    override fun getItemCount = testItemsList.size()
}

```
and usage of this adapter in activity or fragment would look like this:

```kotlin
    override fun onDataLoaded(testItems: List<TestItem>){
      var testAdapter = TestAdapter(context)
      recyclerView.adapter = testAdapter
      testAdapter.onDataChange(testItems)
    }
```

It's simple, it really is. I haven't shown the `TestViewHolder` implementation, but it's really just initialization of TextView's. However,
if we wan't to write a unit test for adapter above, we'll be in trouble. There is a android `Context` in the constructor, we have
`LayoutInflater` when creating the ViewHolder and we have to mock all of it just to check if the list is loading.

We won't, so we'll write a `presenter` for it.
Adapter presenter should be:
- keeping the list of items
- providing an item per index/position
- providing the items count
- notifying adapter when the change occur.

```kotlin

class TestAdapterPresenter {

    var view: View? = null
    private var testItems = mutableListOf<TestItem>()

    fun onDataChange(testItems: List<TestItem>) {
        testItems.forEach {
          if(!testItemsList.contains(it)){
            testItemsList.add(it)
          }
        }
        view?.notifyAdapter()
    }

    fun getCount(): Int = testItems.size

    fun getItemAt(position: Int) = testItems[position]

    infix fun itemAtPosition(position: Int): TestItem = testItems[position]

    interface View {
        fun notifyAdapter()
    }
}

```

and we'll change the adapter according to these changes:

```kotlin

class TestAdapter(val context: Context) : RecyclerView.Adapter<TestViewHolder>(), TestAdapterPresenter.View{

    var testAdapterPresenter = TestAdapterPresenter()

    init {
        testAdapterPresenter.view = this
    }

    override fun notifyAdapter() {
        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): TestViewHolder =
      TestViewHolder(LayoutInflater.fromContext(context).inflate(R.layout.test_list_item, parent, false))

    override fun onBindViewHolder(holder: TestViewHolder?, position: Int) {
      var testItem = testItemsList[position]
      holder?.title.text = testItem.title
      holder?.description.text = testItem.description
    }

    override fun getItemsCount() = testAdapterPresenter.getCount()
}

```

In this case, adapter initialization is mostly unchanged. Except, we won't be passing the list items to it, but to its presenter.
```kotlin
    override fun onDataLoaded(testItems: List<TestItem>){
      var testAdapter = TestAdapter(context)
      recyclerView.adapter = testAdapter
      testAdapter.testAdapterPresenter.onDataChange(testItems)
    }
```

We have created a `View` interface which `TestAdapter` implemented. `TestAdapterPresenter` does not know to whom is he talking to,
when calling `notifyAdapter()`, which is very important as we can mock it easily and test its behavior when list is received and data passed from it.

So we improved the testability of the app. But it can be better, right? If we want to test the behavior of the `TestViewHolder`,
our hands are tight and we have to do some changes in order to help ourself. We'll be still following the MVP pattern and create a presenter for the `TestViewHolder`. Also, we'll move the `title` and `description` related calls to ViewHolder as well.

```kotlin

class TestViewHolderPresenter {

  var testAdapterPresenter: TestAdapterPresenter? = null
  var position: Int? = null
  var view: View? = null

  fun bind(){
    if(position != null && testAdapterPresenter != null){
      var testItem = testAdapterPresenter.getItemAt(position)
      view?.setTitle(testItem.title)
      view?.setDescription(testItem.description)
    }
  }

  interface View {
    fun setTitle(title: String)
    fun setDescription(description: String)
  }
}
```

As each ViewHolder is bound to some position in the list, it is perfectly natural to set that `position` in the ViewHolder presenter.
Also, in order to enable the communication between the Adapter presenter and ViewHolder presenters, each ViewHolder presenter needs to have the reference to Adapter presenter as shown above with `testAdapterPresenter` variable.

Below, `TestViewHolder` implements `TestViewHolderPresenter.View` and it's functions to set the `title` and `description` values.

```kotlin

class TestViewHolder(val view: View) : RecyclerView.ViewHolder(view), TestViewHolderPresenter.View {

    var testViewHolderPresenter = TestViewHolderPresenter()

    @BindView(R.id.test_list_item_tv_title) lateinit var tvTitle: TextView
    @BindView(R.id.test_list_item_tv_description) lateinit var tvDescription: TextView

    init {
      ButterKnife.bind(this, view)
      testViewHolderPresenter.view = this
    }

    override fun setTitle(title: String){
      tvTitle.text = title
    }

    override fun setDescription(description: String){
      tvDesription.text = description
    }
}
```

It is obvious that `TestViewHolder` really does not know anything about the data, or its position in the adapter. Only thing ViewHolder worries about
is to set received values (`title`, `description`) to its components. Whole logic is moved to its presenter and since we have moved ViewHolder logic to its class and Adapter implementation changes again.

```kotlin

class TestAdapter(val context: Context) : RecyclerView.Adapter<TestViewHolder>(), TestAdapterPresenter.View{

    var testAdapterPresenter = TestAdapterPresenter()

    init {
        testAdapterPresenter.view = this
    }

    override fun notifyAdapter() {
        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): TestViewHolder =
      TestViewHolder(LayoutInflater.fromContext(context).inflate(R.layout.test_list_item, parent, false))

    override fun onBindViewHolder(holder: TestViewHolder?, position: Int) {
      holder.testViewHolderPresenter.position = position
      holder.testViewHolderPresenter.testAdapterPresenter = testAdapterPresenter
      holder.testViewHolderPresenter.bind()
    }

    override fun getItemsCount() = testAdapterPresenter.getCount()
}

```

And after these changes, `TestAdapter` also just worries about initialization of its presenter, ViewHolder and passing the current `position` to it.

To make it easier to understand, here is a simple sequence diagram that shows how these presenters fits into the app.
![Sequence Diagram]({{ site.url }}/assets/img/posts/2017/2017-07-21-recyclerview_mvp.svg)

Problem we haven't described above (nor implemented) is callback implementation from ViewHolder through the presenter all the way to Fragment or Activity. Kaushik Gopal mentioned this problem when talking about the state of EventBus today, in [Eposide 61 of Fragmented Podcast](http://fragmentedpodcast.com/episodes/061/). So he decided that EventBus could find its usage here. However, in Part 2 of RecyclerView with MVP, well be addressing this implementation, without EventBus though.
