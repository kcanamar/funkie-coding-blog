---
author: "KCanamar"
date: "2023-03-08"
title: "Building a CRUD API with Golang, Gin, Gorm, Bit.io, Bonus Deployment to Render.com"
lang: "Go"
---

So you want to build an api with GO? Let's do it. The following is a step by step guide on how to build a full CRUD api using Gin. Then we will go one step futher and deploy our api to Render.com.
<!-- more -->
### Prerequisites
There are not that many just the normal housekeeping so we can code together.
- Have VsCode or a text editor - [Download VsCode](https://code.visualstudio.com/)
- Need Golang installed on your machine - [Download Golang](https://go.dev/dl/)
- Register for a FREE account with [Bit.io](https://bit.io/)
- Register for a FREE account with [Render.com](https://render.com/)
- Have Postman or equivalent installed on your machine [Postman](https://www.postman.com/downloads), Dont use the browser based Api Tests they will not work with `localhost` development
- Need a Github Account to host your code. [Github](https://github.com/signup)

I will be linking along the way the links to relevant documentation in case you run into any errors or need futher clarification. Look for these [_Check The Docs_]() along the way.

# We're gonna learn Today!

### Goals
1. Be able to write a RESTful API using Golang, Gin.
2. Connect to bit.io to persist our data.
3. Deploy our api to render.com

##### Take Note - when working with Go is that there is a specific way we have to structure our directories and subsequet files. I am not going to be covering how or why in this post but feel free to [_Check The Docs_](https://go.dev/doc/gopath_code)


Open terminal and navigate to where go code is stored - you can check to see where after installing Go by running command `go env` 
   ```bash
   cd <where ever you have go installed>
   ```

Next run the bash command 
   ```bash
   mkdir github.com
   ```

Now we are going to move our terminal into that newly created directory 
```bash
   cd github.com
```

Bring it back now y'all, okay enough fun. How about we create another directory this time named after our github profile, run the bash command 
   ```bash
   mkdir username
   ```

Now lets navigate into the newly created directory with the command.
```bash
   cd username
```

Alright now let's do that process on more time but for what we want our project to be called, for our purposes let us call this `go-gin-bit-crud`
```bash
mkdir go-gin-bit-crud
```

Now, you guessed it we are going to change directories again to our newly created project directory.
```bash
cd go-gin-bit-crud
```

Now that we are in the correct directory lets go ahead and open up this directory in vscode, in our terminal enter the following command. *If you are unable to open vscode this way [Check the Docs](https://stackoverflow.com/questions/29963617/how-to-call-vs-code-editor-from-terminal-command-line) *

```bash
code .
```

***Note*** - Be sure to have the Go extension for vscode _[Check the Docs](https://code.visualstudio.com/docs/languages/go)_

### Start here if you are familiar with setting up your go projects, and having it open in VsCode

Now since we are going to be deploying this on render.com we are going to need to make sure that we initialize a git repo in our project directory. We do this by running the folowing command.

```bash
git init
```

Now we should make our first commit to start our git history. rRuning the following commands

```bash
git add .
git commit -m "first commit"
```

Now we need to create the go equivalent of a `package.json` , might seem similar if you are coming from `node.js` . The `go.mod` acts as our package manager, this is where we will be installing all of our dependencies for this project. So let us opne up a terminal in vscode in our project directory and run the following command.
```bash
go mod init
```

### Here come the Packages

We are goin to be using `CompileDaemon` to watch `.go` files and invokes `go build` if a file changed, _[Check the Docs](https://github.com/githubnemo/CompileDaemon)_ , again another `node.js` reference this will be similar to `nodemon` 
```bash
go get github.com/githubnemo/CompileDaemon
```

Now we want to be able to use `CompileDaemon` from the command line, so we are going to have to install it.
```bash    
go install github.com/githubnemo/CompileDaemon
```

Alright now we are going to want to setup our environment variables, for this we are going to need the package `godotenv` - _[Check the Docs](https://github.com/joho/godotenv)_

```bash
go get github.com/joho/godotenv
```

Let's keep it moving by adding our web framework package `GIN` - _[Check the Docs](https://gin-gonic.com/docs/quickstart/)_

```bash
go get -u github.com/gin-gonic/gin
```

Now that we have our web framework we are also going to want a way to talk with our database so for this project we are going to be using the ORM `GORM` - _[Check the Docs](https://gorm.io/docs/index.html)_
```bash
go get -u gorm.io/gorm
```

We are going to need add a second part of `GORM` this will be the postgreSQL driver that way we can communicated with bit.io - _[Check the Docs](https://gorm.io/docs/connecting_to_the_database.html#PostgreSQL)_

```bash
go get -u gorm.io/driver/postgres
```

### Finally, we can start writing some code

Lets keep it going by staying in the terminal for a little longer, create a main.go file in the projects root directory.

```bash
touch main.go
```

Here is the code that will be in `main.go` to make sure that we have all of our packages in our `go.mod` setup correctly.

```go
// main.go

// delcare package main
package main

import (	
	"fmt"
)

// declare main function
func main() {
	fmt.Println("Hola")
}
```

Test to make sure that compiledaemon is working with command.

```bash
CompileDaemon -command="./project_name"
```

Let us verify that the canges are happening by changing the return value to `fmt.Println("Hi {your name}")`

Since we have verified that `CompileDaemon` is running and our server is refreshing with changes let us setup our basic server listening on our home route. _[Check the Docs](https://gin-gonic.com/docs/quickstart/)_

```go
// main.go

import "github.com/gin-gonic/gin"

func main() {
	// setup a gin router
	router := gin.Default()
	// Declare home get route
	router.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{
		"message": "Hello from the other side!",
		})
	})
	// setup our server listener and serve on PORT 0.0.0.0:8080
	router.Run()
}
```

In our browser, I am using Chrome, lets navigate to `localhost:3000` and confirm we see our message.

Since we are going to deploying this we are going to want to setup our `dotenv` package by touching a `.env` file into the root directory along with a `.gitignore` that way our `git commit` doesn't expose our environment variables, if you have a global `.gitignore` you are ahead of the game.

```bash
touch .env .gitignore
```

Define a `PORT` variable in the `.env` file and set to `PORT=3000` we are also going to be adding in an `ENV` variable to help with deployment, this will come in handy later.

```env
// .env

PORT=3000
ENV="development"
```

Lets make sure to setup our `.gitignore` too. 

```env
// .gitignore

.env
```

How about we connect the `.env` to our `main.go` . We are going to have to create a `func init()` that will load our environment variables.

```go
import (
	"log"
	"github.com/gin-gonic/gin"
	"github.com/joho/godotenv"
)

func init() {
	err := godotenv.Load()
	
	if err != nil {
		log.Fatal("Error loading .env file")
	}
}

func main(){ ...
```

Now to keep our code looking polished lets move this env section in our `init()` to its own file, we can start this by adding a new directory into root directory of the project. 

```bash 
mkdir initializers
```

Then lets add a file to hold our code.

```bash
touch initializers/loadEnv.go
```

Now we can relocate the init functions contents to `initializers/loadEnv.go` and beef it up a little more, we are going to check our file path for our environment variables since deploying on render.com has its own challenges, and this will make sure that our deployment goes smooth. _[Check the Docs](https://stackoverflow.com/questions/54456186/how-to-fix-environment-variables-not-working-while-running-from-system-d-service)_

```go
//loadEnv.go

package initializers

import (
	"log"
	"os"
	"path/filepath"
	"github.com/joho/godotenv"
)

// Besure to use PascalCase when naming your Helper functions
func LoadEnv() {

	// assign dir to the root directory of our preject
	dir, err := filepath.Abs(filepath.Dir(os.Args[0]))

	// check for errors
	if err != nil {
		log.Fatal("Error loading .env file")
	}

	// assign our environment path to the .env file of the root directory
	environmentPath := filepath.Join(dir, ".env")

	// reassign error to the return value of godotenv
	err = godotenv.Load(environmentPath)

	// Check for errors
	if err != nil {
		log.Fatal("Error loading .env file")
	}
}
```

After we have setup  provide our environment variables to our server, for this we will need to import the `initializers` package into main.go.

```go
//main.go
...
import (
	"github.com/gin-gonic/gin"
	"github.com/kcanamar/go-gin-bit-crud/initializers"
)

func init() {
	// this will safegaurd our production deployment
	if os.Getenv("ENV") != "production" {
		initializers.LoadEnv()
	}
}
...
```
# Connect to Bit.io

Now lets connect a database, touch another file into the initializers directory

```bash
touch initializers/database.go
```

For this connection we will be using `bit.io` as our database, bit.io software is _a platform used to secure Postgres database_. 
If you do not have a bit.io account head over to bit.io and create a free account [here](https://bit.io/)

Now you that you have created/ logged in to your `bit.io` account, you will see a button by the top left of the dashboard labeled `+ new`. For the purposes of this walkthough you can name this database however you like. 

Once the database is setup we are going to select the connect opiton. You will find this near the top right of the screen, once selected our dashboard will now display our connection information.
Be sure to leave this tab open.

Now that we have our database setup and on the connection screen lets setup `database.go`, first we should delcare what package this file belongs to, and import our Gorm dependencies.

```go
//database.go

package initializers

import (
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

func ConnectDB() {

	dsn := "{your postgresql connection string}?sslmode=allow"

}
```
_*IMPORTANT*_ - we need to add the query parameter `sslmode` and set the value to `allow`, this is due to the security setting associated with `bit.io` [_Check the Docs_](https://www.postgresql.org/docs/current/libpq-ssl.html)

Next we will create a global variable to access in any of the files `var DB *gorm.DB`
Then we need to impement some error handling local scoped to our `ConnectDB()` function `var err error` 
Next we are going to asign the variables we have created to the result of our connection attempt, handling our errors if any with a fatal response `log.Fatal("Failed to connect to the database")` [_Check the Docs_](https://gorm.io/docs/connecting_to_the_database.html#PostgreSQL)

```go
//database.go
package initializers

import (
	"log"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)
// create a global variable for Database
var DB *gorm.DB

func ConnectDB() {
	// create local scoped error
	var err error
	
	dsn := "{your postgresql connection string}?sslmode=allow"
	DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
	
	// error handling
	if err != nil {
		log.Fatal("Failed to connect to database")
	}
}
```

Now that we have all of this running when we check our termial running compiledaemon we should see no errors. However if you do encounter one please go back and check for syntaxing errors, or consult the supporting documentation provided at each step.

No error, let's go! Since we are currently exposing our database connection string, a huge security risk, let's relocate that string into our `.env` file with a key name of `DATABASE_URL`

```env
// .env

PORT=3000
ENV="development"
DATABASE_URL={your postgresql connection string}

```

Now we use that variable is safely hidden in our environment variables, lets up date our code to use our connection string with `os.Getenv("Variable_Name")`

```go
//database.go
package initializers

import (
	"log"
	"os"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)
var DB *gorm.DB

func ConnectDB() {
	var err error

	// .env variable
	dsn := os.Getenv("DATABASE_URL")
	DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})

	if err != nil {
		log.Fatal("Failed to connect to database")
	}
}
```

With the database connected, lets move on and create some models for our data, for this we will need to add a models directory in the project's root directory.

```bash
mkdir models
```

Next 

```bash
touch models/postModel.go
```

We are going to be embedding the `gorm.model` to include our `ID, CreatedAt, UpdatedAt, DeletedAt` fields to our struct with our haing to write them all out, thank you gorm. [_Check the Docs_](https://gorm.io/docs/models.html#gorm-Model)

We are going to add two more fields to our struct, since this is a `Post` it should have at minimum a `Title` and `Body` both as string datatypes.

```go
// postModel.go
package models

import "gorm.io/gorm"

type Post struct {
	gorm.model
	Title string
	Body string
}
```

Now that we have a model setup, we need to migrate to our database. Todo this we are going to create, you guessed it, another directory in the root directory `mkdir migrate`, then `touch migrate/migrate.go`

```bash
mkdir migrate
```

Then

```bash
touch migrate/migrate.go
```

Now that we have our file created lets add some dependencies. First we are going to need our `initializers` package, this will enable us to access both `LoadEnv()` and `ConnectDB()` funcs from with in `migrate.go`'s init func. Second we are going to need our `models` package, pretty straight forward, this will describe the shape our data(schema) must be in to reach the database.
Finally in our main func we are going to utilize our global `DB` variable to `AutoMigrate` our data. [_Check the Docs_](https://gorm.io/docs/index.html#Quick-Start)

```go
// migrate.go
package main

import (
	// Import dependencies
	"github.com/kcanamar/go-gin-bit-crud/initializers"
	"github.com/kcanamar/go-gin-bit-crud/models"
)

func init() {
	// Import initializers
	initializers.LoadEnv()
	initializers.ConnectDB()
}

func main() {
	// AutoMigrate takes a model struct argument
	initializers.DB.AutoMigrate(&models.Post{})
}
```

Once the migrte file is all setup lets migrate with command 

``` bash
go run migrate/migrate.go
```
# Create

So long as there are no errors we will see the table created in the `data` section of bit.io. Take a moment to celebrate, we have just now completed setting up our API with Bit.io. Now off to do some CRUD operations with our `Post` model.

I know what you are going to say, "KCan, are we really going to make another directory?", short answer - Yup. With our terminal in the project root directory

```bash
mkdir controllers
``` 

and then

```bash
touch controllers/postCtrl.go
```

In `postCtrl.go` we will be defining the route controls for GET, POST, PUT, and DELETE requests related to our post model. We will start off by gathering our dependencies, from `Gin`. Then start our process with the "C" in CRUD, Create. Let's take a small step first by, sending back a successful message when the `PostsCreate` controller is called.

```go
// postCtrl.go
package controllers

import "github.com/gin-gonic/gin"

func PostsCreate(c *gin.Context) {
	c.JSON(200, gin.H{
		"message": "Hello from the other side!",
	})
}
```

Next, we need to connect the server to our database, then mount the controllers within our server. We are going to need to import our `controllers` package. Since we already have the `initializers` package imported we already have access to `ConnectDB()`. Now in our home route, lets add in our newly created controller and check to make sure that everything is function as intended.

```go
// main.go
package main

import (
	"os"
	"github.com/gin-gonic/gin"

	// import controller package
	"github.com/kcanamar/go-gin-bit-crud/controllers"
	"github.com/kcanamar/go-gin-bit-crud/initializers"
)

func init() {
	if os.Getenv("ENV") != "production" {
		initializers.LoadEnv()
	}
	// connect the database
	initializers.ConnectDB()
}

func main() {
	router := gin.Default()
	
	// include the contoller
	router.GET("/", controllers.PostsCreate)
	
	router.Run()
}
```

Lets head over to `localhost:3000` to make sure the server is giving use the feedback we want. 
Now lets modify our code to send some data using a POST request

```go
// main.go
...

func main() {
	router := gin.Default()
	
	// Create Route should use a POST http verb
	router.POST("/posts", controllers.PostsCreate)
	
	router.Run()
}
```

Now that we are using the right Http grammer, lets head to `postCtrl.go` to refactor our code so we can send data to our database with a payload.

First let's go ahead and create a new struct `reqBody` based on our model to hold the request information. Once we have our `reqBody` struct we need to bind our JSON data with it, we achieve this by calling the `Bind` method from `*gin.context`, passing a pointer to our `reqBody` struct. [_Check the Docs_](https://gin-gonic.com/docs/examples/binding-and-validation/)

Second we are going to use our `reqBody` to populate a `newPost` struct based on our `Post` model[_Check the Docs_](https://gorm.io/docs/create.html#Create-Record). Now that we have our `newPost` built from our `reqBody` we need to send this data to our database, creating the new post. This will be achieved by creating a variable `result` and assinging it the result of Gorm's `.Create` method with the argument of a pointer to `newPost`.

Third, we need to implement some error handling incase something unexpected happens.

Lastly, we will respond with a 200 status and provide postive feedback with the newly created posts data.

```go
// postCtrl.go
package controllers

import (
	"github.com/gin-gonic/gin"
	"github.com/kcanamar/go-gin-bit-crud/initializers"
	"github.com/kcanamar/go-gin-bit-crud/models"
)

func PostsCreate(c *gin.Context) {

	// Get data from request body
	// create variable
	var reqBody struct {
		Title string
		Body string
	}

	// parse the request body as JSON then binds to reqBody via a pointer
	c.Bind(&reqBody)

	// Create a newPost 
	newPost := models.Post{Title: reqBody.Title, Body: reqBody.Body}
	result := initializers.DB.Create(&newPost)
  
	// handle error from result
	if result.Error != nil {
		c.Status(400)
		return
	}
	// return post
	c.JSON(200, gin.H{
		// send back a confirmation of the post
		"newPost": newPost,
	})
}
```

Time to test, open up postman so we can send a JSON body to our local host server. In post man we are going to create a new `POST` request to `localhost:3000/posts` sending a `body` as `raw` data in a `JSON` format. Once we have the proper settings selected, and populated our request body payload, send the request.
*Note* - If you are struggling to send your request [_Check the Docs_](https://www.youtube.com/watch?v=qyYAOty_bDs)

We see that it is created! Happy dance!

# Read

Let's keep that momentum going, onto the "R" in CRUD, this is going to be split up into two different routes. One to find all of the posts, and another to find just a single post.

Let's kick it off by making a new function in our `postCtrl.go` file called `PostsIndex` and here we will be getting all of the posts from our database and sending them back in the response as JSON.

Similar to our `PostsCreate` func we need to create an array variable `allPosts` typed of our `Post` model, this will hold all of our posts. Once we have our variable we are going to use the Gorm method `.Find` to query our database, passing the argument of a pointer to `allPosts`. Finally we will respond with all of our posts. [_Check the Docs_](https://gorm.io/docs/query.html#Retrieving-all-objects)

```go
// postCtrl.go

...

func PostsIndex(c *gin.Context){
	
	// variable to hold posts
	var allPosts []models.Post

	// Query the database
	initializers.DB.Find(&allPosts)
	
	// Response with posts
	c.JSON(200, gin.H{
		"posts": allPosts,
	})
}
```

Let's now update our router with this new route, back to `main.go`.

```go
// main.go

func main() {
	router := gin.Default()

	// Index Route
	router.GET("/posts", controllers.PostsIndex)

	router.POST("/posts", controllers.PostsCreate)
	
	router.Run()
}
```

Check to make sure this works by sending a `GET` request, in postman to `localhost:3000/posts`. 
We should see a JSON object with a key of "posts" with a value of an array of posts. 

*Air Guitar* Most excellent!

Now that we can get all of the posts, let focus in to go and get a sinlge post for our Show route, head back to `postCtrl.go` and lets create another func `PostShow`. 

Now that we have seen the pattern a few times, lets recreate it one more time. Create a variable `post` type of `models.Post`. Next we will use the Gorm method `.First` passing in two arguments. the first argument with be a pointer to the `foundPost` variable, and the second argument will be the `id` of the post provided by the request parameters. Finally responding with the `foundPost`. [_Check the Docs_](https://gorm.io/docs/query.html#Retrieving-objects-with-primary-key)

```go
// postCtrl.go

...

func PostShow(c *gin.Context){
	
	// variable to hold post
	var foundPost models.Post

	// Query the database
	initializers.DB.First(&foundPost, c.Param("id"))

	// Response
	c.JSON(200, gin.H{
		"post": foundPost
	})
}
```

How about we add the show controller to our router. We're going, going, back, back to `main.go`.

```go
// main.go

... 

func main() {

	router := gin.Default()

	// Added in the home route just to have a route route for testing the API
	router.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "Hello from the other side!",
		})
	})

	router.GET("/posts", controllers.PostsIndex)
	router.POST("/posts", controllers.PostCreate)

	// Show Route
	router.GET("/posts/:id", controllers.PostShow)

	router.Run()

}
```

Time to test that out by sending a GET request to `localhost:3000/posts/1` and we should expect to see the post with `id=1`. 

# Update

Go ahead and Celebrate 2/4 of the way through CRUD.

Almost there lets update a post now, back to `postCtrl.go` to create our update controller. In this example lets showcase that we can obtain the request parameter another way, for this we will create a variable `id` assigned with the request param `id`. Next we need to create a variable for our `reqBody` again referencing the `Post` models properties. We are also going to need a variable to represnt our `Post` model.

First we query the database to find the post to updated we will be using Gorm's `.Find` method with two arguements, first argument will be a pointer to the `Post` model variable, second argument will be the `id` variable.

Then we are going to chain some Gorm methods together starting with `.Model` with the argument of a pointer to `updatedPost`. Next method in this chain will be `.Update` passing the argument of a struct typed of our model `Post` populated with our data from the `reqBody` variable.[_Check the Docs_](https://gorm.io/docs/update.html#Updates-multiple-columns)

Finishing this off by responding with the `updatedPost`.

```go
// postsCtrl.go

...

func PostUpdate(c *gin.Context){
	// Get the id from params
	id := c.Param("id")
	
	// Get the request body
	var reqBody struct {
		Title string
		Body string
	}
	
	// Find post to be updated
	var updatedPost models.Post
	initializers.DB.Find(&updatedPost, id)
	
	// Update Post
	initializers.DB.Model(&updatedPost).Updates(models.Post{
		Title: reqBody.Title,
		Body: reqBody.Body,
	})
	
	// Respond with updated post
	c.JSON(200, gin.H{
		"post": updatedPost
	})
}
```

Now, you guessed it, update our router in `main.go` with the new controller

```go
// main.go

...
	router.GET("/posts", controllers.PostsIndex)
	router.POST("/posts", controllers.PostCreate)

	// Update Route
	r.PUT("/posts/:id", controllers.PostUpdate)

	r.GET("/posts/:id", controllers.PostShow)
...
```

Time to test, using postman send a `JSON` body as a `PUT` request to `localhost:3000/posts/1`. 
```js
// Request body
{
	"title": "updated"
}
```
_Note_- You can update any number of properties defined in the request body, or just one.

*Air Horn* Let's GO, we should see our post updated with the new title.
We are right there 3/4 of CRUD complete, 4th Quarter, 90th Minute, let finish strong.

# Delete

Now lets `DELETE` a post to round out our C.R.U.D. Operations, back to `postCtrl.go`. 
This is a fairly straight forward controller, we are going to access the Grom method `.Delete` passing two arguments. First argument will be a pointer to our `Post` model, with the second argument being the request param `id` of the post to be deleted. All that is left is responding with a success method. [_Check the Docs_](https://gorm.io/docs/delete.html#Delete-with-primary-key)

```go
// postCtrl.go

...

func PostDestroy(c *gin.Context) {
	// Delete the post
	initializers.DB.Delete(&models.Post{}, c.Param("id"))

	// Response
	c.JSON(200, gin.H{
		"message": "Successful Destory",
	})

}
```

Another one bites the dust, time to add out controller to our router in `main.go`.

```go
// main.go
...
	r.POST("/posts", controllers.PostCreate)
	r.PUT("/posts/:id", controllers.PostUpdate)
	
	// Delete Route
	r.DELETE("/posts/:id", controllers.PostDestroy)

	r.GET("/posts/:id", controllers.PostShow)
...
```

Time to test, using postman send a `DELETE` request to `localhost:3000/posts/1`.

# CRUD Complete

Take a moment and celebrate, we have just built a full CRUD API using Golang, Gin, Gorm, and persiting our data using Bit.io. Hats off. 

Ready to take it one step futher in the Bonus Round?

# Bonus Round - Deployment

Now that the app is built lets deploy it to render.com. Let's run some command in our terminal to prep our git repo, if you recall way back at the beginning we ran command `git init` this turned our project into a git repo. If you haven't been commiting your code, no worries we are going to do it together.

First sequence of command will stage all of our files to be commited, followed by the actual commit with a message of "CRUD API".

```bash
git add .
git commit -m "CRUD API"
```

Now we need to create a new repo on GitHub, so head over to your github dashboard and in the top right corner next to your avatar icon you will see a `+` sign select that and chose from the dropdown the `New Repository` option. 

This will bring up the create a new repository screen we need to provide a name for the repo my suggestion would be "Kcanamar-GO-CRUD-API", but you can name it what ever you want.

Make sure it is set to public, and *DO NOT* Select to add a README file, the only option you will need to change is provide a repo name. Go a head and smash the `Create Repository` button at the bottom of the page.

This will take us to the code tab in our newly created repo, we are going to copy the bottom section of code, and paste it into our terminal.

```bash
git remote add origin {this will be you connection string}
git branch -M main
git push -u origin main
```

Go a head and refresh the page, and you will see you code on Github.

### Moving on to Render

Create a render.com account or login to your account.

We are going to create a new web service, once you are on your dashboard look to the top of you screen and select the `New +` button and select `Web Service` this will promt you to signin to your github account, connecting your github to render, we need to do this.

Armed with our newly created github repo, we can import our repo directly from github, scroll through the options until you find your project name and click connect.

Render will ask you to give your porject a name.

Now let's chose the advanced setting to set up our environment variables [_Check the Docs_](https://render.com/docs/environment-variables), We need to define our enviornment variables, so in YOUR local `.env` we have these variables defined. We do not have to define the `PORT` variable as render.com will do this for us.

Select the `Add Environment Variable` for each of the following
| Key          | Value                                    | Note |
| ------------ | ---------------------------------------- | ---- |
| DATABASE_URL | {use your connection string from bit.io} ||
| ENV          | "production"                             ||
| GO111MODULE  | on                                       |[_Check the Docs_](https://render.com/docs/environment-variables#go)|
| GOPATH       | /opt/render/project/go                   |[_Check the Docs_](https://render.com/docs/environment-variables#go)|

Finally we can click create Web Service and Render will build our project, provided we did everything correct we should have a successful deployment.

# You did it!

I you enjoyed this content or having any recommendations on what I could imporve or adjust please reach out to me through my [portfolio](https://kcanamar-portfolio.vercel.app/contact). 

Follow me on [Twitter](https://twitter.com/funcy_koder) or connect with me through [Linkedin](https://www.linkedin.com/in/kyle-canamar/)