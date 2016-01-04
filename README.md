# RxAndroid
From this tutorial your will learn how to make a REST API call using RxAndroid. You will also learn some interesting functions of RxAnroid.

## Introduction

RxAndroid is an enhancement of RxJava, specially designed for Android application development. RxJava follows Observer design pattern. _(In this tutorial I am not going to teach you Observer design pattern. Please do some self studies if you do not know what is Observer design patter.)_ There are four main components in RxJava or reactive programming:

1. Observable
2. Observer
3. Subscriber and
4. Subjects

__Observer__ and __Subjects__ are called 'producers' who produce or emit data. Whereas __Observers__ and __Subscribers__ are called 'consumers' who consumes data that are produced by the producers.

### Key concepts of RxJava

1. _Easy concurrency to better use server's power_
2. _Easy conditional asynchronous execution_
3. _A proper way to escape the callback hell_#
4. _A reactive appraoch_
 
#### What's diffferent in RxJava

_From a pure Java point of view , the Rxjava Observable class extends the classic Gand of Four Observer pattern concept. It adds three missing abilities:_

1. _The producer can now signal that there is no more data available: the onCompleted() event._
2. _The producer can now signal that an error occured: the onError() event._
3. _RxJava Observables can be composed instead of nested, saving the developer from the callback hell.

Source : Ivan Mrogillo, RxJava Essentials, Packt Publishing 2015





## Project Description

### Prerequisites

* Your dependencies tag in project's build.gradle file should look like this
```java
dependencies {
    classpath 'com.android.tools.build:gradle:1.3.0'
    // Assists in working with annotation processors for Android Studio.
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    //Assists using retrolambda feature of java 8. Retrolambda makes your code more readable by hiding a lot of boilerplate code.
    classpath 'me.tatarka:gradle-retrolambda:3.2.0'
}
  ```
* Your dependencies tag in module's build.gradle file should look like this
```java
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  testCompile 'junit:junit:4.12'
  compile 'com.android.support:appcompat-v7:23.0.1'
  //Needed for Injecting Views
  compile 'com.jakewharton:butterknife:6.1.0'

  //Needed for managing image loading and for easy caching
  compile 'com.squareup.picasso:picasso:2.5.2'

  // Needed for managing DI
  compile 'com.google.dagger:dagger:2.0.1'
  apt "com.google.dagger:dagger-compiler:2.0.1"


  // Needed to annotated field variables of POJO/Model class for easy mapping with JSON fields
  provided 'javax.annotation:jsr250-api:1.0'
  compile 'javax.annotation:jsr250-api:1.0'

  // Needed to make network based API call
  compile 'com.squareup.retrofit:retrofit:2.0.0-beta2'
  compile 'com.squareup.retrofit:converter-gson:2.0.0-beta2'

  //"Response caching avoids the network completely for repeat requests" - http://square.github.io/okhttp/
  compile 'com.squareup.okhttp:okhttp:2.5.0'

  //For Reactive Programming. For replacing ASYNCTASK.
  compile 'io.reactivex:rxandroid:1.0.1'
  compile 'io.reactivex:rxjava:1.0.14'
  
  //In order to convert the API response type Observable, we have to set the call adapter to RxJavaCallAdapter.
  compile 'com.squareup.retrofit:adapter-rxjava:2.0.0-beta2'
}
  ```
* JDK 8 is required to use Retrolambda feature. So if you are using JDK 7 or 8 then I would recommend you to update it and change your JAVA_HOME and path variable accordingly. Do not forget to change JDK location in andnroid studio also.



In this turotial I have extended one of my github projects https://github.com/shohrabuddin/Dagger2-ButterKnife-Retrofit-OkHttp-Picasso-Gson_Annotations.git. If you have already gone through this project then you might know that I have used Retrofit in that project to make Rest API call. I keep the whole project as it is and just change the following classes to enable RxAnroid enabled API call. However the project goals are:
The aim of this project is to use all of the above mentioned libraries. The project itself is very simple. The taks are:

1. Create an app for Smartphones and Tablets that queries a web service API for catalog
content, parses the JSONformatted response and displays the data. The data consists of a list of products that should be shown in a vertical scrolling list.
2. In the product list show the product's image, name, price and brand
3. Add UI for sort functionality on all sort orders (newest, oldest, price_low etc.).
4. Add offline functionality (Cache images, save data within a local persistence store, etc.)


__I have added a lot of comments in the project. So I hope you will easily understand all code easily.__

Here I have added some important classes for you from the project:

