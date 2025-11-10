# Part 11 — Advanced Compose: Animations, Theming (Material 3) & Adaptive UI

**Objective:** Learn expressive animations in Jetpack Compose, implement Material 3 theming (light/dark), build adaptive layouts that work across phone/tablet, and apply performance best practices.

---

## 🔥 1. Animations in Jetpack Compose

Compose provides high-level animation APIs that are concise and declarative.

### 1.1 `animate*AsState` (Simple property animation)

```kotlin
@Composable
fun SimpleScaleButton() {
    var big by remember { mutableStateOf(false) }
    val scale by animateFloatAsState(if (big) 1.5f else 1f)

    Button(onClick = { big = !big }, modifier = Modifier.graphicsLayer(scaleX = scale, scaleY = scale)) {
        Text("Tap me")
    }
}

RN parallel: CSS/Animated API Animated.timing for basic animations.

1.2 updateTransition (Coordinated animations)
enum class CardState { Collapsed, Expanded }

@Composable
fun TransitionCard() {
    var expanded by remember { mutableStateOf(false) }
    val transition = updateTransition(targetState = if (expanded) CardState.Expanded else CardState.Collapsed)

    val height by transition.animateDp { state ->
        if (state == CardState.Collapsed) 120.dp else 300.dp
    }

    Card(modifier = Modifier
        .height(height)
        .clickable { expanded = !expanded }) {
        // content
    }
}

1.3 Physics-based animations (spring, animate* with damping)
val offset by animateDpAsState(
    targetValue = if (visible) 0.dp else 300.dp,
    animationSpec = spring(dampingRatio = Spring.DampingRatioMediumBouncy, stiffness = Spring.StiffnessLow)
)

1.4 Gesture-driven animations (drag + settle)

Combine pointerInput, Animatable, and LaunchedEffect:

@Composable
fun DraggableBox() {
    val offsetX = remember { Animatable(0f) }
    Box(modifier = Modifier
        .offset { IntOffset(offsetX.value.roundToInt(), 0) }
        .pointerInput(Unit) {
            detectDragGestures { change, dragAmount ->
                change.consume()
                launch {
                    offsetX.snapTo(offsetX.value + dragAmount.x)
                }
            }
        })
}


RN parallel: PanResponder + Animated.Value & spring.

1.5 Lottie Animations

Use lottie-compose to play JSON animations from After Effects.

val composition by rememberLottieComposition(LottieCompositionSpec.RawRes(R.raw.my_animation))
LottieAnimation(composition)

🎨 2. Material 3 Theming (Light & Dark)

Jetpack Compose supports Material 3. Setup a theme and toggle dark mode.

2.1 Add dependency
implementation("androidx.compose.material3:material3:1.1.0")

2.2 Create theme
private val LightColors = lightColorScheme(
    primary = Color(0xFF6750A4),
    onPrimary = Color.White,
    background = Color(0xFFFFFBFE)
)

private val DarkColors = darkColorScheme(
    primary = Color(0xFFD0BCFF),
    onPrimary = Color.Black,
    background = Color(0xFF1C1B1F)
)

@Composable
fun MyAppTheme(darkTheme: Boolean = isSystemInDarkTheme(), content: @Composable () -> Unit) {
    MaterialTheme(colorScheme = if (darkTheme) DarkColors else LightColors, typography = Typography, content = content)
}


Wrap setContent { MyAppTheme { AppNavHost() } }.

2.3 Dynamic color (Android 12+)

Use dynamicLightColorScheme(LocalContext.current) when available to match device wallpaper.

📱 3. Adaptive UI — Responsive layouts

Make UIs that adapt to screen size/orientation.

3.1 Use WindowSizeClass (Jetpack WindowManager)

Small: phones

Medium: large phones / small tablets

Large: tablets / desktop

@Composable
fun AdaptiveLayout() {
    val windowSize = calculateWindowSizeClass(activity)
    when (windowSize.widthSizeClass) {
        WindowWidthSizeClass.Compact -> PhoneLayout()
        WindowWidthSizeClass.Medium -> TwoPaneLayout()
        WindowWidthSizeClass.Expanded -> TabletLayout()
    }
}


Design tips:

Use Adaptive navigation: bottom nav for compact, rail or permanent nav for large screens.

Use two-pane master-detail on tablets: Row { MasterPane(); DetailPane() }.

3.2 Compose ConstraintLayout & BoxWithConstraints

BoxWithConstraints gives breakpoints inside a composable:

@Composable
fun ResponsiveCard() {
    BoxWithConstraints {
        if (maxWidth < 600.dp) Column { /* mobile */ } else Row { /* tablet */ }
    }
}

⚡ 4. Performance Best Practices for Compose

Use remember for expensive initializations.

Prefer derivedStateOf for computed values to avoid recomposition loops.

Avoid creating new lambdas/objects inside frequently recomposed scopes — hoist them or use remember.

Use LazyColumn and Lazy* composables for lists.

Use snapshotFlow to convert Compose state to Flow for heavy processing.

Prefer key in items to provide stable keys for lists.

val rememberedExpensive = remember { expensiveInit() }
val derived = remember { derivedStateOf { computeFrom(count) } }

♿ 5. Accessibility & Localization

Use contentDescription for images and icons.

Use semantics { } for complex components.

Support font scaling and RTL using LocalLayoutDirection.

Provide strings.xml for translations and use stringResource(id = R.string.some_text) in Composables.

✅ 6. Summary & What to Practice

Build small screens that use animate*AsState, updateTransition, and Animatable.

Implement a theme switch (light/dark) and dynamic color.

Convert a single-pane app to a two-pane tablet layout.

Profile recompositions using Layout Inspector and trace system.
