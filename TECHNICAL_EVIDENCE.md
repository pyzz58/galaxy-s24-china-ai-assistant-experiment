# Samsung Galaxy S24 AI Assistant — Public Technical Evidence Appendix

**Evidence cut-off:** 2026-11-01  
**Extraction status:** No source changes, rebuild, device connection, network API request, or GitHub publication was performed during the original evidence extraction. This appendix was subsequently published as reference documentation.

**Publication note:** This is a technical evidence appendix compiled from retained experiment notes, source-code excerpts, APK metadata, and filtered logs. Publishing the appendix did not involve new builds or device tests. The evidence classifications below are those of the original extraction; the private source tree and hardware were not independently re-tested during publication. The Kotlin/XML examples are excerpts, not a standalone buildable application.

**Main report:** [Trying to Replace the AI Assistant on a China-Market Samsung Galaxy S24](README.md).

## Evidence labels

| Label | Meaning |
|---|---|
| **Observed** | Reproduced on the test device and retained in a local report or sanitized log excerpt. |
| **Source-backed** | Established from the checked-out Android source/Manifest, the compiled APK, or cited upstream documentation reviewed during the experiment. |
| **Inference** | A conclusion supported by multiple observations, but not directly logged by the OEM. |
| **Hypothesis** | A plausible cause that was not established. It must not be presented as a confirmed root cause. |

---

## 1. Experiment Environment

### Device and OS

| Item | Recorded value | Evidence status |
|---|---|---|
| Device model | Samsung Galaxy S24, `SM-S9210` | Observed |
| Product device | `e1q` | Observed |
| Android version | Android 16, API level 36 | Observed |
| Android build ID | `BP4A.251205.006.S9210ZCS6DZI1` | Observed |
| One UI property | `ro.build.version.oneui=80500`, reported as One UI 8.5 | Observed |
| CSC sales code | `CHC` | Observed |
| Bootloader/build suffix | `S9210ZCS6DZI1` | Observed |
| ABI used by test builds | `arm64-v8a` | Source-backed |
| ADB connection used for runtime tests | One authorized device, state `device` | Observed |
| ADB host tools | Platform-Tools 37.0.1; `adb` 1.0.41 | Observed |
| Host used for build/ADB | macOS 27.2 on arm64 | Observed |

No serial number, IMEI, Android ID, ADB transport identifier, account identifier, or other
device-unique value is included.

### Application and assistant baseline

| Item | Recorded value |
|---|---|
| Test application ID | `com.app.assistant` |
| Independent Activity-only experiment ID | `com.app.assistant.activityonly` |
| Build SDK | `minSdk=24`, `targetSdk=35`, `compileSdk=35` |
| Signing used for experiments | Android Debug certificate; APK Signature Scheme v2 |
| Original default assistant holder | `com.openai.chatgpt` |
| Restored default assistant holder | `com.openai.chatgpt` |
| Original long-press routing value | `power_button_long_press=101` |
| Recorded side-button selector | `function_key_config_longpress_selected_item=long_press_wake_bixby` |
| Bixby packages during baseline | Bixby Agent `4.1.20.19`; Wakeup `4.1.30.18`; Bixby Touch `4.3.01.15` |
| Root/Shizuku/Accessibility | Not used |
| `WRITE_SECURE_SETTINGS` | Not granted |

The test package was based on the MIT-licensed upstream project `ai-assistant-android` at commit
`0f01923057956068654f4b9a99bf659d48f0e4f4`. The experiment added assistant routing, speech
routing, command parsing, and tests. The retained experiment notes state that the test applications were subsequently uninstalled
by the device owner; this publication did not independently check the device's later state.

---

## 2. Build and Test Matrix

The matrix separates build/test success from real device success. A successful Gradle build or
unit test is **not** evidence that an assistant capability worked on the phone.

