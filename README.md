# joya
Api based search engine Android App
این نسخه یک پروژه اندروید با Jetpack Compose است (پیشرفته، با استفاده از Bing Web Search API)، ظاهر مدرن الهام‌گرفته از رنگ‌های گوگل و با افکت Blur واقعی مبتنی بر RenderEffect.

اسکریپت Bash ساخت پروژه Compose با Blur (قابل اجرا در اوبونتو)
مراحل راه‌اندازی مشابه قبل:

نام فایل مثلاً: create_joya_compose.sh
سپس اجرا:
sh

chmod +x create_joya_compose.sh
./create_joya_compose.sh
سپس پروژه را در Android Studio باز کن و API Key را جایگزین کن.

bash

#!/bin/bash

PROJECT_NAME="JoyaCompose"
PACKAGE_NAME="ir.joya.compose"
API_KEY_PLACEHOLDER="YOUR_BING_API_KEY_HERE"

mkdir -p $PROJECT_NAME/app/src/main/java/ir/joya/compose
mkdir -p $PROJECT_NAME/app/src/main/res/values

cat > $PROJECT_NAME/build.gradle <<EOF
buildscript {
    ext {
        compose_ui_version = '1.4.3'
        kotlin_version = "1.8.22"
    }
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.4.2'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:\$kotlin_version"
    }
}
allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
EOF

cat > $PROJECT_NAME/settings.gradle <<EOF
rootProject.name = '$PROJECT_NAME'
include ':app'
EOF

cat > $PROJECT_NAME/app/build.gradle <<EOF
plugins {
    id 'com.android.application'
    id 'kotlin-android'
}

android {
    namespace '$PACKAGE_NAME'
    compileSdk 33

    defaultConfig {
        applicationId "$PACKAGE_NAME"
        minSdk 27
        targetSdk 33
        versionCode 1
        versionName "1.0"
    }

    buildFeatures {
        compose true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = '1.4.3'
    }

    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib"
    implementation "androidx.activity:activity-compose:1.7.0"
    implementation "androidx.compose.ui:ui:1.4.3"
    implementation "androidx.compose.material:material:1.4.3"
    implementation "androidx.compose.ui:ui-tooling-preview:1.4.3"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.6.1"
    implementation "androidx.lifecycle:lifecycle-viewmodel-compose:2.6.1"
    implementation "com.squareup.okhttp3:okhttp:4.11.0"
    implementation "androidx.core:core-ktx:1.10.1"
    implementation "androidx.compose.foundation:foundation:1.4.3"
    implementation "androidx.compose.ui:ui-graphics:1.4.3"
}
EOF

cat > $PROJECT_NAME/app/src/main/AndroidManifest.xml <<EOF
<manifest package="$PACKAGE_NAME"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <application
        android:name=".JoyaApp"
        android:label="جویا"
        android:theme="@style/Theme.Joya">
        <activity android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
EOF

cat > $PROJECT_NAME/app/src/main/res/values/colors.xml <<EOF
<resources>
    <color name="google_red">#EA4335</color>
    <color name="google_yellow">#FBBC05</color>
    <color name="google_green">#34A853</color>
    <color name="google_blue">#4285F4</color>
    <color name="background_blur">#D9FFFFFF</color> <!-- opacity: 85% -->
    <color name="black">#000000</color>
    <color name="white">#FFFFFF</color>
</resources>
EOF

cat > $PROJECT_NAME/app/src/main/res/values/styles.xml <<EOF
<resources>
    <style name="Theme.Joya" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <item name="android:windowBackground">@color/white</item>
    </style>
</resources>
EOF

cat > $PROJECT_NAME/app/src/main/java/ir/joya/compose/JoyaApp.kt <<EOF
package ir.joya.compose

import android.app.Application

class JoyaApp : Application()
EOF

cat > $PROJECT_NAME/app/src/main/java/ir/joya/compose/MainActivity.kt <<EOF
package ir.joya.compose

