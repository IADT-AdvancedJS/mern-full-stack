# mern-full-stack

This is an example of a full stack web application using React, Node.js, Express, MongoDB and Webpack. Try a running example at https://mern-users-example.herokuapp.com/.

![Gif of app in action](https://i.imgur.com/XLRUE8P.gif)

It is based on the [Simple React Full Stack Boilerplate project](https://github.com/crsandeep/simple-react-full-stack).

- [mern-full-stack](#mern-full-stack)
  - [Introduction](#introduction)
    - [Development mode](#development-mode)
    - [Production mode](#production-mode)
  - [Quick Start](#quick-start)
  - [MongoDB](#mongodb)
    - [Local Database](#local-database)
    - [Cloud Database](#cloud-database)
    - [Working with your DB](#working-with-your-db)
  - [Express](#express)
  - [React](#react)
  - [Heroku](#deploying-to-heroku)


## Introduction

[Create React App](https://github.com/facebook/create-react-app) is a quick way to get started with React development and it requires no build configuration. But it completely hides the build config which makes it difficult to extend. It also requires some additional work to integrate it with an existing Node.js/Express/MongoDB backend application.

This is a simple full stack [React](https://reactjs.org/) application with a [Node.js](https://nodejs.org/en/), [Express](https://expressjs.com/) and MongoDB backend. Client side code is written in React and the backend API is written using Express.

### Development mode

In the development mode, we will have 2 servers running. The front end code will be served by the [webpack dev server](https://webpack.js.org/configuration/dev-server/) which helps with hot and live reloading. The server side Express code will be served by a node server using [nodemon](https://nodemon.io/) which helps in automatically restarting the server whenever server side code changes.

### Production mode

In the production mode, we will have only 1 server running. All the client side code will be bundled into static files using webpack and it will be served by the Node.js/Express application.

## Quick Start

```bash
# Clone the repository
git clone https://github.com/IADT-AdvancedJS/mern-full-stack

# Go inside the directory
cd mern-full-stack

# Install dependencies
yarn (or npm install)

# Start development server
yarn dev (or npm run dev)

# ABOVE is sufficient for development
# BELOW ONLY if preparing for production:

# Build for production
yarn build (or npm run build)

# Start production server
yarn start (or npm start)
```

## Documentation

### Folder Structure

All the source code will be inside **src** directory. Inside src, there is client and server directory. All the frontend code (react, css, js and any other assets) will be in client directory. Backend Node.js/Express code will be in the server directory.

## MongoDB

You will need access to a server running MongoDB. You may use a local server for development, but should move to a cloud database if you wish to host your project online.

### Local Database

Install MongoDB locally on your machine following [these instructions](https://docs.mongodb.com/manual/installation/). Note: you are required to create a folder `/data/db` on Mac or `C:\data\db` on Windows to act as a data store for MongoDB. Once installed you need to have `mongod` running on your machine and listening for connections.

You can use [Robot3T](https://robomongo.org/) or [Compass](https://www.mongodb.com/products/compass) as a GUI to manage the data in MongoDB. You should create database that contains a collection named `users`.

Your database connection URL will be similar to `mongodb://localhost:27017/your-db-name`

### Cloud Database

***Note:*** you will be unable to connect to a MongoDB Cloud database from within IADT as traffic is blocked by the firewall 😞. You can access the [website](https://cloud.mongodb.com/) to setup and manage a DB.

1. Create a free account at [MongoDB Cloud](https://cloud.mongodb.com/).
2. Once you have created an account you should create a database containing a collection named `users`.
3. [Add a new database user](https://docs.atlas.mongodb.com/security-add-mongodb-users/#add-mongodb-users) with access to this collection. Make note of the user name and password.  
4. [Modify the IP whitelist](https://docs.atlas.mongodb.com/security-whitelist/) to allow connections from any IP address (`0.0.0.0`). *This is not advisable for real-world projects, but will suffice for our purposes*
5. You can manage the contents of your database via the [MongoDB Cloud dashboard](https://cloud.mongodb.com). Alternatively, you can connect to it from [Robot3T](https://robomongo.org/) or [Compass](https://www.mongodb.com/products/compass).
6. Within the [MongoDB Cloud dashboard](https://cloud.mongodb.com), click 'connect', select 'connect your application' and then 'short SRV string connection'. Make note of the connection string. It will be of the form `mongodb+srv://aerrity:<PASSWORD>@cluster0-qecqg.mongodb.net/test?retryWrites=true`.

### Working with your DB

Once you have got one of the above setup you can proceed to connecting from your server-side JavaScript code.

If you wish to populate your collection with some records to get started they should be of the format

```javascript
{
    "_id": "5c797f8d9a1fe12d86cca4f6",
    "name": "Johnny",
    "picture": "https://randomuser.me/api/portraits/med/men/54.jpg"
},
{
    "_id": "5c797fe69a1fe12d86cca4f7",
    "name": "Jane",
    "picture": "https://randomuser.me/api/portraits/med/women/12.jpg"
},
{
    "_id": "5c79802e9a1fe12d86cca4f8",
    "name": "Anita",
    "picture": "https://randomuser.me/api/portraits/med/women/42.jpg"
}
```
## Express

We will be using Express to implement a [REST API](https://restfulapi.net/) that will provide access to our database.

Code for a sample Express server is located in `src/server/index.js`. The code listing is provided below. You will need to change the `dbname` to match your database name in MongoDB.

***Important*** At this point you should check that the server is working correctly and can connect to your DB. First ensure the server is running (`npm run dev`, or `npm run server`). Then access http://localhost:8080/api/users. You should see the contents of your `users` collection, i.e. either an empty array (`[]`) or an array of user objects. You could also test this using a tool such as [Postman](https://www.getpostman.com/).

If you receive an error, check your server console for error logs. It is likely that your DB connection details are incorrect. Check the `dbname` and `dbroute` variables are correct. In particular, if you are using MongoDB cloud the `dbroute` should be of the form `mongodb+srv://<USERNAME>:<PASSWORD>@<SUBDOMAIN>.mongodb.net/test?retryWrites=true`.

```javascript
const express = require('express');
const MongoClient = require('mongodb').MongoClient;
const ObjectID = require('mongodb').ObjectID;
const bodyParser = require('body-parser');

const server = express();
const DBname = 'MyMongoDB'; // change to match your database name

// serve files from the dist directory
server.use(express.static('dist'));

// URL to our DB - will be loaded from an env variable or will use local DB
const dbRoute = process.env.MONGODB_URL || `mongodb://localhost:27017/${DBname}`;

let db;

// connect to the db and start the express server
MongoClient.connect(dbRoute, (err, client) => {
  if (err) throw err;

  db = client.db(DBname);
  // start the express web server listening
  server.listen(process.env.PORT || 8080, () => console.log(`Listening on port ${process.env.PORT || 8080}!`));
});

// bodyParser, parses the request body to be a readable json format
server.use(bodyParser.urlencoded({ extended: false }));
server.use(bodyParser.json());

// define the various endpoints

// retrieve all user objects from DB
server.get('/api/users', (req, res) => {
  db.collection('users').find().toArray((err, result) => {
    if (err) throw err;

    console.log(result);
    res.send(result);
  });
});

// retrieve user with specific ID from DB
server.get('/api/users/:id', (req, res) => {
  db.collection('users').findOne({_id: new ObjectID(req.params.id) }, (err, result) => {
    if (err) throw err;

    console.log(result);
    res.send(result);
  });
});

// delete user with specific ID from DB
server.delete('/api/users', (req, res) => {
  db.collection('users').deleteOne( {_id: new ObjectID(req.body.id) }, err => {
    if (err) return res.send(err);

    console.log('deleted from database');
    return res.send({ success: true });
  });
});

// create new user based on info supplied in request body
server.post('/api/users', (req, res) => {
  db.collection('users').insertOne(req.body, (err, result) => {
    if (err) throw err;

    console.log('created in database');
    res.redirect('/');
  });
});

// update user based on info supplied in request body
server.put('/api/users', (req, res) => {
  // get the ID of the user to be updated
  const id  = req.body._id;
  // remove the ID so as not to overwrite it when updating
  delete req.body._id;
  // find a user matching this ID and update their details
  db.collection('users').updateOne( {_id: new ObjectID(id) }, {$set: req.body}, (err, result) => {
    if (err) throw err;

    console.log('updated in database');
    return res.send({ success: true });
  });
});
```

Take note of the five endpoints implement in this code:
```javascript
GET /api/users
GET /api/users/:id
DELETE /api/users
POST /api/users
PUT /api/users
```

## React

At this point you should have confirmed that your server is accepting connections and can access your database. The next step is to connect the client code to the server. A sample React app is located in `src/client`. This code interfaces with the backend via the five REST API endpoints (listed above). You will find `axios` requests to these endpoints, e.g. in `UserList`:

```javascript
// make a GET request to the server for the user data, store it in state
axios.get('api/users')
  .then(response => {
    this.setState({ users: response.data });
  })
  .catch(error => {
    console.log(error);
  });
```

The REST API is used at various points in the client to ***Create, Read, Update and Delete*** documents from the MongoDB collection named `users`. I encourage you to explore each of these.

## Deploying to Heroku

In order to move our code from development to production and to make it available online, we will deploy it to [Heroku](https://www.heroku.com/) - a popular cloud hosting platform that provides free services for experimentation.

In order to deploy to Heroku:
1. Ensure you are using a [cloud hosted database](#cloud-database) and not a local database.
2. Create an account at [Heroku](https://www.heroku.com/) and login.
3. Create a new application within Heroku. Choose the free plan. Pick a suitable name for the app.
4. Install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).
5. Run the following commands to connect your project to the Heroku app and to push your code to the Heroku servers where it will automatically run. *Note: The last command (running in dev mode) is not advisable for real-world projects, but will suffice for our purposes.*

```bash
heroku login
heroku git:clone -a your-app-name
heroku config:set NPM_CONFIG_PRODUCTION=false YARN_PRODUCTION=false

cd your-app-name
git add .
git commit -am "first commit"
git push heroku master
```

6. As the MongoDB connection URL contains your username and password you should not hard code it in `server/index.js`. If you did so it would be visible in your version control system (e.g. GitHub) and would not be secure. To avoid this you can store the MongoDB connection URL as an environmental variable in Heroku by running the command below (ensuring you modify this as appropriate). This URL is then available in your server code via the variable `process.env.MONGODB_URL`.

```bash
heroku config:set MONGODB_URL=mongodb+srv://<USERNAME>:<PASSWORD>@<SUBDOMAIN>.mongodb.net/test?retryWrites=true
```

7. Your app should now be available at https://your-app-name.heroku.com. For example, this sample project has been deployed at https://mern-users-example.herokuapp.com/.
8. To deploy an updated version of your application

```bash
git add .
git commit -am "first commit"
git push heroku master
```
