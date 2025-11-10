🧱 Part 8: Advanced Jetpack Compose & App Architecture
🧩 1. Clean Architecture Overview

Clean Architecture = layered separation of concerns:

Layer	Purpose	React Native Equivalent
UI (Presentation)	Compose screens, UI state, events	Screens / Components
Domain	Business logic, Use cases	Services or Redux Thunks
Data	Data sources (local + network)	AsyncStorage + API calls
// Folder structure
com.example.myapp
│
├── data
│   ├── repository
│   ├── local
│   └── remote
│
├── domain
│   ├── model
│   └── usecase
│
└── presentation
    ├── ui
    ├── viewmodel
    └── navigation

🧠 2. Managing UI State (Compose way)

Compose encourages single source of truth for UI.

Example: UiState pattern
data class ProfileUiState(
    val name: String = "",
    val age: Int = 0,
    val loading: Boolean = false,
    val error: String? = null
)


Then in your ViewModel:

class ProfileViewModel : ViewModel() {
    var uiState by mutableStateOf(ProfileUiState())
        private set

    fun loadProfile() {
        viewModelScope.launch {
            uiState = uiState.copy(loading = true)
            delay(1000)
            uiState = uiState.copy(name = "Vinayak", age = 25, loading = false)
        }
    }
}


And in your Composable:

@Composable
fun ProfileScreen(viewModel: ProfileViewModel = viewModel()) {
    val state = viewModel.uiState

    if (state.loading) CircularProgressIndicator()
    else Text(text = "Hi ${state.name}, Age ${state.age}")
}


🟢 RN Parallel: Similar to managing component state with useState or Redux global state.

🔄 3. Handling Side Effects

Side effects in Compose = anything that affects outside the composable function.

Compose Function	Use Case	React Native Equivalent
LaunchedEffect	Run coroutine when key changes	useEffect
rememberCoroutineScope	Launch coroutine tied to composition	useEffect + useCallback combo
DisposableEffect	Cleanup when leaving composition	useEffect cleanup

Example:

@Composable
fun TimerScreen() {
    var time by remember { mutableStateOf(0) }

    LaunchedEffect(Unit) {
        while(true) {
            delay(1000)
            time++
        }
    }

    Text(text = "Seconds passed: $time")
}

🧱 4. UI Performance Optimization

Use remember to avoid recomposition

Use derivedStateOf for computed properties

Hoist state upwards (just like lifting state up in React)

Avoid creating heavy objects inside Composables

Prefer LazyColumn for large lists (like React Native’s FlatList)

⚡ 5. Building a Modular Compose App

You can split your Compose project into modules (like RN monorepo with separate packages).

Example:

app/
feature-login/
feature-profile/
core-network/
core-ui/


Each module has its own dependencies and can be tested independently. This improves scalability and build times.

🔧 6. Example – Modular Compose App Flow

Let’s simulate a simple Login → Dashboard flow.

1️⃣ Domain Layer

data class User(val id: Int, val name: String)


2️⃣ UseCase

class LoginUseCase {
    suspend fun execute(username: String, password: String): User {
        delay(1000)
        return User(1, username)
    }
}


3️⃣ ViewModel

class LoginViewModel(
    private val loginUseCase: LoginUseCase = LoginUseCase()
) : ViewModel() {

    var uiState by mutableStateOf(LoginUiState())
        private set

    fun login(username: String, password: String) {
        viewModelScope.launch {
            uiState = uiState.copy(loading = true)
            val user = loginUseCase.execute(username, password)
            uiState = uiState.copy(loading = false, user = user)
        }
    }
}


4️⃣ Compose UI

@Composable
fun LoginScreen(viewModel: LoginViewModel = viewModel()) {
    val state = viewModel.uiState

    Column {
        if (state.loading) CircularProgressIndicator()
        else Text("Welcome ${state.user?.name ?: "Guest"}")

        Button(onClick = { viewModel.login("Vinayak", "1234") }) {
            Text("Login")
        }
    }
}


🟢 RN Equivalent: This is very similar to how you’d structure logic in React Native with:

Context + Reducers or Redux store for data

Component hooks for lifecycle (useEffect)

Navigation between screens using React Navigation

📦 7. Recommended Libraries to Learn Next
Category	Library	Purpose
Navigation	Jetpack Navigation	Manage app routes
DI	Hilt	Inject dependencies
Async	Kotlin Coroutines, Flow	Handle async data
Network	Retrofit, OkHttp	API calls
DB	Room, DataStore	Local persistence
Testing	JUnit, Espresso, Compose UI Test	Unit/UI testing
