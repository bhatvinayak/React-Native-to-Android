🧠 Part 5: Architecture, ViewModel, LiveData, Flow & Dependency Injection in Android (with Jetpack Compose)
🎯 Objective

Learn how to structure your Android app like a pro — separating UI, logic, and data.
Understand ViewModel, LiveData, Flow, and how Hilt (Dependency Injection) ties everything together.

🧩 1. MVVM Architecture Overview

In React Native, you usually keep:

UI (JSX) → Components

Logic → Hooks / Context

Data → APIs / Redux / Zustand

Android follows a similar structure using MVVM (Model-View-ViewModel):

Layer	Responsibility	React Native Analogy
Model	Data sources, repository, API, DB	Redux store / Backend API
ViewModel	Holds UI state, business logic	Hooks + Context
View (Compose UI)	UI components observing ViewModel	React components
🏗️ Folder Structure Example
app/
 ├── data/
 │    ├── model/
 │    ├── repository/
 │    └── remote/
 ├── ui/
 │    ├── viewmodel/
 │    ├── screens/
 │    └── components/
 └── di/
      └── AppModule.kt

⚙️ 2. ViewModel — UI Logic Holder

A ViewModel survives configuration changes (like screen rotation).
It holds and exposes data to the UI layer.

🧩 Example
// CounterViewModel.kt
package com.example.counterapp.ui.viewmodel

import androidx.lifecycle.ViewModel
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue

class CounterViewModel : ViewModel() {
    var count by mutableStateOf(0)
        private set

    fun increment() {
        count++
    }

    fun decrement() {
        count--
    }
}


Then in your Compose screen:

// CounterScreen.kt
@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text("Count: ${viewModel.count}", fontSize = 30.sp)
        Row {
            Button(onClick = { viewModel.decrement() }) { Text("-") }
            Spacer(modifier = Modifier.width(20.dp))
            Button(onClick = { viewModel.increment() }) { Text("+") }
        }
    }
}


🧠 React Native analogy:
This is like:

const [count, setCount] = useState(0);


but defined in a shared logic layer that survives re-renders.

🔁 3. LiveData & StateFlow — Observing Data
🔹 LiveData

A lifecycle-aware observable data holder.
Used before Compose; now Compose prefers State or Flow.

Example
val _username = MutableLiveData<String>()
val username: LiveData<String> = _username

_username.value = "Vinayak"


To observe it:

val name by viewModel.username.observeAsState("")
Text("Hello $name")

🔹 StateFlow

StateFlow is a newer alternative — Kotlin’s native way to handle streams of data (reactive).

class ProfileViewModel : ViewModel() {
    private val _username = MutableStateFlow("Guest")
    val username = _username.asStateFlow()

    fun updateUsername(newName: String) {
        _username.value = newName
    }
}

@Composable
fun ProfileScreen(viewModel: ProfileViewModel = viewModel()) {
    val name by viewModel.username.collectAsState()
    Text("Hello, $name")
}


🧠 React Native analogy:
Think of StateFlow as Recoil or Zustand store with a live stream of updates.

🌍 4. Repository Pattern — Decoupling Data Source

The Repository abstracts where data comes from (API, database, etc.).

Example
// UserRepository.kt
class UserRepository {
    suspend fun fetchUser(): String {
        delay(1000)
        return "Vinayak"
    }
}

// ProfileViewModel.kt
class ProfileViewModel(private val repository: UserRepository) : ViewModel() {
    private val _username = MutableStateFlow("")
    val username = _username.asStateFlow()

    init {
        viewModelScope.launch {
            _username.value = repository.fetchUser()
        }
    }
}


🧠 React Native analogy:
Like separating your data layer in a services/userService.js.

🧩 5. Dependency Injection (DI) with Hilt

DI helps inject dependencies automatically instead of manually creating them.
This keeps code cleaner and easier to test.

Install Hilt in build.gradle:

implementation("com.google.dagger:hilt-android:2.52")
kapt("com.google.dagger:hilt-compiler:2.52")


Annotate your Application class:

@HiltAndroidApp
class MyApp : Application()


Provide dependencies:

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    fun provideUserRepository(): UserRepository = UserRepository()
}


Inject ViewModel:

@HiltViewModel
class ProfileViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    ...
}


🧠 React Native analogy:
Like using a Context Provider or a dependency injection library such as inversify or react-di.

🧱 6. Putting It All Together

Here’s a minimal MVVM + Hilt + Compose flow:

@HiltAndroidApp
class MyApp : Application()

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides fun provideRepo() = UserRepository()
}

@HiltViewModel
class MainViewModel @Inject constructor(private val repo: UserRepository) : ViewModel() {
    val username = MutableStateFlow("Loading...")

    init {
        viewModelScope.launch {
            username.value = repo.fetchUser()
        }
    }
}

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val viewModel: MainViewModel = hiltViewModel()
            val name by viewModel.username.collectAsState()
            Text("Hello, $name")
        }
    }
}

🚀 Summary
Concept	Description	React Native Analogy
ViewModel	Holds UI state & logic	Hooks / Context
LiveData / StateFlow	Reactive data streams	useState / useEffect
Repository	Data abstraction layer	API service file
Hilt	Dependency Injection	Context provider / DI lib
MVVM	Standard architecture	Clean separation of UI & logic