| Version | Core change | Test condition | Actual result | Evidence source | Remaining issue |
|---|---|---|---|---|---|
| `2.1` / vc `2` | Initial modified MIT baseline with Assistant Activity and added Voice Interaction services | Local build and static Manifest inspection | 104 MB arm64 APK built; no full side-key acceptance in this record | Replacement review report; APK metadata | Pre-optimization build, oversized dependency set |
| `3.0-mvp1` / vc `3` | Pre-3.1 fallback artifact; Voice Interaction services and recognition placeholder present | Binary Manifest inspection; later superseded by `3.1-mvp2` | Build retained, but no independently preserved full side-key acceptance result | APK metadata; MVP report | No distinct successful system-routing record |
| `3.1-mvp2` / vc `4` | Added zh-CN request, RecognitionService discovery, Chinese command parsing, AppLaunchResolver, alarm-date policy, DeepSeek default/API corrections | Installed with `adb install -r`; text-mode command test; later assistant-role/side-key test | Text “打开计算器” traversed the internal parser and started Samsung Calculator. With role set to the app, long press still invoked Bixby at value `101`. At value `5`, routing reached the app but SessionService binding returned `not found`. | MVP report; sanitized logs | Voice Interaction session never started; native STT unavailable |
| `3.2-mvp3` / vc `5` | Added CTS-style SessionService `:session` process and lifecycle diagnostics | Clean build, 28 unit tests, installed, side button with value `5` | System selected the current assistant, then ActivityManager returned `not found` before SessionService creation | Session fix report; session A/B report | Samsung/OEM-side filtering condition unknown |
| `3.2.1-mvp4` / vc `6` | Same functional package with `debuggable=false` only | One-variable A/B test with value `5` | Identical `not found` failure | Session A/B report | `debuggable=false` was not sufficient |
| `3.2.1-mvp4-activityonly` / vc `6`, separate application ID | Activity-only candidate; Voice Interaction services removed from the built Manifest | Installed as independent app, assigned Assistant role, side button set to `5`, user long-pressed once | Full chain succeeded: SystemUI → `ACTION_ASSIST` → `AssistActivity` → `MainActivity` | Activity-only report; sanitized logs | One-device result only |
| `3.3-mvp5` / vc `7` | Merged Activity-only architecture into the main `com.app.assistant` package; removed service registration | Installed with `adb install -r`, assigned role, side button set to `5`, user long-pressed once | Full chain succeeded; Activity-only candidate selected, voice-interaction service setting empty | Activity integration report; sanitized logs | Voice input still unavailable without a usable STT backend |
| `3.4-mvp6` / vc `8` | Moved synchronous OkHttp transcription off the main thread | Real microphone capture and cloud STT attempt | Capture succeeded, request path advanced, then generic `IOException`; no transcription | Activity integration report; sanitized log | HTTP status was hidden by diagnostic logging |
| `3.4.1-mvp7` / vc `9` | Added status-code-only HTTP diagnostics | Real microphone capture and repeated credential checks | Request reached the service and returned `HTTP 401`; no transcription, parser, or calculator launch | Activity integration report; sanitized logs | Credential authentication failed; experiment stopped |

### Saved APK fingerprints

| Artifact | SHA-256 |
|---|---|
| `2.1` baseline | `660247dd008ee8af54d7e2fa46b00f7da509ac169dc96e58d4f6bca87c857660` |
| `3.0-mvp1` | `6b88e94aaba294a11eaaace2661dbe73e5c43498a7b94f236eb82ccef9a879d7` |
| `3.1-mvp2` | `94156a792363550758b7c09024061a9b8cefc54769ca764b012df6f041524a4c` |
| `3.2-mvp3` | `b6c75d22c2e0f0e63c719605c124c6d1fd0c7d74324d6717caaacc0c5cbcfa47` |
| `3.2.1-mvp4`, service route A/B | `43b55487d6eb62103af6fda4b69521f3924f5e7253e18b1c71914d58d1509886` |
| `3.2.1-mvp4-activityonly` | `679f71a2d0fd701b77c9e38344cea20b5930d1b4dc5f2214af4705654a84a571` |
| `3.3-mvp5` | `dcaa42cbe94169e189cde8a156d8e8846af8c2664feabc568e8f4f97b400ae6c` |
| `3.4-mvp6` | `47be098c355f48ed930c54ff0fd7688e282895834d8533d0dd9c4280939704e5` |
| `3.4.1-mvp7` | `ce847d8702859542459e946f0e51c87cd4bc9e99396211c90345ef5918244ce3` |

