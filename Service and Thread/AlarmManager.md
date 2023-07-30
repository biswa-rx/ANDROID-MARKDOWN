# ALARM MANAGER
### Alarms (based on the AlarmManager class) give you a way to perform time-based operations outside the lifetime of your application. For example, you could use an alarm to initiate a long-running operation, such as starting a service once a day to download a weather forecast.

[Official Documentation](https://developer.android.com/training/scheduling/alarms, "Google")

`To set a notification from alarm manager we have to create a notification service inside a broadcast service`

```java
public class MyNotificationReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // Create and show the notification here
        showNotification(context, "Your Notification Title", "Your Notification Content");
    }

    private void showNotification(Context context, String title, String content) {
        // Create and show the notification using NotificationCompat.Builder
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, "channel_id")
                .setSmallIcon(R.drawable.ic_notification)
                .setContentTitle(title)
                .setContentText(content)
                .setPriority(NotificationCompat.PRIORITY_DEFAULT);

        NotificationManagerCompat notificationManager = NotificationManagerCompat.from(context);
        notificationManager.notify(1, builder.build());
    }
}
```

`To receive broadcast form system we have to add on manifest file`

```xml
<receiver android:name=".MyNotificationReceiver"/>
```



**To setup alarm manager**

```java
// Set the alarm time to 8:00 AM
Calendar calendar = Calendar.getInstance();
calendar.set(Calendar.HOUR_OF_DAY, 8);
calendar.set(Calendar.MINUTE, 0);
calendar.set(Calendar.SECOND, 0);

// Create a PendingIntent to launch the BroadcastReceiver
Intent intent = new Intent(context, MyNotificationReceiver.class);
PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, intent, 0);

// Get the AlarmManager and set the alarm
AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
if (alarmManager != null) {
    // Use setExact for precise timing (requires the alarm to be reset after each firing)
    // For daily repeating alarms, use setRepeating instead
    alarmManager.setExact(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pendingIntent);
}
```

**Set the alarm time to 5 minutes from now**

```java
long alarmTime = System.currentTimeMillis() + 5 * 60 * 1000;

// Create an Intent to launch a BroadcastReceiver
Intent intent = new Intent(context, MyAlarmReceiver.class);
PendingIntent pendingIntent = PendingIntent.getBroadcast(context, ALARM_REQUEST_CODE, intent, PendingIntent.FLAG_UPDATE_CURRENT);

// Get the AlarmManager and set the alarm
AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
if (alarmManager != null) {
    // Use setExact for precise timing (requires the alarm to be reset after each firing)
    // For inexact timing, use set instead
    alarmManager.setExact(AlarmManager.RTC_WAKEUP, alarmTime, pendingIntent);
}
```

`To cancel an alarm that was previously set using AlarmManager, you need to use the same PendingIntent that was used to schedule the alarm.`

```java
// Create a PendingIntent to launch the BroadcastReceiver
Intent intent = new Intent(context, MyNotificationReceiver.class);
PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, intent, 0);

// Get the AlarmManager
AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
if (alarmManager != null) {
    // Cancel the alarm using the same PendingIntent
    alarmManager.cancel(pendingIntent);
}
```

## IMPORTANT NOTE:
    `Keep the device awake while it's running, preventing the device from going into sleep mode. You have to require WAKE_LOCK permission. When your app acquires a wake lock, it ensures that the device's screen and CPU stay on, even if the device's screen timeout is reached or the device tries to enter a low-power state. This can be useful when you want to perform background tasks that require the device to be awake, such as playing media, downloading data, or scheduling alarms.`

```xml
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

`Set the alarm to start after 5 seconds and repeat every 30 seconds`

```java
    long initialDelay = System.currentTimeMillis() + 5000; // 5 seconds from now
    long repeatInterval = 30000; // 30 seconds

    // Set the repeating alarm
    alarmManager.setRepeating(AlarmManager.RTC_WAKEUP, initialDelay, repeatInterval, pendingIntent);
```


    

