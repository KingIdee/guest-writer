---
layout: post
title: "Creating Infinite Lists in Android"
description: "Learn how to develop endless scrolling lists in your Android apps using the Paging library"
date: "2019-07-26 08:30"
author:
  name: "Idorenyin Obong"
  url: "kingidee"
  mail: "idee4ril@gmail.com"
  avatar: "https://twitter.com/kingidee/profile_image?size=original"
related:
- 2017-11-15-an-example-of-all-possible-elements
---

**TL;DR:** In the third part of these series, you will learn how to create infinite lists with the Android Paging Library. You can find the final code developed throughout the article in this GitHub repository.


## Prerequisites

Since this is the third part of a series, it is recommended that you have completed the first two parts of the series, however, if you have not done so, you can still jump on this. It is still expected that you possess some prior understanding of Android development. The prerequisites from the [first part](https://auth0.com/blog/android-tutorial-building-and-securing-your-first-app-part-1/#Prerequisites) still stand also.

## Introduction

Infinite scrolling is a scenario where data is continuously loaded as a user scrolls down a page or list. Since the size is not definite, these lists are often referred to as infinite lists. They are used when you have a large data set to present to your users.

Infinite lists are becoming increasingly popular among apps because scrolling is easier and more engaging for a user. As much as this looks like an almighty reason to always use it, it comes with some cons, like performance consequences if not properly implemented. Also, users can easily get lost while scrolling, among other reasons.

In the article, you will learn how to implement infinite lists in your android applications. You will be building on top of the project from the second part of the series which can be found [here](https://github.com/auth0-blog/to-do-android-app-part-2).

## Paging library introduction 

Before now, implementing infinite lists in Android required adding a scroll listener added to the `RecyclerView` to monitor the scroll position and distance to the end of the list. This approach was not so effective as you can see from a response to a [StackOverflow question here](https://stackoverflow.com/questions/47718270/why-do-i-need-to-use-the-new-paging-library-android-architecture-components):

“*You need to detect that the user has scrolled close enough to the end of the list to need to fetch data. You need to fetch that data. You need a `RecyclerView.Adapter` that can deal with incremental additions to that data. You need some sort of LRU-style caching rules to get rid of older data (that the user has scrolled past) to limit overall memory consumption. You need to handle the scenario where the user scrolls past your current data before additional data gets loaded. And so on.*”

And so the Android team at Google built the Paging library. The Paging library is majorly made up of three components:

### PagedList
This is the component that loads data in chunks or loads data in pages. It makes sure that data is loaded asynchronously in the proper thread and the updates are delivered seamlessly. You can configure this component with custom initial load size, page size, prefetch distance, etc.

### PageListAdapter
This is the base adapter you need to extend when building an adapter for your list. This component works hand in hand with the PagedList.

### DataSource
This is an interface you need to implement when using this library. It is used to determine how data is fetched. When implementing a DataSource, it is advisable to make use of one of the following subclasses made available:

- [PageKeyedDataSource](https://developer.android.com/reference/androidx/paging/PageKeyedDataSource.html): This is used where the keys for the previous and next page are returned. This is a popular use-case because most backends are built to deliver keys for large data sets. 


- [ItemKeyedDataSource](https://developer.android.com/reference/androidx/paging/ItemKeyedDataSource.html): This is used if you want to use data from item `N-1` (previous item) to load the `N` (current item).


- [PositionalDataSource](https://developer.android.com/reference/androidx/paging/PositionalDataSource.html): This is used to load data within a specific position, say from item 10 to item 50. Using this class requires that you know the size of your data. Because of this, the class is usually used you are fetching from the local storage.

## Paging library in action

In this tutorial, you will build an infinite list for your Android app using the Paging library. The app will fetch data from a server in little chunks. While implementing this, you will equally learn how to handle errors and retry network requests for your infinite lists.

### Adding dependencies
For this part, you will need two additional libraries namely; the RecyclerView library to manage the lists, and the Paging library to handle infinite scrolling.

To install these libraries, open the `./app/build.gradle` in your project, and update the `dependencies` section as follows:

```groovy
// ./app/build.gradle
dependencies {
    // ... leave the rest untouched and add ...
    implementation 'androidx.recyclerview:recyclerview:1.0.0'
    implementation "androidx.paging:paging-runtime:2.1.0"
}
```

You might already be wondering why you are switching from a `ListView` to a `RecyclerView`. Well, the Paging library is built to work with a `RecyclerView` instead of a `ListView`. The `RecyclerView` is generally preferred because it is an upgrade on the `ListView` which comes with more optimizations.

Still in the `./app/build.gradle` file, enable Java 8 for the project so that you can take advantage of lambda expressions. Lambda expressions are anonymous functions that don’t belong to any class. They are used for creating functional interfaces(interfaces with only one abstract method). They can be assigned to variables and can be passed to methods as parameters. A typical lambda expression looks like this:

```
(parameters) -> {action}
```

So, go ahead and update the `gradle` file as follows:

```groovy
android {
    // ...

    compileOptions {
        sourceCompatibility = '1.8'
        targetCompatibility = '1.8'
    }

}
```

After adding these libraries to your project, you will need to click the *Sync Now* link that the IDE shows to download the libraries you have added and make them available.

### Adding a network state
When frequently making network requests to load data, a few things can go wrong, such as poor internet connection, server downtime, etc. You need to be able to monitor the status of your network requests to know the next appropriate action to take. 

In this regard, you will create an enum that holds the status of a network request. First, create a new package called `network` inside `com.auth0.todo`. After that, create the enum named `Status` and add the following code to it:

```kotlin
// ./app/src/main/java/com/auth0/todo/network/Status.java
enum Status {
    LOADING,
    SUCCESS,
    FAILED
}
```

From the code, the enum has three constants which represent the status of any network request. 

### Modifying your List Adapter
Since you have switched from a `ListView`, you need to revamp your `ListAdapter` class. Open your `ListAdapter` class and replace it with this snippet:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/ToDoListAdapter.java

package com.auth0.todo.util;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import com.auth0.todo.R;
import com.auth0.todo.ToDoItem;
import com.auth0.todo.network.Status;
import androidx.annotation.NonNull;
import androidx.core.util.Consumer;
import androidx.paging.PagedListAdapter;
import androidx.recyclerview.widget.DiffUtil;
import androidx.recyclerview.widget.RecyclerView;

public class ToDoListAdapter extends PagedListAdapter<ToDoItem,RecyclerView.ViewHolder> {

    private Status networkState;
    private Consumer retryFunction;

    public ToDoListAdapter(@NonNull DiffUtil.ItemCallback<ToDoItem> diffCallback, Consumer function) {
        super(diffCallback);
        this.retryFunction = function;
    }

    // Other methods...

}
```

In this class, you changed the parent class from `BaseAdapter` to `PagedListAdapter`. Then, you added new class variables - `networkState` to tell the current network state, and `retryFunction` to hold the retry function. The retry function will be passed from the constructor.

The `PagedListAdapter` class extends the `RecyclerView.Adapter` class and so, you have to implement some methods that typically come with it. Some of which include:  `getItemViewType`, `getItemCount`, `onCreateViewHolder`, `onBindViewHolder`. You will now add these methods to your adapter.

The `getItemViewType` method is used to return view type of an item at a particular position. The method provides you with the current position of the list and you can use that to perform your logic. Add the method to your `ToDoListAdapter` class:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/ToDoListAdapter.java
@Override
public int getItemViewType(int position) {
    if(hasExtraRow() && position == getItemCount() - 1){
        return R.layout.loading_item;
    } else {
        return R.layout.to_do_item;
    }
}
```

Here in this method, you are checking to know when to display the normal todo item or a loading item on the list. In this method, you have the `loading_item` layout file and the `hasExtraRow()` method missing. Now, create a new layout named `loading_item` and add this: 

```xml
<!-- ./app/src/main/res/layout/loading_item.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" 
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ProgressBar
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/progress_bar"
        android:layout_gravity="center"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/retry_button"
        android:text="Retry"
        android:layout_gravity="center"/>

</LinearLayout>
```

This layout contains a progress bar that will be displayed when you are loading data from the server and a button to retry network requests if an error occurred. After that, add the missing `hasExtraRow()` method still in the `ToDoListAdapter` class like so:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/ToDoListAdapter.java

private Boolean hasExtraRow(){
    return networkState!=null && networkState != Status.SUCCESS;
}
```

This method checks to know if an extra row should be shown. That is simply known if the `networkState` is not type `Status.SUCCESS`.

The next method you will add to the adapter is the `onCreateViewHolder` method. This method is used by the adapter when a new ViewHolder of the given view type is needed. The method creates a new ViewHolder using a layout reference you pass to it. Go ahead and add the method to your class like so:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/ToDoListAdapter.java

@Override
public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {

    switch (viewType){

        case R.layout.to_do_item:
            View todoItemView = LayoutInflater.from(parent.getContext()).inflate(R.layout.to_do_item,parent,false);
            return new ToDoViewHolder(todoItemView);

        case R.layout.loading_item:
            View loadingItemView = LayoutInflater.from(parent.getContext()).inflate(R.layout.loading_item,parent,false);
            return new LoaderViewHolder(loadingItemView,retryFunction);

    }

    throw new IllegalArgumentException("Wrong view type");

}
```

Based on the return value of the `getItemViewType` method you created earlier, you know the layout resource to use in creating your ViewHolder. There are two separate ViewHolders used here; `ToDoViewHolder`  and `LoaderViewHolder`.

> A ViewHolder describes an item view and metadata about its place within the `RecyclerView`. So, a ViewHolder keeps track of a view and the data you want to bind to that view.

Create a new class named `ToDoViewHolder` in the `com.auth0.todo.util` package and add this:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/ToDoViewHolder.java

package com.auth0.todo.util;

import android.view.View;
import android.widget.TextView;
import com.auth0.todo.R;
import com.auth0.todo.ToDoItem;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

public class ToDoViewHolder extends RecyclerView.ViewHolder {

    private TextView textView;

    public ToDoViewHolder(@NonNull View itemView) {
        super(itemView);
        textView = itemView.findViewById(R.id.to_do_message);
    }

    public void bind(ToDoItem toDoItem){
        textView.setText(toDoItem.getMessage());
    }

}
```

This ViewHolder will be used when you want to display a todo item. In the constructor, it is initialized with a view as you did in the penultimate snippet. In the constructor, you also initialized the `textView` with the view’s id. The `bind` method uses the message item from the  `toDoItem` object to set the text on the `textView`. 

After that, create the second ViewHolder named `LoaderViewHolder` in the `com.auth0.todo.util` directory and add this:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/LoaderViewHolder.java

package com.auth0.todo.util;

import android.view.View;
import android.widget.Button;
import android.widget.ProgressBar;
import com.auth0.todo.R;
import com.auth0.todo.network.Status;
import androidx.annotation.NonNull;
import androidx.core.util.Consumer;
import androidx.recyclerview.widget.RecyclerView;

public class LoaderViewHolder extends RecyclerView.ViewHolder {

    private ProgressBar progressBar = itemView.findViewById(R.id.progress_bar);
    private Button retryButton = itemView.findViewById(R.id.retry_button);

    public LoaderViewHolder(@NonNull View itemView, Consumer retryFunction) {
        super(itemView);
        retryButton.setOnClickListener(v -> retryFunction.accept(null));
    }

    public void bind(Status networkState){

        if(networkState==Status.LOADING){
            progressBar.setVisibility(View.VISIBLE);
            retryButton.setVisibility(View.GONE);
        } else if(networkState== Status.SUCCESS){
            progressBar.setVisibility(View.GONE);
            retryButton.setVisibility(View.GONE);
        } else {
            progressBar.setVisibility(View.GONE);
            retryButton.setVisibility(View.VISIBLE);
        }

    }

}
```

This ViewHolder is similar to the  `ToDoViewHolder` you created earlier. This is one is used to display a loading item. Apart from the `itemView`, the `retryFunction` is also passed to the constructor. The `retryFunction` is triggered when the `retryButton` is clicked. The `bind` method takes a `networkState` value to know what should be displayed. 

The next method you will add to your `ToDoListAdapter` is the `onBindViewHolder` method. This method is in charge of binding data to a particular position on the list. Add the method to your adapter like so:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/ToDoListAdapter.java

@Override
public void onBindViewHolder(@NonNull RecyclerView.ViewHolder holder, int position) {

    switch (getItemViewType(position)) {

        case R.layout.loading_item:
            ((LoaderViewHolder) holder).bind(networkState);
            break;

        case R.layout.to_do_item:
            ((ToDoViewHolder) holder).bind(getItem(position));
            break;

    }

}
```

This method still makes use of the `getItemViewType` method you created earlier. Depending on the view type at any position on the list, the `bind` method of the corresponding ViewHolder is called. 

Finally, add these two methods in the `ToDoListAdapter`:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/ToDoListAdapter.java

@Override
public int getItemCount() {
    int count = super.getItemCount();
    if(hasExtraRow()){
        count++;
    }
    return count;
}

public void updateNetworkState(Status networkState) {
    Status previousState = this.networkState;
    Boolean hadExtraRow = hasExtraRow();

    this.networkState = networkState;
    Boolean hasExtraRow = hasExtraRow();

    if (hadExtraRow!=hasExtraRow) {
        if (hadExtraRow){
            notifyDataSetChanged();
        } else {
            notifyItemInserted(super.getItemCount());
        }
    } else if (hasExtraRow && previousState!=networkState){
        notifyItemChanged(getItemCount() - 1);
    }

}
```

In this snippet, you have two methods; `getItemCount()` and `updateNetworkState()`. The `getItemCount()` returns the size of the list, so here, you are overriding the method to increment the list by one when a loading item is supposed to be shown. The `updateNetworkState()` method is a public method that will be used to update the status of the `networkState` object and refresh the list after doing so. 

### Creating a DataSource
As mentioned earlier, you need to implement a DataSource which determines how the data is fetched. In the `com.auth0.todo.network` package, create a new class named `ToDoDataSource` and set it up like so:

```kotlin
// ./app/src/main/java/com/auth0/todo/network/ToDoDataSource.java

package com.auth0.todo.network;

import android.content.Context;
import com.android.volley.RequestQueue;
import com.android.volley.toolbox.JsonArrayRequest;
import com.android.volley.toolbox.Volley;
import com.auth0.todo.ToDoItem;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;
import java.util.ArrayList;
import java.util.List;
import androidx.annotation.NonNull;
import androidx.core.util.Consumer;
import androidx.lifecycle.MutableLiveData;
import androidx.paging.PageKeyedDataSource;

public class ToDoDataSource extends PageKeyedDataSource<String, ToDoItem> {

    private int pageCount = 1;
    private String url = "http://10.0.2.2:3001";
    private RequestQueue queue;
    public MutableLiveData<Status> status = new MutableLiveData<>();
    private Consumer retry;

    public ToDoDataSource(Context context) {
        queue = Volley.newRequestQueue(context);
    }

    @Override
    public void loadInitial(@NonNull PageKeyedDataSource.LoadInitialParams<String> params, @NonNull final LoadInitialCallback<String, ToDoItem> callback) {}

    @Override
    public void loadBefore(@NonNull LoadParams<String> params, @NonNull final LoadCallback<String, ToDoItem> callback) {}

    @Override
    public void loadAfter(@NonNull LoadParams<String> params, @NonNull final LoadCallback<String, ToDoItem> callback) {}

}
```

In this class, you are extending one of the `DataSource` interface implementations - `PageKeyedDataSource`. This interface provides three methods to implement; `loadInitial()`, `loadBefore()`, and `loadAfter()`. 

The `loadInitial()` method is called when data is supposed to be loaded initially. This method is typically called once, at the start of the list. Add this snippet to your `loadInitial()` method:

```kotlin
// ./app/src/main/java/com/auth0/todo/network/ToDoDataSource.java

status.postValue(Status.LOADING);
JsonArrayRequest microPostsRequest = new JsonArrayRequest(url, response -> {
    pageCount++;
    callback.onResult(transformResponse(response), null, String.valueOf(pageCount));
    status.postValue(Status.SUCCESS);

}, error -> {
    status.postValue(Status.FAILED);
    retry = x -> loadInitial(params, callback);
});

queue.add(microPostsRequest);
```

Here, you first set the status of the `status` object to loading, then you make a network request to fetch the first page from the server. If the request is successful, the page count is incremented, the `DataSource` is notified through the `callback` object and the `status` object is updated too. 

If the request fails, the `status` object is updated accordingly and the `loadInitial()` method is assigned to the `retry` object. This means that if a request retry is to be performed, it will attempt the `loadInitial()` method again.

The next method `loadBefore()` will be left empty because you do not need it. The `loadAfter()` method is used to load subsequent pages from the server. Add this snippet to the `loadAfter()` method:

```kotlin
// ./app/src/main/java/com/auth0/todo/network/ToDoDataSource.java

status.postValue(Status.LOADING);
String updatedUrl = url + "?page=" + pageCount;
JsonArrayRequest microPostsRequest = new JsonArrayRequest(updatedUrl, response -> {
    pageCount++;
    callback.onResult(transformResponse(response), String.valueOf(pageCount));
    status.postValue(Status.SUCCESS);

}, error -> {
    status.postValue(Status.FAILED);
    retry = x -> loadAfter(params, callback);
});

queue.add(microPostsRequest);
```

This snippet is similar to what you had in the `loadInitial()` method. The difference here is that you are appending the `pageCount` to the url so that the server can know which page to fetch.

The `loadInitial()` and `loadAfter()` methods make use of a `transformResponse()` method. Create the method inside the `ToDoDataSource` class:

```kotlin
// ./app/src/main/java/com/auth0/todo/network/ToDoDataSource.java

private List<ToDoItem> transformResponse(JSONArray response) {

    List<ToDoItem> toDoItems = new ArrayList<>(response.length());
    for (int i = 0; i < response.length(); i++) {
        JSONObject item;
        String id = null;
        String message = null;
        try {
            item = response.getJSONObject(i);
            id = item.getString("_id");
            message = item.getString("message");
        } catch (JSONException e) {
            e.printStackTrace();
        }

        toDoItems.add(new ToDoItem(id, message));
    }

    return toDoItems;

}
```

The method takes the response from the server and parses it to a list of `ToDoItem` items. After that, add the `retryFailedRequest()` method to the class:

```kotlin
// ./app/src/main/java/com/auth0/todo/network/ToDoDataSource.java

public void retryFailedRequest() {
    Consumer previousRetry = retry;
    retry = null;
    if (previousRetry != null) {
        previousRetry.accept(null);
    }
}
```

Here, you have a public method that will execute the function stored in the `retry` object. Next, you will create a factory class in charge of providing an instance of the `ToDoDataSource` class you have just created. Inside the `com.auth0.todo.network` package, create a new class called `ToDoDataSourceFactory` and add this snippet:

```kotlin
// ./app/src/main/java/com/auth0/todo/network/ToDoDataSourceFactory.java

package com.auth0.todo.network;

import android.content.Context;
import com.auth0.todo.ToDoItem;
import androidx.annotation.NonNull;
import androidx.lifecycle.MutableLiveData;
import androidx.paging.DataSource;

public class ToDoDataSourceFactory extends DataSource.Factory<String, ToDoItem> {

    public MutableLiveData<ToDoDataSource> todoDataSource = new MutableLiveData<>();
    private ToDoDataSource source;

    public ToDoDataSourceFactory(Context context){
        source = new ToDoDataSource(context);
    }

    @NonNull
    @Override
    public DataSource<String, ToDoItem> create() {
        todoDataSource.postValue(source);
        return source;
    }
}
```

This class is used to create a DataSource (`ToDoDataSource`). In this snippet, you have a publicly available `todoDataSource` object which gives the latest value of the instance of the `ToDoDataSource` created in the factory class.

### Wrapping up your app

Now that you have created the little bits, you will now stitch things up in your `MainActivity` class. Before that, you have to do some cleanups. Remove the interfaces implemented by the class and keep the parent activity. It should look like this:

```kotlin
public class MainActivity extends AuthAwareActivity {
```

After that, remove the `onErrorResponse()` and the `onResponse()` methods in the class. Next, replace the `onCreate()` method with this:

```kotlin
// ./app/src/main/java/com/auth0/todo/MainActivity.java

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    ToDoDataSourceFactory factory =  new ToDoDataSourceFactory(this);

    this.toDoListAdapter = new ToDoListAdapter(new DiffUtilCallback(), x -> factory.todoDataSource.getValue().retryFailedRequest());
    RecyclerView microPostsListView = findViewById(R.id.to_do_items);
    microPostsListView.setLayoutManager(new LinearLayoutManager(this));
    microPostsListView.setAdapter(toDoListAdapter);

    PagedList.Config config = new PagedList.Config.Builder()
    .setPageSize(15)
    .setEnablePlaceholders(false)
    .build();

    new LivePagedListBuilder<>(factory, config).build().observe(this, toDoListAdapter::submitList);

    LiveData<Status> networkStateLiveData =
    Transformations.switchMap(factory.todoDataSource, input -> input.status);

    networkStateLiveData.observe(this, toDoListAdapter::updateNetworkState);

}
```

Here in this method, you started off creating an instance of  the `ToDoDataSourceFactory` class, then you initialize your `toDoListAdapter` object. Two parameters are passed to the `ToDoListAdapter` constructor, a `DiffUtilCallback` instance and the retry function which is the public function made available in the `ToDoDataSource` class.

After that, you initialized the `RecyclerView` and assigned the adapter to it. Then, you built a `PagedList` which updates the `toDoListAdapter` when there is an update. Finally, you setup a `LiveData` object to monitor network status and update the `ToDoListAdapter` class through the `updateNetworkState()` method.

Now, go ahead and create the missing `DiffUtilCallback` class. Create the class  `DiffUtilCallback` in the `com.auth0.todo.util` package:

```kotlin
// ./app/src/main/java/com/auth0/todo/util/DiffUtilCallback.java

package com.auth0.todo.util;

import com.auth0.todo.ToDoItem;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.DiffUtil;

public class DiffUtilCallback extends DiffUtil.ItemCallback<ToDoItem> {

    @Override
    public boolean areItemsTheSame(@NonNull ToDoItem oldItem, @NonNull ToDoItem newItem) {
        return oldItem.getId().equals(newItem.getId());
    }

    @Override
    public boolean areContentsTheSame(@NonNull ToDoItem oldItem, @NonNull ToDoItem newItem) {
        return oldItem.getId().equals(newItem.getId()) &&
            oldItem.getMessage().equals(newItem.getMessage());
    }

}
```

This class is used by the adapter to check the difference between two non-null items in the list. 


## Conclusion

In this article, you have learned about infinite lists and how to implement them in your Android apps. You used the extensive Paging library to achieve this. The Paging library introduces the separation of concerns to help you easily manage how your data is fetched and displayed. There is so much more you can achieve with this library if you dive deeper. I'm looking forward to that, catch ya!






