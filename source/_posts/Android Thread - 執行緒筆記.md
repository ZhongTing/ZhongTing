---
title: Android Thread - 執行緒筆記
date: 2013-08-29 21:43:00
update: 2017-04-27 19:47:23
category: android
tags: thread
---
這裡會介紹 Thread, Runnable, Handler, AsyncTask等相關的android thread class。
<!-- more -->
android 程式一開啟的時候，系統會創造並執行一個main thread。所有UI顯示或者widget的event都在這個thread完成，所以又名 UI thread。有時候我們需要進行一些比較花時間的運算，為了避免UI thread的阻塞，我們必須另外使用另外一個thread來處理(worker thread)。而使用thread的方法很簡單，只需要定義run()方法再叫用run()函式即可。
```java
    new Thread(){
        @Override
        public void run() {
            // do something
        }
    }.run();
```
為了閱讀性及彈性，Android API有另外提供一個 [Runnable](https://developer.android.com/reference/java/lang/Runnable.html) 的介面。Runnable 裡面定義了 run() function，我們僅需要把Runnable 物件丟給thread執行就可以了。就像是把工作(Runnable)交給工人(worker thread)一樣。

```java
    Runnable task = new Runnable() {
        public void run() {
            // do some task here
        }
    };
    new Thread(task).start();
```
如果task不需要重複利用，也可以不要存成變數而直接在new thread時再宣告(如下)。其實跟第一個方法很像，只是我們run定義在runable interface。
```java
    new Thread(new Runnable() {
        public void run() {
            // do something
        }
    }).start();
```
Thread使用上滿簡單的，不過還是有一些要注意的地方。[Android API Guide](https://developer.android.com/guide/components/processes-and-threads.html#Threads) 提到Thread有兩個原則要遵守：
1. Do not block the UI thread 
2. Do not access the Android UI toolkit from outside the UI thread 

所以在worker thread 千萬不能直接使用UI的元件，否則會造成run-time exception error 或者其他未知的問題。雖然不能直接存取UI，但是可以使用間接的方式來存取。thread之間彼此溝通方式有很多，其中一種就是message。Android提供一個 [Handler](https://developer.android.com/reference/android/os/Handler.html)類別來充當thread間的溝通介面，在這裡舉的例子是 worker thread 與 UI thread(activity) 之間的溝通。

Handler 的 功用主要有兩個：
1. to enqueue an action to be performed on a different thread than your own.
2. to schedule messages and runnables to be executed as some point in the future

第一個功能是藉由 message queue 來處理其他 thread 的訊息
```java
    final private int MESSAGE_SET_TITLE = 1;

    private Handler handler = new Handler() {
        public void handleMessage(android.os.Message msg) {
            switch (msg.what) {
                case MESSAGE_SET_TITLE:
                    somebutton.setTitle("sometitle");
                    break;
            }
        }
    };

    public void onClick(View v) {
        new Thread() {
            public void run() {
                handler.sendEmptyMessage(MESSAGE_SET_TITLE);
            }
        }.start();
    }
```
另外一個功能則是排程工作或訊息
```java
 private Handler handler = new Handler();
    //or handler = new Handler(anotherThread.getLooper());

    public void onClick(View v) {
        handler.post(new Runnable() {
            public void run() {
                //do some task here
            }
        });
    }
```
除了post以及sendEmptyMessage之外，Handler另外也提供了一些函式可以用在訊息排程：
- [postAtTime(Runnable, long)](https://developer.android.com/reference/android/os/Handler.html#postAtTime(java.lang.Runnable,%20long%29)
- [postDelayed(Runnable, long)](https://developer.android.com/reference/android/os/Handler.html#postDelayed(java.lang.Runnable,%20long%29)
- [sendMessage(Message)](https://developer.android.com/reference/android/os/Handler.html#sendMessage(android.os.Message%29)
- [sendMessageAtTime(Message, long)](https://developer.android.com/reference/android/os/Handler.html#sendMessageAtTime(android.os.Message,%20long%29)
- [sendMessageDelayed(Message, long)](https://developer.android.com/reference/android/os/Handler.html#sendMessageDelayed(android.os.Message,%20long%29)

如果你是要直接使用UI Thread的元件，也可以直接透過下列的method操作：
- [Activity.runOnUiThread(Runnable)](https://developer.android.com/reference/android/app/Activity.html#runOnUiThread(java.lang.Runnable%29)
- [View.post(Runnable)](https://developer.android.com/reference/android/view/View.html#post(java.lang.Runnable%29)
- [View.postDelayed(Runnable, long)](https://developer.android.com/reference/android/view/View.html#postDelayed(java.lang.Runnable,%20long%29)

臨時性的操作不同thread可以利用上面的作法，不過大量使用程式碼會變得不容易閱讀。Android 這裡有提供一個 [AsyncTask Class](https://developer.android.com/reference/android/os/AsyncTask.html) 來簡化thread與handler之間的操作。你可以在文件裡面可以找到更詳細介紹以及使用方法。使用AsyncTask可以讓程式碼變得很簡潔：
```java
    new DownloadFilesTask().execute(url1, url2, url3);
```
當然AsyncTask還是有一些限制的：
1. The AsyncTask class must be loaded on the UI thread. This is done automatically as of JELLY_BEAN.
2. The task instance must be created on the UI thread. 
3. execute(Params...) must be invoked on the UI thread.
4. Do not call onPreExecute(), onPostExecute(Result), doInBackground(Params...), onProgressUpdate(Progress...) manually. 
5. The task can be executed only once (an exception will be thrown if a second execution is attempted.)

AsyncTask的設計適合用來作短時間且一次性的工作。如果想處理長時間的工作，文件建議我們使用 java.util.concurrent pacakge的類別，如[Executor](https://developer.android.com/reference/java/util/concurrent/Executor.html), [ThreadPoolExecutor](https://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html) 或是 [FutureTask](https://developer.android.com/reference/java/util/concurrent/FutureTask.html)。若想要處理重複有週期性的工作，則可以使用[Timer](https://developer.android.com/reference/java/util/Timer.html)與[TimerTask](https://developer.android.com/reference/java/util/TimerTask.html)。