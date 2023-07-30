# RxJava-RxAndroid


---

>Dependency of RxJava and RxAndroid

* Click here for dependency of [RxJava](https://github.com/ReactiveX/RxJava, "RxJava Dependency")

* Click here for dependency of [RxAndroid](https://github.com/ReactiveX/rxandroid, "RxAndroid Dependency")

* Click here for my repository [RxJava-RxAndroid](https://github.com/biswa-rx/RxJava-RxAndroid, "biswa-rx")


## Topic Explained

* Create Operators
  * Create
  * Just
  * Range
  * Repeat
  * Interval
  * Timer
  * fromArray
  * fromIterable
  * from Callable
  * [fromFuture](https://github.com/biswa-rx/Retrofit-RxJava)
  * [fromPublisher](https://github.com/biswa-rx/Retrofit-RxJava/tree/fromPublisher)
* Filter Operators
  * Filter
  * Distinct
  * Take
  * TakeWhile
* Transformation Operators
  * Map
  * Buffer
  * Debounce
  * ThrottleFirst
  * FlatMap
  * ConcatMap
  * SwitchMap

---
## Detailed Transformation Operators

1. Map Operator\
   `Applies a function to each emitted item. It transforms each emitted item by applying a function to it.`
   ![Map Operator](https://codingwithmitch.s3.amazonaws.com/static/courses/lectures/286/map.png)
   ```java
   Observable<String> extractDescriptionObservable = Observable
                .fromIterable(DataSource.createTasksList())
                .subscribeOn(Schedulers.io())
                .map(new Function<Task, String>() {
                    @Override
                    public String apply(Task task) throws Throwable {
                        Log.d(TAG, "apply: doing work on thread: " + Thread.currentThread().getName());
                        return task.getDescription();
                    }
                })
                .observeOn(AndroidSchedulers.mainThread());

        extractDescriptionObservable.subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
            }
            @Override
            public void onNext(String s) {
                Log.d(TAG, "onNext: extracted description: " + s);
            }
            @Override
            public void onError(Throwable e) {
            }
            @Override
            public void onComplete() {
            }
        });
   ```

2. Buffer Operator\
   `Periodically gather items from an Observable into bundles and emit the bundles rather than emitting items one at a time.`
   ![Buffer Operator](https://codingwithmitch.s3.amazonaws.com/static/courses/lectures/287/Buffer.png)
   ```java
    Observable<Task> taskObservable = Observable
                .fromIterable(DataSource.createTasksList())
                .subscribeOn(Schedulers.io());

        taskObservable
                .buffer(2) // Apply the Buffer() operator
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<List<Task>>() { // Subscribe and view the emitted results
                    @Override
                    public void onSubscribe(Disposable d) {
                    }
                    @Override
                    public void onNext(List<Task> tasks) {
                        Log.d(TAG, "onNext: bundle results: -------------------");
                        for(Task task: tasks){
                            Log.d(TAG, "onNext: " + task.getDescription());
                        }
                    }

                    @Override
                    public void onError(Throwable e) {
                    }
                    @Override
                    public void onComplete() {
                    }
                });
   ```
   
3. Debounce Operator\
   `The Debounce operator filters out items emitted by the source Observable that are rapidly followed by another emitted item.`
   ![Debounce Operator](https://codingwithmitch.s3.amazonaws.com/static/courses/lectures/289/debounce.png)
   ```java
   Observable<String> observableQueryText = Observable
                .create(new ObservableOnSubscribe<String>() {
                    @Override
                    public void subscribe(final ObservableEmitter<String> emitter) throws Exception {

                        // Listen for text input into the SearchView
                        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
                            @Override
                            public boolean onQueryTextSubmit(String query) {
                                return false;
                            }

                            @Override
                            public boolean onQueryTextChange(final String newText) {
                                if(!emitter.isDisposed()){
                                    emitter.onNext(newText); // Pass the query to the emitter
                                }
                                return false;
                            }
                        });
                    }
                })
                .debounce(500, TimeUnit.MILLISECONDS) // Apply Debounce() operator to limit requests
                .subscribeOn(Schedulers.io());

        // Subscribe an Observer
        observableQueryText.subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                disposables.add(d);
            }
            @Override
            public void onNext(String s) {
                Log.d(TAG, "onNext: time  since last request: " + (System.currentTimeMillis() - timeSinceLastRequest));
                Log.d(TAG, "onNext: search query: " + s);
                timeSinceLastRequest = System.currentTimeMillis();

                // method for sending a request to the server
                sendRequestToServer(s);
            }
            @Override
            public void onError(Throwable e) {
            }
            @Override
            public void onComplete() {
            }
        });
   ```

4. ThrottleFirst Operator\
   `The ThrottleFirst() operator filters out items emitted by the source Observable that are within a timespan.`
   ![ThrottleFirst Operator](https://codingwithmitch.s3.amazonaws.com/static/courses/lectures/290/throttleFirst.png)
   ```java
   RxView.clicks(findViewById(R.id.button))
                .throttleFirst(500, TimeUnit.MILLISECONDS) // Throttle the clicks so 500 ms must pass before registering a new click
                .observeOn(io.reactivex.android.schedulers.AndroidSchedulers.mainThread())
                .subscribe(new io.reactivex.Observer<Unit>() {
                    @Override
                    public void onSubscribe(io.reactivex.disposables.Disposable d) {
                        disposable2.add(d);
                    }
                    @Override
                    public void onNext(Unit unit) {
                        Log.d(TAG, "onNext: time since last clicked: " + (System.currentTimeMillis() - timeSinceLastRequest));
                        timeSinceLastRequest = System.currentTimeMillis();
                        sendRequestToServer("Some Request"); // Execute some method when a click is registered
                    }
                    @Override
                    public void onError(Throwable e) {
                    }
                    @Override
                    public void onComplete() {
                    }
                });
   ```

5. FlatMap Operator\
   `Transform the item(s) emitted by an Observable into Observables, then flatten the emissions from those into a single Observable. If you're familiar with LiveData, MediatorLiveData can do something very similar.`
   ![FlatMap Operator](https://codingwithmitch.s3.amazonaws.com/static/courses/lectures/291/flatmap.png)

   *Order is not maintained*

6. ConcatMap Operator\
   `Transform the item(s) emitted by an Observable into Observables, then flatten the emissions from those into a single Observable. This operator is essentially the same as the Flatmap operator, but it emits the object(s) while maintaining order.`
   ![ConcatMap Operator](https://codingwithmitch.s3.amazonaws.com/static/courses/lectures/293/concatmap.png)

   *Order is maintained*

7. SwitchMap Operator\
   `SwitchMap() will transform items emitted by an Observable into an Observable just like ConcatMap() and FlatMap(). The difference being that it will unsubscribe previous observers once a new Observer has subscribed. Essentially this solves a limitation that both ConcatMap() and FlatMap() have.`
   ![SwitchMap Operator](https://codingwithmitch.s3.amazonaws.com/static/courses/lectures/293/concatmap.png)

   *Order is maintained*
   
  