---

## 3. Failed VoiceInteractionService Configuration

### Manifest used by the failed service route

The following structure was present in the `3.2.1-mvp4` service-route APK:

```xml
<service
    android:name=".AssistantVoiceInteractionService"
    android:permission="android.permission.BIND_VOICE_INTERACTION"
    android:exported="true">

    <meta-data
        android:name="android.voice_interaction"
        android:resource="@xml/assistant_voice_interaction" />

    <intent-filter>
        <action android:name="android.service.voice.VoiceInteractionService" />
    </intent-filter>
</service>

<service
    android:name=".AssistantVoiceInteractionSessionService"
    android:permission="android.permission.BIND_VOICE_INTERACTION"
    android:enabled="true"
    android:exported="true"
    android:process=":session" />

<service
    android:name=".AssistantRecognitionService"
    android:permission="android.permission.BIND_RECOGNITION_SERVICE"
    android:exported="true">

    <intent-filter>
        <action android:name="android.speech.RecognitionService" />
    </intent-filter>

    <meta-data
        android:name="android.speech"
        android:resource="@xml/assistant_recognition_service" />
</service>
```

The Voice Interaction metadata resource was:

```xml
<voice-interaction-service
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:recognitionService=".AssistantRecognitionService"
    android:sessionService=".AssistantVoiceInteractionSessionService"
    android:supportsAssist="true"
    android:supportsLaunchVoiceAssistFromKeyguard="true" />
```

The recognition service was a declaration placeholder:

```xml
<recognition-service
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:selectableAsDefault="false" />
```

It did not provide speech recognition.

### Observed failure

The parent Voice Interaction service reached:

```text
VoiceInteractionService onCreate
VoiceInteractionService onReady
```

The system then attempted to bind the session service and logged:

```text
Unable to start service Intent {
  act=android.service.voice.VoiceInteractionService
  cmp=com.app.assistant/.AssistantVoiceInteractionSessionService
}: not found

Failed binding to voice interaction session service
ComponentInfo{com.app.assistant/.AssistantVoiceInteractionSessionService}
```

No lifecycle callback from the session service or session was observed:

```text
VoiceInteractionSessionService onCreate       // absent
VoiceInteractionSessionService onNewSession   // absent
VoiceInteractionSession onCreate              // absent
VoiceInteractionSession onShow                // absent
```

There was no `SecurityException`, application crash, or ANR.

### Hypothesis audit

| Candidate explanation | Result | Evidence |
|---|---|---|
| SessionService class is missing | Excluded | Component existed in PackageManager, `dumpsys voiceinteraction`, APK Manifest, and DEX |
| Required bind permission is wrong | Excluded | Declared `BIND_VOICE_INTERACTION`; no permission exception was raised |
| Service is disabled or not exported | Excluded | `enabled=true`, `exported=true`; package and component were both visible |
| Intent filter is required for SessionService | Excluded by source review | Android 16 AOSP binds via explicit `ComponentName`; the CTS SessionService has no intent filter |
| Shared process with the parent service is the cause | Excluded as a sufficient cause | `:session` process did not change the result |
| The service starts and crashes | Excluded | No session service/session lifecycle callback occurred |
| `debuggable=true` is the only cause | Excluded | A/B build with `debuggable=false` failed identically |
| CHC firmware blocks all third-party sessions | Excluded | ChatGPT's SessionService bound and completed on the same device and routing value |
| Samsung applies another package-specific filter | **Hypothesis** | ChatGPT succeeded while this package failed before session creation; exact filter was not exposed |
| Signature/target SDK/vendor allowlist is responsible | **Hypothesis** | These differ from ChatGPT, but were not isolated |

**Inference:** this failure occurred in the system-side explicit service resolution/binding path,
before application session code could run. The exact OEM condition remains unproven.

---

## 4. Working ACTION_ASSIST Implementation

### Proven path

The Activity-only route was accepted by Android and One UI 8.5:

