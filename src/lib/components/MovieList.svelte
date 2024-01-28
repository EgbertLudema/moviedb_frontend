<script>
    export let movies;
    export let watchlist;
    export let watched;
    import { createEventDispatcher } from 'svelte';
    import { onMount } from 'svelte';

    let editingRatingMovieId = null;
    const dispatch = createEventDispatcher();

    // Fetch Ratings
    onMount(async () => {
        // This code runs after the component is mounted to the DOM

        // Fetching individual ratings
        const ratingsResponse = await fetch('http://localhost:1337/ratings');
        if (ratingsResponse.ok) {
            const ratingsData = await ratingsResponse.json();
            // Updating the 'movies' array with the fetched ratings
            movies = movies.map(movie => ({
                ...movie,
                // Adding a 'rating' property to each movie
                // It searches for a corresponding rating in the fetched data
                // If no rating is found, it defaults to 'null'
                rating: ratingsData.find(r => r.movie_id === movie.id)?.rating || null
            }));
            // The modification of 'movies' here triggers Svelte's reactivity
        }
    });

    const handleWatchlist = async (movieId, isInWatchlist) => {
        const url = `http://localhost:1337/${isInWatchlist ? 'remove-from-watchlist' : 'add-to-watchlist'}`;
        const response = await fetch(url, {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({ movie_id: movieId })
        });

        // Update UI if the watchlist is updated correctly
        if (response.ok) {
            dispatch('updateWatchlist', { id: movieId, action: isInWatchlist ? 'remove' : 'add' });
        }
    };

    const handleWatched = async (movieId, isInWatched) => {
        const url = `http://localhost:1337/${isInWatched ? 'remove-from-watched' : 'add-to-watched'}`;
        const response = await fetch(url, {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({ movie_id: movieId })
        });

        // Update UI if the watched is updated correctly
        if (response.ok) {
            dispatch('updateWatched', { id: movieId, action: isInWatched ? 'remove' : 'add' });
        }
    };

    function submitRating(movieId, rating) {
        fetch('http://localhost:1337/add-update-rating', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({ movie_id: movieId, rating: rating }),
        })
        .then(response => response.text())
        .then(result => {
            console.log(result);
            // Updates movies array with ratings, triggering UI refresh
            const movie = movies.find(m => m.id === movieId);
            if (movie) {
                movie.rating = rating;
                movies = [...movies];
            }
        })
        .catch(error => console.error('Error:', error));
    }

    function removeRating(movieId) {
        fetch('http://localhost:1337/remove-rating', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({ movie_id: movieId }),
        })
        .then(response => response.text())
        .then(result => {
            console.log(result);
            // Updates movies array with ratings, triggering UI refresh
            const movie = movies.find(m => m.id === movieId);
            if (movie) {
                movie.rating = null;
                movies = [...movies];
            }
        })
        .catch(error => console.error('Error:', error));
    }

    function handleWatchedChange(movieId, isInWatched) {
        handleWatched(movieId, isInWatched);
        // The CASCADE deletion in the database will automatically delete the rating for the movie as well.
    }

    function formatRating(rating) {
        return rating ? `${rating}/10` : 'Not rated';
    }
</script>

<ul>
    {#each movies as movie}
        <li>
            {movie.title}
            {#if !watched.has(movie.id)}
                <button on:click={() => handleWatchlist(movie.id, watchlist.has(movie.id))}>
                    {watchlist.has(movie.id) ? 'Remove from Watchlist' : 'Add to Watchlist'}
                </button>
                <button on:click={() => handleWatchedChange(movie.id, false)}>Add to Watched</button>
            {:else}
                <button on:click={() => handleWatchedChange(movie.id, true)}>Remove from Watched</button>
                {#if editingRatingMovieId === movie.id}
                    <div class="rating-container">
                        <div class="current-rating">{formatRating(movie.userRating)}</div>
                        <input type="range" min="1" max="10" step="1" bind:value={movie.userRating} />
                    </div>
                    <button on:click={() => { submitRating(movie.id, movie.userRating); editingRatingMovieId = null; }}>Submit</button>
                {:else}
                    {#if typeof movie.rating === 'number'}
                        <p>Rating: {formatRating(movie.rating)}</p>
                        <button on:click={() => { movie.userRating = movie.rating; editingRatingMovieId = movie.id; }}>Change Rating</button>
                        <button on:click={() => removeRating(movie.id)}>Remove Rating</button>
                    {:else}
                        <div class="rating-container">
                            <div class="current-rating">{formatRating(movie.userRating)}</div>
                            <input type="range" min="1" max="10" step="1" bind:value={movie.userRating} />
                        </div>
                        <button on:click={() => submitRating(movie.id, movie.userRating)}>Rate</button>
                    {/if}
                {/if}
            {/if}
        </li>
    {/each}
</ul>