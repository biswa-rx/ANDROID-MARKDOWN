# Looper MessageQueue Handler

![Looper-MessageQueue-Handler](https://i.ytimg.com/vi/TN-CGfzvBhc/maxresdefault.jpg)

`We know that once our java thread is terminated then we can't start it again, if we start it again by button click it give us a run time error. To solve this problem we introduced by looper and message queue where looper is continuously looping inside a thread and we have put runnable to execute that piece of code`

---

### To create looper we have to call looper.prepare() and looper.loop() inside thread run().

```java
import android.os.Handler;
import android.os.Looper;
import android.util.Log;

public class ExampleLooperThread extends Thread {
    private static final String TAG = "ExampleLooperThread";

    public Looper looper;
    public Handler handler;

    @Override
    public void run() {
        Looper.prepare();

        looper = Looper.myLooper();

        handler = new ExampleHandler();

        Looper.loop();

        Log.d(TAG, "End of run()");
    }
}
```

`Inside MainActivity we have to get handler or looper variable to create handler to post runnable in it.`

```java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";

    private ExampleLooperThread looperThread = new ExampleLooperThread();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void startThread(View view) {
        looperThread.start();
    }

    public void stopThread(View view) {
        looperThread.looper.quit();
    }

    public void taskA(View view) {
        Handler threadHandler = new Handler(looperThread.looper);
        threadHandler.post(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 5; i++) {
                    Log.d(TAG, "run: " + i);
                    SystemClock.sleep(1000);
                }
            }
        });
    }

    public void taskB(View view) {

    }
}
```

`But it causes memory leak due to we post task to background thread from main ui thread. Actually in this case runnable introduces a potential memory leak. A anonymous inner class is same as non static inner class and this non static inner class have  a reference to the outer class. So our runnable here has reference to out MainActivity. To prove we can access Activity variables or Activity method (Ex. startThread etc.) in this runnable class. This is called a implicit reference because we need activity variables in that.`

`As long as we have reference to that activity, the activity can't be garbage collected. Example in a situation where out activity got destroyed but out thread is still alive. So we are inside the looper.loop() , for example user rotate his device then activity go destroyed so out activity is just a garbage. But a system can only clean up a object if there is no reference to it. But here problem is out runnable is here has a implicit reference to out MainActivity so as long as it is running our activity can't be cleaned up. And this runnable and thread inner classes are very common for memory leaks.`

>One way to fix it to make this runnable in a static inner class
```java
static class MyRunnable extends Runnable {
}
```
`The differences here is since it is a static inner class it can't reference to the Activity so It means inside the runnable you can't access the Activity variables directly anymore.`

`If you want to access these variable then you can use weak references.`
*"A weakly referenced object is cleared by the Garbage Collector when it's weakly reachable."*

`If you start this looper thread and don't quit then it also cause of memory leak. To cancel this looper we have 2 methord below`

```java
looperThread.looper.quit(); // to sudden cancel the looper thread
looperThread.looper.quitSafe(); // to cancel after completion of rest of message queue
```
*"I we don't quit it and destroy out activity then we are end up with infinitely long running zombie therad which we can't not able to stop"*

---
`Another topic is here it post a Runnable and called as message queue, why it doesn't call a runnable queue.`

```java
threadHandler.post(new Runnable() {});
```
`Because this runnable queue actually post a message internally... To understand it let's have an look what other opetions we have.`
\
&nbsp;
\
&nbsp;

>Instead of posting a runnable we can also send a message directly to queue

`Example Looper Class`
```java
import android.os.Handler;
import android.os.Looper;
import android.util.Log;

public class ExampleLooperThread extends Thread {
    private static final String TAG = "ExampleLooperThread";

    public Looper looper;
    public Handler handler;

    @Override
    public void run() {
        Looper.prepare();

        looper = Looper.myLooper();

        handler = new ExampleHandler();

        Looper.loop();

        Log.d(TAG, "End of run()");
    }
}
```

*"The difference betweeen message and runnable is in runnable we send a piece of work ot exicute but the message contain data. But the default our handler doesn't know how to interpret this value. It know what to do with runnable just exicute it but it does not know what to with message. Ofcourse we have to tell it some way. So this we have to create out own handler class."*

`Custom Handler Class`
```java
import android.os.Handler;
import android.os.Message;
import android.util.Log;

public class ExampleHandler extends Handler {
    private static final String TAG = "ExampleHandler";

    public static final int TASK_A = 1;
    public static final int TASK_B = 2;

    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case TASK_A:
                Log.d(TAG, "Task A executed");
                break;
            case TASK_B:
                Log.d(TAG, "Task B executed");
                break;
        }
    }
}
```

`MainActivity`
```java
import android.os.Handler;
import android.os.Message;
import android.os.SystemClock;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;

import static com.codinginflow.looperthreadexample.ExampleHandler.TASK_A;
import static com.codinginflow.looperthreadexample.ExampleHandler.TASK_B;

public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";

    private ExampleLooperThread looperThread = new ExampleLooperThread();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void startThread(View view) {
        looperThread.start();
    }

    public void stopThread(View view) {
        looperThread.looper.quit();
    }

    public void taskA(View view) {
        Message msg = Message.obtain();
        msg.what = TASK_A;
        looperThread.handler.sendMessage(msg);
    }

    public void taskB(View view) {
        Message msg = Message.obtain();
        msg.what = TASK_B;
        looperThread.handler.sendMessage(msg);
    }
}
```

## IMPORTANT NOTE

`When we take another look at the sendMessage() which we can see it internally called sendMessageDelayed() with delay of zero miliseconds, which make sense because we didn't define a delay.`

```java
public final boolean sendMessage(Message msg)
{
    return sendMessageDelayed(msg, 0); // delayMillis: 0
}
```
`But when we take a look of post() method of handler, we can see that it also called sendMessageDelayed() with delayMillis of zero. But this time for the message it passed getPostMessage() where it forwards a runnable to it.`

```java
public final boolean post(Runnable r)
{
    return sendMessageDelayed(getPostMessage(r), 0);
}
```
`When we take a look at getPostMessage() method we can see it create a message object similarly how we did it before and it simply attach a runnable as a callback to this message. So both casses we send a message where we call sendMessage() or we call post(). The only difference is the message we send with the post has runnable attached to it.`

```java
private static Message getPostMessage(Runnable r)
{
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```
`Funny thing is when we take another look the looper class into dispatchMessage() method`

```java
try {
    msg.target.dispatchMessage(msg);
    ...
} finally {
    ...
}
```
`This is where out message will end up where it exicuted`
```java
/**
 * Subclassed must implement this to receive messages.
 * */
public void handleMessage(Message msg) {
}
/**
 * Handle system messages here.
 */
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if(mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```

`We can see it check if there is a callback attached to this Mesage object in this case it calls handleCallback(). And this simply exicutes the runnalbe`

```java
private static void handleCallback(Message message) {
    message.callback.run();
}
```

`If there is no callback attatched to the message only when we did not send a runnable then else blocks will be executed where it called handleMessage() which we defined in out custom Handler class. We have to @Override handleMesssage() in our Custom Handler class.`

---

