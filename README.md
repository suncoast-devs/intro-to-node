# Intro to Node

Welcome! This is the step by step guide to following along with the Intro to Node.js Crash course at TIY Tampa/St Pete. We will be creating a Top 100 Album website with node and express that pulls data from the iTunes API. 


## Setup

### Installation
We need to install a few things first:

- Node (the runtime)
- Express-Generator (the web framework generator)
- Yarn (package manager)


#### Mac
```
brew install node
npm install express-generator -g
npm install yarn -g
```

#### On Windows

Install node via [https://nodejs.org/en/download/](https://nodejs.org/en/download/) and then use powershell & npm to install express and yarn

``` 
npm install express-generator -g
npm install yarn -g
```

#### Other

For a complete list of ways to install node for other systems. Check this out [https://nodejs.org/en/download/package-manager/](https://nodejs.org/en/download/package-manager/)

##  Step one

Lets use the generator to create a new app

```
mkdir CrashCourse
cd CrashCourse
express Top100
cd Top100 && yarn
```

You should now be able to run to with 

```
yarn start
```

See the default website at http://localhost:3000

### Add a watcher task

Lets add a watcher to the package json so we dont need to restart our all the time by ourselves


``` 
yarn add nodemon
```

add to project.json under the scripts section

``` json
 "watch":"nodemon ./bin/www"
```

so your project json should look like this:

 ``` json
{
  "name": "top100",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www", 
    "watch":"nodemon ./bin/www"
  },
  "dependencies": {
    "body-parser": "~1.17.1",
    "cookie-parser": "~1.4.3",
    "debug": "~2.6.3",
    "express": "~4.15.2",
    "jade": "~1.11.0",
    "morgan": "~1.8.1",
    "nodemon": "^1.11.0",
    "serve-favicon": "~2.4.2"
  }
}

 ```

We should be able to run it now 

```
yarn watch
```

This will start the app at http://localhost:3000 and now anytime you make changes to the server it restart and show the latest

## Need to get the data

Instead of a database, we will using the ITunes API to get our data, lets add a new module to our program. Reading from a database would use the exact same paradigm

```
yarn add request
```

Now in the `routes/index.js`, lets add a refernce to that module

``` js
const request = require("request");
```

And we need to make the call to the API

``` js
const _url = "https://rss.itunes.apple.com/api/v1/us/apple-music/top-albums/all/100/explicit.json"
  request.get(_url, (error, response, body) => {
    const _json = JSON.parse(body);
    const _top100 = _json.feed.results;
    console.log({_top100: _top100[0]})
    res.render('index', { title: 'TIY Top 100 of the day', data: _top100 });
  });
```


## Jade!

### Add some css (bootstrap!)
Lets add bootstrap to our site using a cdn. Since we want this on every page, we should must put this on our layout page. Open `views/layout.jade` and add this to head

``` jade
    link(rel='stylesheet', href='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css')
```

And add this to the end of body

``` jade
script(src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js")
script(src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js")
    
```


### Add some html

Now we have the data in the view, lets add some html to display the data:

Open `views/index.jade`  and replace it with the following pug

``` jade 
extends layout

block content
  .row
    .col-xs-6.col-xs-offset-3
      div
         h1= title
      br
      br
  .row
    .col-xs-10.col-xs-offset-1
      .row
      - for (let i = 0; i < data.length; i++)
          .col-sm-6.col-md-4
            .thumbnail
              img(src=data[i].artworkUrl100, alt=data[i].artistName, class="img-responsive img-thumbnail")
              .caption
                h4(class="text-nowrap ellipsis")!= data[i].name
                p 
                 .labels
                   - for (let j = 0; j < (data[i].genres.length > 3 ? 3 : data[i].genres.length) ; j++)
                    span(class="label label-info genre-label" )= data[i].genres[j].name
                p
                  a.btn.btn-primary(href=data[i].url, target="_blank") More Detail
        
        
```


### add some custom CSS
Now lets add some custom css to `public/stylesheets/style.css`

```css
.genre-label{
  margin: 0.5em;
  display: inline-block;
}

.ellipsis{
     white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.ablum-cover{
  max-height: 14em;
}
```

Now if we refresh the browser we should see the top 100 in not  too bad of a format

## (Bonus) Search

Now that we have our basic site created, lets create a way for a user to search by a term.

### add new route
We need to add a new route to our `index.js` in order to add the new page

Add this to `routes/index.js`

```js
router.get('/search', function (req, res, next) {
  let _term = req.query.needle;
  let _url = "https://itunes.apple.com/search?&entity=album&term=" + _term;
  request.get(_url, (error, response, body) => {
    let _json = JSON.parse(body);
    let _top100 = _json.results;
    res.render('index', { title: 'TIY Top 100 of the day', data: _top100, needle:_term });
  });
});
```

### Add HTML

No we need to update the pug to have a search bar & render the new data format

``` jade
extends layout

block content
  .row
    .col-xs-6.col-xs-offset-3
      div
         h1= title
      br
      br
  .row
    .col-xs-10.col-xs-offset-1
      form(method="GET", action="/search", )
        .form-group
          .input-group
            input#searchBar.form-control(type='text', placeholder='Search', name="needle", value=needle || "")
            span.input-group-btn
              button(class="btn btn-primary", type="submit") Search!

  .row
    .col-xs-10.col-xs-offset-1
      .row
        - for (let i = 0; i < data.length; i++)
          .col-sm-6.col-md-4.result-container
            .thumbnail
              img(src=data[i].artworkUrl100.replace("100x100bb", "200x200bb"), alt=data[i].artistName, class="img-responsive img-thumbnail ablum-cover")
              .caption
                h4(class="text-nowrap ellipsis")!= data[i].name || data[i].collectionName
                p 
                 .labels
                   - if (data[i].genreNames)
                    - for (let j = 0; j < (data[i].genreNames.length > 3 ? 3 : data[i].genreNames.length) ; j++)
                      span(class="label label-info genre-label" )= data[i].genreNames[j]
                   - else 
                    span(class="label label-info genre-label")= data[i].primaryGenreName
                p
                  a.btn.btn-primary(href=data[i].url || data[i].collectionViewUrl, target="_blank") More Detail
        
```
