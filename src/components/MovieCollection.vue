<template>
  <div class="movie-app">
    <h2> Filter Genres:</h2>

    <!-- Genre Filter Tags -->
    <div>
     <span v-for="genre in uniqueGenres" :key="genre" @click="filterByGenre(genre)" class="genre-tag" :class="{ active: genre === selectedGenre }">
      {{ genre }} ({{ genreCounts[genre] }})
      </span>
    </div>

    <!-- Movies/Series -->
    <ul>
      <li v-for="entry in filteredMovies" :key="entry.id" @click="showModal(entry)">
        <img :src="entry.imageUrl" :alt="entry.title" class="movie-cover">
        <div class="movie-info">
          <h2>{{ entry.title }}</h2>
          <p>{{ entry.description }}</p>
          <p>Genre: {{ entry.genre }}</p>
        </div>
      </li>
    </ul>

    <!-- Pagination Controls -->
    <div class="pagination">
      <button @click="previousPage" :disabled="currentPage <= 1">Forrige</button>
      <span class="m-2">Side {{ currentPage }} af {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage >= totalPages">Næste</button>
    </div>

    <!--  Modal structure -->
    <div v-if="isModalVisible" class="modal">
      <div class="modal-content">
        <span class="close" @click="isModalVisible = false">&times;</span>
        <h2>{{ selectedMovie.title }}</h2>
        <img :src="selectedMovie.imageUrl" :alt="selectedMovie.title" class="modal-image">
        <p>{{ selectedMovie.description }}</p>
        <p><strong>Actors:</strong> {{ selectedMovie.actors.join(', ') }}</p>
        <p><strong>Directors:</strong> {{ selectedMovie.directors.join(', ') }}</p>
        <p><strong>Year:</strong> {{ selectedMovie.year }}</p>
        <p><strong>Genre:</strong> {{ selectedMovie.genre }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MovieApp',
  data() {
    return {
      movies: [],
      isModalVisible: false,
      selectedMovie: null,
      selectedGenre: null,
      placeholderImage: 'https://t3.ftcdn.net/jpg/02/48/42/64/360_F_248426448_NVKLywWqArG2ADUxDq6QprtIzsF82dMF.jpg',
      currentPage: 1,
      pageSize: 10
    };
  },
  computed: {
  genreCounts() {
    const count = {};
    this.movies.forEach(movie => {
      if (count[movie.genre]) {
        count[movie.genre]++;
      } else {
        count[movie.genre] = 1;
      }
    });
    return count;
  },
  uniqueGenres() {
    return Object.keys(this.genreCounts);
  },
  filteredMovies() {
    return this.selectedGenre ? this.movies.filter(movie => movie.genre === this.selectedGenre) : this.movies;
  },
  totalPages() {
    return Math.ceil(this.filteredMovies.length / this.pageSize);
  },
  paginatedMovies() {
    const start = (this.currentPage - 1) * this.pageSize;
    const end = start + this.pageSize;
    return this.filteredMovies.slice(start, end);
  }
},
  methods: {
    async fetchMovies() {
      const response = await fetch('https://feed.entertainment.tv.theplatform.eu/f/jGxigC/bb-all-pas?form=json&lang=da&byProgramType=series');
      const data = await response.json();
      this.movies = data.entries.map(entry => {
        return {
          id: entry.guid,
          title: entry.title,
          description: entry.description,
          year: entry.plprogram$year,
          imageUrl: entry.media$thumbnails ? entry.media$thumbnails[0].url : this.placeholderImage,
          actors: entry.plprogram$credits.filter(credit => credit.plprogram$creditType === "actor").map(actor => actor.plprogram$personName),
          directors: entry.plprogram$credits.filter(credit => credit.plprogram$creditType === "director").map(director => director.plprogram$personName),
          genre: entry.plprogram$tags && entry.plprogram$tags.length > 0 && entry.plprogram$tags[0].plprogram$title ? entry.plprogram$tags[0].plprogram$title : "unknown"
        };
      });
      this.selectedGenre = null; // Reset filter when fetching new movies
    },
    showModal(movie) {
      this.selectedMovie = movie;
      this.isModalVisible = true;
    },
    filterByGenre(genre) {
      this.selectedGenre = this.selectedGenre === genre ? null : genre; // Toggle genre filter
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    }
  },
  mounted() {
    this.fetchMovies();
  }
};
</script>

<style>
.movie-app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  color: #333;
  text-align: center;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  margin-top: 10px;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  display: flex;
  align-items: center;
}

@media only screen and (max-width: 801px) {
  li {
   flex-wrap: wrap;
    justify-content: center;
  }
}

li:hover {
  background-color: #f1f1f1;
}

.movie-cover {
  width: 100px;
  height: 150px;
  object-fit: cover;
  margin-right: 20px;
}

.movie-info h2 {
  font-size: 20px;
  color: #2a2a2a;
  margin: 0 0 10px 0;
}

.movie-info p {
  font-size: 16px;
  color: #666;
  margin: 0;
}
.modal {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 10px; 
  width: 90%; 
  max-width: 500px; 
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  transition: transform 0.3s ease-in-out;
}

@media (max-width: 768px) {
  .modal-content {
    width: 85%; 
    margin: 10% auto; 
    transform: scale(0.7);
  }
}

@media (max-width: 480px) {
  .modal-content {
    width: 95%; 
    padding: 15px; 
    margin: 20% auto;
  }
}

.close {
  float: right;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
}

.modal-image {
  width: 100%;
  height: auto;
  margin-top: 10px;
}

.m-2{
  margin: 0 12px;
}

.genre-tag {
  cursor: pointer;
  color: white;
  background: #646464;
  padding: 0px 4px;
  margin: 3px;
  border-radius: 2px;
}
.genre-tag.active {
  font-weight: bold;
  text-decoration: underline;
  background: #41B884
}

</style>
