📘 Part 10 – Building a Real-World App: Weather Forecast

Goal: Build a complete app using Retrofit + Room + Flow + Hilt + Jetpack Compose
React Native Equivalent:
Think of it as building a Weather app using Axios (API calls), AsyncStorage (local cache), Redux (state flow), and useEffect (lifecycle).

🧱 1. App Overview

We’ll build a Weather Forecast app that:

Fetches weather from an API (e.g., OpenWeatherMap)

Caches last fetched weather data using Room

Displays it using Jetpack Compose

Handles offline mode gracefully

App layers:

data/
 ├── local/ (Room)
 ├── remote/ (Retrofit)
 └── repository/
domain/
 ├── model/
 └── usecase/
presentation/
 ├── ui/
 └── viewmodel/

⚙️ 2. Dependencies (Gradle)
dependencies {
    implementation "androidx.compose.ui:ui:1.7.0"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.8.0"
    implementation "androidx.activity:activity-compose:1.9.0"

    // Networking
    implementation "com.squareup.retrofit2:retrofit:2.11.0"
    implementation "com.squareup.retrofit2:converter-gson:2.11.0"
    implementation "com.squareup.okhttp3:logging-interceptor:4.12.0"

    // Room
    implementation "androidx.room:room-runtime:2.7.0"
    kapt "androidx.room:room-compiler:2.7.0"
    implementation "androidx.room:room-ktx:2.7.0"

    // Hilt (Dependency Injection)
    implementation "com.google.dagger:hilt-android:2.52"
    kapt "com.google.dagger:hilt-compiler:2.52"

    // Coroutines
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0"
}

🌐 3. Remote Data Source (Retrofit)
interface WeatherApi {
    @GET("weather")
    suspend fun getWeather(
        @Query("q") city: String,
        @Query("appid") apiKey: String = YOUR_API_KEY,
        @Query("units") units: String = "metric"
    ): WeatherResponse
}

data class WeatherResponse(
    val name: String,
    val main: Main,
)

data class Main(val temp: Double, val humidity: Int)


🟢 RN Parallel:
Just like defining an Axios API helper and returning a JSON response.

💾 4. Local Data Source (Room)
@Entity(tableName = "weather")
data class WeatherEntity(
    @PrimaryKey val city: String,
    val temperature: Double,
    val humidity: Int
)

@Dao
interface WeatherDao {
    @Query("SELECT * FROM weather WHERE city = :city")
    fun getWeather(city: String): Flow<WeatherEntity?>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertWeather(weather: WeatherEntity)
}


🟢 RN Parallel:
Like saving API results to AsyncStorage or SQLite.

🧩 5. Repository

Handles logic between remote + local data.

class WeatherRepository @Inject constructor(
    private val api: WeatherApi,
    private val dao: WeatherDao
) {
    fun getWeather(city: String): Flow<WeatherEntity?> = dao.getWeather(city)

    suspend fun refreshWeather(city: String) {
        val response = api.getWeather(city)
        val entity = WeatherEntity(city, response.main.temp, response.main.humidity)
        dao.insertWeather(entity)
    }
}


🟢 RN Parallel:
Equivalent to a Redux thunk that fetches and caches data locally.

🧠 6. ViewModel
@HiltViewModel
class WeatherViewModel @Inject constructor(
    private val repository: WeatherRepository
): ViewModel() {

    private val _city = MutableStateFlow("Bangalore")
    val weather = _city.flatMapLatest { city ->
        repository.getWeather(city)
    }.stateIn(viewModelScope, SharingStarted.WhileSubscribed(), null)

    fun refresh(city: String) {
        viewModelScope.launch {
            repository.refreshWeather(city)
            _city.value = city
        }
    }
}


🟢 RN Parallel:
Like using a Redux selector or custom hook to fetch + refresh data.

🎨 7. Compose UI
@Composable
fun WeatherScreen(viewModel: WeatherViewModel = hiltViewModel()) {
    val weather by viewModel.weather.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.Center
    ) {
        Text(text = "Weather in ${weather?.city ?: "Loading..."}")
        weather?.let {
            Text(text = "Temperature: ${it.temperature}°C")
            Text(text = "Humidity: ${it.humidity}%")
        }

        Spacer(Modifier.height(16.dp))
        Button(onClick = { viewModel.refresh("Bangalore") }) {
            Text("Refresh")
        }
    }
}


🟢 RN Parallel:
Equivalent to a React Native screen with:

const [weather, setWeather] = useState()
useEffect(() => fetchWeather(), [])

🔌 8. Dependency Injection with Hilt
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides @Singleton
    fun provideRetrofit(): WeatherApi = Retrofit.Builder()
        .baseUrl("https://api.openweathermap.org/data/2.5/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()
        .create(WeatherApi::class.java)

    @Provides @Singleton
    fun provideDatabase(app: Application) = Room.databaseBuilder(
        app, WeatherDatabase::class.java, "weather_db"
    ).build()

    @Provides
    fun provideDao(db: WeatherDatabase) = db.weatherDao()
}

🧩 9. Final Flow Summary
WeatherScreen
    ↓ observes
WeatherViewModel
    ↓ triggers
WeatherRepository
    ↓ combines
Retrofit (network) + Room (cache)


🟢 React Native Analogy:

Screen -> Redux thunk -> API + AsyncStorage -> Update store -> UI refreshes

✅ Summary

You now have:

A working app architecture (data + UI)

Reactive updates via Flow

Dependency injection with Hilt

Offline caching via Room
