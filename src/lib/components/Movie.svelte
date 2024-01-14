<script>
    import { onMount } from "svelte";
    import MovieList from "./MovieList.svelte";

    let movies = [];
    let watchlist = new Set();
    let watched = new Set();

    onMount(async () => {
        // Fetch all movies
        const moviesResponse = await fetch("http://localhost:1337/movies");
        movies = await moviesResponse.json();

        // Fetch watchlist
        const watchlistResponse = await fetch("http://localhost:1337/watchlist");
        const watchlistData = await watchlistResponse.json();
        watchlist = new Set(watchlistData.map(item => item.movie_id));

        // Fetch watched movies
        const watchedResponse = await fetch("http://localhost:1337/watched");
        const watchedData = await watchedResponse.json();
        watched = new Set(watchedData.map(item => item.movie_id));
    });

    const updateWatchlist = ({ detail }) => {
        const { id, action } = detail;
        if (action === 'add') {
            watchlist = new Set(watchlist.add(id));
        } else {
            watchlist.delete(id);
            watchlist = new Set(watchlist);
        }
    };

    const updateWatched = ({ detail }) => {
        const { id, action } = detail;
        if (action === 'add') {
            watched.add(id);
            watchlist.delete(id); // also remove from watchlist
            watched = new Set(watched);
            watchlist = new Set(watchlist); // reassign to trigger reactivity
        } else {
            watched.delete(id);
            watched = new Set(watched);
        }
    };
</script>

<MovieList 
    movies={movies} 
    watchlist={watchlist} 
    watched={watched} 
    on:updateWatchlist={updateWatchlist} 
    on:updateWatched={updateWatched} 
/>