# socialx
 To create the e-commerce website
 // 1.Set Up Your Next.js Project//
//First, make sure you have Node.js and npm installed. Then create a new Next.js project://
bash 
    npx create-next-app ecommerce-website
     cd ecommerce-website
// 2. Install Required Packages//
//Install the necessary dependencies including Tailwind CSS, Redux, and Axios for API calls.//
     npm install tailwindcss @reduxjs/toolkit react-redux axios
//3. Set Up Tailwind CSS//
//Follow the official guide to set up Tailwind CSS with Next.js: Tailwind CSS Next.js//
//4. Create Redux Store//
//Set up the Redux store and create slices for managing the state.//
//store.js//
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './slices/cartSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer,
  },
});
//slices/cartSlice.js//
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  items: [],
};

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addToCart(state, action) {
      state.items.push(action.payload);
    },
    removeFromCart(state, action) {
      state.items = state.items.filter(item => item.id !== action.payload.id);
    },
  },
});

export const { addToCart, removeFromCart } = cartSlice.actions;

export default cartSlice.reducer;
//_app.js//
import { Provider } from 'react-redux';
import { store } from '../store';
import '../styles/globals.css';

function MyApp({ Component, pageProps }) {
  return (
    <Provider store={store}>
      <Component {...pageProps} />
    </Provider>
  );
}

export default MyApp;
//services/api.js//
import axios from 'axios';

const API_KEY = 'your_rapidapi_key'; // Replace with your RapidAPI key

const api = axios.create({
  baseURL: 'https://real-time-amazon-data.p.rapidapi.com',
  headers: {
    'x-rapidapi-key': API_KEY,
    'x-rapidapi-host': 'real-time-amazon-data.p.rapidapi.com',
  },
});

export const getProducts = async () => {
  const response = await api.get('/products');
  return response.data;
};
//components/Product.js//
import { useDispatch } from 'react-redux';
import { addToCart } from '../slices/cartSlice';

const Product = ({ product }) => {
  const dispatch = useDispatch();

  const handleAddToCart = () => {
    dispatch(addToCart(product));
  };

  return (
    <div className="product-card">
      <img src={product.image} alt={product.title} />
      <h2>{product.title}</h2>
      <p>{product.price}</p>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
};

export default Product;
//components/Cart.js//
import { useSelector, useDispatch } from 'react-redux';
import { removeFromCart } from '../slices/cartSlice';

const Cart = () => {
  const cartItems = useSelector(state => state.cart.items);
  const dispatch = useDispatch();

  const handleRemoveFromCart = (item) => {
    dispatch(removeFromCart(item));
  };

  return (
    <div className="cart">
      {cartItems.map(item => (
        <div key={item.id} className="cart-item">
          <h3>{item.title}</h3>
          <p>{item.price}</p>
          <button onClick={() => handleRemoveFromCart(item)}>Remove</button>
        </div>
      ))}
    </div>
  );
};

export default Cart;
 //Implement Infinite Scroll//
 npm install react-infinite-scroll-component
//pages/index.js//

import { useState, useEffect } from 'react';
import InfiniteScroll from 'react-infinite-scroll-component';
import { getProducts } from '../services/api';
import Product from '../components/Product';

const Home = () => {
  const [products, setProducts] = useState([]);
  const [hasMore, setHasMore] = useState(true);

  useEffect(() => {
    fetchMoreProducts();
  }, []);

  const fetchMoreProducts = async () => {
    const newProducts = await getProducts();
    setProducts(prevProducts => [...prevProducts, ...newProducts]);
    if (newProducts.length === 0) {
      setHasMore(false);
    }
  };

  return (
    <InfiniteScroll
      dataLength={products.length}
      next={fetchMoreProducts}
      hasMore={hasMore}
      loader={<h4>Loading...</h4>}
    >
      <div className="products-grid">
        {products.map(product => (
          <Product key={product.id} product={product} />
        ))}
      </div>
    </InfiniteScroll>
  );
};

export default Home;