```text
Side-button long press
→ SystemUI AssistManager
→ android.intent.action.ASSIST
→ <applicationId>/.AssistActivity
→ <applicationId>/.MainActivity
```

This route was first proven under the independent application ID
`com.app.assistant.activityonly`, then merged into `com.app.assistant`.

The Android 16 CDD and AOSP SystemUI behavior reviewed during the experiment permit an
`ACTION_ASSIST` Activity as a user-selected assistant. When the selected assistant is not an
active VoiceInteractionService, SystemUI uses its Assist Activity path rather than creating a
Voice Interaction session.

### Minimal Manifest configuration

This is the routing configuration from the successful main application:

```xml
<application>
    <activity
        android:name=".AssistActivity"
        android:launchMode="singleTask"
        android:exported="true"
        android:theme="@style/Theme.TransparentActivity">

        <intent-filter>
            <action android:name="android.intent.action.ASSIST" />
            <category android:name="android.intent.category.DEFAULT" />
        </intent-filter>
    </activity>

    <activity
        android:name=".MainActivity"
        android:exported="false"
        android:windowSoftInputMode="adjustResize"
        android:theme="@style/Theme.TransparentActivity"
        android:showOnLockScreen="true" />
</application>
```

The successful APK also declared `ACTION_VOICE_COMMAND` and `ACTION_VOICE_ASSIST` on the same
Activity and retained a launcher intent-filter. Those additional filters are not required for
the proven SystemUI `ACTION_ASSIST` route.

### Actual `AssistActivity` routing code

The following is the relevant routing subset from the successful source:

```kotlin
package com.app.assistant

import android.content.Intent
import android.os.Bundle
import androidx.activity.ComponentActivity

class AssistActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        if (
            Intent.ACTION_ASSIST == intent.action ||
            Intent.ACTION_VOICE_COMMAND == intent.action
        ) {
            moveToMainActivity()
        } else {
            startDefaultFlow()
        }
    }

    private fun startDefaultFlow() {
        val intent = Intent(this, MainActivity::class.java)
        intent.flags = Intent.FLAG_ACTIVITY_REORDER_TO_FRONT
        startActivity(intent)
        finish()
    }

    private fun moveToMainActivity() {
        val intent = Intent(this, MainActivity::class.java)
        intent.flags =
            Intent.FLAG_ACTIVITY_NEW_TASK or
            Intent.FLAG_ACTIVITY_CLEAR_TASK or
            Intent.FLAG_ACTIVITY_NO_USER_ACTION
        intent.putExtra("speak", true)
        startActivity(intent)
        finish()
    }
}
```

The successful source also included lock-screen presentation handling; that code is not required
to reproduce the plain `ACTION_ASSIST` routing result.

### App-specific automatic speech handoff

`speak=true` is an application-defined extra, not an Android assistant contract:

```kotlin
// AssistActivity
intent.putExtra("speak", true)

// MainViewModel initialization
if (speak) {
    _isCustomUI.value = true
    _isCustomUIHalfPage.value = true
    _isListening.value = true
    startSpeechRecognition()
}
```

This reliably started the app's speech path after assistant invocation. It did not make speech
recognition itself succeed, because the device had no usable native recognition service and the
cloud STT credential later returned HTTP 401.

### Selecting the default assistant

The application checked role ownership through `RoleManager`:

```kotlin
val roleManager = context.getSystemService(Context.ROLE_SERVICE) as? RoleManager
val isDefault =
    roleManager?.isRoleHeld(RoleManager.ROLE_ASSISTANT) == true
```

The in-app button opened the system voice-input/default-app settings rather than directly
assigning the role:

```kotlin
val intent = Intent(Settings.ACTION_VOICE_INPUT_SETTINGS)
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
context.startActivity(intent)
```

For an owner-authorized test device, the shell equivalent used during the experiment was:

```bash
adb shell cmd role add-role-holder --user 0 \
  android.app.role.ASSISTANT com.app.assistant

adb shell cmd role get-role-holders --user 0 android.app.role.ASSISTANT
adb shell settings get secure assistant
adb shell settings get secure voice_interaction_service
```

After selecting the Activity-only build, the effective state was:

