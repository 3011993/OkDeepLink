# OkDeepLink
[![Build Status](https://travis-ci.org/HongJun2046/OkDeepLink.svg?branch=master)](https://travis-ci.org/HongJun2046/OkDeepLink)
[ ![Download](https://api.bintray.com/packages/zmanchina/maven/okdeeplink-gradle/images/download.svg) ](https://bintray.com/zmanchina/maven/okdeeplink-gradle/_latestVersion)

[Cn Introduction](http://www.jianshu.com/p/8a3eeeaf01e8)

OkDeepLink provides a annotation-based api to manipulate app deep links.

- register deep links  with annotation `@Path`、`@Activity`
- start  deep links by service which generates an implementation by `Annotation-processor`, auto inject to activity with annotation `@Service`
- url or bundle parameters auto inject to activity , restore  when activity recreate  by annotation `@Query`
- async intercept deep links in ui  thread  with annotation `@Intercept`
- support activity result  with `rxJava`


### Config
**In Root Gradle**

```groovy
repositories {
   jcenter()
}
dependencies {
    classpath 'com.hongjun:okdeeplink-gradle:1.0.0'
}
```

**In App Or Lib Gradle**

```groovy
apply plugin: 'okdeeplink.plugin'
```
if use multiple apt lib, you must use

```groovy
packagingOptions {
        exclude 'META-INF/services/javax.annotation.processing.Processor'
}
```

### Example

If you want define `old://app/main?key=value` uri, you can do like this

#### Define Host And Scheme

you can define dispatch activity which receive deep links in your `AndroidManifest.xml` file (using `odl` as an example):

**In App AndroidManifest**
```xml
 <activity
            android:name="okdeeplink.DeepLinkActivity"
            android:theme="@android:style/Theme.Translucent.NoTitleBar">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data
                    android:host="app"
                    android:scheme="odl" />
            </intent-filter>
 </activity>
```
In the future, I will use annotation `@AppLink` which define scheme and host  instead of `AndroidManifest.xml`. I already have a simple way.

**In Service File**

```java
public interface SampleService {


    @Path("/main")
    @Activity(MainActivity.class)
    void startMainActivity(@Query("key") String key);
}
```
path must start with "/"
when app receive uri like `old://app/main?key=value`, `DeepLinkActivity` will start ,then dispatch the deep link to the appropriate `Activity`.


**In Activity Or Fragment File**

You also start MainActivity by service in app.
```java
public class SecondActivity extends AppCompatActivity {

       @Service
       SampleService sampleService;
       @Query("key")
       String key;


       sampleService.startMainActivity("value");
}

```

### Intercept
If you want Intercept `old://app/second`, you can use `@Intercept`

```java
@Intercept(path = "/second")
public class SecondInterceptor extends Interceptor {
    @Override
    public void intercept(final Call call) {
        Request request = call.getRequest();
        final Intent intent = request.getIntent();
        Context context = request.getContext();

        StringBuffer stringBuffer = new StringBuffer();
        stringBuffer.append("Intercept\n");
        stringBuffer.append("URL: " + request.getUrl() + "\n");

        AlertDialog.Builder builder = new AlertDialog.Builder(context,R.style.Theme_AppCompat_Dialog_Alert);
        builder.setTitle("Notice");
        builder.setMessage(stringBuffer);
        builder.setNegativeButton("cancel", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                call.cancel();
            }
        });
        builder.setPositiveButton("ok", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                intent.putExtra("key1", "value3");
                call.proceed();
            }
        });
        builder.setOnDismissListener(new DialogInterface.OnDismissListener() {
            @Override
            public void onDismiss(DialogInterface dialog) {
                call.cancel();
            }
        });
        builder.show();
    }
}
```

![intercept`old://app/second` ](https://raw.githubusercontent.com/HongJun2046/OkDeepLink/master/snapshot/intercept_preview.png)


### Log
I define `LogInterceptor`, this can log deep links which start、notFound、error , log tag is `OkDeepLink`

### Other Way
You can start  intent action by service

```java
public interface SampleService {

    @Action(MediaStore.ACTION_IMAGE_CAPTURE)
    Observable<Response> startImageCapture();

     @Action(Intent.ACTION_DIAL)
     @Uri("tel:{phone}")
     void startTel(@UriReplace("phone") String phone);
}

```
```java
    public void startImageCapture(){
        sampleService
                .startImageCapture()
                .subscribe(new Action1<Response>() {
                    @Override
                    public void call(Response response) {
                        Intent data = response.getData();
                        int resultCode = response.getResultCode();
                        if (resultCode == RESULT_OK) {
                            Bitmap imageBitmap = (Bitmap) data.getExtras().get("data");
                            ImageView imageView = (ImageView) findViewById(R.id.ImageView_Capture_Image);
                            imageView.setImageBitmap(imageBitmap);
                        }
                    }
                });
    }

    public void startTel(String phone){
            sampleService
                    .startTel(phone);
        }
```

### Words

It is so simple，then you just do it like `sample`

License
-------

    Copyright 2017 Hongjun2046

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.


