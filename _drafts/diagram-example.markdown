---
layout: "post"
title: "diagram example"
date: "2017-06-14 17:35"
---
sequenceDiagram
     participant Activity
     participant SimpleFragmentManager
     participant FragmentManager
     participant FragmentTagStack
     participant Fragment
     Activity->>SimpleFragmentManager: Initialize
     activate Activity
     Activity->>+Fragment: Creates new instance
     Fragment->>-Activity: Return IFragment instance
     Activity->>SimpleFragmentManager: Adds new IFragment
     deactivate Activity
     activate SimpleFragmentManager
     SimpleFragmentManager->>FragmentManager: Commits transaction
     SimpleFragmentManager->>FragmentTagStack: Pushes the tag
     deactivate SimpleFragmentManager
     Note over Activity,Fragment: User interaction
     Fragment->>Activity: (via FragmentChannel) Open some other fragment
     activate Activity
     Activity->>+Fragment: Create some other instance
     Fragment->>-Activity: Return some other IFragment instance
     Activity->>SimpleFragmentManager: Replace existing IFragment with newly created
     deactivate Activity
     activate SimpleFragmentManager
     SimpleFragmentManager->>FragmentManager: Commits transaction
     SimpleFragmentManager->>FragmentTagStack: Pushes the tag
     deactivate SimpleFragmentManager
     Note over Activity,Fragment: User interaction
     Fragment->>+Activity: User pressed the back button
     Activity->>-SimpleFragmentManager: Checking for the state of fragment stack
     alt is not last fragment
       activate SimpleFragmentManager
       SimpleFragmentManager->>FragmentManager: Pop Immediate
       SimpleFragmentManager->>FragmentTagStack: Pop up the tag
       SimpleFragmentManager->>+FragmentTagStack: Get tag of current fragment
       FragmentTagStack->>-SimpleFragmentManager: Returns tag of current fragment
       SimpleFragmentManager->>+FragmentManager: Find fragment by tag
       FragmentManager-->>-SimpleFragmentManager: Returns the fragment
       SimpleFragmentManager-->>+Fragment: Set the title
       Fragment-->>-Activity: (via FragmentChannel) Set the title
       Note over Activity: Setting the title.
       SimpleFragmentManager->>Activity: TRUE
       deactivate SimpleFragmentManager
     else is last fragment
       SimpleFragmentManager->>Activity: FALSE
       Note over Activity: Finish
       Activity->>SimpleFragmentManager: Disposing on destroy
       activate Activity
       SimpleFragmentManager->>FragmentTagStack: Get tag of current fragment
       activate FragmentTagStack
       FragmentTagStack->>SimpleFragmentManager: Returns tag of current fragment
       deactivate FragmentTagStack
       SimpleFragmentManager->>FragmentManager: Find fragment by tag
       activate FragmentManager
       FragmentManager-->>SimpleFragmentManager: Returns the fragment
       deactivate FragmentManager
       SimpleFragmentManager-->>Fragment: Dispose
       activate Fragment
       Note over Fragment: Disposing all RX streams for example
       Fragment-->>SimpleFragmentManager: Dispose completed
       deactivate Fragment
       SimpleFragmentManager->>Activity: Everything disposed
       deactivate Activity
       Note over Activity: Finished
     end



     recycler view

sequenceDiagram
participant Fragment
participant TestAdapter
participant TestAdapterPresenter
participant TestViewHolder
participant TestViewHolderPresenter
Fragment->>TestAdapter: Initializes
TestAdapter->>TestAdapterPresenter: Initializes
activate TestAdapter
TestAdapter->>TestAdapterPresenter: Sets View
deactivate TestAd
Fragment->>TestAdapterPresenter: onDataChange(list)
TestAdapterPresenter->>TestAdapter: view.notifyAdapter()
TestAdapter->>TestAdapterPresenter: getItemCount()
TestAdapter->>TestViewHolder: Creates
TestViewHolder->>TestViewHolderPresenter: Initializes
activate TestViewHolder
TestViewHolder->>TestViewHolderPresenter: Sets View
deactivate TestViewHolder
TestAdapter->>TestViewHolder: Binds
activate TestAdapter
TestViewHolder->>TestAdapter: Get TestViewHolderPresenter instance
TestAdapter->>TestViewHolderPresenter: Sets TestAdapterPresenter instance
TestAdapter->>TestViewHolderPresenter: Sets current position
deactivate TestAdapter
TestAdapterPresenter->>TestViewHolderPresenter: getItemAt(position)
activate TestViewHolderPresenter
TestViewHolderPresenter->>TestViewHolder: setTitle(title)
TestViewHolderPresenter->>TestViewHolder: setDescription(description)
deactivate TestViewHolderPresenter
TestViewHolder->>TestViewHolder: Set values to TextViews
