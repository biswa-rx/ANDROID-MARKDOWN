# JOB SCHEDULER

### Background Execution Limits
Whenever an app runs in the background, it consumes some of the device's limited resources, like RAM. This can result in an impaired user experience, especially if the user is using a resource-intensive app, such as playing a game or watching video. To improve the user experience, Android 8.0 (API level 26) imposes limitations on what apps can do while running in the background. [Official Documentation](https://developer.android.com/about/versions/oreo/background)

![Job Scheduler Service](https://user.oc-static.com/upload/2018/05/16/15264852994194_1516973769774_JobScheduler.png)

`JobScheduler is more inteligent alarm manager, Instead of scheduling out background tasks based on time it can define different criteria (Like if the device is charging or connected to wifi) and the system will deside when exactly out service will run our job.`

NOTE : _System will decide when our job will exicuted but we can control it little bit (Example deadline or minimum waiting time)_

`To impliment this we have to create a Job Service class and extends to JobService`

```java
public class ExampleJobService extends JobService {

    @Override
    public boolean onStartJob(JobParameters params) {
        //Our task goes here

        //Then it returns false to indicate that our job is completed successfully and device goes to sleep...

        return false; // For long running background operation we return true to keep device WAKE_LOCK so that our job is finished and it is our responsibility to cancel that job manually by calling jobFinished() with jobParams as parameter and second parameter for needsReschedule which is boolean.
    }

    @Override
    public boolean onStopJob(JobParameters params) {

        //This method is called when the job is cancelled by the system (e.g. we give requred wifi connection but user turn off the wifi connection)

        return false; //it returns boolean for you want to reschedule the job or not. If true then it will rescheduled.
    }
}
```

IMPORTANT NOTE: _This JobService run out task on UI thread and it is our responsibility to if out task is long running to do it in background thread._

## Demonstration of JobScheduler in a simple way

`ExampleJobService.java`

```java
public class ExampleJobService extends JobService {
    private static final String TAG = "ExampleJobService";
    private boolean jobCancelled = false;

    @Override
    public boolean onStartJob(JobParameters params) {
        Log.d(TAG, "Job started");
        doBackgroundWork(params);

        return true; //Return true for long running jobs
    }

    private void doBackgroundWork(final JobParameters params) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    Log.d(TAG, "run: " + i);
                    if (jobCancelled) {
                        return;
                    }

                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                Log.d(TAG, "Job finished");
                jobFinished(params, false);
            }
        }).start();
    }

    @Override
    public boolean onStopJob(JobParameters params) {
        Log.d(TAG, "Job cancelled before completion");
        jobCancelled = true;
        return true; // return true for reschedule it
    }
}
```
`MainActivity.java`
```java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private static final int jobId = 123;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    //scheduleJob Button
    public void scheduleJob(View v) {
        ComponentName componentName = new ComponentName(this, ExampleJobService.class);
        JobInfo info = new JobInfo.Builder(jobId, componentName)
                .setRequiresCharging(true)
                .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED)
                .setPersisted(true) // It keeps out job alive even if we reboot our device
                .setPeriodic(15 * 60 * 1000)
                .build();

        JobScheduler scheduler = (JobScheduler) getSystemService(JOB_SCHEDULER_SERVICE);
        int resultCode = scheduler.schedule(info); // This is intiger only for ensured that everything is successful
        if (resultCode == JobScheduler.RESULT_SUCCESS) {
            Log.d(TAG, "Job scheduled");
        } else {
            Log.d(TAG, "Job scheduling failed");
        }
    }

    //cancelJob Button
    public void cancelJob(View v) {
        JobScheduler scheduler = (JobScheduler) getSystemService(JOB_SCHEDULER_SERVICE);
        scheduler.cancel(jobId);
        Log.d(TAG, "Job cancelled");
    }
```

`For adding setPersisted(true) // It keeps out job alive even if we reboot our device we have to add RECEIVE_BOOT_COMPLETED permission`

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

`All over we have to resister out service call inside Manifest file`

```xml
<service android:name=".ExampleJobService"
    android:permission="android.permission.BIND_JOB_SERVICE" />
/>
```