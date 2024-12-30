Composition is basically drawing UI.
Jetpack compose basically re executes your composable (the ui code) when the stat of app changes.
This re execution is triggered with change of **State<T>** object.
Each composable has its own lifecycle. So when you have 2 different Text() composable they will have 2 different lifecycle.
There is a term **call site** which means where the code is live, in a tree structure. And jetpack compose identify each composable with this call site and only re execute the needed part.
### Smart recomposition
The **key** composable can be used for identifying instances in composition. Remember the example with movies; there is movies list that shown in column and if you add new element at top of this list you will recompose everything but with **key** only newly added item will be composed.

For example; first block recompose everytihng when a movie added on top of movies list.
``` Kotlin
@Composable
fun MoviesScreen(movies: List<Movie>) {
    Column {
        for (movie in movies) {
            // MovieOverview composables are placed in Composition given its
            // index position in the for loop
            MovieOverview(movie)
        }
    }
}
```
However if I use ***key*** composable re composition won't trigger again for the other
``` Kotlin
@Composable
fun MoviesScreenWithKey(movies: List<Movie>) {
    Column {
        for (movie in movies) {
            key(movie.id) { // Unique ID for this movie
                MovieOverview(movie)
            }
        }
    }
}
```
Why do you think this recomposition is bad thing? Because if MovieOverview makes network calls inside like below we'll call every movies again, usually I'll make these calls in viewmodels scope but there is also cpu load however most devices can handle easily.
``` Kotlin
@Composable
fun MovieOverview(movie: Movie) {
    Column {
        // Side effect explained later in the docs. If MovieOverview
        // recomposes, while fetching the image is in progress,
        // it is cancelled and restarted.
        val image = loadNetworkImage(movie.url)
        MovieHeader(image)

        /* ... */
    }
}
```
### Skipping recomposition
Compose skips the recomposition if all the inputs are stable and haven't change. The comparison uses the **equals** method.
But sometimes Compose not able the skip so we can use **@Stable** annotation.