#### Carmudi_API.java
```java
public interface CarmudiAPI {
    @Headers("Cache-Control: max-age=640000, s-maxage=640000 , max-stale=2419200") // This is optional
    @GET("/api/cars/") //the complete URL is https://www.carmudi.ae/api/cars/ (https://www.carmudi.ae is the base URL which is available in StaticValue class)
   /*
    Retrofit interface 'Call<T>' causes the API call (https://www.carmudi.ae/api/cars). The URL takes a query parameter called 'sort'.
    Including the paramter the URL looks like https://www.carmudi.ae/api/cars/sort:[value] (value could be newest, oldest etc). You can add query parameter by
    using @Query(parametername) annotation.
    */
    //Call<CarmudiModel> getCarmudiModel(@Query("sort") String sortingCriteria);

   //Observable class makes it Reactive :) (RxAndroid)
    Observable<CarmudiModel> getCarmudiModel(@Query("sort") String sortingCriteria);

}
```

### Utility.java
```java
/**
 * Created by shohrab.uddin on 22.11.2015.
 * Retrofit Interface for API calls like GET/POST etc.
 */
public Retrofit provideRetrofit(Application app, Context context) {

    isNetworkAvailable=isNetworkAvailable(context);
    Cache cache = provideOkHttpCache(app);
    OkHttpClient okHttpClient = provideOkHttpClient(cache);

    Retrofit retrofit = new Retrofit.Builder()
            .baseUrl(StaticValue.BASE_URL)
            .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) //In order to convert the API response type Observable, we have to set the call adapter to RxJavaCallAdapter.
            .addConverterFactory(GsonConverterFactory.create())
            .client(okHttpClient)
            .build();
    return retrofit;
}
```

#### MainActivity.java
```java
/**
* This method is used to fetch data from remote API using RxAndroid
* @param sortingCriteria
*/
private void apiCallUsingRxAndroid(final String sortingCriteria) {
  dialog = ProgressDialog.show(this, "Wait", "Awesome cars are loading for you...");

  //converting the API response to Observable
  Observable<CarmudiModel> observableResponse = carmudiAPI.getCarmudiModel(sortingCriteria);

  //A new thread is subscribed to observable
  observableResponse.subscribeOn(Schedulers.newThread())
          .observeOn(AndroidSchedulers.mainThread())
          .subscribe(response -> {
              if (response.isSuccess()) {
                  txtErrorMsg.setVisibility(View.GONE);
                  txtTitle.setText(sortingCriteria.toUpperCase()+" Cars");
                  awesomeCars = response.getMetaData().getResults(); //Now you can access all fields of CarmudiModel class in familiar OOP Style
                  Log.d("MainActivity", "Items = " + awesomeCars.size());
                  carAdapter = new CarAdapter(MainActivity.this, awesomeCars);
                  listView_Car.setAdapter(carAdapter);

                  //Caching data into SQLite
                  dbHelper.addCachedCars(awesomeCars, sortingCriteria); //Traditional Way

                  //Caching timestamp. This timestamp is used to update/snychronize local SQLite DB with remote DB
                  mSharedPref.setCached_time_stamp(mUtility.getUnixTimeStamp());
                  dialog.dismiss();


              } else {  //If response is other than 200 (Ok)
                  dialog.dismiss();
                  txtErrorMsg.setText("Something went wrong");

              }
          });

}

/**
* This method is used to fetch data from remote API using Retrofit
* @param sortingCriteria
*/
private void apiCallUsingRetrofit(final String sortingCriteria) {

  call.enqueue(new Callback<CarmudiModel>() { //call.enqueue makes asynchronous call of API, call.create makes synchronous call
      @Override
      public void onResponse(Response<CarmudiModel> response, Retrofit retrofit) {
          dialog.dismiss();
          txtTitle.setText(sortingCriteria.toUpperCase()+" Cars");
          Log.d("MainActivity", "Status Code = " + response.code());
          if (response.isSuccess()) {
              txtErrorMsg.setVisibility(View.GONE);
              CarmudiModel result = response.body(); //Get the JSON response which is mapped to CarmudiModel POJO class (MAGIC!!). No JSON parsing is needed at all!!
              Log.d("MainActivity", "response = " + new Gson().toJson(result));
              awesomeCars = result.getMetaData().getResults(); //Now you can access all fields of CarmudiModel class in familiar OOP Style
              Log.d("MainActivity", "Items = " + awesomeCars.size());
              carAdapter = new CarAdapter(MainActivity.this, awesomeCars);
              listView_Car.setAdapter(carAdapter);

              //Caching data into SQLite
              dbHelper.addCachedCars(awesomeCars, sortingCriteria); //Traditional Way

              //Caching timestamp. This timestamp is used to update/snychronize local SQLite DB with remote DB
              mSharedPref.setCached_time_stamp(mUtility.getUnixTimeStamp());

          } else {  //If response is other than 200 (Ok)
              ResponseBody responseBody = response.errorBody();
              if (responseBody != null) {
                  try {
                      txtErrorMsg.setText("responseBody = " + responseBody.string());
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              } else {
                  txtErrorMsg.setText("responseBody = null");
              }
          }

      }

      @Override

      public void onFailure(Throwable t) {


          dialog.dismiss();
          txtErrorMsg.setText("t = " + t.getMessage());
      }

  });
}
```

