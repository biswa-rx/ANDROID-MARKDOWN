# JOB INTENT SERVICE

`JobIntentService combine two different service`

1. IntentService
2. JobService

So why it is necessary:- Since form android oreo (API 26) we can't run normal service in the background any more because system will kill them of around one minute and through a exception when our app self in background. This happens to save system resources and to prevent drain battary so quickly.

This intent service is just a sub class of normal service that exicute the incomming work sequentially in background thread.And when it finished all work then it automatically stops itself.

`As a service it also affected by background execution limits. One way to handle this to promote intent service to foreground service for api >= 26 , we can run it without any restrictions, but it also means we have to display a persistence notification which show whole duration while our service is running.But it also notify use that something runnning in background.`

`You can use the previous Intent service approch if you want your service must get exicuted imidiately with out any interuption. But in most cases you can use JobIntentService instead which uses a different approch.`

`On pre Oreo devices it starts as normal intent service because there it don't have any restrictions, But on Oreo unwords it uses JobScheduler to achieve a similar behavior as IntentService.`

### There are much more about it you can find from official documentation. [Click here](https://developer.android.com/reference/androidx/core/app/JobIntentService)


`ExampleJobIntentService`
```java
public class ExampleJobIntentService extends JobIntentService {
    private static final String TAG = "ExampleJobIntentService";

    static void enqueueWork(Context context, Intent work) {
        enqueueWork(context, ExampleJobIntentService.class, 123, work); // To start service we have to enqueue that Work but there is two value fixed so we can do this work here instead in MainActivity
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate");
        //Intent service we aquire a wakelock in onCreate to keep CPU running even device screen turn off. But intent service takes care of this automatically. So we don't have to do this.
    }

    @Override
    protected void onHandleWork(@NonNull Intent intent) {
        Log.d(TAG, "onHandleWork"); // This method is run in both cases when executing as intent service or job service in background thread.

        String input = intent.getStringExtra("inputExtra");

        for (int i = 0; i < 10; i++) {
            Log.d(TAG, input + " - " + i);

            // When our job has been stoped it doesn't automatically stop exicuting onHandleWork() method. We have to stop it manually. If we don't stop this method from executing then system will simply kill our service, so better you stop it in a control manner. And job will release wakelock after it has been stopped.

            // so after our job is stopped, we should stop our onHandleMethod(). In JobService we use a boolean variable to stop the onHandleMethod() but here we have isStoped() method to know that our job is stopped or running.
            if (isStopped()) return;

            SystemClock.sleep(1000);
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy");
    }

    @Override
    public boolean onStopCurrentWork() {
        Log.d(TAG, "onStopCurrentWork"); // This method will be treagered when the current job will be stopped if it using the job scheduler so on api 26 onwords. This happens when the device have strong memory pressure, when go to DOSE and simply when it running too long because job scheduler have a time limit is around 10 minutes, after that it will be stopped and return value is boolean value which is defined if you want to resume work letter or just drop it.

        return super.onStopCurrentWork(); // Default value of super class implementation is true, if your job is stopped then it will leter start again with the same intent which is processing so nothing get lost.
    }
}
```

`MainActivity`

```java
public class MainActivity extends AppCompatActivity {
    private EditText editTextInput;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        editTextInput = findViewById(R.id.edit_text_input);
    }

    public void enqueueWork(View v) {
        String input = editTextInput.getText().toString();

        Intent serviceIntent = new Intent(this, ExampleJobIntentService.class);
        serviceIntent.putExtra("inputExtra", input);

        ExampleJobIntentService.enqueueWork(this, serviceIntent);
    }
}
```

`We have WAKE_LOCK permission in Manifest for API <= 25 and API >= 26 it will do automatically for us.`

```xml
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

`Resister the service`
```xml
 <service
            android:name=".ExampleJobIntentService"
            android:permission="android.permission.BIND_JOB_SERVICE" />
```

#### NOTE : In JobScheduler we do lot of configuration (e.g. wifi connectivity, is device charging). We define specificly some condition to stop our job.But in JobIntentService we can't do this constraints. Instead it is basically start as soon as possible because the point of JobIntentService is to mimic the behavior of the normal IntentService. If you want some constraints then JobIntentService is not suitable for you.

_This class is deprecated.
This class has been deprecated in favor of the Android Jetpack WorkManager library, which makes it easy to schedule deferrable, asynchronous tasks that are expected to run even if the app exits or the device restarts._