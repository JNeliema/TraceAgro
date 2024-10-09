# TraceAgro
PROJECT STRUCTURE
agriculture-supply-chain-tracking/
│
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── App.js
│   │   ├── index.js
│   └── package.json
│
├── backend/
│   ├── models/
│   ├── routes/
│   ├── server.js
│   └── package.json
│
└── smart_contracts/
    ├── AgricultureSupplyChain.sol
    └── truffle-config.js


USER REGISTRATION COMPONENT
// src/components/Register.js
import React, { useState } from 'react';
import axios from 'axios';

const Register = () => {
  const [name, setName] = useState('');
  const [userType, setUserType] = useState('farmer'); // 'farmer' or 'buyer'

  const handleRegister = async () => {
    // Make API call to backend to register user
    await axios.post('/api/register', { name, userType });
    alert('Registered successfully');
  };

  return (
    <div>
      <h2>Register</h2>
      <input type="text" placeholder="Name" onChange={(e) => setName(e.target.value)} />
      <select onChange={(e) => setUserType(e.target.value)}>
        <option value="farmer">Farmer</option>
        <option value="buyer">Buyer</option>
      </select>
      <button onClick={handleRegister}>Register</button>
    </div>
  );
};

export default Register;

Product Listing Component
// src/components/ProductList.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const ProductList = () => {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    const fetchProducts = async () => {
      const res = await axios.get('/api/products');
      setProducts(res.data);
    };
    fetchProducts();
  }, []);

  return (
    <div>
      <h2>Available Products</h2>
      {products.map(product => (
        <div key={product.id}>
          <h4>{product.name}</h4>
          <p>Price: {product.price} ETH</p>
          <p>Quality: {product.qualityCertification}</p>
        </div>
      ))}
    </div>
  );
};

export default ProductList;

MAIN APP COMPONENT
// src/App.js
import React from 'react';
import Register from './components/Register';
import ProductList from './components/ProductList';

const App = () => {
  return (
    <div>
      <h1>Agriculture Supply Chain Tracking</h1>
      <Register />
      <ProductList />
    </div>
  );
};

export default App;

BACKEND DEVELOPMENT
// backend/server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const bodyParser = require('body-parser');

const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(bodyParser.json());

mongoose.connect('mongodb://localhost/agriculture', { useNewUrlParser: true, useUnifiedTopology: true });

const UserSchema = new mongoose.Schema({
  name: String,
  userType: String // 'farmer' or 'buyer'
});
const User = mongoose.model('User', UserSchema);

app.post('/api/register', async (req, res) => {
  const { name, userType } = req.body;
  const newUser = new User({ name, userType });
  await newUser.save();
  res.send('User registered successfully');
});

app.get('/api/products', async (req, res) => {
  // Fetch products from the database (for demo, return mock data)
  const products = [
    { id: 1, name: 'Tomatoes', price: 0.5, qualityCertification: 'Organic' },
    { id: 2, name: 'Potatoes', price: 0.3, qualityCertification: 'Non-GMO' }
  ];
  res.json(products);
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});


SMART CONTRACT DEVELOPMENT
// smart_contracts/AgricultureSupplyChain.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AgricultureSupplyChain {
    struct Product {
        string name;
        address farmer;
        uint256 price;
        string qualityCertification;
        bool isDelivered;
    }

    mapping(uint256 => Product) public products;
    uint256 public productCount;

    function registerProduct(string memory _name, uint256 _price, string memory _qualityCertification) public {
        productCount++;
        products[productCount] = Product(_name, msg.sender, _price, _qualityCertification, false);
    }

    function confirmDelivery(uint256 _productId) public {
        require(msg.sender == products[_productId].farmer, "Only farmer can confirm delivery");
        products[_productId].isDelivered = true;
    }

    function getProduct(uint256 _productId) public view returns (string memory, address, uint256, string memory, bool) {
        Product memory product = products[_productId];
        return (product.name, product.farmer, product.price, product.qualityCertification, product.isDelivered);
    }
}

CONNECTING FRONTEND TO SMART CONTRACTS
import Web3 from 'web3';

// Inside your component
const web3 = new Web3(window.ethereum);
const contract = new web3.eth.Contract(contractABI, contractAddress);

// Call functions from the smart contract
const fetchProduct = async (productId) => {
    const product = await contract.methods.getProduct(productId).call();
    console.log(product);
};

