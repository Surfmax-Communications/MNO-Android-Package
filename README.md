# Surfmax Communications Liveness SDK — Android

Native Android SDK for **Liveness Detection** and **Facial Comparison**. Supports both front and rear camera modes, passive and active liveness types, and optional device metadata collection.

Get your credentials from [Surfmax](https://surfmaxcomm.com/).

---

## Features

### Liveness Detection Only (`SurfmaxLivenessDetectionOnlyBuilder`)

Verifies that the person in front of the camera is a live human being. Three detection types:

| Type | Description |
|------|-------------|
| `ACTIVE` | User performs a prompted action (e.g. open mouth, turn head). Strictest. |
| `PASSIVE` | User holds still within the frame. Less strict, faster. |
| `FLASH` | Increases screen brightness to enhance facial point detection. Front camera only. |

### Liveness with Facial Comparison (`SurfmaxLivenessFacialComparisonBuilder`)

Runs liveness detection and then compares the captured face against a provided reference image. Two comparison types:

| Type | Description |
|------|-------------|
| `ACTIVE` | Dynamic liveness (user performs an action) + face match. |
| `PASSIVE` | Static liveness (user holds still) + face match. |

---

## Installation

### Step 1 — GitHub credentials

Create a `github.properties` file in the root of your Android project (do **not** commit this file):

```properties
USERNAME_GITHUB=YourGitHubUsername
TOKEN_GITHUB=YourClassicPersonalAccessToken
```

The token needs at minimum: `read:packages`.

### Step 2 — Project-level `build.gradle`

```gradle
buildscript {
    ext.kotlin_version = '1.9.+'

    dependencies {
        classpath 'com.android.tools.build:gradle:8.9.+'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

allprojects {
    def githubPropertiesFile = rootProject.file("github.properties")
    def githubProperties = new Properties()
    githubProperties.load(new FileInputStream(githubPropertiesFile))

    repositories {
        google()
        mavenCentral()
        maven {
            name "GitHubPackages"
            url 'https://maven.pkg.github.com/Surfmax-Communications/MNO-Android-Package'
            credentials {
                username githubProperties['USERNAME_GITHUB']
                password githubProperties['TOKEN_GITHUB']
            }
        }
    }
}
```

### Step 3 — App-level `build.gradle`

Minimum SDK version is **24**.

```gradle
android {
    defaultConfig {
        minSdkVersion 24
        targetSdkVersion 36
    }
}

dependencies {
    implementation 'com.surfmaxcomm:liveness_android_data:1.4-1'
    implementation 'com.surfmaxcomm:liveness_android_core:1.4-1'
}
```

### Step 4 — AndroidManifest.xml

Add all required permissions and hardware features:

```xml
<!-- Camera hardware features -->
<uses-feature
    android:name="android.hardware.camera"
    android:required="false" />
<uses-feature
    android:name="android.hardware.camera.autofocus"
    android:required="false" />
<uses-feature
    android:glEsVersion="0x00020000"
    android:required="false" />

<!-- Camera: required for liveness capture (front and rear) -->
<uses-permission android:name="android.permission.CAMERA" />

<!-- Network: required for SDK API calls -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />

<!-- Audio: required for liveness voice prompts -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />

<!-- Storage: required for reading/writing captured images -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

<!-- Device metadata: required when setCollectDeviceId(true) is used -->
<uses-permission android:name="android.permission.READ_PHONE_STATE" />

<!-- Haptic feedback during liveness prompts -->
<uses-permission android:name="android.permission.VIBRATE" />

<!-- Location: required when setCollectDeviceLocation(true) is used -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

<application android:requestLegacyExternalStorage="true">
    ...
</application>
```

### Step 5 — ProGuard (if minification is enabled)

Add to `proguard-rules.pro`:

```proguard
-keep class com.megvii.** { *; }
-keep class com.face.** { *; }
-keep class com.zenith.** { *; }
-keep class com.neutralbase.** { *; }
-keep class com.surfmaxcomm.liveness_native.** { *; }
```

Enable in `build.gradle`:

```gradle
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

---

## Usage

### Liveness Detection Only

Verify that a live person is in front of the camera — no reference image needed.

```java
import com.surfmaxcomm.liveness_native.Core.SurfmaxLiveness;
import com.surfmaxcomm.liveness_native.Core.CameraType;
import com.surfmaxcomm.liveness_native.Core.LivenessCompletedCallBack;
import com.surfmaxcomm.liveness_native.Core.LivenessDetectionOnlyType;
import com.surfmaxcomm.liveness_native.Core.Model.DeviceMetadataConfig;
import com.surfmaxcomm.liveness_native.Core.Model.LivenessSuccessful.LivenessSuccess;

new SurfmaxLiveness.SurfmaxLivenessDetectionOnlyBuilder(activity)
        .setClientId("your-client-id")
        .setApiKey("your-api-key")
        .setAppName("your-app-name")
        .setLivenessType(LivenessDetectionOnlyType.ACTIVE)  // ACTIVE, PASSIVE, or FLASH
        .setCameraType(CameraType.FRONT)                    // FRONT or REAR
        .setIsDev(false)                                    // true for sandbox/dev environment
        .setShowLivenessResult(true)                        // show SDK result screen after check
        .setStartProcessOnGettingToFirstScreen(false)       // auto-start on screen load
        .setDeviceMetadataConfig(new DeviceMetadataConfig.Builder()
                .setCollectDeviceId(true)
                .setCollectDeviceLocation(true)
                .build())
        .setListener(new LivenessCompletedCallBack() {
            @Override
            public void onSuccess(LivenessSuccess response) {
                boolean passed = response.getIsProcedureValidationPassed();
                double confidence = response.getConfidencePercent();
                double threshold = response.getThresholdPercent();
                // handle success
            }

            @Override
            public void onFailure(int statusCode, String errorObject) {
                // handle failure
            }
        })
        .build();
```

#### `LivenessDetectionOnlyType` values

| Value | Description |
|-------|-------------|
| `ACTIVE` | User performs a prompted gesture (strictest) |
| `PASSIVE` | User holds face still in frame |
| `FLASH` | Like PASSIVE but with full-brightness screen flash (front camera only) |

#### `LivenessSuccess` fields (detection only)

| Method | Type | Description |
|--------|------|-------------|
| `getIsProcedureValidationPassed()` | `Boolean` | Whether the liveness check passed |
| `getConfidencePercent()` | `Double` | Confidence score of the liveness result |
| `getThresholdPercent()` | `Double` | Threshold used for the pass/fail decision |

---

### Liveness with Facial Comparison

Runs liveness detection and compares the captured face against a reference image you provide.

```java
import com.surfmaxcomm.liveness_native.Core.SurfmaxLiveness;
import com.surfmaxcomm.liveness_native.Core.CameraType;
import com.surfmaxcomm.liveness_native.Core.LivenessCompletedCallBack;
import com.surfmaxcomm.liveness_native.Core.LivenessFacialComparisonType;
import com.surfmaxcomm.liveness_native.Core.Model.DeviceMetadataConfig;
import com.surfmaxcomm.liveness_native.Core.Model.LivenessSuccessful.LivenessSuccess;
import com.surfmaxcomm.liveness_native.Core.Model.ThresholdConfig;
import com.surfmaxcomm.liveness_native.Core.ThresholdPriority;

// referenceImageBytes: byte[] from a JPEG/PNG image of the person to compare against
new SurfmaxLiveness.SurfmaxLivenessFacialComparisonBuilder(activity)
        .setClientId("your-client-id")
        .setApiKey("your-api-key")
        .setAppName("your-app-name")
        .setImageByte(referenceImageBytes)                  // reference face image as byte[]
        .setLivenessType(LivenessFacialComparisonType.ACTIVE) // ACTIVE or PASSIVE
        .setCameraType(CameraType.FRONT)                    // FRONT or REAR
        .setIsDev(false)
        .setShowLivenessResult(true)
        .setStartProcessOnGettingToFirstScreen(false)
        .setThresholdConfig(new ThresholdConfig.Builder()
                .setThresholdPriority(ThresholdPriority.SERVER_ONLY)
                .build())
        .setDeviceMetadataConfig(new DeviceMetadataConfig.Builder()
                .setCollectDeviceId(true)
                .setCollectDeviceLocation(true)
                .build())
        .setListener(new LivenessCompletedCallBack() {
            @Override
            public void onSuccess(LivenessSuccess response) {
                boolean faceMatched = response.getComparisonData().getIsPassFaceComparison();
                double confidence = response.getConfidencePercent();
                double threshold = response.getThresholdPercent();
                // handle success
            }

            @Override
            public void onFailure(int statusCode, String errorObject) {
                // handle failure
            }
        })
        .build();
```

#### Loading a reference image from the gallery

```java
// Register the picker in your Fragment's field (before onCreateView)
private byte[] referenceImageBytes;

private final ActivityResultLauncher<String> imagePickerLauncher =
        registerForActivityResult(new ActivityResultContracts.GetContent(), uri -> {
            if (uri == null) return;
            try {
                InputStream inputStream = requireContext().getContentResolver().openInputStream(uri);
                Bitmap bitmap = BitmapFactory.decodeStream(inputStream);
                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                bitmap.compress(Bitmap.CompressFormat.JPEG, 100, baos);
                referenceImageBytes = baos.toByteArray();
            } catch (Exception e) {
                // handle error
            }
        });

// Launch in a click listener
button.setOnClickListener(v -> imagePickerLauncher.launch("image/*"));
```

#### `LivenessFacialComparisonType` values

| Value | Description |
|-------|-------------|
| `ACTIVE` | Dynamic liveness gesture + face match |
| `PASSIVE` | Static face hold + face match |

#### `ThresholdPriority` values

| Value | Description |
|-------|-------------|
| `SERVER_ONLY` | Use the threshold configured on the server |
| `CLIENT_ONLY` | Use a threshold set by the client |
| `HIGHEST` | Use whichever threshold value is highest |

#### `LivenessSuccess` fields (facial comparison)

| Method | Type | Description |
|--------|------|-------------|
| `getComparisonData().getIsPassFaceComparison()` | `Boolean` | Whether the faces matched |
| `getConfidencePercent()` | `Double` | Confidence score of the face match |
| `getThresholdPercent()` | `Double` | Threshold used for the pass/fail decision |

---

## Builder Reference

### Common options (both builders)

| Method | Type | Required | Description |
|--------|------|----------|-------------|
| `setClientId(String)` | `String` | Yes | Your Surfmax client ID |
| `setApiKey(String)` | `String` | Yes | Your Surfmax API key |
| `setAppName(String)` | `String` | Yes | Your registered app name |
| `setCameraType(CameraType)` | `CameraType` | No | `FRONT` (default) or `REAR` |
| `setIsDev(boolean)` | `boolean` | No | `true` for sandbox, `false` for production |
| `setShowLivenessResult(boolean)` | `boolean` | No | Show the SDK's built-in result screen |
| `setStartProcessOnGettingToFirstScreen(boolean)` | `boolean` | No | Auto-start capture on first screen |
| `setDeviceMetadataConfig(DeviceMetadataConfig)` | `DeviceMetadataConfig` | No | Configure device ID and location collection |
| `setListener(LivenessCompletedCallBack)` | callback | Yes | Success/failure callbacks |

### Detection-only options

| Method | Type | Description |
|--------|------|-------------|
| `setLivenessType(LivenessDetectionOnlyType)` | `LivenessDetectionOnlyType` | `ACTIVE`, `PASSIVE`, or `FLASH` |

### Facial comparison options

| Method | Type | Description |
|--------|------|-------------|
| `setImageByte(byte[])` | `byte[]` | Reference face image bytes (JPEG/PNG) |
| `setLivenessType(LivenessFacialComparisonType)` | `LivenessFacialComparisonType` | `ACTIVE` or `PASSIVE` |
| `setThresholdConfig(ThresholdConfig)` | `ThresholdConfig` | Configure comparison threshold priority |

---

## Camera Type Behaviour

| `CameraType` | Liveness Detection | Facial Comparison |
|---|---|---|
| `FRONT` | All types supported (ACTIVE, PASSIVE, FLASH) | Supports ACTIVE and PASSIVE |
| `REAR` | ACTIVE and PASSIVE only | Requires a reference image for comparison |

When using `CameraType.REAR` for facial comparison, a reference image (`setImageByte`) must be provided before starting.

---

## Runtime Permissions

Starting from Android 6.0 (API 23), dangerous permissions must be requested at runtime. Request these before invoking the SDK:

```java
ActivityCompat.requestPermissions(activity, new String[]{
    Manifest.permission.CAMERA,
    Manifest.permission.RECORD_AUDIO,
    Manifest.permission.READ_EXTERNAL_STORAGE,
    Manifest.permission.WRITE_EXTERNAL_STORAGE,
    Manifest.permission.READ_PHONE_STATE,
    Manifest.permission.ACCESS_FINE_LOCATION,
    Manifest.permission.ACCESS_COARSE_LOCATION
}, REQUEST_CODE_PERMISSIONS);
```

---

## Troubleshooting

**"Unauthorized" during Gradle sync**
Generate a new GitHub personal access token with `read:packages` scope (or tick all boxes if unsure). Update `github.properties`.

**Crashes or timeouts in release builds**
Add the ProGuard rules from Step 5 and ensure `minifyEnabled true` is set only in release build type.

**Location not collected**
Ensure `ACCESS_FINE_LOCATION` and `ACCESS_COARSE_LOCATION` are declared in the manifest and granted at runtime before calling `.build()`.

**Device ID not collected**
Ensure `READ_PHONE_STATE` is declared and granted. On Android 10+, this permission is required for IMEI access.
