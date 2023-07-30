# INTENT SERVICE

`Intent service is a subclass of a normal service but the difference is that it exicutes the incoming task in background thread, where as a normal service runs on main thread. So it is a class for do threading and service make easier to handle`

* Since intent service are just services so it also affected same android oreo background exicution limits applied. (Which means you can't run it as normal background service because the system will either kill it or throw exception)...
* Recommended alternative to use JobIntentService instead, JobIntentService will work API 26 and higher and if (API<= 25) then it will run as normal background service which system will not kill either way.
* JobIntentService is also subclass of service class but with a jobScheduler api which decides correct time for job to execute.

>NOTE: Intent service is better work on API lower than 26 if higher then use ForegroundService instead for do heavy work or use JobIntentService for do lite work on background thread(default). Intent service will only we have to start but it stop it self rather than normal foreground service which we stop it manually.

>API <= 25 --> IntentService

>API > 26 and lite work --> JobIntentService

>API > 26 and High priority service and can't kill by system --> Foreground service

_If we use JobIntentService then if we run it in API<26 then it will automatically switch to IntentService. Here we impliment a IntentService but if this service run API>=26 then write condition for switch to Foreground service to prevent to killed by OS. This is enough information let's jump to code_

 `Notification channel created on App`
 ```java
 public class App extends Application {
    public static final String CHANNEL_ID = "exampleServiceChannel";

    @Override
    public void onCreate() {
        super.onCreate();

        createNotificationChannel();
    }

    private void createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel serviceChannel = new NotificationChannel(
                    CHANNEL_ID,
                    "Example Service Channel",
                    NotificationManager.IMPORTANCE_DEFAULT
            );

            NotificationManager manager = getSystemService(NotificationManager.class);
            manager.createNotificationChannel(serviceChannel);
        }
    }
}
 ```

 `ExampleIntentService`
 ```java
 import static com.codinginflow.intentserviceexample.App.CHANNEL_ID;

public class ExampleIntentService extends IntentService {
    private static final String TAG = "ExampleIntentService";

    private PowerManager.WakeLock wakeLock; // wake lock instance

    public ExampleIntentService() {
        super("ExampleIntentService");
        setIntentRedelivery(true); // here false indicates that if system kill the service it will never created automatically again (it is same like NON_STICKY in foreground service) or if it is true means it will Redeliver intent again. Remember in IntentService all work will done in the basis of Intent
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate");

        //We start out WAKE_LOCK here. A wake-lock is keep the CPU running when the screen is turned off
        PowerManager powerManager = (PowerManager) getSystemService(POWER_SERVICE);
        wakeLock = powerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
                "ExampleApp:Wakelock"); //ExampleApp:Wakelock only for debugging purposes

        wakeLock.acquire();  //Some time it require a number wakeLock.acquire(600000); 600000 defines 10 minutes it will wakeLock, It ensures that if you forget to release the wakeLock then it will automatically release after 10 minutes because wakeLock drain battery of our phone. We have to release the wakeLock on onDestroy() to release when our job is finished.

        Log.d(TAG, "Wakelock acquired");

        //Condition for checking if this version is greater than equal to Oreo then it just start normal Forgroud Service because Intent service does not make any sense for higher android version
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            Notification notification = new NotificationCompat.Builder(this, CHANNEL_ID)
                    .setContentTitle("Example IntentService")
                    .setContentText("Running...")
                    .setSmallIcon(R.drawable.ic_android)
                    .build();

            startForeground(1, notification);
        }
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        Log.d(TAG, "onHandleIntent"); // All out task  goes here in OnHandleIntent. This method call by service in a background thread. User this intent to send data to the service. Each incomming intent is executed sequentially one after another on one single thread. If you need multiple theread in a single time then you have to extend the service class directly because IntentService can't do that. However in most cases a single thread is sufficient to do our job.

        String input = intent.getStringExtra("inputExtra"); // Intent send from where service is started

        for (int i = 0; i < 10; i++) {
            Log.d(TAG, input + " - " + i);
            SystemClock.sleep(1000);
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy");

        wakeLock.release();  // Here we release our wake lock
        Log.d(TAG, "Wakelock released");
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

    public void startService(View v) {
        String input = editTextInput.getText().toString();

        Intent serviceIntent = new Intent(this, ExampleIntentService.class);
        serviceIntent.putExtra("inputExtra", input);

        // ContextCompat will run startForegroundService from Oreo onwords and startService for lower API
        ContextCompat.startForegroundService(this, serviceIntent);
    }
}
 ```

> And finally don't forget to add manifest permissions

```xml
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

> To resister our service

```xml
<service android:name=".ExampleIntentService" />
```


`IMPORTANT NOTE: In api<26 we don't have notification because we can start as normal background service but of course you can start your service as Foreground service in lower api versions, For this you have to simple remove if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){} this line and just call startForeground(1, notification) in any case. And the benifit of foreground service over a normal background service are that is less likely kill in low memory situations and also not affected by dose mode. The phone start its DOSE_MODE when it stay a while when it's screen turned off. And when it will happened our wakeLock.acquire(); will actually ignored which not work any more in dose mode. But it not case for Foreground services`