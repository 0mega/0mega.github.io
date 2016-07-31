---
layout: post
title: Using Firebase alongside Google Analytics on Android
date: 2016-06-25 10:00 UTC
author: Oleksandr Kruk
tags: android, firebase, google analytics, google tag manager
---

During the last I/O, Google announced their new super product, the Firebase Analytics (FA). FA, compared to Google Analytics (GA) is a huge improvement for cross platform mobile analytics, crash reports, marketing campaigns coordination, mobile notifications, among other useful features.

Given the amount of new useful goodies, it's only logical to consider moving to Firebase ASAP. The only problem lies in a small but very important detail, you can not migrate your analytics data from GA to FA :(.

A reasonable approach [appointed by Google](https://developers.google.com/tag-manager/ios/v5/), is to setup FA with Google Tag Manager (GTM) for your app and use GTM to "replay" the events to the GA. This will allow you to take advantage of the new FA features and at the same time keep the GA features.

### What is Google Tag Manager?

Google Tag Manager can be thought of as an event listener. Very shortly, it receives all the events from your application and redirects them to other systems, but it also acts like a variables data store. This means at least two things:

0. It acts like a proxy to the analytics systems, allowing the user to process the events in the most appropriate way

0. It allows to change application runtime using these variables without having to redeploy the application

The important point for this post is 1., given that we are interested in "intercepting" the FA events and resending them to GA.

### Firebase and GTM setup

Adding the Firebase and GTM libs to your application is quite simple process. It is well documented in the link referenced above so you can follow the instructions to set it up in the application.

#### Configuring Screen View

Using Google Analytics you probably had something similar to the following to track page views:

{% highlight java %}

GoogleAnalytics analytics = GoogleAnalytics.getInstance(this);
tracker = analytics.newTracker(R.xml.global_tracker);
tracker.setScreenName(MyActivity.class.getSimpleName());
tracker.send(new HitBuilders.ScreenViewBuilder().build());

{% endhighlight %}

To do the equivalent in Firebase Analytics you would write something like:

{% highlight java %}
mFirebaseAnalytics = FirebaseAnalytics.getInstance(this);
Bundle screenView = new Bundle();
screenView.putString(
  FirebaseAnalytics.Param.ITEM_NAME,
  MyActivity.class.getSimpleName()
);
mFirebaseAnalytics.logEvent(
  FirebaseAnalytics.Event.VIEW_ITEM,
  screenView
);
{% endhighlight %}

Now what we need to do to make this screen view available in GA is to create a trigger in GTM which extracts the data logged by FA and send that same data to GA.
We start by creating a new Tag and choosing Googla Analytics as the product.

![tag for GA]({{ site.url }}/images/tag_product.png)

The next step is to create a new variable of type Constant holding the Google Analytics Tracking ID, let's call it `GA Tracking ID`

![var for tracking id]({{ site.url }}/images/tracking_id_var.png)

Now we select `Screen View` for `Track Type` and expand `More settings` and `Fields to Set`. This field will hold the screen name for our screen view event. For the `Field Name` we select `screenName` from the dropdown and for the `Value` we will create a new variable of type `Event Parameter`. Lets call the variable `Item Name` and select `view_item` for the `Suggested Event` and `item_name` as `Event Parameter`. We can also set a default value in case our event doesn't bring data in `item_name` parameter (I just set it to "not set" as usually seen in GA).

![item name var]({{ site.url }}/images/item_name_var.png)

Having those in place, we can create a trigger, which will be the mechanism that intercepts the type of event we are interested in. For that, in the `Fire On` section, we select `Custom`

![screen view trigger]({{ site.url }}/images/screen_view_trigger.png)

After the trigger is created, the only thing left is to confirm the tag creation.

![tag creation complete](/images/tag_complete.png)

And we have a complete integration for Screen View events in FA and GA using TGM as a middleman :).

#### Configuring Custom Event

Similarly to screen views, to track user actions with Google Analytics, you would have something like:

{% highlight java %}

tracker.send(new HitBuilders.EventBuilder()
  .setCategory("my_category")
  .setAction("my_action")
  .build()
);

{% endhighlight %}

Firebase

{% highlight java %}

Bundle eventAction = new Bundle();
eventAction.putString(
  FirebaseAnalytics.Param.ITEM_CATEGORY,
  "my_category"
);
eventAction.putString(
  FirebaseAnalytics.Param.ITEM_ID,
  "my_action"
);
mFirebaseAnalytics.logEvent(
  FirebaseAnalytics.Event.SELECT_CONTENT,
  eventAction
);

{% endhighlight %}

#### Things to bear in mind

0. Google Analytics lib can be removed `com.google.android.gms:play-services-analytics`
0. You might need to update `Google Play Services` and `Google Repository` using `Android SDK Manager` in order to have access to FA and GTM libraries.
0. Make sure you publish the container after creating the tags before you download the container json file.
0. Make sure you choose the correct type of container in Google Tag Manager. There are two type of containers, `Legacy Android` and `Firebase Android`. The one we have to choose is `Firebase Android`, as shown in the next screenshoot.

![firebase container]({{ site.url }}/images/firebase_container.png)