import android.graphics.RenderEffect
import android.graphics.Shader
import android.os.Build
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.annotation.RequiresApi
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.text.KeyboardActions
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material.*
import androidx.compose.material.MaterialTheme.typography
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.drawWithContent
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.asAndroidBitmap
import androidx.compose.ui.graphics.graphicsLayer
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalView
import androidx.compose.ui.text.input.ImeAction
import androidx.compose.ui.unit.dp
import androidx.core.content.ContextCompat.startActivity
import androidx.lifecycle.viewmodel.compose.viewModel
import android.content.Intent
import android.net.Uri
import com.google.accompanist.systemuicontroller.rememberSystemUiController
import kotlinx.coroutines.launch

@OptIn(ExperimentalMaterialApi::class)
class MainActivity : ComponentActivity() {
    @RequiresApi(Build.VERSION_CODES.S)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            JoyaAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = Color.White
                ) {
                    val viewModel: SearchViewModel = viewModel()
                    val uiState = viewModel.uiState.collectAsState().value

                    // افکت Blur واقعی با RenderEffect
                    val blurModifier =
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) Modifier.graphicsLayer {
                            renderEffect = RenderEffect.createBlurEffect(
                                40f, 40f, Shader.TileMode.CLAMP
                            )
                        } else Modifier.background(Color(0xD9FFFFFF))

                    Box(
                        modifier = Modifier.fillMaxSize().then(blurModifier)
                    ) {
                        Column(
                            modifier = Modifier
                                .fillMaxSize()
                                .padding(top = 64.dp),
                            horizontalAlignment = Alignment.CenterHorizontally
                        ) {
                            Row(
                                verticalAlignment = Alignment.CenterVertically
                            ) {
                                Text(
                                    text = "ج",
                                    style = typography.h2,
                                    color = Color(0xFF4285F4),
                                )
                                Text(
                                    text = "و",
                                    style = typography.h2,
                                    color = Color(0xFF34A853),
                                )
                                Text(
                                    text = "ی",
                                    style = typography.h2,
                                    color = Color(0xFFFBBC05),
                                )
                                Text(
                                    text = "ا",
                                    style = typography.h2,
                                    color = Color(0xFFEA4335),
                                )
                            }
                            Spacer(Modifier.height(24.dp))
                            var text by remember { mutableStateOf("") }
                            OutlinedTextField(
                                value = text,
                                onValueChange = { text = it },
                                label = { Text("عبارت خود را وارد کنید...") },
                                modifier = Modifier
                                    .fillMaxWidth(0.85f),
                                singleLine = true,
                                keyboardOptions = KeyboardOptions(imeAction = ImeAction.Search),
                                keyboardActions = KeyboardActions(
                                    onSearch = {
                                        viewModel.search(text.trim())
                                    }
                                )
                            )
                            Spacer(Modifier.height(10.dp))
                            Button(
                                onClick = { viewModel.search(text.trim()) },
                                colors = ButtonDefaults.buttonColors(
                                    backgroundColor = Color(0xFF4285F4)
                                )
                            ) {
                                Text("جستجو", color = Color(0xFFFFF176))
                            }
                            Spacer(Modifier.height(16.dp))
                            when {
                                uiState.loading -> {
                                    CircularProgressIndicator(color = Color(0xFF34A853))
                                }
                                uiState.results.isNotEmpty() -> {
                                    Column(
                                        verticalArrangement = Arrangement.spacedBy(8.dp),
                                        modifier = Modifier
                                            .fillMaxWidth(0.9f)
                                            .weight(1f)
                                    ) {
                                        uiState.results.forEach { item ->
                                            Card(
                                                elevation = 6.dp,
                                                backgroundColor = Color(0xD9FFFFFF),
                                                modifier = Modifier
                                                    .fillMaxWidth()
                                                    .clickable {
                                                        val intent = Intent(Intent.ACTION_VIEW, Uri.parse(item.url))
                                                        startActivity(LocalContext.current, intent, null)
                                                    }
                                            ) {
                                                Column(Modifier.padding(12.dp)) {
                                                    Text(item.title, color = Color(0xFFEA4335), style = typography.subtitle1)
                                                    Text(item.url, color = Color(0xFF4285F4), style = typography.body2)
                                                }
                                            }
                                        }
                                    }
                                }
                                uiState.error.isNotEmpty() -> {
                                    Text(uiState.error, color = Color.Red)
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
EOF

cat > $PROJECT_NAME/app/src/main/java/ir/joya/compose/SearchViewModel.kt <<EOF
package ir.joya.compose

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch
import okhttp3.OkHttpClient
import okhttp3.Request
import org.json.JSONObject

data class SearchResult(val title: String, val url: String)
data class UiState(
    val loading: Boolean = false,
    val results: List<SearchResult> = emptyList(),
    val error: String = ""
)

class SearchViewModel: ViewModel() {
    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> get() = _uiState
    private val apiKey = "$API_KEY_PLACEHOLDER"

    fun search(query: String) {
        if (query.isBlank()) return
        _uiState.value = UiState(loading = true)
        viewModelScope.launch {
            try {
                val url = "https://api.bing.microsoft.com/v7.0/search?q=\$query"
                val client = OkHttpClient()
                val request = Request.Builder()
                    .url(url)
                    .addHeader("Ocp-Apim-Subscription-Key", apiKey)
                    .build()
                val response = client.newCall(request).execute()
                if (!response.isSuccessful) throw Exception("HTTP Error: \${response.code}")
                val json = JSONObject(response.body!!.string())
                val list = mutableListOf<SearchResult>()
                val webPages = json.optJSONObject("webPages")
                val value = webPages?.optJSONArray("value")
                if (value != null) {
                    for (i in 0 until value.length()) {
                        val obj = value.getJSONObject(i)
                        list.add(SearchResult(obj.getString("name"), obj.getString("url")))
                    }
                }
                if (list.isEmpty()) list.add(SearchResult("نتیجه‌ای پیدا نشد!", ""))
                _uiState.value = UiState(results = list)
            } catch (e: Exception) {
                _uiState.value = UiState(error = e.localizedMessage ?: "خطا!")
            }
        }
    }
}
EOF

cat > $PROJECT_NAME/app/src/main/java/ir/joya/compose/JoyaAppTheme.kt <<EOF
package ir.joya.compose

import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material.MaterialTheme
import androidx.compose.material.darkColors
import androidx.compose.material.lightColors
import androidx.compose.runtime.Composable
import androidx.compose.ui.graphics.Color

private val LightColors = lightColors(
    primary = Color(0xFF4285F4),
    primaryVariant = Color(0xFF34A853),
    secondary = Color(0xFFEA4335),
    background = Color.White,
    surface = Color(0xFFF4F4F4),
    onPrimary = Color.White,
    onSecondary = Color.White
)

@Composable
fun JoyaAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    MaterialTheme(
        colors = LightColors,
        typography = androidx.compose.material.Typography(),
        shapes = androidx.compose.material.Shapes(),
        content = content
    )
}
EOF

echo "پروژه Compose '$PROJECT_NAME' ساخته شد!"
echo "ساختار شاخه‌ها:"
echo "
$PROJECT_NAME/
├── app/
│   ├── build.gradle
│   └── src/
│       └── main/
│           ├── AndroidManifest.xml
│           ├── java/
│           │   └── ir/
│           │       └── joya/
│           │           └── compose/
│           │               ├── MainActivity.kt
│           │               ├── JoyaApp.kt
│           │               ├── SearchViewModel.kt
│           │               └── JoyaAppTheme.kt
│           └── res/
│               └── values/
│                   ├── colors.xml
│                   └── styles.xml
├── build.gradle
└── settings.gradle
"
echo -e "
نکته: مقدار 'YOUR_BING_API_KEY_HERE' را در فایل‌های MainActivity.kt و SearchViewModel.kt با Bing API Key خودت جایگزین کن.
پروژه را در Android Studio باز کن، کامپایل کن و اجرا بگیر!
(حداقل اندروید ۱۲ برای blur واقعی لازم است. روی نسخه پایین‌تر بک‌گراند نیمه‌شفاف می‌گیرد)
"
نکات ویژه این پروژه
افکت Blur با RenderEffect فقط روی اندروید ۱۲ (SDK 31، Build.VERSION_CODES.S) به بالا امکان‌پذیر است.
تم و ترکیب رنگ‌ها الهام‌گرفته از رنگ‌های گوگل، با لوگوی شخصی‌سازی‌شده (متن "جویا" با رنگ هر حرف).
نمایش نتایج جستجو و کلیک‌پذیری هر کارت برای باز کردن لینک در مرورگر.
ساختار کاملاً Jetpack Compose و مدرن؛ مناسب یادگیری.