```text
secure assistant=com.app.assistant/.AssistActivity
secure voice_interaction_service=<empty>
```

### Observed successful routing log

Sanitized excerpt:

```text
DigitalAssistantApp=com.app.assistant
InvocationType=Side button
AssistManager: startAssistInternal

START {
  act=android.intent.action.ASSIST
  cmp=com.app.assistant/.AssistActivity
} from com.android.systemui

START com.app.assistant/.MainActivity
```

The final foreground Activity was `com.app.assistant/.MainActivity`. Main Activity display took
approximately 590 ms in the recorded run. No crash or ANR was observed.

### Permissions

The Activity-only routing itself required:

- `exported=true` on the `ACTION_ASSIST` Activity;
- a `DEFAULT` category in its intent filter;
- no custom signature permission;
- no `BIND_VOICE_INTERACTION` permission;
- no Accessibility, Shizuku, Root, or `WRITE_SECURE_SETTINGS`.

Microphone use requires the normal runtime `RECORD_AUDIO` permission, but this is separate from
assistant-route eligibility. The final test APK's declared functional permissions were
`INTERNET`, `RECORD_AUDIO`, `VIBRATE`, and `SET_ALARM`, plus AndroidX-generated entries.

### Why Activity-only worked while the Session route failed

| VoiceInteractionService route | Activity-only route |
|---|---|
| Requires a discoverable `VoiceInteractionService` and valid metadata | Requires an `ACTION_ASSIST` Activity |
| System creates/binds a Voice Interaction session | System starts the selected Assist Activity directly |
| Failure occurred while resolving/binding the SessionService | No SessionService exists to bind |
| `secure voice_interaction_service` was populated | `secure voice_interaction_service` was empty |
| App code did not run in the failed session route | `AssistActivity` executed and forwarded to `MainActivity` |

This is both **observed** and consistent with the reviewed Android 16 CDD/SystemUI behavior.

---

## 5. Samsung Side-Key Routing Evidence

### Values and behavior on this device

| Setting | Value | Observed result |
|---|---|---|
| Original `power_button_long_press` | `101` | Long press invoked Bixby even when another package held the Android Assistant role |
| AOSP-style Assistant value | `5` | Long press invoked the current Android digital assistant through SystemUI |
| `function_key_config_longpress_selected_item` | `long_press_wake_bixby` | Retained during tests; changing it was not required |

At `101`, sanitized logs contained Bixby-specific entries such as:

```text
BixbyService: startBixbyService ... longPress=true
PhoneWindowManagerExt: run wake Bixby
PhoneWindowManagerExt: consume powerLongPress
BixbyKeyLService ... processLongPress()
```

The third-party `MainActivity` was not started.

At `5`, the Bixby long-press path was absent and the following path appeared:

```text
EID_INVOKE_DIGITAL_ASSISTANT
DigitalAssistantApp=<selected assistant package>
InvocationType=Side button
AssistManager: startAssistInternal
```

**Inference:** on this One UI 8.5 build, `101` selects the Samsung/Bixby-specific long-press
branch, while `5` selects the current Android assistant dispatch path. This must not be
generalized to all Samsung models or firmware versions without independent testing.

### Exact owner-authorized test commands

Record the original value before any change:

```bash
adb shell settings get global power_button_long_press
adb shell settings get global function_key_config_longpress_selected_item
adb shell settings get secure assistant
adb shell settings get secure voice_interaction_service
adb shell cmd role get-role-holders --user 0 android.app.role.ASSISTANT
```

Temporary route:

```bash
adb shell cmd role add-role-holder --user 0 \
  android.app.role.ASSISTANT com.app.assistant
adb shell settings put global power_button_long_press 5
```

Restore the recorded values:

```bash
adb shell settings put global power_button_long_press 101
adb shell cmd role add-role-holder --user 0 \
  android.app.role.ASSISTANT com.openai.chatgpt

adb shell settings get global power_button_long_press
adb shell settings get global function_key_config_longpress_selected_item
adb shell cmd role get-role-holders --user 0 android.app.role.ASSISTANT
adb shell settings get secure assistant
adb shell settings get secure voice_interaction_service
```

