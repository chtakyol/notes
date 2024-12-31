State is value in app that change.

### `remember` API

```Kotlin
val mutableState = remember { mutableStateOf(default) }
var value by remember { mutableStateOf(default) }
val (value, setValue) = remember { mutableStateOf(default) }
```
We use this to hold data in composition (mostly state).
The **remember** api stores objets in memory during compositions. 
And the data held with remember is removed when composition is removed because remember is stores data in Composition.
The **mutableStateOf** creates an observable MutableState(compose runtimes observable type).
rememberSaveable saves value for persisting between configuration changes. Just like Bundle.
`ArrayList<T>` and `mutableListOf()` are not observable so using these as state cause show wrong data or not trigger recomposition. Prefer immutable objects.
You don't have to use MutableState to hold state always. Also Flow can be used and collect with `collectAsStateWithLifecycle()`.
### Stateful/less
If composable uses `remember` to store data -> stateful
If not using `remember` -> stateless
Stateless are easier to preview and reuse.
### State hoisting
Basically moving state to a composable's caller to make composable stateless with higher order functions.
While hoisting the state  this the basic pattern;
- **`value: T`:** the current value to display
- **`onValueChange: (T) -> Unit`:** an event that requests the value to change, where `T` is the proposed new value
For example when you are implementing a new screen we use; HomeScreen and HomeContent, and pass states and event between these. HomeContent is stateless and we can preview etc.