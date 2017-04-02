<template>
<div id="movie-list">
    <div v-if="filteredMovies.length">
        <movie-item v-for="movie in filteredMovies" v-bind:movie="movie.movie"
         v-bind:sessions="movie.sessions" v-bind:day="day" v-bind:time="time" ></movie-item>
    </div>
    <div class="no-results" v-else-if="movies.length">
        {{ noResultMsg }}
    </div>
    <div v-else class="no-results">
        Loading...
    </div>
</div>
</template>
<script>
import genres from '../util/genres';
import MovieItem from './MovieItem.vue';
import times from '../util/times';

export default  {
            props : ['genre' , 'time', 'movies', 'day'],
            methods: {
                movieFilter (movie) {
                    if(!this.genre.length) {
                        return true;
                    }else {
                        let movieGenres = movie.movie.Genre.split(", ");
                        let matched = true;
                        this.genre.forEach(genre => {
                            if(movieGenres.indexOf(genre) === -1) {
                                matched = false;
                            }
                        });
                        return matched;
                    }
                    
                },
                sessionFilter(session) {
                    if(!this.day.isSame(this.$moment(session.time), 'day')){
                        return false;
                    }
                    if(this.time.length === 0 || this.time.length == 2 ) {
                        return true;
                    } else if (this.time[0] === times.AFTER_6PM ) {
                        return this.$moment(session.time).hour() >= 18;
                    } else {
                        return this.$moment(session.time).hour() < 18;
                    }
                }
            },
            computed : {
                filteredMovies () {
                    return this.movies
                        .filter(this.movieFilter)
                        .filter(movie => movie.sessions.find(this.sessionFilter));
                },
                noResultMsg () {
                    let times = this.time.join(", ");
                    let genres = this.genre.join(", ");
                    return `No results for ${times} ${times.length && genres.length ? ',' : ''} ${genres}`
                }
            },
            components: {
                MovieItem
            }
}
</script>