If the original `power_button_long_press` value is absent, restore the absence instead of
guessing:

```bash
adb shell settings delete global power_button_long_press
```

### Persistence and authorization limits

- The setting persisted for the duration of the observed tests.
- **Not tested:** whether reboot, reopening Samsung's Side-button settings, One UI updates, or
  security updates rewrite the value from `5` back to `101`.
- The SideKey project was reviewed as a risk reference. SideKey was not installed, and this
  experiment did not grant `WRITE_SECURE_SETTINGS`.
- Changing assistant role or global settings must be performed only with the device owner's
  explicit authorization. It requires an owner-controlled ADB/debug environment or equivalent
  system authority; it does not require Android Enterprise Device Owner mode.
- The application itself cannot promise to repair this OEM setting after it is reset.

---

## 6. Speech Recognition Failure Analysis

### Native Android RecognitionService

Observed device state:

```text
settings get secure voice_recognition_service
→ empty
```

Excluding the app's own placeholder, no enabled external `android.speech.RecognitionService`
was found. The app could not obtain Chinese recognition through the standard native route.

Samsung private speech components were visible, but they use private actions, protected binding,
or platform signing constraints. They were not bypassed.

### Implemented routing policy

The source implements:

```kotlin
object SttRoutePolicy {
    fun select(
        mode: SttMode,
        nativeRecognitionAvailable: Boolean,
        apiConfigured: Boolean,
    ): SttRoute = when {
        mode == SttMode.API -> SttRoute.API
        nativeRecognitionAvailable -> SttRoute.NATIVE
        apiConfigured -> SttRoute.API
        else -> SttRoute.NATIVE
    }
}
```

When neither native nor configured cloud STT was available, the app reported that Chinese
speech recognition was unavailable before recording.

### Audio capture and WAV format

The API route used `AudioRecord` with:

```text
source:        VOICE_RECOGNITION
sample rate:   16,000 Hz
channels:      mono
encoding:      signed PCM 16-bit little-endian
silence stop:  1,200 ms after detected speech
maximum capture: 15 seconds
transport file: speech.wav
```

The PCM data was converted into an in-memory RIFF/WAV byte array:

```kotlin
out.write("RIFF".toByteArray())
le32(36 + pcm.size)
out.write("WAVEfmt ".toByteArray())
le32(16)
le16(1)                 // PCM
le16(1)                 // mono
le32(sampleRate)        // 16,000
le32(sampleRate * 2)    // byte rate
le16(2)                 // block align
le16(16)                // bits per sample
out.write("data".toByteArray())
le32(pcm.size)
out.write(pcm)
```

The microphone was stopped and released in a `finally` block. No audio file was intentionally
written to persistent device storage by this path.

### `NetworkOnMainThreadException`

The first `3.4` attempt recorded audio but failed before a network response:

```text
W SpeechRecognizerManager:
Configured speech recognition failed: NetworkOnMainThreadException
```

**Root cause:** `transcribe()` used OkHttp's synchronous `execute()` from a coroutine launched
on the lifecycle/main dispatcher.

The exact fix was:

```kotlin
// Before
val text = transcribe(pcm)

// After
val text = withContext(Dispatchers.IO) { transcribe(pcm) }
```

After this change, the app reached the network path and no longer produced
`NetworkOnMainThreadException`.

### HTTP authentication result

The diagnostic build changed HTTP failures from a generic exception name to a status-code-only
diagnostic:

```kotlin
if (!response.isSuccessful) {
    throw SttHttpException(response.code)
}
```

The retained sanitized log was:

```text
W SpeechRecognizerManager:
Configured speech recognition failed: STT HTTP 401
```

AudioService recorded genuine microphone capture before each request. Replacing the
configuration value in the app and re-running still produced `HTTP 401`.

**Confirmed:** capture, WAV encoding, worker-thread network execution, and HTTPS request dispatch
worked.  
**Confirmed:** the tested service response rejected authentication with `401`.  
**Not proven:** whether this was due to a wrong platform key, incomplete copy, revocation,
account/region restrictions, or another account-side cause.  
**Not achieved:** a non-empty transcript, Chinese command parsing from STT, or calculator launch
through the voice path.

