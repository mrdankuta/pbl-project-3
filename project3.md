# Project 3 - MERN Stack Implementation
---


## Step 1 - Backend Configuration: NodeJS

### Install Nodejs & NPM
- Update Ubuntu:
    ```
    sudo apt update
    ```
- Upgrade Ubuntu:
    ```
    sudo apt upgrade
    ```
- Add location of Node.js in NodeSource's Ubuntu Repository:
    ```
    curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -
- Run installation of Node.js and NPM:
    ```
    sudo apt-get install -y nodejs
    ```
- Verify installation and version of Nodejs:
    ```
    node -v
    ```
    ![Node Version](./images/node-version.png)
- Verify NPM version:
    ```
    npm -v
    ```
    ![NPM Version](./images/npm-version.png)


### Application Project Setup
- Create project directory:
    ```
    mkdir todo
    ```
- Navigate into project directory:
    ```
    cd todo
    ```
- Initialize node project to follow prompts & generate `package.json` file:
    ```
    npm init
    ```
    ![NPM Init](./images/npm-init.png)
- View created file:
    ```
    ls
    ```
    ![NPM Init Package File](./images/npm-init-package.png)


---


## Step 2 - Backend Configuration: ExpressJS

### Install & Configure ExpressJS
- Use npm to install express:
    ```
    npm install express
    ```
    ![NPM Express Installed](./images/npm-express-installed.png)
- Install `dotenv` module:
    ```
    npm install dotenv
    ```
    ![NPM Dotenv Installed](./images/npm-dotenv-installed.png)
- Create & edit `index.js` file:
    ```
    nano index.js
    ```
- Insert the following code:
    ```
    const express = require('express');
    require('dotenv').config();

    const app = express();

    const port = process.env.PORT || 5000;

    app.use((req, res, next) => {
        res.header("Access-Control-Allow-Origin", "\*");
        res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        next();
    });

    app.use((req, res, next) => {
        res.send('Welcome to Express');
    });

    app.listen(port, () => {
        console.log(`Server running on port ${port}`)
    });
    ```
- Save and exit editor with `CTRL/CMD + X`, then `Y`, then `ENTER`
- Start Node server:
    ```
    node index.js
    ```
    ![Node Running on 5000](./images/node-running-5000.png)


### Open EC2 Port
- Open port 5000 in EC2 Instance attached security group by adding a new inbound rule
    ![EC2 Port 5000](./images/ec2-open-5000.png)
- Check with browser using server's Public DNS: `ec2-18-215-153-64.compute-1.amazonaws.com:5000` OR using Public IP Address: `18.215.153.64:5000`
![Port 5000 Browser Test](./images/ec2-port-5000-browser-test.png)


### Routes Configuration
Create `routes` directory:
    ```
    mkdir routes
    ```
- Navigate into `routes` directory:
    ```
    cd routes
    ```
- Create & edit `api.js` file where `POST`, `GET`, and `DELETE` endpoints will be defined:
    ```
    nano api.js
    ```
- Insert the following code:
    ```
    const express = require ('express');
    const router = express.Router();

    router.get('/todos', (req, res, next) => {

    });

    router.post('/todos', (req, res, next) => {

    });

    router.delete('/todos/:id', (req, res, next) => {

    });

    module.exports = router;
    ```
- Save and exit editor with `CTRL/CMD + X`, then `Y`, then `ENTER`


---


## Step 3 - Backend Configuration: Models

### Install Mongoose & Setup Schema
- First navigate back to application root directory:
    ```
    cd ..
    ```
- Install Mongoose:
    ```
    npm install mongoose
    ```
- Create `models` directory, navigate into `models` directory, and create & edit `todo.js` file:
    ```
    mkdir models && cd models && nano todo.js
    ```
- Insert the following code:
    ```
    const mongoose = require('mongoose');
    const Schema = mongoose.Schema;

    //create schema for todo
    const TodoSchema = new Schema({
        action: {
            type: String,
            required: [true, 'The todo text field is required']
        }
    });

    // create model for todo
    const Todo = mongoose.model('todo', TodoSchema);

    module.exports = Todo;
    ```
- Save and exit editor with `CTRL/CMD + X`, then `Y`, then `ENTER`


### Update Routes to use new model
- Navigate back to application root directory:
    ```
    cd ..
    ```
- Open `api.js` file in editor:
    ```
    nano routes/api.js
    ```
- Update code to the following:
    ```
    const express = require ('express');
    const router = express.Router();
    const Todo = require('../models/todo');

    router.get('/todos', (req, res, next) => {
        // return all data, expose only id and action fields to client
        Todo.find({}, 'action')
        .then(data => res.json(data))
        .catch(next)
    });

    router.post('/todos', (req, res, next) => {
        if(req.body.action){
            Todo.create(req.body)
            .then(data => res.json(data))
            .catch(next)
        }else {
            res.json({
                error: "The input field is empty"
            })
        }
    });

    router.delete('/todos/:id', (req, res, next) => {
        Todo.findOneAndDelete({"_id": req.params.id})
        .then(data => res.json(data))
        .catch(next)
    });

    module.exports = router;
    ```
- Save and exit editor with `CTRL/CMD + X`, then `Y`, then `ENTER`


---


## Backend Configuration: MongoDB

### Setup MongoDB Account, Database, & Collection
- Visit [MongoDB Atlas](https://www.mongodb.com/atlas-signup-from-mlab) to sign-up/login
- Click "Build a Database" button
![Build A Database](./images/mongodb-setup/mongodb-setup-0.jpeg)
- Select Free Shared Cloud Database tier
![Select Free Tier](./images/mongodb-setup/mongodb-setup-1.jpeg)
- Select AWS as cloud provider, select region closest to you
![Cloud Provider](./images/mongodb-setup/mongodb-setup-2.jpeg)
- Ensure all configurations are in the free tier, Name  and create cluster.
![Free Configurations](./images/mongodb-setup/mongodb-setup-3.jpeg)
- Create new database user
![Create DB User](./images/mongodb-setup/mongodb-setup-4.jpeg)
![Auth Details](./images/mongodb-setup/mongodb-setup-5.jpeg)
- Open network access
![Network Access](./images/mongodb-setup/mongodb-setup-6.jpeg)
![Allow Access](./images/mongodb-setup/mongodb-setup-7.jpeg)
- Create database collection
![Database Collections](./images/mongodb-setup/mongodb-setup-8.jpeg)
![Add New Collection](./images/mongodb-setup/mongodb-setup-9.jpeg)
![Name database and collection](./images/mongodb-setup/mongodb-setup-91.jpeg)


### Connect Application to Database
- Get connection string for database access from MongoDB dashboard
![Get Connection String](./images/mongodb-setup/mongodb-setup-92.jpeg)
![Connection Method](./images/mongodb-setup/mongodb-setup-93.jpeg)
![Copy String](./images/mongodb-setup/mongodb-setup-94.jpeg)
- Create and edit `.env` file in application root directory `todo` to hold all environment variables:
    ```
    nano .env
    ```
- Add connection string with correct user details:
    ```
    DB = 'mongodb+srv://pbl_merner:pbl_pass_001@pblcluster.mujjvx6.mongodb.net/?retryWrites=true&w=majority'
