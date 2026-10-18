# AppMetricaSDKGodotAndroid  
AppMetrica Android SDK plugin for Godot 4 and AppMetrica 8  

Contains essential functions: **init()**, **reportEvent()**, **reportAppOpen()**, **reportError()**, and an extended version for events with parameters: **reportEventWithParams()**.  

---

### 📁 Plugin Structure  
- Uses **file system for v1 plugins** in folders: `android/plugins` and `gdap file`.  
- If using [**v2 Godot plugin format**](https://docs.godotengine.org/en/4.4/tutorials/platform/android/android_plugin.html#packaging-a-v2-android-plugin), metrics dependencies may fail to load (likely a Yandex issue).  

**Important**: The `gdap` file name must match the plugin name returned by `getPluginName()` function.  

---

### 📚 Setup Guide  
For configuring your Godot project, refer to:  
- [Godot Android Deployment Docs](https://docs.godotengine.org/en/stable/tutorials/xr/deploying_to_android.html#deploying-to-android)  
- [Detailed Plugin Setup Tutorial (Russian)](https://kovardin.ru/articles/godot/mytacker-and-app-metrica/#%D0%BF%D0%BB%D0%B0%D0%B3%D0%B8%D0%BD-appmetrica) *(for Godot 4.2)*  

---

### 🧾 Example Usage  
```gdscript
var metrica: Object
func _ready():
	if Engine.has_singleton("AppMetricaSDK"):
		metrica = Engine.get_singleton("AppMetricaSDK")

		metrica.init(apiKey)
		metrica.reportAppOpen()
		metrica.reportEvent("Initialized")
		metrica.reportEventWithParams("TestEvent", 1, 2, 3)
		print("AppMetrica is on line")
	else:
		metrica.reportError("Initializing error", "AppMetrica isnt online")
		printerr("AppMetrica isnt online")
```

---

### 📦 Plugin Code Preview  
```kotlin
package com.peerless183.appmetricasdk

import io.appmetrica.analytics.AppMetrica
import android.content.Context
import io.appmetrica.analytics.AppMetricaConfig
import org.godotengine.godot.Godot
import org.godotengine.godot.plugin.GodotPlugin
import org.godotengine.godot.plugin.UsedByGodot

class AppMetricaGodotPlugin(godot: Godot?) : GodotPlugin(godot) {
    private var isInitialized = false
    lateinit var config: AppMetricaConfig.Builder

    override fun getPluginName(): String {
        return "AppMetricaSDK"
    }

    @UsedByGodot
    fun init(apiKey: String) {
        val currentActivity = activity ?: return
        if (godot == null) return

        config = AppMetricaConfig.newConfigBuilder(apiKey).withSessionsAutoTrackingEnabled(true)
            .withLocationTracking(false)

        AppMetrica.activate(currentActivity.applicationContext, config.build())
        AppMetrica.setAdvIdentifiersTracking(true)
        AppMetrica.resumeSession(currentActivity)
        isInitialized = true
    }

    @UsedByGodot
    fun reportAppOpen() {
        if (!isInitialized) return

        activity?.let {AppMetrica.reportAppOpen(it)}
    }

    @UsedByGodot
    fun reportEvent(name: String) {
        if (!isInitialized) return

        AppMetrica.reportEvent(name)
    }

    @UsedByGodot
    fun reportError(name: String, message: String) {
        if (!isInitialized) return

        val exception = Exception(message)
        AppMetrica.reportError(name, message, exception)
    }

    @UsedByGodot
    fun reportEventWithParams(name: String, par1: Any, par2: Any, par3: Any) {
        if (!isInitialized) return

        val params = mapOf<String, Any>(
            "par1" to par1,
            "par2" to par2,
            "par3" to par3
        )
        AppMetrica.reportEvent(name, params)
    }
}
```

---

### 📌 Notes  
1. **reportEvent** and **reportEventWithParams** are **template functions** demonstrating how to create custom event-logging methods.  
2. I do not recommend using AppMetrica as a metrics service for production. Consider using [GameAnalytics](https://www.gameanalytics.com/) for better reliability and features.
