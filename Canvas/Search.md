# The Canvas App Search box
Scraping up the the internet, forums, etc. there are loads of examples of how to connect a simple text box with a Gallery and turn it into what it looks like some basic search box. But what if your needs are more complex and maybe you need to let's say search through a combined collection of fields or maybe have your search feature take into account some filter functionality?
I've literally hit this bump into the road with every app I have built.

Let's face it, any app these days has some pretty well defined expectations for the search experience, and if haven't gone through loads of development scenarios, you would either create a non-scalable mess, or perhaps create something that takes the user back in 1995.

Here's what I think search should be in any app, from both the Dev and User perspective:

**User**
* Search should lookup all relevant fields
* Should take into account filters
* Should happen as the user starts typing in
* It should be obvious that the field is a Search Box

**Dev**
* It must be obvious how this works
* It must be easy to add more fields into the search soup
* It must have a single location where changes would be made
* It must be documented and well tested

So lets build a search experience that satisfies the above criteria.

## Pre-requisites
Here's what I've prepared for this session:

**SharePoint List**
* Create a SharePoint List with the following fields (*Single line of text*): `Title`, `Names`, `Description`
* Ad a couple of items with various text. Some items should have unique text, some should not.

**Canvas App**
* Create a new Canvas App from Blank (I've created a Tablet app)
* In your new App, in `Screen1`, create the the following:
    * Add a new Data Source. Make this the SharePoint List we have created earlier (mine is called `Test_Search`)
    * A blank gallery (you can leave its name the default `Gallery1`). Resize it to about 3/4 of your app width, and 3/4 your app height. Drag it to the left bottom corner.
    * An Icon that you can place somewhere in the far right upper corner (I used the Reload Icon). Name the Icon `ICN_Refresh`
    * A Text Input control. Name it `TXT_Search`. Place this somewhere in the far left upper corner
    * Another Icon that you resize to match the height of `TXT_Search` and you place it to the left of the control. Name it `ICN_Search`

We will come back to adding some labels into `Gallery1` but after we load the `Test_Search` data source into a Collection.
We could have just worked with the `Test_Search` data source directly, but if this would have been a real word app with loads of data, you would have had some Delegation issues as the data source filtering uses `in` to look at a combined dataset.

> Touching base on that, me personally, I never ever add the data source directly into a Gallery. This is incosiderate from an Architecture point of view, and it will get you in trouble soon regardless of which angle you view this from. Requirements often change, and most likely at some point you need to do some `Filter` here and there. Most operators will end in Delegation issues when used in `Filter`, so just to have evrything future proof and a breeze to work with, I always implement a strategy that allows me to load the data locally in Collections. I will post a full topic about "Working with Data inside Canvas Apps" at a later date though.

So, select the `ICN_Refresh` icon, go to the `OnSelect` property, and paste the below snippet in:

```
ClearCollect(COL_Search, Test_Search)
```

Now `Play` your app. And click on the Refresh Icon. This has now loaded the `Test_Search` data into a Collection called `COL_Search`.

Let do some tidy up as well. I often tend to be a perfectionist, so let's select the `TXT_Search` Search Box, go to the `Hint Text` property, and change it to "Type to search"

> Another thing that has drawn my attention on Icons is the fact that eventhough the icon does nothing, on hover it displays a hand. This gives the impression that the Icon is selectable and it should do something. I've fixed that by insterting a new Label control, remove the text, and place it on top of the icon. Not very elegant, but until we get more control on these things from Microsoft, well, its a fix.

Now let's sort out `Gallery1`:

**Data Source**

Select `Gallery1`, go to the `Items` property, and paste the below expression:
```
If(
    IsBlank(TXT_Search.Text),
    COL_Search,
    Filter(
        COL_Search,
        TXT_Search.Text in Concatenate(
            Title,
            " ",
            Names,
            " ",
            Description
        )
    )
)
```

This is what happens in the above expression:
* We start with an `If` statement and validate if the `TXT_Search` search box has any text
* If the statement evaluates as `true` then the user is not searching for anything, so we give him/her unfiltered information (that is data from the `COL_Search` Collection)
* If the statement evaluates as `false` then the user has typed something in the Search Box, so we need to see if what the user has typed in so far exists in any of the 3 fields. We do this applying a `Filter` expression on `COL_Search` where we check to see if the text typed by the user so far exists in any of the 3 fields. To combine the text values of the 3 fields, we use the `Concatenate` function and we make sure we add a space between each field (that is so words don't join together)

We need to be able to see this happening in `Gallery1` so let's add some labels there to display data:
* Add 3 labels and name them as follows: `LBL_Description_Label`, `LBL_Name_Label`, `LBL_Title_Label`. Change their Fill to Blue and Color to White
* Add 3 more labels and name them as follows: `LBL_Description_Value`, `LBL_Name_Value`, `LBL_Title_Value`

Set the `Text` property of the below labels as follows:
* `LBL_Description_Value` = `ThisItem.Description`
* `LBL_Name_Value` = `ThisItem.Names` (Please be carefull here. `ThisItem.Names` it is Names not Name. So easy to make a mistake really)
* `LBL_Title_Value` = `ThisItem.Title`

Change the Text of the following labels to the following static text values:
* `LBL_Description_Label` = "Description"
* `LBL_Name_Label` = "Name"
* `LBL_Title_Label` = "Title"

Stack the actual labels on top of the dynamic data labels so your app looks at least close to the below:

<img src="Test_Search_Image.jpg"></img>


Now try the Search Box. If you have followed the instructions correctly, the Gallery should filter results out as you type. The Search sould also look into all 3 fields. And guess what, because we have used a Collection, your App Checker is a happy chap with no errors or Delegation warnings.

But the real gain here is that you have a way to Refresh your data, a search box that is lightning fast, and a single location where you can maintain your search functionality (the `Items` proper of `Gallery1`). And not to mention the fact that you are not doing additional API calls when Delegating to the data source, and that the whole User Experience is really fast.

So in the future, if your SharePoint Data Source gets more columns, you can easily add these to the `Concatenate` function in the `Items` property in `Gallery1`. Just make sure you separate columns with a space.

Also, if you are then being asked to add a section with DropDown filters, you can again just add these to the `Concatenate` function. But in this scenario you might need to consider that additional filters would have to work regardless of whether the Search Box has text or not. So let's see how we do that.
