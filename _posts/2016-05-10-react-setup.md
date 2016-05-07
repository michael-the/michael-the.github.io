---
layout: post
title:  "React setup"
date:   2016-05-7 07:00:00 +0300
categories: tutorials
---

# Introduction

I know that there are tones of tutorials on how to do this; but it happens that i have my own opinion on how to setup and here it is.
Also, I assume that nodejs and npm are installed and ready to role.
You can get the whole project ready to clone from [github](https://github.com/michaelthe/react-boilerplate).


# Useful commands

``` bash
# build the project
webpack

# watch for changes and build 
webpack --watch

# dev server startup
webpack-dev-server --progress --colors 
```

# Setting up for development 

First lets install the tools we need.
This will make them available though command line.

``` bash
npm install -g webpack webpack-dev-server
```

Lets start with the initialization of our directory and installing some packages we need. 

``` bash 
mkdir react-demo
mkdir react-demo/src
mkdir react-demo/public
mkdir react-demo/src/components
cd react-demo
npm init
touch webpack.config.js
touch src/entry.jsx
touch src/components/hello.jsx
touch src/components/hello.scss 
npm install --save-dev webpack  
npm install --save-dev react react-dom  
npm install --save-dev style-loader css-loader sass-loader node-sass
npm install --save-dev babel-core babel-loader  
npm install --save-dev babel-preset-react babel-preset-es2015  
```

Our package.json should look something like this 

``` json
{
  "name": "react-demo",
  "version": "1.0.0",
  "description": "react demo",
  "main": "index.js",
  "dependencies": {
    "react-dom": "^15.0.2",
    "react": "^15.0.2"
  },
  "devDependencies": {
    "babel-core": "^6.8.0",
    "babel-loader": "^6.2.4",
    "babel-preset-es2015": "^6.6.0",
    "babel-preset-react": "^6.5.0",
    "css-loader": "^0.23.1",
    "node-sass": "^3.7.0",
    "react": "^15.0.2",
    "react-dom": "^15.0.2",
    "sass-loader": "^3.2.0",
    "style-loader": "^0.13.1",
    "webpack": "^1.13.0"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "react",
    "demo"
  ],
  "author": "michael theodorides",
  "license": "GPL-3.0"
}
```

Now we can write some code.
First we will fill up the webpack.config.js file, that is responsible for the build of the source 

``` js 
var path = require('path');

module.exports = {
    entry: path.resolve(__dirname, "src/entry.jsx"),
    output: {
        path: path.resolve(__dirname, "public"),
        filename: "bundle.js"
    },
    module: {
        loaders: [
            {
                test: /\.s?css$/,
                loaders: ['style', 'css', 'sass']
            },
            {
                test: /\.jsx?$/,
                loader: ['babel'],
                exclude: /node-modules/,
                include: path.resolve(__dirname, 'src'),
                query: {
                    presets: ['react', 'es2015'],
                    cacheDirectory: true
                }
            }
        ]
    }
};
```

Add this lines into the src/entry.jsx.
This is where the application starts.

``` jsx
import React from 'react';
import {render} from 'react-dom';
import Hello from './components/hello.jsx';

document.body.innerHTML = '<div id="react-root">';

render(<Hello />, document.getElementById('react-root'));
```

Add this lines to src/component/hello.jsx
To create some

``` jsx
import React from 'react';
import './hello.scss'

export default class Hello extends React.Component {
    render() {
        return <div className="hello">Hello, world!</div>;
    }
}

export default Hello;
```

Lets add some styles too

``` scss
.hello{
    font-size: 20px;
    font-style: italic;
}
```

At this point we are ready to develop just run the dev server 

``` bash 
webpack-dev-server --progress --colors
```

And browse to [http://localhost:8080/webpack-dev-server/bundle](http://localhost:8080/webpack-dev-server/bundle). 
Here the app will automatically update it self whenever you make changes to your code.   


# Preparing the production setup

Lets add some dependencies and setup an nodejs web server. 

``` bash
touch index.js 
touch public/index.html
touch production.webpack.config.js
npm install --save express
```

Lets code our server. 
add this lines to index.js

``` js
var express = require('express');
var path = require('path');

var app = express();

app.use(express.static(path.join(__dirname, 'public')));

app.use(function (req, res, next) {
    var err = new Error('Not Found');
    err.status = 404;
    next(err);
});

if (app.get('env') === 'production') {
    app.use(function (err, req, res) {
        res.status(err.status || 500);
        res.render('error', {
            message: err.message,
            error: {}
        });
    });
} else {
    app.use(function (err, req, res) {
        res.status(err.status || 500);
        res.render('error', {
            message: err.message,
            error: err
        });
    });
}

var port = process.env.PORT || 3000;
app.listen(port, function () {
    console.log('Browse to: localhost:' + port);
});
```

Add this lines into the public/index.html 

``` html
<html>
<head>
    <meta charset="utf-8">
</head>
<body>
<script type="text/javascript" src="/bundle.js" charset="utf-8"></script>
</body>
</html>
```

Now build and run you application

``` bash
webpack --config production.webpack.config.js 
node index.js
```
