🧭 Part 7: Navigation in Jetpack Compose (with React Native Comparison)
🎯 Objective

Understand how navigation works in Jetpack Compose — similar to React Native’s React Navigation.
We’ll cover:

Navigation setup

Moving between screens

Passing arguments

Bottom navigation

Navigation with ViewModel

🧱 1. Adding Navigation Dependency

In your build.gradle (module-level):

implementation("androidx.navigation:navigation-compose:2.8.3")

🧩 2. Basic Navigation Setup

Jetpack Compose uses a NavHost and a NavController.
Think of this like React Native’s NavigationContainer and Stack.Navigator.

🧠 React Native Analogy:
<NavigationContainer>
  <Stack.Navigator>
    <Stack.Screen name="Home" component={HomeScreen} />
    <Stack.Screen name="Detail" component={DetailScreen} />
  </Stack.Navigator>
</NavigationContainer>

🧩 Jetpack Compose Equivalent
@Composable
fun AppNavigation() {
    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = "home"
    ) {
        composable("home") { HomeScreen(navController) }
        composable("details") { DetailsScreen(navController) }
    }
}


Each composable route is like a React Native screen.

🧩 Home Screen Example
@Composable
fun HomeScreen(navController: NavController) {
    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text("🏠 Home Screen", fontSize = 24.sp)
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = { navController.navigate("details") }) {
            Text("Go to Details")
        }
    }
}

🧩 Details Screen Example
@Composable
fun DetailsScreen(navController: NavController) {
    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text("📄 Details Screen", fontSize = 24.sp)
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = { navController.popBackStack() }) {
            Text("Back to Home")
        }
    }
}

🧩 3. Passing Arguments Between Screens

In React Native:

navigation.navigate("Details", { userId: 42 })


In Compose Navigation:

navController.navigate("details/42")


Define your route with a placeholder:

NavHost(navController, startDestination = "home") {
    composable("home") { HomeScreen(navController) }
    composable("details/{userId}") { backStackEntry ->
        val userId = backStackEntry.arguments?.getString("userId")
        DetailsScreen(userId)
    }
}


Screen receiving argument:

@Composable
fun DetailsScreen(userId: String?) {
    Text("User ID: $userId", fontSize = 22.sp)
}


🧠 React Native analogy:
route.params.userId

🧩 4. Type-Safe Navigation (Optional)

Instead of manually typing route strings, define routes as constants or sealed classes.

sealed class Screen(val route: String) {
    object Home : Screen("home")
    object Details : Screen("details/{userId}") {
        fun createRoute(userId: Int) = "details/$userId"
    }
}


Usage:

navController.navigate(Screen.Details.createRoute(101))

🧭 5. Bottom Navigation in Jetpack Compose

React Native analogy:
createBottomTabNavigator() from @react-navigation/bottom-tabs.

🧩 Define Screens
sealed class BottomNavScreen(val route: String, val label: String, val icon: ImageVector) {
    object Home : BottomNavScreen("home", "Home", Icons.Default.Home)
    object Profile : BottomNavScreen("profile", "Profile", Icons.Default.Person)
    object Settings : BottomNavScreen("settings", "Settings", Icons.Default.Settings)
}

🧩 Bottom Navigation Bar
@Composable
fun BottomNavBar(navController: NavController) {
    val items = listOf(
        BottomNavScreen.Home,
        BottomNavScreen.Profile,
        BottomNavScreen.Settings
    )
    NavigationBar {
        val currentRoute = navController.currentBackStackEntryAsState().value?.destination?.route
        items.forEach { screen ->
            NavigationBarItem(
                selected = currentRoute == screen.route,
                onClick = { navController.navigate(screen.route) },
                icon = { Icon(screen.icon, contentDescription = screen.label) },
                label = { Text(screen.label) }
            )
        }
    }
}

🧩 Host Navigation + Bottom Bar
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    Scaffold(
        bottomBar = { BottomNavBar(navController) }
    ) { padding ->
        NavHost(
            navController = navController,
            startDestination = BottomNavScreen.Home.route,
            modifier = Modifier.padding(padding)
        ) {
            composable(BottomNavScreen.Home.route) { Text("Home") }
            composable(BottomNavScreen.Profile.route) { Text("Profile") }
            composable(BottomNavScreen.Settings.route) { Text("Settings") }
        }
    }
}

🧩 6. Navigation with ViewModel

Each screen can have its own ViewModel that persists across navigation — just like React Context shared between screens.

Example:

@HiltViewModel
class ProfileViewModel @Inject constructor() : ViewModel() {
    var username by mutableStateOf("Guest")
    fun updateName(newName: String) { username = newName }
}


Screen:

@Composable
fun ProfileScreen(viewModel: ProfileViewModel = hiltViewModel()) {
    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text("Hello, ${viewModel.username}")
        Button(onClick = { viewModel.updateName("Vinayak") }) {
            Text("Change Name")
        }
    }
}


🧠 React Native analogy:
This is like using useContext or useStore hook that persists across screens.

🧩 7. Navigation Transitions (Optional)

You can use AnimatedNavHost for custom screen transitions using the accompanist-navigation-animation library:

implementation("com.google.accompanist:accompanist-navigation-animation:0.36.0")


Example:

AnimatedNavHost(
    navController,
    startDestination = "home",
    enterTransition = { slideInHorizontally() },
    exitTransition = { slideOutHorizontally() }
) {
    composable("home") { HomeScreen(navController) }
    composable("details") { DetailsScreen(navController) }
}


🧠 React Native analogy:
Like using createStackNavigator with custom cardStyleInterpolator.

🧠 8. Back Navigation

Use:

navController.popBackStack()


To clear backstack:

navController.navigate("home") {
    popUpTo("home") { inclusive = true }
}


🧠 React Native analogy:
navigation.popToTop() or reset().

🚀 9. Complete Example
@Composable
fun AppNavHost() {
    val navController = rememberNavController()
    NavHost(navController, startDestination = "home") {
        composable("home") { HomeScreen(navController) }
        composable("details/{userId}") { backStackEntry ->
            val userId = backStackEntry.arguments?.getString("userId")
            DetailsScreen(userId)
        }
    }
}

@Composable
fun HomeScreen(navController: NavController) {
    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text("🏠 Home Screen", fontSize = 22.sp)
        Button(onClick = { navController.navigate("details/7") }) {
            Text("View Details for User 7")
        }
    }
}

@Composable
fun DetailsScreen(userId: String?) {
    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text("📄 Details for User ID: $userId", fontSize = 22.sp)
    }
}

✅ Summary
Feature	Compose Equivalent	React Native Equivalent
Navigation Host	NavHost	NavigationContainer
Screen Registration	composable("route")	Stack.Screen
Navigate to screen	navController.navigate("route")	navigation.navigate("route")
Pass params	Route arguments	route.params
Go back	popBackStack()	navigation.goBack()
Bottom Tabs	NavigationBar	createBottomTabNavigator()
Animated transitions	AnimatedNavHost	Custom stack animation
