

### ` src/pages/Watchlist.tsx (Type Simulator)`
  ```javascript
import React from 'react'; //[pause]
import { useSelector, useDispatch } from 'react-redux'; //[pause]
import { Grid, Container, Typography, Button } from '@mui/material'; //[pause]
import { RootState } from '../redux/store'; //[pause]
import { removeFromWatchlist } from '../redux/movieSlice'; //[pause]
import MovieCard from '../components/MovieCard'; //[pause]

const Watchlist: React.FC = () => { //[pause]
  const dispatch = useDispatch(); //[pause]
  const watchlist = useSelector((state: RootState) => state.movies.watchlist); //[pause]

  const handleRemoveFromWatchlist = (movieId: number) => { //[pause]
    const movieToRemove = watchlist.find((movie) => movie.id === movieId); //[pause]
    if (movieToRemove) { //[pause]
      dispatch(removeFromWatchlist(movieToRemove)); //[pause]
    } //[pause]
  }; //[pause]

  return ( //[pause]
    <Container sx={{ py: 4 }}> //[pause]
      <Typography variant="h4" gutterBottom> //[pause]
        My Watchlist //[pause]
      </Typography> //[pause]
      <Grid container spacing={3}> //[pause]
        {watchlist.length === 0 ? ( //[pause]
          <Typography variant="h6" color="textSecondary"> //[pause]
            Your watchlist is empty. //[pause]
          </Typography> //[pause]
        ) : ( //[pause]
          watchlist.map((movie) => ( //[pause]
            <Grid key={movie.id}> //[pause]
              <MovieCard movie={movie} /> //[pause]
              <Button //[pause]
                variant="contained" //[pause]
                color="secondary" //[pause]
                onClick={() => handleRemoveFromWatchlist(movie.id)} //[pause]
                sx={{ mt: 2 }} //[pause]
              > //[pause]
                Remove from Watchlist //[pause]
              </Button> //[pause]
            </Grid> //[pause]
          )) //[pause]
        )} //[pause]
      </Grid> //[pause]
    </Container> //[pause]
  ); //[pause]
}; //[pause]

export default Watchlist; //[pause]
  ```

### ` src/components/NavBar.tsx(main)` 
  
  ```javascript
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { AppBar, Toolbar, Typography, Box, InputBase, IconButton } from '@mui/material';
import { styled, alpha } from '@mui/material/styles';
import SearchIcon from '@mui/icons-material/Search';
import BookmarkIcon from '@mui/icons-material/Bookmark'; // Import Icon for Watchlist

// Styled components for the search bar
const Search = styled('div')(({ theme }) => ({
  position: 'relative',
  borderRadius: theme.shape.borderRadius,
  backgroundColor: alpha(theme.palette.common.white, 0.15),
  '&:hover': {
    backgroundColor: alpha(theme.palette.common.white, 0.25),
  },
  marginLeft: theme.spacing(2),
  width: 'auto',
}));

const SearchIconWrapper = styled('div')(({ theme }) => ({
  padding: theme.spacing(0, 2),
  height: '100%',
  position: 'absolute',
  pointerEvents: 'none',
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'center',
}));

const StyledInputBase = styled(InputBase)(({ theme }) => ({
  color: 'inherit',
  '& .MuiInputBase-input': {
    padding: theme.spacing(1, 1, 1, 0),
    paddingLeft: `calc(1em + ${theme.spacing(4)})`,
    transition: theme.transitions.create('width'),
    width: '12ch',
    [theme.breakpoints.up('md')]: {
      width: '20ch',
    },
  },
}));

const Navbar: React.FC = () => {
  const [searchQuery, setSearchQuery] = useState<string>('');
  const navigate = useNavigate();

  const handleSearch = (e: React.FormEvent) => {
    e.preventDefault();
    if (searchQuery.trim()) {
      navigate(`/search?q=${encodeURIComponent(searchQuery.trim())}`);
      setSearchQuery(''); // Clear the search input after submitting
    }
  };

  return (
    <AppBar position="fixed">
      <Toolbar>
        <Typography variant="h6" component="div">
          Movie App
        </Typography>
        <Box component="form" onSubmit={handleSearch} sx={{ ml: 'auto', display: 'flex', alignItems: 'center' }}>
          <Search>
            <SearchIconWrapper>
              <SearchIcon />
            </SearchIconWrapper>
            <StyledInputBase
              placeholder="Search..."
              inputProps={{ 'aria-label': 'search' }}
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
            />
          </Search>
          <IconButton //[pause]
  onClick={() => navigate('/watchlist')} //[pause]
  sx={{ //[pause]
    ml: 2, //[pause]
    color: 'white', 
  }} //[pause]
  aria-label="Watchlist" //[pause]
> //[pause]
  <BookmarkIcon />WatchList //[pause]
</IconButton> //[pause]

        </Box>
      </Toolbar>
    </AppBar>
  );
};

export default Navbar;

  ```


