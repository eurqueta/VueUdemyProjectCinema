<template>
<div id="movie-list">
    <div v-if="filteredMovies.length">
        <movie-item v-for="movie in filteredMovies" v-bind:movie="movie.movie"
          >
            <div class="movie-sessions">
                <div v-for="session in filteredSessions(movie.sessions)" 
                    class="session-time-wrapper tooltip-wrapper"
                     v-tooltip="{seats : session.seats}"
                    >
                    <div class="session-time">
                        {{ formatSessionTime(session.time)}}
                    </div>
                </div>
            </div>
         </movie-item>
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
                formatSessionTime(raw) {
                    return this.$moment(raw).format('h:mm A');
                },
                filteredSessions(sessions) {
                    return sessions.filter(this.sessionFilter);
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