No API key, Authorization header, audio, response body, or personal speech content is retained
in this appendix.

---

## 7. Reproduction and Rollback Procedure

This procedure reproduces only the **working Activity-only assistant entry**. It does not claim
to reproduce successful Chinese cloud STT.

### Safety prerequisites

1. Use a test device or explicit owner approval.
2. Record the current assistant holder and side-button settings before changes.
3. Use a test application ID and test signing key.
4. Do not request Accessibility, Shizuku, Root, or system-signing privileges.
5. Do not place credentials in source, build files, logs, or bug reports.
6. Keep a known-working assistant selected and available for rollback.

### Build shape

- `targetSdk=35` was installed successfully on Android 16/API 36.
- Build only `arm64-v8a` for this device test.
- Include one exported `ACTION_ASSIST` Activity.
- Do not register a `VoiceInteractionService` or `VoiceInteractionSessionService` in the
  Activity-only test build.
- Keep the actual interaction Activity non-exported.
- Use a unique FileProvider authority if an experiment package coexists with the main package.

### Verify candidate registration

```bash
adb shell cmd package query-activities \
  -a android.intent.action.ASSIST \
  -c android.intent.category.DEFAULT \
  --brief
```

Confirm that the test Activity appears. Confirm that the package does not appear in the
VoiceInteractionService query:

```bash
adb shell cmd package query-services \
  -a android.service.voice.VoiceInteractionService \
  --brief
```

### Select and trigger the test assistant

The user may assign the app from the system default-assistant UI. For an authorized automated
test:

```bash
adb shell cmd role add-role-holder --user 0 \
  android.app.role.ASSISTANT com.app.assistant
adb shell settings put global power_button_long_press 5
```

The user must then perform the side-button long press manually. Collect only minimal filtered
evidence:

```bash
adb logcat -d -v brief | grep -E \
  'DigitalAssistantApp|InvocationType|startAssistInternal|ACTION_ASSIST|AssistActivity|MainActivity|FATAL EXCEPTION|ANR'
```

Expected evidence:

```text
DigitalAssistantApp=com.app.assistant
InvocationType=Side button
AssistManager: startAssistInternal
act=android.intent.action.ASSIST
cmp=com.app.assistant/.AssistActivity
com.app.assistant/.MainActivity
```

### Rollback

```bash
adb shell settings put global power_button_long_press 101
adb shell cmd role add-role-holder --user 0 \
  android.app.role.ASSISTANT com.openai.chatgpt

adb shell cmd role get-role-holders --user 0 android.app.role.ASSISTANT
adb shell settings get global power_button_long_press
adb shell settings get global function_key_config_longpress_selected_item
adb shell settings get secure assistant
adb shell settings get secure voice_interaction_service
```

If the original value was absent:

```bash
adb shell settings delete global power_button_long_press
```

Remove a test build only after assigning the role back to a known assistant:

```bash
adb uninstall <test.application.id>
```

Uninstalling removes the test application's private data.

---

## 8. Confirmed Results vs Unresolved Questions

### Confirmed results

1. **Observed:** Android 16/One UI 8.5 accepted an `ACTION_ASSIST` Activity as the selected
   assistant and launched it from a physical side-button long press.
2. **Observed:** The complete chain
   `SystemUI AssistManager → ACTION_ASSIST → AssistActivity → MainActivity` worked on
   `SM-S9210`, CSC `CHC`.
3. **Observed:** The Activity-only route worked without a VoiceInteractionService,
   SessionService, Accessibility, Shizuku, Root, or `WRITE_SECURE_SETTINGS`.
4. **Observed:** On this device, `power_button_long_press=101` invoked Bixby, while `5` invoked
   the current Android assistant.
5. **Observed:** The VoiceInteraction route repeatedly failed before session creation with
   `not found`, even when the component was visible to PackageManager and `debuggable=false`.
6. **Observed:** ChatGPT's VoiceInteractionSessionService bound successfully under the same
   device and side-button value, excluding a blanket ban on every third-party session.
7. **Observed:** The local text path parsed “打开计算器” and launched Samsung Calculator before
   the voice work; this does not prove the voice-to-command path.
