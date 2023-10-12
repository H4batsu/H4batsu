

// server.js
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const axios = require('axios');
const Pusher = require('pusher');

// create a new Pusher instance
const pusher = new Pusher({
  appId: 'your-app-id',
  key: 'your-app-key',
  secret: 'your-app-secret',
  cluster: 'your-app-cluster',
  useTLS: true
});

// create a new Express app
const app = express();

// use bodyParser to parse application/json content-type
app.use(bodyParser.json());

// enable CORS
app.use(cors());

// create a route for fetching all news
app.get('/news', (req, res) => {
  // get the query parameters
  const { category, country } = req.query;

  // construct the news API URL
  const url = `https://newsapi.org/v2/top-headlines?apiKey=your-api-key&category=${category}&country=${country}`;

  // make a request to the news API
  axios.get(url)
    .then(response => {
      // send the response data as JSON
      res.json(response.data);
    })
    .catch(error => {
      // log the error
      console.error(error);
    });
});

// create a route for updating news
app.post('/update-news', (req, res) => {
  // get the data from the request body
  const { category, country } = req.body;

  // construct the news API URL
  const url = `https://newsapi.org/v2/top-headlines?apiKey=your-api-key&category=${category}&country=${country}`;

  // make a request to the news API
  axios.get(url)
    .then(response => {
      // trigger a pusher event with the updated news data
      pusher.trigger('news-channel', 'update-news', {
        articles: response.data.articles
      });
    })
    .catch(error => {
      // log the error
      console.error(error);
    });

  // send a success response
  res.json({
    status: 'success'
  });
});

// start the app on port 3000
app.listen(3000, () => {
  console.log('Server started on port 3000');
});

// client.js
import React, { Component } from 'react';
import Pusher from 'pusher-js';
import NewsList from './NewsList';
import './App.css';

class App extends Component {
  constructor(props) {
    super(props);

    // initialize the state with an empty articles array and default category and country values
    this.state = {
      articles: [],
      category: 'general',
      country: 'br'
    };
  }

  componentDidMount() {
    // create a new Pusher instance with your app key and cluster
    this.pusher = new Pusher('your-app-key', {
      cluster: 'your-app-cluster',
      useTLS: true
    });

    // subscribe to the news channel
    this.channel = this.pusher.subscribe('news-channel');

    // bind to the update-news event and update the state with the new articles data
    this.channel.bind('update-news', data => {
      this.setState({
        articles: data.articles
      });
    });

    // fetch the initial news data based on the default category and country values
    this.fetchNews(this.state.category, this.state.country);
  }

  componentWillUnmount() {
    // unsubscribe from the news channel when the component is unmounted
    this.pusher.unsubscribe('news-channel');
  }

  fetchNews(category, country) {
    // construct the URL for fetching news based on the category and country values
    const url = `http://localhost:3000/news?category=${category}&country=${country}`;

    // make a GET request to the server and set the state with the response data
    fetch(url)
      .then(response => response.json())
      .then(data => {
        this.setState({
          articles: data.articles
        });
      })
      .catch(error => console.error(error));
  }

  handleCategoryChange = e => {
    // get the selected category value from the event object
    const category = e.target.value;

    // update the state with the new category value
    this.setState({
      category
    });

    // fetch the news data based on the new category value
    this.fetchNews(category, this.state.country);

    // send a POST request to the server to update the news data for other clients
    fetch('http://localhost:3000/update-news', {
      method: 'post',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        category,
        country: this.state.country
      })
    });
  };

  handleCountryChange = e => {
    // get the selected country value from the event object
    const country = e.target.value;

    // update the state with the new country value
    this.setState({
      country
    });

    // fetch the news data based on the new country value
    this.fetchNews(this.state.category, country);

    // send a POST request to the server to update the news data for other clients
    fetch('http://localhost:3000/update-news', {
      method: 'post',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        category: this.state.category,
        country
      })
    });
  };

  render() {
    return (
      <div className="App">
        <div className="App-header">
          <h1>Hao</h1>
          <p>O seu site de notícias em tempo real</p>
          <div className="App-filters">
            <label>Categoria:</label>
            <select value={this.state.category} onChange={this.handleCategoryChange}>
              <option value="general">Geral</option>
              <option value="business">Negócios</option>
              <option value="entertainment">Entretenimento</option>
              <option value="health">Saúde</option>
              <option value="science">Ciência</option>
              <option value="sports">Esportes</option>
              <option value="technology">Tecnologia</option>
            </select>
            <label>País:</label>
            <select value={this.state.country} onChange={this.handleCountryChange}>
              <option value="ar">Argentina</option>
              <option value="au">Austrália</option>
              <option value="br">Brasil</option>
              <option value="ca">Canadá</option>
              <option value="cn">China</option>
              <option value="fr">França</option>
              <option value="de">Alemanha</option>
              <option value="in">Índia</option>
              <option value="it">Itália</option>
              <option value="jp">Japão</option>
              <option value="mx">México</option>
              <option value="ru">Rússia</option>
              <option value="gb">Reino Unido</option>
              <option value="us">Estados Unidos</option>
            </select>
          </div>
        </div>
        <div className="App-body">
          {/* render the NewsList component and pass the articles as props */}
          <NewsList articles={this.state.articles} />
        </div>
      </div>
    );
  }
}

export default App;

/* App.css */
.App {
  font-family: Arial, Helvetica, sans-serif;
}

.App-header {
  background-color: #222;
  color: white;
  padding: 20px;
  text-align: center;
}

.App-filters {
  display: flex;
  justify-content: center;
  align-items: center;
}

.App-filters label {
  margin-right: 10px;
}

.App-filters select {
  margin-right: 20px;
}

.App-body {
  max-width: 1200px;
  margin: auto;
}

.NewsList {
  display: flex;
  flex-wrap: wrap;
}

.NewsItem {
  width: 33%;
  padding: 10px;
}

.NewsItem-image {
  width: 100%;
  height: auto;
}

.NewsItem-title {
  font-weight: bold;
}

.NewsItem-source {
  color: gray;
}

.NewsItem-link {
  text-decoration: none;
}