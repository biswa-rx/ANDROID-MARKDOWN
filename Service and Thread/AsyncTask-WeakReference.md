# ASYNC TASK

*AsyncTask* `is a class in Android that provides an easy way to perform background operations and update the UI thread with the results. It is commonly used for short-lived tasks that don't require complex threading management. However, starting from Android API level 30 (Android 11), AsyncTask has been deprecated.`

```java
public class MyAsyncTask extends AsyncTask<Params, Progress, Result> {
    @Override
    protected void onPreExecute() {
        // Perform setup tasks before starting the background task
        // For example, show a progress dialog or update UI elements
    }
    // Override the doInBackground method to perform the background task
    @Override
    protected Result doInBackground(Params... params) {
        // Perform background tasks here
        // This method runs on a separate worker thread
        // You can publish progress using publishProgress() and update UI in onProgressUpdate()
        // The result of the background task is returned and passed to onPostExecute()
        return result;
    }

    // Optional: Override onProgressUpdate to update UI with progress
    @Override
    protected void onProgressUpdate(Progress... progress) {
        // Update UI with progress information
    }

    // Override onPostExecute to handle the result returned from doInBackground
    @Override
    protected void onPostExecute(Result result) {
        // Update UI or perform other actions with the result
    }
}
```
`To Start this Async Task we to exicute it`

```java
MyAsyncTask myAsyncTask = new MyAsyncTask();
myAsyncTask.execute(params);
```

`Now we have one problem here, It show a warning that "This Async Task class should be static or leaks might occur". The problem is that when we set this class as notstatic inner class then it has a implicit reference to the outer class which is out main activity. Therefore we can access Activity variable and function inside this class because we have a reference to our main activity. And the problem is here that when out activity is destroyed(Ex. we click out back button or rotate our screen) in that case out activity is a garbage and normal cases it removed form out memory but here async task could leave longer that out main activity. Since we have reference to our main activity it can't be garbage collected. So even we can use our main activity any more it still have in out memory, this is called as memory leaks. There are different ways to handle memory leaks.`

1. Some say AsyncTask do not leave longer that few mili-seconds so memory leaks should not be a problem.
2. Make async class a separate top level class
3. Make it a static inner class (In this we can remove warning but we can directly access our UI variable, Because we have no reference which cause out memory leak)
4. Create a interface that communicates our background to main activity
5. Use WeakReference
\
&nbsp;
>Example of using WeakReference

```java
public class MainActivity extends AppCompatActivity {
    private ProgressBar progressBar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        progressBar = findViewById(R.id.progress_bar);
    }

    public void startAsyncTask(View v) {
        ExampleAsyncTask task = new ExampleAsyncTask(this);
        task.execute(10);
    }

    private static class ExampleAsyncTask extends AsyncTask<Integer, Integer, String> {
        private WeakReference<MainActivity> activityWeakReference;

        ExampleAsyncTask(MainActivity activity) {
            activityWeakReference = new WeakReference<MainActivity>(activity);
        }

        @Override
        protected void onPreExecute() {
            super.onPreExecute();

            MainActivity activity = activityWeakReference.get(); // Converting weak references to strong references
            if (activity == null || activity.isFinishing()) {
                return;
            }

            activity.progressBar.setVisibility(View.VISIBLE);
        }

        @Override
        protected String doInBackground(Integer... integers) {
            for (int i = 0; i < integers[0]; i++) {
                publishProgress((i * 100) / integers[0]);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            return "Finished!";
        }

        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);

            MainActivity activity = activityWeakReference.get(); // Converting weak references to strong references
            if (activity == null || activity.isFinishing()) {
                return;
            }

            activity.progressBar.setProgress(values[0]);
        }

        @Override
        protected void onPostExecute(String s) {
            super.onPostExecute(s);

            MainActivity activity = activityWeakReference.get(); // Converting weak references to strong references
            if (activity == null || activity.isFinishing()) {
                return;
            }

            Toast.makeText(activity, s, Toast.LENGTH_SHORT).show();
            activity.progressBar.setProgress(0);
            activity.progressBar.setVisibility(View.INVISIBLE);
        }
    }
}
```