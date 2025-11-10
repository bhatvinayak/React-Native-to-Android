# 🌐 Part 4: Networking and Data Persistence in Jetpack Compose

This section covers how to fetch data from an API using **Retrofit** and store it locally using **Room Database**, all while following best practices with **Coroutines** and **ViewModel**.

If you know React Native, you can think of this as learning how to do **`fetch()` + local async storage + Redux store updates** — but the **native way** in Kotlin.

---

## 📦 1. Dependencies Setup

Add the following in your `app/build.gradle`:

```gradle
// Retrofit for Networking
implementation("com.squareup.retrofit2:retrofit:2.11.0")
implementation("com.squareup.retrofit2:converter-gson:2.11.0")

// Coroutines for async
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")

// Lifecycle ViewModel & Compose integration
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.0")

// Room for local persistence
implementation("androidx.room:room-runtime:2.6.1")
kapt("androidx.room:room-compiler:2.6.1")
implementation("androidx.room:room-ktx:2.6.1")

⚡ 2. Networking with Retrofit (API Calls)
🧱 Define a Data Model
data class Post(
    val userId: Int,
    val id: Int,
    val title: String,
    val body: String
)

🌐 Create Retrofit API Interface
import retrofit2.http.GET

interface ApiService {
    @GET("posts")
    suspend fun getPosts(): List<Post>
}

⚙️ Create Retrofit Instance
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {
    private const val BASE_URL = "https://jsonplaceholder.typicode.com/"

    val api: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}

🧠 React Native analogy:

In React Native, this would look like:

const response = await fetch("https://jsonplaceholder.typicode.com/posts");
const posts = await response.json();

🧩 3. Using ViewModel + Coroutines for API Calls
class PostViewModel : ViewModel() {
    private val _posts = mutableStateOf<List<Post>>(emptyList())
    val posts: State<List<Post>> = _posts

    private val _isLoading = mutableStateOf(false)
    val isLoading: State<Boolean> = _isLoading

    init {
        fetchPosts()
    }

    private fun fetchPosts() {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                val response = RetrofitClient.api.getPosts()
                _posts.value = response
            } catch (e: Exception) {
                println("Error: ${e.message}")
            } finally {
                _isLoading.value = false
            }
        }
    }
}


🧠 React Native analogy:

const [posts, setPosts] = useState([]);
useEffect(() => {
  fetchPosts();
}, []);

🖥️ UI to Display Posts
@Composable
fun PostListScreen(viewModel: PostViewModel = viewModel()) {
    val posts = viewModel.posts.value
    val loading = viewModel.isLoading.value

    if (loading) {
        Box(modifier = Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
            CircularProgressIndicator()
        }
    } else {
        LazyColumn {
            items(posts) { post ->
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(8.dp),
                    elevation = CardDefaults.cardElevation(4.dp)
                ) {
                    Column(modifier = Modifier.padding(16.dp)) {
                        Text(post.title, fontSize = 18.sp, fontWeight = FontWeight.Bold)
                        Text(post.body, fontSize = 14.sp)
                    }
                }
            }
        }
    }
}


🧠 React Native analogy:

<FlatList
  data={posts}
  renderItem={({ item }) => (
    <View>
      <Text>{item.title}</Text>
      <Text>{item.body}</Text>
    </View>
  )}
/>

💾 4. Room Database (Local Persistence)

Room is Android’s native database layer over SQLite.

It’s like AsyncStorage or MMKV in React Native — but type-safe, efficient, and built-in.

🧱 Step 1: Define Entity
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "posts")
data class PostEntity(
    @PrimaryKey val id: Int,
    val title: String,
    val body: String
)

📜 Step 2: Define DAO (Data Access Object)
import androidx.room.*

@Dao
interface PostDao {
    @Query("SELECT * FROM posts")
    suspend fun getAllPosts(): List<PostEntity>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(posts: List<PostEntity>)
}

🏗️ Step 3: Create Database Class
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import android.content.Context

@Database(entities = [PostEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun postDao(): PostDao

    companion object {
        @Volatile private var INSTANCE: AppDatabase? = null

        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "posts_db"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}

🧠 React Native analogy:

Using local DB via SQLite or AsyncStorage:

await AsyncStorage.setItem("posts", JSON.stringify(posts));

💡 Step 4: Combine API + Room in ViewModel
class CachedPostViewModel(application: Application) : AndroidViewModel(application) {
    private val dao = AppDatabase.getDatabase(application).postDao()

    private val _posts = mutableStateOf<List<PostEntity>>(emptyList())
    val posts: State<List<PostEntity>> = _posts

    init {
        fetchAndCachePosts()
    }

    private fun fetchAndCachePosts() {
        viewModelScope.launch {
            try {
                val cachedPosts = dao.getAllPosts()
                if (cachedPosts.isNotEmpty()) {
                    _posts.value = cachedPosts
                }

                val remotePosts = RetrofitClient.api.getPosts()
                val postEntities = remotePosts.map { PostEntity(it.id, it.title, it.body) }
                dao.insertAll(postEntities)
                _posts.value = postEntities
            } catch (e: Exception) {
                println("Error: ${e.message}")
            }
        }
    }
}


🧠 React Native analogy:

useEffect(() => {
  const cached = JSON.parse(await AsyncStorage.getItem("posts"));
  if (cached) setPosts(cached);
  const data = await fetchFromAPI();
  setPosts(data);
  AsyncStorage.setItem("posts", JSON.stringify(data));
}, []);

🖥️ Step 5: UI for Cached Posts
@Composable
fun CachedPostScreen(viewModel: CachedPostViewModel = viewModel()) {
    val posts = viewModel.posts.value

    LazyColumn {
        items(posts) { post ->
            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(8.dp),
                elevation = CardDefaults.cardElevation(4.dp)
            ) {
                Column(modifier = Modifier.padding(16.dp)) {
                    Text(post.title, fontSize = 18.sp, fontWeight = FontWeight.Bold)
                    Text(post.body, fontSize = 14.sp)
                }
            }
        }
    }
}

⚙️ 5. Architecture Summary
View (Composable UI)
       ↓
ViewModel (holds UI state + business logic)
       ↓
Repository (handles API + DB logic)
       ↓
Retrofit (Remote API) & Room (Local DB)


🧠 React Native analogy:

Component
  ↓
useContext / Redux store
  ↓
Service / API layer
  ↓
fetch() + AsyncStorage

✅ Summary
Feature	Jetpack Compose	React Native Equivalent
Network request	Retrofit + Coroutines	fetch / axios
Local storage	Room Database	AsyncStorage / SQLite
Global state	ViewModel	Redux / Context API
Async handling	Coroutines	Promises / async-await
Offline cache	Room + DAO	AsyncStorage + API sync
