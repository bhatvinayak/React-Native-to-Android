📘 Part 9 – Coroutines, Flow & Asynchronous Architecture

Focus: Kotlin concurrency, async data streams, error handling, offline patterns
Relates to: React Native’s async/await, Promises, and Observables (RxJS)

🧠 1. Kotlin Coroutines – Overview

Coroutines = lightweight threads for asynchronous programming in Kotlin.

🟢 React Native Equivalent:

coroutineScope.launch {} ≈ async function / await

viewModelScope.launch {} ≈ using async calls within React hooks or Redux Thunks

Example: Simple coroutine
fun main() = runBlocking {
    launch {
        delay(1000)
        println("Hello after 1 second!")
    }
    println("Hi immediately!")
}


Output:

Hi immediately!
Hello after 1 second!

🧩 2. Coroutine Builders
Builder	Purpose	React Native Equivalent
launch	Fire-and-forget	async function without return
async	Return a result	Promise / async function return
runBlocking	Blocking coroutine (testing)	not common in RN

Example:

val result = async { fetchData() }
println(result.await())

⚙️ 3. Scopes

Scopes define lifecycle for coroutines.

Scope	Usage	React Native Analogy
GlobalScope	App-wide (avoid for production)	Background JS thread
viewModelScope	Cancels on ViewModel clear	Component unmount cleanup
lifecycleScope	Tied to Activity/Fragment	useEffect cleanup
🌊 4. Kotlin Flow – Asynchronous Data Stream

Flow emits data over time (like RxJS Observables).

🟢 React Native Equivalent:
Like an observable emitting state changes that components subscribe to.

Example:

fun getNumbers(): Flow<Int> = flow {
    for (i in 1..5) {
        emit(i)
        delay(1000)
    }
}


Collect it:

viewModelScope.launch {
    getNumbers().collect { value ->
        println("Received: $value")
    }
}

🔁 5. Combining Flow + Room + Retrofit

Flows work beautifully for reactive architecture:

Room can expose database queries as Flow

Retrofit can fetch data and update the DB

Compose UI can observe those flows automatically

@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
}


In ViewModel:

val users = userDao.getAllUsers()


In Compose:

@Composable
fun UserList(viewModel: UserViewModel = viewModel()) {
    val users by viewModel.users.collectAsState(initial = emptyList())
    LazyColumn {
        items(users) { user ->
            Text(user.name)
        }
    }
}


🟢 RN Parallel:
Similar to using a subscription in Redux or Recoil selector that re-renders when data changes.

⚡ 6. Error Handling in Coroutines

Use try/catch inside coroutine:

viewModelScope.launch {
    try {
        val user = repository.getUser()
    } catch (e: Exception) {
        _state.value = UiState(error = e.message)
    }
}


Or use Flow operators:

repository.getUserFlow()
    .catch { emit(Result.Error(it)) }
    .collect { emit(Result.Success(it)) }

🧱 7. Offline-first Architecture

Steps:

Fetch from local DB (Room)

Fetch from network (Retrofit)

Update DB

UI observes DB flow → always shows latest data

RN analogy → Like AsyncStorage + API call + Redux state

🧪 8. Testing Coroutines & Flows

Use runTest {} for coroutine testing.

@Test
fun testFlowEmitsValues() = runTest {
    val values = getNumbers().toList()
    assertEquals(listOf(1,2,3,4,5), values)
}

✅ Summary

You learned:

Coroutines (async programming)

Flow (reactive streams)

Scopes, error handling

Offline-first pattern
