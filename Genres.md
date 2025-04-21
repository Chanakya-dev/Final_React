
### ` src/components/GenreDrawer.tsx(Type Simulator)`

```javascript

import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { List, ListItemButton, ListItemText, Typography, Box } from '@mui/material';
import { fetchGenres, fetchMoviesByGenre, setSelectedGenre } from '../redux/movieSlice';
import type { AppDispatch } from '../redux/store';

// Define the Genre type for better type safety
interface Genre {
  id: number;
  name: string;
}


const GenreDrawer: React.FC = () => {
  const dispatch = useDispatch<AppDispatch>();
  const genres = useSelector((state: any) => state.movies.genres); // `any` is used here for simplicity, replace with RootState
  const selectedGenre = useSelector((state: any) => state.movies.selectedGenre); // Same here, replace with RootState

  useEffect(() => {
    dispatch(fetchGenres());
  }, [dispatch]);

  const handleGenreClick = (genre: Genre) => {
    dispatch(setSelectedGenre(genre));
    dispatch(fetchMoviesByGenre(genre.id)); // Fetch movies by selected genre
  };

  return (
    <Box sx={{ p: 2, borderBottom: 1, borderColor: 'divider' }}>
      <Typography variant="h6">Genres</Typography>
      <List>
        {genres.map((genre: Genre) => (
          <ListItemButton
            key={genre.id}
            selected={selectedGenre?.id === genre.id}
            onClick={() => handleGenreClick(genre)}
          >
            <ListItemText primary={genre.name} />
          </ListItemButton>
        ))}
      </List>
    </Box>
  );
};

export default GenreDrawer;

```

### ` src/redux/MovieSlice.ts(main) `

```javascript
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import api from '../utils/api';

// Define the Movie and Genre types
interface Movie {
  id: number;
  title: string;
  poster_path: string;
  vote_average: number | null;
}

interface Genre {
  id: number;
  name: string;
}

interface MovieState {
  popularMovies: Movie[];
  trendingMovies: Movie[];
  searchResults: Movie[];
  watchlist: Movie[];
  genres: Genre[];
  genreMovies: Movie[];
  selectedGenre: Genre | null;
  loading: boolean;
  error: string | null;
}

const initialState: MovieState = {
  popularMovies: [],
  trendingMovies: [],
  searchResults: [],
  watchlist: [],
  genres: [],
  genreMovies: [],
  selectedGenre: null,
  loading: false,
  error: null,
};

// Async actions using createAsyncThunk
export const fetchGenres = createAsyncThunk<Genre[]>(
  'movies/fetchGenres',
  async () => {
    const response = await api.get('/genre/movie/list');
    return response.data.genres;
  }
);

export const searchMoviesAsync = createAsyncThunk<Movie[], string>(
  'movies/search',
  async (query) => {
    const response = await api.get(`/search/movie?query=${query}`);
    return response.data.results;
  }
);

export const fetchPopularMovies = createAsyncThunk<Movie[]>(
  'movies/fetchPopularMovies',
  async () => {
    const response = await api.get('/movie/popular');
    return response.data.results;
  }
);

export const fetchTrendingMovies = createAsyncThunk<Movie[]>(
  'movies/fetchTrendingMovies',
  async () => {
    const response = await api.get('/trending/movie/week');
    return response.data.results;
  }
);

export const fetchMoviesByGenre = createAsyncThunk<Movie[], number>(
  'movies/fetchMoviesByGenre',
  async (genreId) => {
    const response = await api.get(`/discover/movie?with_genres=${genreId}`);
    return response.data.results;
  }
);

// Slice definition
const movieSlice = createSlice({
  name: 'movies',
  initialState,
  reducers: {
    addToWatchlist: (state, action: PayloadAction<Movie>) => {
      if (!state.watchlist.find((movie) => movie.id === action.payload.id)) {
        state.watchlist.push(action.payload);
      }
    },
    removeFromWatchlist: (state, action: PayloadAction<Movie>) => {
      state.watchlist = state.watchlist.filter(
        (movie) => movie.id !== action.payload.id
      );
    },
    setSelectedGenre: (state, action: PayloadAction<Genre>) => {
      state.selectedGenre = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      // Fetch genres
      .addCase(fetchGenres.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchGenres.fulfilled, (state, action) => {
        state.loading = false;
        state.genres = action.payload;
      })
      .addCase(fetchGenres.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch genres';
      })
      // Fetch popular movies
      .addCase(fetchPopularMovies.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchPopularMovies.fulfilled, (state, action) => {
        state.loading = false;
        state.popularMovies = action.payload;
      })
      .addCase(fetchPopularMovies.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch popular movies';
      })
      // Fetch trending movies
      .addCase(fetchTrendingMovies.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchTrendingMovies.fulfilled, (state, action) => {
        state.loading = false;
        state.trendingMovies = action.payload;
      })
      .addCase(fetchTrendingMovies.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch trending movies';
      })
      // Search movies
      .addCase(searchMoviesAsync.fulfilled, (state, action) => {
        state.searchResults = action.payload;
      })
      // Fetch movies by genre
      .addCase(fetchMoviesByGenre.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchMoviesByGenre.fulfilled, (state, action) => {
        state.loading = false;
        state.genreMovies = action.payload;
      })
      .addCase(fetchMoviesByGenre.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch movies by genre';
      });
  },
});

// Export actions and reducer
export const { addToWatchlist, removeFromWatchlist, setSelectedGenre } =
  movieSlice.actions;

export default movieSlice.reducer;

```


