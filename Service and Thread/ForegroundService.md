# FOREGROUND SERVICE

It run our service on UI thread

First we have to create a notification channel

`App`
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

`ExampleService`
```java
import static com.codinginflow.foregroundserviceexample.App.CHANNEL_ID;


public class ExampleService extends Service {

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        String input = intent.getStringExtra("inputExtra");

        Intent notificationIntent = new Intent(this, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(this,
                0, notificationIntent, 0);

        Notification notification = new NotificationCompat.Builder(this, CHANNEL_ID)
                .setContentTitle("Example Service")
                .setContentText(input)
                .setSmallIcon(R.drawable.ic_android)
                .setContentIntent(pendingIntent)
                .build();

        startForeground(1, notification); // If we do not probide startForeground() method then it will run only for one minute as a normal service and system will kill it after one minute but if we call startForegroundService() for initiate service(sdk>=26) there will be a time window which is for 5 seconds and if we do not provide notification then it will automatically killed by OS.

        //do heavy work on a background thread
        //stopSelf(); //stopself() method stop it after out work is over by system

        return START_NOT_STICKY; //It tell to system that what to do when system kill it for low memory
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
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

        Intent serviceIntent = new Intent(this, ExampleService.class);
        serviceIntent.putExtra("inputExtra", input);

        // startService() also create service but it only works when our application is open/running because after sdk >=26 startForegroundService() method will introduces for start service either app is alive or not but if sdk < 26 then startService() work always but if higher then app will crash as iligal exception.

        // startService(serviceIntent);

        // We can mennually add a check that
        //if (Build.VERSION.SDK_INT >= 26){ context.startForegroundService(serviceIntent);} else {context.startService(serviceIntent);}
        
        //Otherwise use ContextCompat.startForegroundService() which same condition is applied by internally [You can check it by pressing alt + enter]
        ContextCompat.startForegroundService(this, serviceIntent);
    }

    public void stopService(View v) {
        Intent serviceIntent = new Intent(this, ExampleService.class);
        stopService(serviceIntent);
    }
}
```

