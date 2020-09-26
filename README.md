# Node.js 

# Table of contents

- [Table of contents](#table-of-contents)
- [Bascis](#basics)
  - [Modules in Node](#modules-in-node)
  - [File System module in Node](#file-system-module-in-node)
  - [Environmental Variables](#environmental-variables)
  - [Storing User Passwords Securely](#storing-user-passwords-securely)
  - [Building a Server](#building-a-server)
- [Introduction to Express JS](#introduction-to-express-js)
  - [Building a Server via Express](#building-a-server-via-express)
  - [Loading Static Files](#loading-static-files)
- [Express Middleware](#express-middleware)
- [RESTful API](#restful-api)
  - [Request](#request)
  - [Response](#response)
- [Cross-Origin Resource Sharing - CORS](#cors)
  - [What is CORS ?](#what-is-cors)
  - [CORS Request Types](#cors-request-types)
  - [Identifying a CORS Response](#Identifying-a-cors-response)
  - [How to Add CORS to a Nodejs Express App](#how-to-add-cors-to-a-nodejs-express-app)
- [Postman](#postman)
- [Production Deployment](#production-deployment)
  - [Back-End Heroku Deploy](#back-end-heroku-deploy)
  

# Basics
## Modules in Node
-  **Common Node Modules** <br/>
    | Module   |      Command      |  Description |
    |----------|:-------------:|------|
    |package.json| npm init **-y**| to create package.json; -y flag to skip input
    |dev mod | npm install <module-name> **--save-dev**| to install module in devDependencies for deveploment only, not production|
    |||
    |express|npm install express| to 3rd party framework to build server, no need to use http module & DRY|
    | fs | require('fs')| to work with [File System](#file-system-module-in-node)|
    | http|require('http')| to build the server|
    | nodemon| npm install nodemon | to auto reload the server when we make some changes in our code <br> `"script" : {"start": "nodemon index.js"}` so when we run npm start ~ npm nodemon index.js|
    | body-parser | included in Express |Body-Parser middleware to parse the body of the request<br/>Otherwise, console.log(req.body) will return `undefined`<br/>How to use:<br> `app.use(express.urlencoded({ extended: false })); //to parse urlencoded`<br> `app.use(express.json()); //to parse json`|
    |[bcrypt-nodejs](https://github.com/kelektiv/node.bcrypt.js)|npm install bcrypt-nodejs|allow us to create secure login via encrypt (hash) the password to hash<br>bcrypt is 15 years old and has been vetted by the crypto community.|
    |cors|npm install cors|An example of a cross-origin request: the front-end JavaScript code served from `https://domain-a.com` uses `XMLHttpRequest` to make a request for `https://domain-b.com/data.json`<br>How to add into Node JS Back-`const cors = require("cors");`<br>`app.use(cors());`|
    ||||
    |Knex| npm install knex|SQL query builder for Postgres, MSSQL, MySQL, MariaDB, SQLite3, Oracle, and Amazon Redshift |
    |PostgreSQL|npm install pg|Install Postg|
-  **How to import/export a Node module:** <br/>
    - *Method 1 [New Way]:* **import - export**<br/>
    
      **app.js**
      ```JavaScript
      import largeNumber, {smallNumber} from 'module.js'
      ```
      **module.js**
      ```JavaScript
      const largeNumber = 356;
      const smallNumber = 1;

      export default largeNumber;
      export smallNumber;
      ```
    - *Method 2 [Conventional Way]:* **require - modules.export**<br/>
      **app.js**
      ```JavaScript
      const c = require('./module.js');
      console.log(c.largeNumber);
      ```
      **module.js**
      ```JavaScript
      const largeNumber = 356;

      module.exports = {
        largeNumber: largeNumber;
      }
      ```
  [(Back to top)](#table-of-contents)

## File System module in Node
- **Usage**: I/O to read data from Excel files, to read from robot to get the data that robot detects
```JavaScript
const fs = require("fs");
```
### Read File
  #### Async - readFile (Recommended for handling massive files when building Server)
  - `Async readFile` starts by reading this `"./public/index.html"` file, and you can continue with other code
  - Once done, the call back function `(err,data) => {...}` will perform the tasks, either give you `error` or `data`, inside that call back function
  ```JavaScript
  fs.readFile("./public/index.html", (err, data) => {
    if (err) {
      console.log(err);
    }
    //Print out data from the file; .toString() to encode the data read
    console.log("Async", data.toString("utf-8"));
  });
  ```
  #### Sync - readFileSync
  - `readFileSync`: going to read this `"./public/index.html"` file, so dont do anything (i.e: below code) until I fininsh reading the file & assign to `file` variable
  ```JavaScript
  const file = fs.readFileSync("./public/index.html");
  console.log("Sync", file.toString("utf-8"));
  ```
  
### Append File
```JavaScript
//APPEND
fs.appendFile("./hello.txt", "This is so cool!", (err) => {
  if (err) {
    console.log(err);
  }
});
```
### Write File
```JavaScript
//WRITE
fs.writeFile("bye.txt", "Sad to see you go", (err) => {
  if (err) {
    console.log(err);
  }
});
```
### Delete File
```JavaScript
//DELTE
fs.unlink("./bye.txt", (err) => {
  if (err) {
    console.log(err);
  }
});
```
 [(Back to top)](#table-of-contents)
 
## Environmental Variables
Node allows to access Environmental Variables via `process.env`

```JavaScript
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`app is running on port ${PORT}`);
});
```

To inject the new Environmental Variables, like in this case is `PORT`, we need to open Bash Shell: Terminal > type "bash"
```
bash-3.2$ PORT=3050 node server.js
app is running on port 3050

```
 [(Back to top)](#table-of-contents)
## Storing User Passwords Securely
- Hashing Function (like SHA256) + add a salt (randomly generated bytes to place in front of the password) for each user
- **Salt Rounds**: This is the cost factor that indicates the amount of time needed to calculate a single bcrypt hash. For example, a cost factor of n (`saltRounds = n`) means that the calculation will be done 2^n times.
  - add a saltRound (10 is the recommended value) which iterates 2^10, or 1024 times over the password in a process called **key stretching**. 
```JavaScript
$ npm install bcrypt
/*
* You can copy and run the code below to play around with bcrypt
* However this is for demonstration purposes only. Use these concepts
* to adapt to your own project needs.
*/
 
import bcrypt from'bcrypt'
const saltRounds = 10 // increase this if you want more iterations  
const userPassword = 'supersecretpassword'  
const randomPassword = 'fakepassword'
 
const storeUserPassword = (password, salt) =>  
  bcrypt.hash(password, salt).then(storeHashInDatabase)
 
const storeHashInDatabase = (hash) => {  
   // Store the hash in your password DB
   return hash // For now we are returning the hash for testing at the bottom
}
 
// Returns true if user password is correct, returns false otherwise
const checkUserPassword = (enteredPassword, storedPasswordHash) =>  
  bcrypt.compare(enteredPassword, storedPasswordHash)
 
 
// This is for demonstration purposes only.
storeUserPassword(userPassword, saltRounds)  
  .then(hash =>
    // change param userPassword to randomPassword to get false
    checkUserPassword(userPassword, hash)
  )
  .then(console.log)
  .catch(console.error)

```
 [(Back to top)](#table-of-contents)
## Building a Server
 
  ```JavaScript
  const http = require("http");

  const server = http.createServer((request, response) => {
    //Request
    console.log("headers", request.headers);
    console.log("method", request.method);
    console.log("url", request.url);

    //Response
    const user = {
      name: "CodeXplore",
      hobby: "Hello World",
    };

    response.setHeader("Content-Type", "application/json");
    
    //Since sending through server -> must be in JSON format
    response.end(JSON.stringify(user));
  });

  server.listen(3000);
  ```
   [(Back to top)](#table-of-contents)

# Introduction to Express JS
  - [Most Popular Back-End Framework Survey](https://2019.stateofjs.com/back-end/)
## Building a Server via Express
  - Express is module to **build the server** instead of re-write the build server code again & again

  ```JavaScript
  const express = require("express");
  const app = express();

  app.get("/", (req, res) => {
    const user = {
      name: "CodeXplore",
      hobby: "Hello World",
    };
   
    res.send(user); //Express help to auto-JSON.stringify
  });

  app.listen(3000);
  ```
   [(Back to top)](#table-of-contents)

## Loading Static Files
- Static Files: **index.htmls, css**
  - Step 1: Create a `public` folder in same directory with `server.js`
  - Step 2: In `server.js`, add this: `app.use(express.static(__dirname + "/public"));`

 [(Back to top)](#table-of-contents)
# Express Middleware

  ```JavaScript
  //Express Middleware: do smtg with request to make it easier to work with
  app.use((req, res, next) => {
    console.log("Middleware");
    //Do something here 
    
    next();
  });
  ```

# RESTful API
- Define a set of functions in the server (URL Param) which developer can perform the requests and receive the response from the server
- RESTFul means creating the rules that everybody agrees on the compatibity between systems
  - GET: receive the resource
  - PUT: update the resource
  - POST: create the resource
  - DELETE: remove the resource
- State-less: call can be made independently, and each call contains all necessary data to complete itself successfully 
  - i.e: Server does not need to remember anything, each comming request has enough information that the server can response 

## Request

   | Type   |     Commands      |  Description |
   |----------|:-------------:|------|
   |`req.query`| https://localhost:3000/ `?name=CodeXPlore&age=25` | `{name: 'CodeXPlore', age: '25' }`|
   |`req.body`| **form-data**, **urlendcoded**, **raw-json**<br/>Please refer Postman/Body part||
   |`req.headers`| `Content-Type : application/json` <br> `name : CodeXplore` | |
   |`req.params`| `app.get("/:id")` | |

## Response
- JSON  : `res.json(user.entries)`
- Status: `res.status(200).send("Sending Res");`

 [(Back to top)](#table-of-contents)
 
 
# CORS
### What is CORS
- Thanks to the **same-origin policy** followed by `XMLHttpRequest` and `fetch`, JavaScript can only make calls to URLs that live on the same origin as the location where the script is running. For example, if a JavaScript app wishes to make an AJAX call to an API running on a different domain, it would be blocked from doing so thanks to the same-origin policy.

- **Cross-Origin Resource Sharing (CORS)** is a protocol that enables scripts running on a browser client to interact with resources from a different origin
  - For example, if you're running a React SPA running on http://localhost:3001  that makes calls to an API backend running on a different domain http://localhost:3000 (in this case different port 3001 & 3000)


### CORS Request Types
- There are *two types of CORS request*: "simple" requests, and "preflight" requests, and it's the browser that determines which is used.
  - **"Simple"** request: 
    - One of these methods is used: `GET`, `POST`, or `HEAD`
    - Using the `Content-Type` header, only the following values are allowed: `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`

  - **"Preflight"** request: 
    - If a request does not meet the criteria for a simple request, the browser will instead make an automatic preflight request using the `OPTIONS` method.
    - The preflight request sets the mode as `OPTIONS` and sets a couple of headers to describe the actual request that is to follow:
      - `Access-Control-Request-Method`: The intended method of the request (e.g., GET or POST)
      - `Access-Control-Request-Headers`: An indication of the custom headers that will be sent with the request
      - `Origin`: The usual origin header that contains the script's current origin
    
      - An example of such a request might look like this: This request basically says "I would like to make a `GET` request to localhost:3001/api/ping  with the `Content-Type` and `Accept` headers from http://localhost:3000 - is that possible?".
    
      ```
      # Request
      curl -i -X OPTIONS localhost:3001/api/ping \
      -H 'Access-Control-Request-Method: GET' \
      -H 'Access-Control-Request-Headers: Content-Type, Accept' \
      -H 'Origin: http://localhost:3000'
      ```

### Identifying a CORS Response
- When a server has been configured correctly to allow cross-origin resource sharing, some special headers will be included. 
- the primary one that determines who can access a resource is `Access-Control-Allow-Origin`

- So a response to the earlier example might look like this:
  ```
  HTTP/1.1 204 No Content
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE
  Vary: Access-Control-Request-Headers
  Access-Control-Allow-Headers: Content-Type, Accept
  Content-Length: 0
  Date: Fri, 05 Apr 2019 11:41:08 GMT
  Connection: keep-alive
  ```
  - `Access-Control-Allow-Origin` header, in this case `*`, allows the request to be made from any origin.
  - `Access-Control-Allow-Methods` header describes only the accepted HTTP methods, in this case: `GET,HEAD,PUT,PATCH,POST,DELETE`
  
### How to Add CORS to a Nodejs Express App
 ```JavaScript
 //In server.js file of Express:
 
 const cors = require("cors"); // bring in the cors library
 
 //replace custom middleware with the cors() middleware
app.use(cors());
 ```
 
 - As an example of how to do this, you can reconfigure the CORS middleware to only accept requests from the origin that the frontend is running on. Modify the cors() setup from the previous example to look like the following:
 
 ```JavaScript
 app.use(
  cors({
    origin: "http://localhost:3000", // restrict calls to those this address
    methods: "GET" // only allow GET requests
  })
);
 ```
 
  [(Back to top)](#table-of-contents)

# Postman
### Body
- **form-data**: similar to form submit in Front-End with Key-Value pair
- **x-www-form-urlendcoded**: similar to form submit in Front-End with Key-Value pair
- **raw**: select JSON

   [(Back to top)](#table-of-contents)
   
# Production Deployment
How to deploy the full-stack application below to Production ?
![Screenshot 2020-09-26 at 12 53 00 PM](https://user-images.githubusercontent.com/64508435/94330453-77321f80-fff7-11ea-9fed-045160d9a132.png)


### Back-End Heroku Deploy

In package.json: On Heroku, dev dependencies never get installed => need to modify start script
- In Heroku, npm start runs by default and loads server.js from a normal node command
- Locally, just run in your terminal `npm run start:dev`
```JavaScript
"scripts": {
    "start": "node server.js",
    "start:dev": "nodemon server.js"
    .....    
  },
```
- [Step 1]: Login to Heroku Web => Create an app "project-name-back-end-app"
- [Step 2]: Go to Terminal @ Back-End Folder
If you haven't already, log in to your Heroku account and follow the prompts to create a new SSH public key.
```
$ heroku login
```
Clone the repository
Use Git to clone smart-brain-codexplore's source code to your local machine.
```
$ heroku git:clone -a project-name-back-end-app
```
Check if git origin move to heroku
```
git remote -v

heroku  https://git.heroku.com/project-name-back-end-app.git (fetch)
heroku  https://git.heroku.com/project-name-back-end-app.git (push)
```

```
cd back-end
git init 

git add .
git commit -m "my First Commit"

#Push the App to Heroku Server
git push heroku master
```
Once the App is uploaded, there is the new Back-End link: https://project-name-back-end-app.herokuapp.com
--> NEED to CHANGE all `fetch(http://localhost:3000)` in Front-End app to `fetch(https://project-name-back-end-app.herokuapp.com)`

### Database Heroku Connection
- [Step 1]: Select Database > create Postgre DB > Install Heroku Postgres with Back-End app
- [Step 2]: Go to Terminal @ Back-End Folder
  
   | Step   |      Command      |  Description |
   |----------|:-------------:|------|
   | 1 | `heroku addons`| to add the **heroku-postgresql** database to the |
   |2|`heroku pg:info`| Check the info of the DB to modify `Back-End Knex` DB Connection|
   |3|`heroku pg:psql`| To access PSQL DB to create Tables|
   |4|`heroku logs --tail`| Check Server log |
   |5|`heroku config`| To check the `DATABASE_URL`|

```
--Create "Users" Table
CREATE TABLE users (
    id serial PRIMARY KEY,
    name VARCHAR(100),
    email text UNIQUE NOT NULL,
    entries BIGINT DEFAULT 0,
    joined TIMESTAMP NOT NULL
);
--Create "Login" Table
CREATE TABLE login (
    id serial PRIMARY KEY,
    hash VARCHAR(100) NOT NULL,
    email text UNIQUE NOT NULL
);
```

- [Step 3] Modify `DB Knex` connection in Back-end `server.js` file

```JavaScript

process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0;   //Need to modify like this to use Free DB on Heroku
const db = knex({
  client: "pg",
  connection: {
    connectionString: process.env.DATABASE_URL, //DATABASE_URL will be in the Environment Variable on Heroku
    //Need to modify like this to use Free DB on Heroku
    ssl: {
      rejectUnauthorized: false,
    },
  },
});
```
### Front-End Heroku Deploy
- [Step 1] To avoid any errors when deploying to Heroku (some students have mentioned that their deployment fails because of a certain version of create react app that they are using), there is a way to make sure your deployment succeeds.
```
npm install serve --s
```
Replace the npm start command  in `package.json` of Front-End folder like this:
```JavaScript
"scripts": {
    "start": "serve -s build",
    // it used to be like this, which you can remove now:
    // "start": "react-scripts start",
```
- [Step 2]: Login to Heroku Web => Create an app "project-name-front-end-app"
- [Step 3]: Go to Terminal @ Back-End Folder
If you haven't already, log in to your Heroku account and follow the prompts to create a new SSH public key.
```
$ heroku login
```
Clone the repository
Use Git to clone smart-brain-codexplore's source code to your local machine.
```
$ heroku git:clone -a project-name-front-end-app
```
Check if git origin move to heroku
```
git remote -v

heroku  https://git.heroku.com/project-name-front-end-app.git (fetch)
heroku  https://git.heroku.com/project-name-front-end-app.git (push)
```

```
cd back-end
git init 

git add .
git commit -m "my First Commit"

#Push the App to Heroku Server
git push heroku master
```


[(Back to top)](#table-of-contents)