### ` src/App.tsx(main) `

```javascript
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { ThemeProvider, CssBaseline, Box, Toolbar } from '@mui/material';
import { Provider } from 'react-redux';
import theme from './styles/theme';
import { store } from './redux/store';
import Search from './pages/Search';
import Home from './components/Home';
import Watchlist from './pages/WatchList';
import Navbar from './components/NavBar';
import GenreDrawer from './components/GenreDrawer';

const drawerWidth = 240; // Must match the width set in GenreDrawer

function App() {
  return (
    <Provider store={store}>
      <ThemeProvider theme={theme}>
        <CssBaseline />
        <Router>
          {/* Navbar: fixed position to remain at the top */}
          <Navbar />
          <Box sx={{ display: 'flex', mt: 8 }}>
            {/* Sidebar: GenreDrawer */}
            <Box
              component="aside"
              sx={{
                width: { xs: '100%', sm: `${drawerWidth}px` }, // Full width on small screens, fixed on larger
                flexShrink: 0,
                position: 'fixed',
                top: '64px', // Move the drawer down to start below the navbar
                height: 'calc(100vh - 64px)', // Subtract Navbar height
                overflowY: 'auto',
                borderRight: '1px solid #e0e0e0',
                bgcolor: 'background.paper',
              }}
            >
              <GenreDrawer />
            </Box>

            {/* Main Content: Adjust content area based on sidebar */}
            <Box
              component="main"
              sx={{
                flexGrow: 1,
                ml: { sm: `${drawerWidth}px` }, // Leave space for the sidebar on larger screens
                p: 3,
              }}
            >
              <Toolbar /> {/* Adds space below the navbar for content */}
              <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/search" element={<Search />} />
                <Route path="/watchlist" element={<Watchlist />} />
              </Routes>
            </Box>
          </Box>
        </Router>
      </ThemeProvider>
    </Provider>
  );
}

export default App;

```

 
### ` src/pages/Home.tsx(main) `

```javascript
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { Container, Grid, Typography } from '@mui/material';
import { fetchPopularMovies, fetchTrendingMovies, fetchMoviesByGenre } from '../redux/movieSlice';
import Loading from '../components/Loading';
import MovieCard from '../components/MovieCard';
import { AppDispatch, RootState } from '../redux/store'; // Import AppDispatch

// Ensure the Movie type matches the one defined in the reducer
interface Movie {
  id: number;
  title: string;
  poster_path: string;
  overview?: string;
  release_date?: string;
  vote_average: number | null; // Updated to match the reducer
}

const Home: React.FC = () => {
  const dispatch = useDispatch<AppDispatch>();
  const {
    popularMovies,
    trendingMovies,
    genreMovies,
    selectedGenre,
    loading,
  } = useSelector((state: RootState) => state.movies);

  useEffect(() => {
    if (selectedGenre) {
      dispatch(fetchMoviesByGenre(selectedGenre.id));
    } else {
      dispatch(fetchPopularMovies());
      dispatch(fetchTrendingMovies());
    }
  }, [dispatch, selectedGenre]);

  if (loading) {
    return <Loading message="Fetching movies..." />;
  }

  console.log('Genre Movies:', genreMovies); // Debugging line to check the data

  // Render movies by genre
  if (selectedGenre) {
    return (
      <Container sx={{ py: 4 }}>
        <Typography variant="h4" gutterBottom>
          {selectedGenre.name} Movies
        </Typography>
        <Grid container spacing={3}>
          {genreMovies.map((movie: Movie) => (
            <Grid item xs={12} sm={6} md={4} lg={3} key={movie.id}>
              <MovieCard movie={movie} />
            </Grid>
          ))}
        </Grid>
      </Container>
    );
  }

  // Render popular and trending movies
  return (
    <Container sx={{ py: 4 }}>
      <Typography variant="h4" gutterBottom>
        Popular Movies
      </Typography>
      <Grid container spacing={3} sx={{ mb: 4 }}>
        {popularMovies.slice(0, 6).map((movie: Movie) => (
          <Grid item xs={12} sm={6} md={4} lg={3} key={movie.id}>
            <MovieCard movie={movie} />
          </Grid>
        ))}
      </Grid>

      <Typography variant="h4" gutterBottom>
        Trending Now
      </Typography>
      <Grid container spacing={3}>
        {trendingMovies.slice(0, 6).map((movie: Movie) => (
          <Grid item xs={12} sm={6} md={4} lg={3} key={movie.id}>
            <MovieCard movie={movie} />
          </Grid>
        ))}
      </Grid>
    </Container>
  );
};

export default Home;

```
