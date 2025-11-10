🧩 Part 6: Room, Retrofit & Clean Architecture in Android (with Jetpack Compose)
🎯 Objective

Learn how to handle data persistence, networking, and app architecture best practices using:

Room (Local DB)

Retrofit (API calls)

Repository pattern

Clean Architecture layers

ViewModel integration

🧱 1. Overview of Data Flow

Your app’s data typically moves like this:

UI (Compose)
   ⬆️
ViewModel (Logic)
   ⬆️
Repository (Abstraction)
   ⬆️
RemoteDataSource (Retrofit) + LocalDataSource (Room)


🧠 React Native Analogy:
This is like having:

Screens = UI

Hooks / Context = ViewModel

services/api.js + db/storage.js = Data Sources

🧩 2. Room — Local Database (Offline Storage)
🗃️ Step 1: Add dependencies
implementation("androidx.room:room-runtime:2.6.1")
kapt("androidx.room:room-compiler:2.6.1")
implementation("androidx.room:room-ktx:2.6.1")

🧩 Step 2: Define Entity
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val name: String,
    val email: String
)

🧩 Step 3: Define DAO (Data Access Object)
@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: UserEntity)

    @Query("SELECT * FROM users")
    suspend fun getAllUsers(): List<UserEntity>
}

🧩 Step 4: Create Database
@Database(entities = [UserEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}

🧩 Step 5: Provide with Hilt
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    @Provides
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "my_app_db"
        ).build()
    }

    @Provides
    fun provideUserDao(db: AppDatabase): UserDao = db.userDao()
}

🌐 3. Retrofit — API Calls
🔹 Step 1: Add dependencies
implementation("com.squareup.retrofit2:retrofit:2.11.0")
implementation("com.squareup.retrofit2:converter-gson:2.11.0")

🔹 Step 2: Define API Interface
interface ApiService {
    @GET("users")
    suspend fun getUsers(): List<UserResponse>
}

data class UserResponse(
    val id: Int,
    val name: String,
    val email: String
)

🔹 Step 3: Provide Retrofit with Hilt
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://jsonplaceholder.typicode.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    @Provides
    fun provideApiService(retrofit: Retrofit): ApiService =
        retrofit.create(ApiService::class.java)
}

🧩 4. Repository — Combine Room + Retrofit
class UserRepository @Inject constructor(
    private val api: ApiService,
    private val dao: UserDao
) {
    suspend fun getUsers(): List<UserEntity> {
        return try {
            val response = api.getUsers()
            val entities = response.map { UserEntity(it.id, it.name, it.email) }
            dao.insertUser(entities.first()) // Insert one for demo
            entities
        } catch (e: Exception) {
            dao.getAllUsers() // fallback to cache
        }
    }
}


🧠 React Native analogy:
Just like you’d fetch from API first, then fallback to AsyncStorage or SQLite.

🧠 5. ViewModel — Manage State
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {

    var users by mutableStateOf<List<UserEntity>>(emptyList())
        private set

    var loading by mutableStateOf(false)
        private set

    var error by mutableStateOf<String?>(null)
        private set

    fun fetchUsers() {
        viewModelScope.launch {
            loading = true
            error = null
            try {
                users = repository.getUsers()
            } catch (e: Exception) {
                error = e.message
            } finally {
                loading = false
            }
        }
    }
}

🧱 6. Compose UI — Display Data
@Composable
fun UserListScreen(viewModel: UserViewModel = hiltViewModel()) {
    val users = viewModel.users
    val loading = viewModel.loading
    val error = viewModel.error

    LaunchedEffect(Unit) {
        viewModel.fetchUsers()
    }

    when {
        loading -> CircularProgressIndicator()
        error != null -> Text("Error: $error")
        else -> LazyColumn {
            items(users) { user ->
                Text("${user.name} (${user.email})", modifier = Modifier.padding(8.dp))
            }
        }
    }
}


🧠 React Native analogy:
This is like:

useEffect(() => { fetchUsers(); }, []);
if (loading) return <ActivityIndicator />;

🧩 7. Clean Architecture Layers Recap
Layer	Responsibility	Example
Domain	Business logic, use cases	FetchUsersUseCase.kt
Data	Repositories, API, DB	UserRepository.kt
Presentation	ViewModel + UI	UserViewModel.kt, UserListScreen.kt
Presentation → Domain → Data


🧠 React Native analogy:
Like separating your project into:

/src/screens
/src/hooks
/src/services

🚀 8. Putting Everything Together (Flow)

User opens app

UserListScreen → calls UserViewModel.fetchUsers()

UserViewModel → calls UserRepository.getUsers()

UserRepository:

Fetches from Retrofit

Saves to Room

Returns data to ViewModel

ViewModel updates state → Compose UI re-renders.

🧠 9. Error Handling & UI States

Create a sealed class for UI states:

sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
}


Use it in ViewModel:

var state by mutableStateOf<UiState<List<UserEntity>>>(UiState.Loading)

fun fetchUsers() {
    viewModelScope.launch {
        try {
            val result = repository.getUsers()
            state = UiState.Success(result)
        } catch (e: Exception) {
            state = UiState.Error(e.message ?: "Unknown error")
        }
    }
}


And in UI:

when (val s = viewModel.state) {
    is UiState.Loading -> CircularProgressIndicator()
    is UiState.Success -> LazyColumn { items(s.data) { Text(it.name) } }
    is UiState.Error -> Text("Error: ${s.message}")
}

🧭 10. Summary
Concept	Tool	Purpose
Local DB	Room	Offline storage
Network	Retrofit	API communication
Data Layer	Repository	Unify DB + API
Logic Layer	ViewModel	State & logic
UI Layer	Compose	Declarative UI
DI	Hilt	Dependency management
