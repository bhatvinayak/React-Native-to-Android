# 🧱 PART 1: Android App Fundamentals (with React Native Comparisons)

## 1. Activity — Like a React Native “Screen”

An Activity is a single screen in an Android app.

In React Native, each “screen” in navigation (like HomeScreen, ProfileScreen) = one Activity or Fragment.

Each Activity has its own lifecycle (similar to React component lifecycle).

```
// MainActivity.kt
package com.example.myfirstapp

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // set the layout for this activity
        setContentView(R.layout.activity_main)
    }
}
```

> Here R.layout.activity_main is like your JSX layout file — it defines what UI to show.


## 2. Intent — Like React Navigation’s navigate()

An Intent is how you move between Activities (or start background actions).
It’s similar to `navigation.navigate('ScreenName')` in React Native.

```
// Moving from MainActivity to SecondActivity
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("username", "Vinayak")
startActivity(intent)
```

In SecondActivity, you can get this data:

```
val name = intent.getStringExtra("username")
```

🧠 React Native analogy:
> navigation.navigate("SecondScreen", { username: "Vinayak" })

## 3. Service — Like a background worker / headless JS task

A Service runs in the background without a UI.
Think of it like background tasks in React Native (e.g. using react-native-background-fetch or Headless JS).

```
class MyService : Service() {
    override fun onBind(intent: Intent?): IBinder? = null

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        println("Service running in background")
        return START_STICKY
    }
}
```

To start:

> startService(Intent(this, MyService::class.java))

🧠 React Native analogy:
> Background task using AppRegistry.registerHeadlessTask().

## 4. Broadcast Receiver — Like event listeners

A BroadcastReceiver listens for system-wide or app-wide events (e.g. battery low, Wi-Fi state, incoming call).

```
class BatteryReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        val level = intent?.getIntExtra("level", 0)
        println("Battery Level: $level")
    }
}
```

Register it:

```
val filter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
registerReceiver(BatteryReceiver(), filter)
```

🧠 React Native analogy:
> DeviceEventEmitter.addListener('batteryLevelChange', callback).

## 5. Content Provider — Like shared data between apps

A Content Provider allows data sharing between apps or within the system (like accessing Contacts, Media, etc).

Think of it like a database API layer exposed to other apps.

```
🧩 Example: Read Contacts
val cursor = contentResolver.query(
    ContactsContract.Contacts.CONTENT_URI,
    null, null, null, null
)

while (cursor?.moveToNext() == true) {
    val name = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME))
    println("Contact Name: $name")
}
cursor?.close()
```

🧠 React Native analogy:
> Using react-native-contacts to access the same contact list — under the hood, that library uses a ContentProvider.

## 6. 🌀 Activity Lifecycle (Very Important)

Just like React has:

```
useEffect(() => {
  // mount
  return () => { /* unmount */ }
}, [])
```

Android Activity has lifecycle methods:


|  Method	   |   When called	                       |   React Native analogy   |
|------------|---------------------------------------|--------------------------|
|onCreate()  | When Activity is created	             |  useEffect(() => {}, []) |
|onStart()	 | When Activity becomes visible	       |   ComponentDidMount-ish  |
|onResume()  |When Activity comes to foreground	     |   Screen gains focus     |
|onPause()	 |When Activity goes partially invisible |	 Screen blur            |
|onStop()	   |When Activity is hidden	               |  componentWillUnmount-ish|
|onDestroy() |When Activity is killed	               |      componentWillUnmount|


### ⚙️ How Android App is Structured

📁 Typical Android Project:

```
app/
 ├── manifests/
 │    └── AndroidManifest.xml     ← App config (permissions, activities)
 ├── java/
 │    └── com.example.myapp/
 │         └── MainActivity.kt
 ├── res/
 │    ├── layout/                 ← XML UI layouts
 │    ├── values/                 ← strings.xml, colors.xml, etc.
 └── build.gradle                 ← Dependencies & build settings
```

🧠 React Native analogy:

> AndroidManifest.xml → like app.json

> res/layout/ → like JSX components

> MainActivity.kt → like your entry point in React Native (App.tsx)

🚀 Summary So Far


| Android Concept |	    RN Equivalent |	          Purpose |
|------------|---------------------------------------|--------------------------|
|Activity	|          Screen / Component	 |   UI container|
|Intent	     |       navigation.navigate()	|  Move between screens|
|Service	   |         Background Task	  |      Background operations|
|BroadcastReceiver|	  Event Listener	  |      React to system events|
|ContentProvider	|    External API / DB	|      Share data between apps|