### ` src/redux/MovieSlice.tsx(main) `
```javascript
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { getPopularMovies, getTrendingMovies } from '../utils/api';
import api from '../utils/api';

interface Movie {
  id: number;
  title: string;
  poster_path: string;
  vote_average: number | null;
}

interface MovieState {
  popularMovies: Movie[];
  trendingMovies: Movie[];
  searchResults: Movie[];
  watchlist: Movie[]; // Added watchlist to the state
  loading: boolean;
  error: string | null;
}

const initialState: MovieState = {
  popularMovies: [],
  trendingMovies: [],
  searchResults: [],
  watchlist: [], // Initialize the watchlist
  loading: false,
  error: null,
};

// Define async actions using createAsyncThunk
export const searchMoviesAsync = createAsyncThunk<Movie[], string>(
  'movies/search',
  async (query) => {
    const response = await api.get(`/search/movie?query=${query}`);
    return response.data.results; // Assuming the response has a `results` key with an array of movies
  }
);

export const fetchPopularMovies = createAsyncThunk<Movie[]>(
  'movies/fetchPopularMovies',
  async () => {
    const response = await api.get('/movie/popular');
    return response.data.results; // Assuming the response has a `results` key with an array of movies
  }
);

export const fetchTrendingMovies = createAsyncThunk<Movie[]>(
  'movies/fetchTrendingMovies',
  async () => {
    const response = await api.get('/trending/movie/week');
    return response.data.results; // Assuming the response has a `results` key with an array of movies
  }
);

// Create the slice
const movieSlice = createSlice({
  name: 'movies',
  initialState,
  reducers: { //[pause]
  addToWatchlist: (state, action: PayloadAction<Movie>) => { //[pause]
    if (!state.watchlist.find((movie) => movie.id === action.payload.id)) { //[pause]
      state.watchlist.push(action.payload); //[pause]
    } //[pause]
  }, //[pause]
  removeFromWatchlist: (state, action: PayloadAction<Movie>) => { //[pause]
    // Remove the movie from watchlist //[pause]
    state.watchlist = state.watchlist.filter( //[pause]
      (movie) => movie.id !== action.payload.id //[pause]
    ); //[pause]
  }, //[pause]
}, //[pause]
  extraReducers: (builder) => {
    builder
      // Handle the popular movies fetch
      .addCase(fetchPopularMovies.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchPopularMovies.fulfilled, (state, action: PayloadAction<Movie[]>) => {
        state.loading = false;
        state.popularMovies = action.payload;
      })
      .addCase(fetchPopularMovies.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch popular movies';
      })
      // Handle the trending movies fetch
      .addCase(fetchTrendingMovies.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchTrendingMovies.fulfilled, (state, action: PayloadAction<Movie[]>) => {
        state.loading = false;
        state.trendingMovies = action.payload;
      })
      .addCase(fetchTrendingMovies.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch trending movies';
      })
      // Handle search movies async action
      .addCase(searchMoviesAsync.fulfilled, (state, action: PayloadAction<Movie[]>) => {
        state.searchResults = action.payload;
      });
  },
});

export const { addToWatchlist, removeFromWatchlist } = movieSlice.actions;

export default movieSlice.reducer;


```    