8. **Observed:** Real microphone capture, WAV creation, and HTTPS dispatch worked after the
   thread fix.
9. **Observed:** Cloud STT returned HTTP 401; no successful transcript was obtained.
10. **Source-backed:** The final Activity-only build did not register VoiceInteraction or
    Session services.

### Unresolved questions

1. Which exact Samsung/OEM filter or policy causes the custom VoiceInteractionSessionService
   bind to return `not found` remains unknown.
2. The effects of package signing identity, target SDK, OEM allowlisting, caller UID visibility,
   and other build attributes are not isolated.
3. Whether `power_button_long_press=5` survives reboot, entering Samsung's Side-button settings,
   OTA updates, or security updates on this exact device is untested.
4. The `ACTION_VOICE_ASSIST` filter was declared, but the Activity's explicit `speak=true`
   branch was verified only for `ACTION_ASSIST` and `ACTION_VOICE_COMMAND`.
5. No successful native or cloud transcription, continuous voice follow-up, TTS response, or
   voice-driven Calculator launch was completed.
6. The text-mode calculator launch is verified; other app launches, alarm creation through the
   final voice loop, real LLM chat, web search citations, and TTS were not accepted end-to-end.
7. Every runtime conclusion comes from one consumer device and one CHC firmware build. None of
   the Samsung side-key findings should be generalized without testing other models and builds.

---

## 9. Privacy and Reproducibility Limitations

### Privacy review

This appendix contains only:

- model/system/build and CSC metadata;
- application IDs, component names, standard Android actions, and setting names;
- public test command phrases;
- sanitized log fragments;
- source excerpts and hashes.

It intentionally excludes:

- device serial numbers, IMEI/MEID, Android ID, ADB transport IDs, phone number, and account IDs;
- user names, email addresses, contacts, messages, private application content, or voice audio;
- API keys, bearer tokens, cookies, Authorization headers, or credential values;
- raw unfiltered logcat, screenshots, or full bug reports;
- local absolute filesystem paths;
- unrelated repository or personal-project material.

According to the retained notes, the test applications were uninstalled after the experiment;
no live device check was conducted while preparing or publishing this appendix.

### Reproducibility limitations

- This is a single-device, single-firmware result, not a Samsung-wide compatibility claim.
- The VoiceInteraction failure is a reproducible observation on this package/device combination,
  not a fully explained Android or Samsung defect.
- The Activity-only result establishes assistant routing and Activity handoff; it does not
  establish a complete AI voice assistant.
- The cloud STT test reached authentication but no successful transcription. `HTTP 401` is an
  observed response, not proof of one specific account-side cause.
- Debug-signed test APKs are suitable for this experiment but are not a production release
  strategy.
- The side-button global setting is an owner-authorized lab intervention. A production design
  should not assume that undocumented OEM values remain stable across updates.

---

## Primary retained evidence

- `samsung_s24_diagnostic.md`
- `samsung_s24_assistant_mvp.md`
- `samsung_s24_assistant_session_fix.md`
- `samsung_s24_assistant_session_ab.md`
- `samsung_s24_activity_only_assistant.md`
- `samsung_s24_assistant_activity_integration.md`

**Implementation sources used for excerpts:**

- `app/src/main/AndroidManifest.xml`
- `app/src/main/java/com/app/assistant/AssistActivity.kt`
- `app/src/main/java/com/app/assistant/MainActivity.kt`
- `app/src/main/java/com/app/assistant/viewmodel/MainViewModel.kt`
- `app/src/main/java/com/app/assistant/viewmodel/SettingsViewModel.kt`
- `app/src/main/java/com/app/assistant/ui/screen/SettingsScreen.kt`
- `app/src/main/java/com/app/assistant/speech/SpeechRecognizerManager.kt`
- `app/src/main/java/com/app/assistant/speech/SttRoutePolicy.kt`
- `app/src/main/java/com/app/assistant/speech/WavEncoder.kt`
- `app/src/main/res/xml/assistant_voice_interaction.xml`
- `app/src/main/res/xml/assistant_recognition_service.xml`