- Save and exit editor with `CTRL/CMD + X`, then `Y`, then `ENTER`
- Edit `index.js` to connect to database with details is `.env` file:
    ```
    nano index.js
    ```
- Update code with the following:
    ```
    const express = require('express');
    const bodyParser = require('body-parser');
    const mongoose = require('mongoose');
    const routes = require('./routes/api');
    const path = require('path');
    require('dotenv').config();

    const app = express();

    const port = process.env.PORT || 5000;

    // connect to database
    mongoose.connect(process.env.DB, {useNewUrlParser: true, useUnifiedTopology: true})
    .then(() => console.log(`Database connected successfully`))
    .catch(err => console.log(err));

    // Override mongoose promise with node's promise, due to deprecation
    mongoose.Promise = global.Promise;

    app.use((req, res, next) => {
        res.header("Access-Control-Allow-Origin", "\*");
        res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        next();
    });

    app.use(bodyParser.json());

    app.use('/api', routes);

    app.use((err, req, res, next) => {
        console.log(err);
        next();
    });

    app.listen(port, () => {
        console.log(`Server running on port ${port}`)
    });
    ```
- Save and exit editor with `CTRL/CMD + X`, then `Y`, then `ENTER`
- Start NodeJS server:
    ```
    node index.js
    ```


### Backend Testing with Postman