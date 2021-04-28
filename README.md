# Demo-iOS-NibLoading
An example of different ways to load NIB/XIB files in iOS.

There's (currently) two options displayed in here; there are more than just these in the world, certainly, but these two both exemplify the idea of loading a "raw" NIB/XIB file, using it purely as a "resource-oriented" way to construct a user interface at runtime.

## Goal
We want a simple app that allows us to add name/age pairs to the main screen, along with a "Happy Birthday" button that, when pressed, adds one to that person's age. The label/button pair is nested inside a UIStackView (horizontal) and stored in a XIB file for easy instantiation.

As the user presses the "Add Person" button at the top, these label/button pairs (inside the horizontal UIStackView) need to be added into the main content body (a vertical UIStackView) of the main screen.

## Option One: Everything in place
Here, we store the data in the hosting ViewController, as an array of (String, Int) tuples. (We could create a full-blown class out of this, but I wanted to mimic the idea that in some rare cases, we wanted to keep the data in primitive types inside the hosting ViewController.) Thus, when the "Add Person" button is pressed, we:
* add the new name/age pair into the array
* instantiate the Nib
* find the label and button inside the Nib-instantiated view hierarchy
* set the label's text to "Name : Age"
* tag the button with the index into the array that this label/button is paired to
* set the button's .touchUpInside target to be the function inside the hosting ViewController
* add the Nib-instantiated view hierarchy into the UIStackView (vertical) that is the "content body" of the hosting ViewController.

Then, when the Nib-instantiated button is pressed, the associated method:
* extracts the button's "tag" value so that it can use that as an index back into the array of data
* modifies the data in the array
* finds the paired label by going to the button's superview (parent), and getting the other control under there (which will be the label)
* updates the label's text

## Option Two: Raw NIB/XIB, wrapped in custom UIView subclass
In this approach, we want to store the data relevant to a given person closer to the actual UI controls that manipulate it. We keep the "raw" NIB/XIB file, but create a custom UIView subclass that "wrappers" the act of manipulating the NIB/XIB and hides it.

The heart of this is the `PersonPanelView` class, which as mentioned inherits from UIView, and holds its data as a public `data` property, which is again a (String, Int) tuple. I choose to do this as a tuple because I want the user of this class to modify the data "all at once", so that we can attach a `didSet` handler to the data to update the UI right after the data is modified; if these were two separate properties, then we'd have to write two separate `didSet` handlers, or force the user of this class to go through a public method to accomplish the same "atomic" update of both data elements at once.

The two initializers are necessary whenever you create a subclass of UIView. No getting around that. They both call to a private "helper" initializer to do the NIB/XIB work.

In that initializer (`initSubViews`) we load the NIB/XIB and instantiate it (using the `UINib` class and its `instantiate` method this time), and again get hold of the top-level UIView instantiated from the NIB/XIB (which, again, will be the horizontal UIStackView) so that we can add it to this (`self`'s) view tree (via `addSubview`), and then wire up the label and button accordingly.

Because the data is held in this object, the button-handler method is proportionally much simpler: grab our data, increment the age portion of it, update the label text.

Note that we're still wiring up the button's target at runtime, rather than doing any configuration in Interface Builder; this is to keep the example relatively similar to what's happening in Option One.
