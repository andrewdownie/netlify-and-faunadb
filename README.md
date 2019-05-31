This tutorial uses this repo as a starting point:  
https://github.com/andrewdownie/standalone-serverless-backend-using-netlify-lambda

## 1) Get a faunadb secret key and save the key into your netlify environment variables
TODO: This is easy, I'll try to find the resource I followed along with.
Save your key to a variable with the name of **FAUNADB_SERVER_SECRET**. This variable will get pulled in during the netlify build, and we will reference it in step 4 using `process.env.FAUNADB_SERVER_SECRET`.

## 2) In the root folder of standalone-serverless-backend-using-netlify-lambda, install the faunadb package:  
```js
npm i faunadb
```  

## 2) In the root folder, create a file named webpack.config.js, and file it with the following:  
This will fix an import error caused by the netlify-lambda package when trying to import node modules
The **mode: "development"** is optional, and will prevent the resulting JS from being uglified, which is needed for debugging code issues
```js
var webpack = require('webpack');

modules.export = {
  plugins: [
    new webpack.DefinePlugin({"global.GENTLY": false})
  ],
  node: {
    __dirname: true
  },
  mode: "development"
}
```

## 3) Modify the start and build scripts in the package.json file to reference the webpack.config.js file you just created:  
Just add ` -c ./webpack.config.js` to the end of each script command
```js
"scripts": {
  ...
  "start": "netlify-lambda serve src -c ./webpack.config.js",
  "build": "netlify-lambda build src -c ./webpack.config.js"
  ...
}
```

## 4) Create a new file in the src folder called 'createDB.js', fill it with the following:  
Fun fact: if you don't setup an entry in indexes, you will be able to add items to your faunadb class no problem, but you wan't be able to see them unless you know their faunadb id. Creation of indexes entry is recommended unless you have a reason not to.
```js
import faunadb from 'faunadb';
const q = faunadb.query;

exports.handler = (event, context, callback) => {
    const client = new faunadb.Client({
        secret: process.env.FAUNADB_SERVER_SECRET
    })

    // create the class Messages
    client.query(q.Create(q.Ref('classes'), {name: 'Messages'}))
    .then(() => {
    
        // create an index so we can look at the contents of the Messages class
        client.query(q.Create(q.Ref('indexes'), {
            name: 'all_messages',
            source: q.Ref('classes/Messages')
        }))
        .then(() => {
            callback(null, {
                statusCode: 200,
                body: 'database meow',
                headers: {
                    'Access-Control-Allow-Origin': '*'
                }
            })
        })
    })
    
};
 
```

## 5) Open your faunadb in the faunadb web UI to confirm that the Message class and the index have been created

