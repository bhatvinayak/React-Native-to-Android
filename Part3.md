# 🧭 Part 3: Navigation & State Management in Jetpack Compose

This section teaches how to handle **multi-screen navigation**, **state hoisting**, and **ViewModel integration** in Jetpack Compose — all with parallels to **React Native concepts** for easier understanding.

---

## 📘 1. Navigation in Jetpack Compose

Jetpack Compose uses the **Navigation Component** (from `androidx.navigation.compose`) to move between screens — similar to **React Navigation**.

### 🧩 Setup

Add dependency in your `build.gradle` (Module: app):

```gradle
implementation("androidx.navigation:navigation-compose:2.8.0")

🧱 Create Screens
@Composable
fun HomeScreen(navController: NavHostController) {
    Column(modifier = Modifier.fillMaxSize(), horizontalAlignment = Alignment.CenterHorizontally, verticalArrangement = Arrangement.Center) {
        Text("Home Screen", fontSize = 24.sp)
        Button(onClick = { navController.navigate("details/Vinayak") }) {
            Text("Go to Details")
        }
    }
}

@Composable
fun DetailsScreen(name: String?) {
    Box(modifier = Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        Text("Hello, $name!", fontSize = 24.sp)
    }
}


🧠 React Native analogy:

function HomeScreen({ navigation }) {
  return (
    <View>
      <Text>Home Screen</Text>
      <Button title="Go to Details" onPress={() => navigation.navigate('Details', { name: 'Vinayak' })} />
    </View>
  );
}

🗺️ Set Up Navigation Host
@Composable
fun AppNavHost() {
    val navController = rememberNavController()

    NavHost(navController, startDestination = "home") {
        composable("home") { HomeScreen(navController) }
        composable("details/{name}") { backStackEntry ->
            val name = backStackEntry.arguments?.getString("name")
            DetailsScreen(name)
        }
    }
}


Now use this in your MainActivity:

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppNavHost()
        }
    }
}


🧠 React Native analogy:

<Stack.Navigator initialRouteName="Home">
  <Stack.Screen name="Home" component={HomeScreen} />
  <Stack.Screen name="Details" component={DetailsScreen} />
</Stack.Navigator>

⚙️ 2. State Management in Jetpack Compose
🧠 Local State (remember, mutableStateOf)

For simple UI state (like form inputs, toggles, counters):

@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }

    Column(horizontalAlignment = Alignment.CenterHorizontally, verticalArrangement = Arrangement.Center) {
        Text("Count: $count", fontSize = 28.sp)
        Button(onClick = { count++ }) { Text("Increase") }
    }
}


🧠 React Native analogy:

const [count, setCount] = useState(0);

📦 State Hoisting (Lifting State Up)

In Compose, state hoisting means passing data + events from parent to child — similar to React props.

@Composable
fun CounterParent() {
    var count by remember { mutableStateOf(0) }
    CounterDisplay(count = count, onIncrement = { count++ })
}

@Composable
fun CounterDisplay(count: Int, onIncrement: () -> Unit) {
    Column(horizontalAlignment = Alignment.CenterHorizontally) {
        Text("Count: $count", fontSize = 24.sp)
        Button(onClick = onIncrement) { Text("Add") }
    }
}


🧠 React analogy:

function CounterParent() {
  const [count, setCount] = useState(0);
  return <CounterDisplay count={count} onIncrement={() => setCount(count + 1)} />;
}

🧰 3. ViewModel (Like Redux or Context in React)

For app-wide state or logic that survives configuration changes (like screen rotation), you use a ViewModel.

Add dependency:

implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.0")

Example:
class CounterViewModel : ViewModel() {
    private val _count = mutableStateOf(0)
    val count: State<Int> = _count

    fun increment() {
        _count.value++
    }
}

@Composable
fun CounterWithViewModel(viewModel: CounterViewModel = viewModel()) {
    Column(horizontalAlignment = Alignment.CenterHorizontally, verticalArrangement = Arrangement.Center) {
        Text("Count: ${viewModel.count.value}", fontSize = 28.sp)
        Button(onClick = { viewModel.increment() }) {
            Text("Increase")
        }
    }
}


🧠 React Native analogy:

const { count, increment } = useCounterStore(); // from Redux/Zustand/Context

🔄 4. Side Effects in Compose

Compose side-effects are similar to React’s useEffect.

Function	Purpose	React Equivalent
LaunchedEffect	Runs coroutine when key changes	useEffect
DisposableEffect	Runs setup & cleanup	useEffect with cleanup
rememberCoroutineScope()	Launch coroutines manually	useEffect(async...)
Example:
@Composable
fun FetchDataScreen() {
    LaunchedEffect(Unit) {
        println("Fetching data...")
    }
}


🧠 React analogy:

useEffect(() => {
  console.log("Fetching data...");
}, []);

🎯 5. Putting It All Together
@Composable
fun AppNavHost() {
    val navController = rememberNavController()

    NavHost(navController, startDestination = "counter") {
        composable("counter") { CounterWithViewModelScreen(navController) }
        composable("summary/{count}") { backStackEntry ->
            val count = backStackEntry.arguments?.getString("count")
            SummaryScreen(count)
        }
    }
}

@Composable
fun CounterWithViewModelScreen(navController: NavHostController) {
    val viewModel: CounterViewModel = viewModel()
    Column(horizontalAlignment = Alignment.CenterHorizontally, verticalArrangement = Arrangement.Center) {
        Text("Count: ${viewModel.count.value}", fontSize = 30.sp)
        Button(onClick = { viewModel.increment() }) { Text("Increase") }
        Button(onClick = { navController.navigate("summary/${viewModel.count.value}") }) {
            Text("View Summary")
        }
    }
}

@Composable
fun SummaryScreen(count: String?) {
    Box(modifier = Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        Text("Final Count: $count", fontSize = 32.sp)
    }
}


🧠 React analogy (navigation + state via context/Redux):

function CounterScreen({ navigation }) {
  const [count, setCount] = useState(0);
  return (
    <View>
      <Text>Count: {count}</Text>
      <Button title="Increase" onPress={() => setCount(count + 1)} />
      <Button title="Summary" onPress={() => navigation.navigate('Summary', { count })} />
    </View>
  );
}

✅ Summary
Concept	Jetpack Compose	React Native Equivalent
Navigation	NavHost + composable()	React Navigation
Local state	remember { mutableStateOf() }	useState()
Shared state	ViewModel	Redux / Context
Side effects	LaunchedEffect, DisposableEffect	useEffect()
State hoisting	Passing props to child composables	Lifting state up
