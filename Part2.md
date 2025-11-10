# 🧱 PART 2: Jetpack Compose — Modern Android UI (React for Android)

Jetpack Compose is Android’s React Native-style UI toolkit.
It replaces XML layouts (activity_main.xml) with Composable functions written in Kotlin.

## ⚛️ 1. What is Jetpack Compose?

In classic Android, you used XML + Activities:

```
<TextView
    android:id="@+id/title"
    android:text="Hello World" />
```

In Jetpack Compose, you use Kotlin functions that describe the UI:

```
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}
```

🧠 React Native analogy:

```
function Greeting({ name }) {
  return <Text>Hello, {name}!</Text>
}
```

Both are declarative — UI = function(state).

## 🧩 2. The Entry Point (Activity + Compose)

In Compose, your `MainActivity` sets the UI using `setContent {}`.

```
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp()
        }
    }
}
```

```
@Composable
fun MyApp() {
    Greeting("Vinayak")
}
```

🧠 React Native analogy:

```
export default function App() {
  return <Greeting name="Vinayak" />;
}
```

## 💡 3. Composable Functions = React Components

Every function annotated with `@Composable` is like a React component.

```
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!", fontSize = 24.sp)
}
```

They can be nested, reused, and stateful — exactly like React components.

## ⚙️ 4. State in Compose (Like React’s useState)

Compose uses `remember` and `mutableStateOf` for reactive state.

```
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }

    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Increase")
        }
    }
}
```

🧠 React Native analogy:

```
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <View>
      <Text>Count: {count}</Text>
      <Button title="Increase" onPress={() => setCount(count + 1)} />
    </View>
  );
}
```

## 🧭 5. Navigation in Compose

Instead of using `Intents`, Compose has a Navigation library (just like React Navigation).

```
@Composable
fun AppNavHost(navController: NavHostController) {
    NavHost(navController, startDestination = "home") {
        composable("home") { HomeScreen(navController) }
        composable("details/{name}") { backStackEntry ->
            val name = backStackEntry.arguments?.getString("name")
            DetailsScreen(name)
        }
    }
}
```

🧠 React Native analogy:

```
<Stack.Navigator initialRouteName="Home">
  <Stack.Screen name="Home" component={HomeScreen} />
  <Stack.Screen name="Details" component={DetailsScreen} />
</Stack.Navigator>
```

Navigation is driven by `navController.navigate("details/Vinayak")`

## 🧱 6. Layouts — Row, Column, Box

In Compose, instead of \<View> and \<Text>, you use Column, Row, and Box.

|Compose	| React Native|
|---------|-------------|
|Column	|<View style={{ flexDirection: 'column' }}>|
|Row	|<View style={{ flexDirection: 'row' }}>|
|Box	|<View style={{ position: 'relative' }}>|
|Text	| \<Text> |
|Button	| \<Button> |

Example:

```
@Composable
fun ProfileCard(name: String) {
    Row(modifier = Modifier.padding(16.dp)) {
        Icon(Icons.Default.Person, contentDescription = null)
        Spacer(modifier = Modifier.width(8.dp))
        Text("Hello, $name")
    }
}
```

🧠 React Native analogy:

```
<View style={{ flexDirection: 'row', padding: 16 }}>
  <Icon name="person" />
  <Text>Hello, {name}</Text>
</View>
```

## 🎨 7. Modifiers — Like Styles / Props in React Native

Compose uses Modifiers to define layout, padding, color, etc.

```
Text(
    text = "Hello Compose!",
    modifier = Modifier
        .padding(16.dp)
        .background(Color.Yellow)
        .fillMaxWidth()
)
```

🧠 React Native analogy:

```
<Text style={{ padding: 16, backgroundColor: 'yellow', width: '100%' }}>
  Hello Compose!
</Text>
```

## 🧠 8. Recomposition (How Compose Reacts to State)

In React, when setState is called → component re-renders.
In Compose, when state changes (mutableStateOf), the Composable recomposes automatically.

Example:

```
@Composable
fun ToggleGreeting() {
    var isVisible by remember { mutableStateOf(true) }

    Column {
        if (isVisible) Text("Hello Vinayak!")
        Button(onClick = { isVisible = !isVisible }) {
            Text("Toggle")
        }
    }
}
```

🧠 Equivalent React code:

```
function ToggleGreeting() {
  const [isVisible, setVisible] = useState(true);

  return (
    <View>
      {isVisible && <Text>Hello Vinayak!</Text>}
      <Button title="Toggle" onPress={() => setVisible(!isVisible)} />
    </View>
  );
}
```

## 🧰 9. Side Effects — Like useEffect

To perform side effects (e.g. fetch data, start animations), Compose uses:

| Function |	Purpose	React Equivalent |
|-----------|---------------------------|
|LaunchedEffect |	Run suspend functions (like async) when key changes	useEffect|
|DisposableEffect |	Cleanup resources	useEffect cleanup|
|rememberCoroutineScope() |	Launch coroutines	useEffect(async...)|

Example:

```
@Composable
fun FetchData() {
    LaunchedEffect(Unit) {
        println("Fetching data from API...")
    }
}
```

## 🔄 10. Jetpack Compose vs React Native — Concept Map
|Concept	| React Native	| Jetpack Compose| 
|---------|---------------|----------------|
|UI component |	JSX	| Composable function|
|State |	useState	| remember { mutableStateOf() }|
|Effect |	useEffect	| LaunchedEffect, DisposableEffect|
|Navigation |	React Navigation |	Jetpack Navigation|
|Styling |	StyleSheet |	Modifier|
|Layout |	Flexbox |	Row / Column / Box|
|Background | thread	JS thread + native bridge |	Kotlin coroutines|
|Re-render |	setState() → re-render |	mutableState → recomposition|


⚡ Example Mini App (like React Native counter app)

```
@Composable
fun CounterApp() {
    var count by remember { mutableStateOf(0) }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text("Count: $count", fontSize = 30.sp)
        Spacer(Modifier.height(16.dp))
        Button(onClick = { count++ }) {
            Text("Increase")
        }
    }
}
```

🧠 React Native equivalent:

```
function CounterApp() {
  const [count, setCount] = useState(0);

  return (
    <View style={styles.centered}>
      <Text style={{ fontSize: 30 }}>Count: {count}</Text>
      <Button title="Increase" onPress={() => setCount(count + 1)} />
    </View>
  );
}
```

## 🏁 Summary
Jetpack Compose gives you:

- Declarative UI like React

- Reactive State Management

- Kotlin-only code (no XML)

- Type-safe and performant
