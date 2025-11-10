
---

```
# Part 12 — Testing, CI/CD, Play Store Publishing & Monitoring (Release)

**Objective:** Finish the course: test your app, set up CI/CD, produce signed release, publish to Play Store, monitor & maintain. Also compare native Android vs React Native deployment considerations.

---

## 🧪 1. Testing Strategy

### 1.1 Unit tests (Kotlin / ViewModel / UseCases)

- Use JUnit + `kotlinx-coroutines-test` for ViewModel & coroutines.
- Keep business logic in `UseCase` classes for easy testing.

```kotlin
@get:Rule
val instantExecutorRule = InstantTaskExecutorRule()

@Test
fun testIncrement() = runTest {
    val vm = CounterViewModel()
    vm.increment()
    assertEquals(1, vm.count)
}

Dependencies:

testImplementation "junit:junit:4.13.2"
testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:1.9.0"

1.2 Instrumented UI tests (Compose UI tests)

Use androidx.compose.ui:ui-test-junit4 and Espresso for broader UI interactions.

@get:Rule
val composeTestRule = createAndroidComposeRule<MainActivity>()

@Test
fun counterIncrements() {
    composeTestRule.onNodeWithText("0").assertExists()
    composeTestRule.onNodeWithText("+").performClick()
    composeTestRule.onNodeWithText("1").assertExists()
}


Dependencies:

androidTestImplementation "androidx.compose.ui:ui-test-junit4:1.7.0"
androidTestImplementation "androidx.test.espresso:espresso-core:3.5.1"

1.3 Integration tests (Repository + DB)

Use an in-memory Room DB for tests (Room.inMemoryDatabaseBuilder).

Mock Retrofit with MockWebServer for predictable responses.

🔁 2. CI/CD Pipeline (GitHub Actions example)

A simple pipeline:

Run unit tests

Run lint & ktlint

Build debug & assemble release

Run UI tests (optional, needs emulator or Firebase Test Lab)

Publish AAB to Play Store (via Gradle + service account)

2.1 Example .github/workflows/android.yml
name: Android CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17
      - name: Cache Gradle
        uses: actions/cache@v4
        with:
          path: ~/.gradle/caches
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
      - name: Build & Run tests
        run: ./gradlew build connectedAndroidTest --no-daemon


Tip: For UI tests use Firebase Test Lab or self-hosted emulators.

🔐 3. App Signing & Release (AAB recommended)
3.1 Create signing key
keytool -genkey -v -keystore release-keystore.jks -alias my_app -keyalg RSA -keysize 2048 -validity 10000

3.2 Configure signingConfigs in build.gradle
signingConfigs {
    release {
        storeFile file("keystores/release-keystore.jks")
        storePassword keystoreProperties['storePassword']
        keyAlias keystoreProperties['keyAlias']
        keyPassword keystoreProperties['keyPassword']
    }
}
buildTypes {
    release {
        signingConfig signingConfigs.release
        isMinifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
}


Security: Do not commit keystore or passwords — store credentials as GitHub Secrets or use Gradle properties excluded from VCS.

3.3 Build an AAB
./gradlew bundleRelease
# output: app/build/outputs/bundle/release/app-release.aab

🚀 4. Publishing to Google Play Console

Create a Play Developer account.

Create an app entry; fill store listing, content rating, privacy policy.

Upload .aab in Production or testing tracks (internal/closed).

Provide signing key (Play App Signing recommended — upload key or opt-in).

Roll out staged releases to monitor crashes.

Notes:

Use internal testing track for fast iterations.

Gradually roll out to a percentage to catch issues before 100% release.

📈 5. Monitoring & Crash Reporting

Use Firebase Crashlytics for crash reporting.

Use Firebase Performance Monitoring for network and rendering traces.

Use Google Analytics / Firebase Analytics for user events.

Integrate log aggregation (Sentry, Bugsnag) if needed.

🛡 6. Privacy & Permissions

Request runtime permissions only when needed.

Provide a clear privacy policy (Play requires it if you access personal data).

Follow Play Store policies on data collection and ads.

⚖ 7. Native Android vs React Native: Deployment & Maintenance Differences
Topic	Native (Kotlin)	React Native
Build artifacts	.aab via Gradle	.aab still produced, but JS bundle included
App size	Usually smaller, fully native	Slightly larger due to JS runtime + native libs
Hotfixes	Rebuild + release (fast with staged rollouts)	Can use CodePush-like services (but policy considerations)
Native APIs	Full access, first-class features	Bridged via native modules (may need to write native code)
Debugging	Android Studio profilers	Flipper + React DevTools
Performance	Best (native rendering)	Very good for most apps; tight loops/native animations may need optimization
CI complexity	Standard Gradle pipelines	Additional JS toolchain + Node/npm steps
Testing	Espresso, Compose UI tests	Jest (JS), Detox (E2E), plus native tests
🧰 8. Release Checklist

 All unit and UI tests pass

 Proguard/R8 rules verified

 Crashlytics initialized

 Privacy & permission texts ready

 Screenshots, icons, marketing assets prepared

 Analytics events set up

 Internal testing + beta rollout successful

✅ 9. Post-Release: Maintenance & Updates

Monitor Crashlytics and performance dashboards.

Use staged rollouts to minimize blast radius.

Patch critical bugs quickly with hotfix release (or JS patch if RN and policy allows).

Regularly update dependencies and SDKs (Android Gradle Plugin, Compose, Hilt).