### ` src/App.tsx(main)` 
```javascript
import { ThemeProvider, CssBaseline, Box } from '@mui/material';
import theme from './styles/theme';
import Navbar from './components/NavBar';
import Home from './components/Home';
import { Provider } from 'react-redux';
import { store } from './redux/store';
import { Route, Routes } from 'react-router-dom';
import Search from './pages/Search';
import { BrowserRouter } from 'react-router-dom'; 
import Watchlist from './pages/WatchList';

function App() {
  return (
    <Provider store={store}>   
      <ThemeProvider theme={theme}>
        <CssBaseline />
        <BrowserRouter>  {/* Wrap with BrowserRouter */}
          <Navbar />
          <Box sx={{ mt: 2 }}>
            <Routes>
              <Route path="/" element={<Home />} />
              <Route path="/search" element={<Search />} />
              <Route path="/watchlist" element={<Watchlist />} />
            </Routes>
          </Box>
        </BrowserRouter>
      </ThemeProvider>
    </Provider>
  );
}

export default App;

```

### ` src/components/MovieCard.tsx(main) `
  
  ```javascript
 import React from 'react';
import { Card, CardContent, Typography, CardMedia, Box, IconButton } from '@mui/material';
import { Bookmark, BookmarkBorder } from '@mui/icons-material';
import { useDispatch, useSelector } from 'react-redux';
import { addToWatchlist, removeFromWatchlist } from '../redux/movieSlice';

// Define the Movie type based on the data structure expected
interface Movie {
  id: number;
  title: string;
  poster_path: string;
  vote_average: number | null;
}

interface MovieCardProps {
  movie: Movie;
}

const MovieCard: React.FC<MovieCardProps> = ({ movie }) => {
  const dispatch = useDispatch();
  
  // Access watchlist state from Redux store
  const watchlist = useSelector((state: any) => state.movies.watchlist);
  const isInWatchlist = watchlist.some((m: Movie) => m.id === movie.id);

  // Handle click for adding/removing movie from watchlist
  const handleWatchlistClick = (e: React.MouseEvent) => {
    e.stopPropagation(); // Prevent the click from triggering the card's onClick
    if (isInWatchlist) {
      dispatch(removeFromWatchlist(movie));
    } else {
      dispatch(addToWatchlist(movie));
    }
  };

  return (
    <Card
      sx={{
        display: 'flex',
        flexDirection: 'column',
        cursor: 'pointer',
        boxShadow: 3,
        borderRadius: 2,
        overflow: 'hidden',
        height: '100%',
        transition: 'transform 0.2s',
        '&:hover': {
          transform: 'scale(1.05)',
        },
        position: 'relative', // To position the icon button inside the card
      }}
    >
      <CardMedia
        component="img"
        image={`https://image.tmdb.org/t/p/w500${movie.poster_path}`}
        alt={movie.title}
        sx={{
          height: 350,
          objectFit: 'cover',
          width: '100%',
        }}
      />
      <CardContent sx={{ flexGrow: 1, padding: 2 }}>
        <Typography
          gutterBottom
          variant="h6"
          component="div"
          sx={{
            fontWeight: 'bold',
            color: 'text.primary',
            fontSize: '1.1rem',
            overflow: 'hidden',
            textOverflow: 'ellipsis',
            whiteSpace: 'nowrap',
          }}
        >
          {movie.title}
        </Typography>
        <Box sx={{ display: 'flex', alignItems: 'center' }}>
          <Typography
            variant="body2"
            color="text.secondary"
            sx={{ fontSize: '0.875rem' }}
          >
            Rating: {movie.vote_average ? movie.vote_average.toFixed(1) : 'N/A'} / 10
          </Typography>
        </Box>
      </CardContent>

      <IconButton
        onClick={handleWatchlistClick}
        sx={{
          position: 'absolute',
          top: 8,
          right: 8,
          color: isInWatchlist ? 'primary.main' : 'text.secondary', // Change color based on watchlist status
        }}
      >
        {isInWatchlist ? <Bookmark /> : <BookmarkBorder />}
      </IconButton>
    </Card>
  );
};

export default MovieCard;

```

