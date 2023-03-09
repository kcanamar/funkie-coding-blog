---
author: "KCanamar"
date: "2023-03-08"
title: "Golang, Gin, Gorm, Bit.io"
lang: "Go"
---

So you want to build an api with GO? Let's do it. The following is a step by step guide on how to build a full CRUD api using Gin. Then we will go one step futher and deploy our api to Render.com.
<!-- more -->
### Prerequisites
There are not that many just the normal housekeeping so we can code together.
- Have VsCode or a text editor - [Download VsCode](https://code.visualstudio.com/)
- You are going to need Golang installed on your machine - [Download Golang](https://go.dev/dl/)

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

## Start here if you are familiar with setting up your go projects, and having it open in VsCode

Now since we are going to be deploying this on render.com we are going to need to make sure that we initialize a git repo in our project directory. We do this by running the folowing command.

```bash
git init
```

Now we need to create the go equivalent of a `package.json` , might seem similar if you are coming from `node.js` . The `go.mod` acts as our package manager, this is where we will be installing all of our dependencies for this project. So let us opne up a terminal in vscode in our project directory and run the following command.
```bash
go mod init
```

## Here come the Packages

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

## Finally, we can start writing some code

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

Now to keep our code looking polished lets move this env section in our `init()` to its own file, we start this by adding a new directory into root directory of the project
```bash 
mkdir initializers
```

Then lets add a file to hold our code.

```bash
touch initializers/loadEnv.go
```

Now we can relocate the init functions contents to `initializers/loadEnv.go` and beef it up a little more, we are going to check our file path for our environment variables since deploying on render.com has its own challenges, and this will. make sure that our deployment goes smooth. _[Check the Docs](https://stackoverflow.com/questions/54456186/how-to-fix-environment-variables-not-working-while-running-from-system-d-service)_

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

We should reconnect our environment variables, for this we will need to import this function into main.go

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

Now lets connect a database, touch another file into the initializers directory `touch initializers/database.go

For this connection we will be using `bit.io` as our database, bit.io software is _a platform used to secure Postgres database_. 
If you do not have a bit.io account head over to 
Once the database is setup we are going to select the connect opiton to display our connection information.

Now that we have our database setup and on the connection screen lets setup `database.go`

```go
//database.go

package initializers

import (
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

func ConnectDB() {

	dsn := "host={Hostname} user={Username} password={API Key/Password} dbname={Database name} port={PORT} sslmode=allow"

}
```

// * Todo Edit From Here Down * //

37. From the bit.io connection information we can see that variables we need to use for completing the connection string, make sure that the `sslmode` is assigned `require` due to bit.io security settings.
38. Next we will create a global variable to access in any of the files `var DB *gorm.DB` 
39. Then we need to create a error variable local scoped to our `ConnectDB()` function `var err error` 
40. Next we need to asign the variables we have created to the connection
41. Now we need to handle our errors if any responding with a `log.Fatal("Failed to connect to the database")`

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
	
	dsn := "host={Hostname} user={Username} password={API Key/Password} dbname={Database name} port={PORT} sslmode=allow"
	DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
	
	// error handling
	if err != nil {
		log.Fatal("Failed to connect to database")
	}
}
```

Now that we have all of this running when we check our termial running compiledaemon we should see no errors. However if you do please go back and check for syntaxing errors.
Now we should not have our database connection string out in the open, so lets relocate that string to our `.env` file under the name `DATABASE_URL`
Now we use that variable instead of the connection string with `os.Getenv("Variable_Name")`

```go
//database.go
package initializers

import (
	"log"
	"os"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)
// create a global variable for Database
var DB *gorm.DB

func ConnectDB() {
	// create local scoped error
	var err error
	// .env variable
	dsn := os.Getenv("DATABASE_URL")
	DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
	// error handling
	if err != nil {
		log.Fatal("Failed to connect to database")
	}
}
```
45. With the database connected, lets move on and create some models for our data, first `mkdir models` in the projects root directory
46. `touch postModel.go` in the `models` directory
```go
// postModel.go
// https://gorm.io/docs/models.html
package models

import "gorm.io/gorm"

type Post struct {
	gorm.model
	Title string
	Body string
}
```
47. Now that we have a model, we need to migrate to our database, in the root directory in terminal  `mkdir migrate`, then `touch migrate/migrate.go`
```go
// migrate.go
package main

import (
	"github.com/kcanamar/go-gin-bit-crud/initializers"
	"github.com/kcanamar/go-gin-bit-crud/models"
)

func init() {
	// Import initializers
	initializers.LoadEnv()
	initializers.ConnectDB()
}

func main() {
	// auto migrate - https://gorm.io/docs/index.html
	// AutoMigrate takes a model struct argument
	initializers.DB.AutoMigrate(&models.Post{})
}
```
48. Once the migrte file is all setup lets migrate with command `go run migrate/migrate.go`, so long as there are no errors we will see the table created in the `data` section of bit.io. Take a moment to celebrate, we have just now setup up our API, now off to create some CRUD.
50. Lets navigate in our terminal to the project root directory and `mkdir controllers` and then`touch controllers/postCtrl.go` , Ctrl is my shorthand for Controller.
51. In `postCtrl.go` we will be defining the route controls for GET, POST, PUT, and DELETE requests related to our post model.
```go
// postCtrl.go
package controllers

import "github.com/gin-gonic/gin"

func PostsCreate (c *gin.Context) {
	c.JSON(200, gin.H{
		"message": "Hello from the other side!",
	})
}
```
51. First we need to connect to our database, the mount the controllers with our server, so we relocate the message from our home route to our `PostsCreate` func, *Important* The naming convention to be able to use your functions in other file, much be PascalCase.
```go
// main.go
package main

import (
	"os"

	"github.com/gin-gonic/gin"
	"github.com/kcanamar/go-gin-bit-crud/controllers"
	"github.com/kcanamar/go-gin-bit-crud/initializers"
)

func init() {
	// load the env variables
	// add ENV="development" to your .env
	if os.Getenv("ENV") != "production" {
		initializers.LoadEnv()
	}
	// connect the database
	initializers.ConnectDB()
}

func main() {
	// setup router
	router := gin.Default()
	
	// include the contoller
	router.GET("/", controllers.PostsCreate)
	
	// server listener
	router.Run()
}
```

Lets make sure that we prepare our init func for depolyment so we are going to wrap the `initializers.LoadEnv()` in a coditional checking for the .env variable `ENV` this will make things easier when we depoly to Render.com later
Check to make sure the server is giving use the feedback we want. 
Now lets modify our code to send some data using a POST request

```go
// main.go
...

func main() {
	// router
	router := gin.Default()
	
	// Create Route
	router.POST("/posts", controllers.PostsCreate)
	
	// server listener
	router.Run()
}
```

Now lets refactor out code in `postCtrl.go` so we can send data to our database with a payload
First step we need to test createing a post with some hardcoded values then we will send requests and use the request body data to create new posts

```go
// postCtrl.go
package controllers

import (
	"github.com/gin-gonic/gin"
	"github.com/kcanamar/go-gin-bit-crud/initializers"
	"github.com/kcanamar/go-gin-bit-crud/models"
)

func PostsCreate(c *gin.Context) {

	// Get data off request body

	// Create a post - https://gorm.io/docs/create.html#Create-Record
	post := models.Post{Title: "Yes", Body: "We did it!"}
	result := initializers.DB.Create(&post)
  
	// handle error from result
	if result.Error != nil {
		c.Status(400)
		return
	}
	// return post
	c.JSON(200, gin.H{
		// send back a confirmation of the post
		"post": post,
	})
}
```

Confirm that we see the data in our database! okay hardocded example out of the way, on to refactoring to use the request body.
First let's go ahead and create a new struct based on our model to hold the request information.

```go
// postCtrl.go
...
func PostCreate(c *gin.Context){

	// Get Data off request body
	// https://gin-gonic.com/docs/examples/binding-and-validation/
	// create variable
	var reqBody struct {
		Title string
		Body string
	}
	// parse the request bodyas JSON then binds to variable reqBody
	c.Bind(&reqBody)

...
}
```

Test, it out by opening up postman, thunderclient or even a curl request to send a JSON body to our local host server. We see that it is created! On to the Index controller!
Let's make a new function in our `postCtrl.go` file called `PostsIndex`  and here we will be getting all of the posts from our database and sending them back in the response as JSON.

```go
// postCtrl.go

...

func PostsIndex(c *gin.Context){
	// https://gorm.io/docs/query.html#Retrieving-all-objects
	
	// Get all posts
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

Let's now include this in our routes

```go
// main.go

func main() {
	// router
	router := gin.Default()

	// Index Route
	router.GET("/posts", controllers.PostsIndex)

	// Create Route
	router.POST("/posts", controllers.PostsCreate)
	
	// server listener
	router.Run()
}
```

Check to make sure this works by sending a GET request to localhost:3000/posts, we should see a JSON object with a key of "posts" with a value of an array of posts. Most excellent!
Now let us go and get a sinlge post for our Show route, head back to `postCtrl.go` and lets create another func `PostShow` 

```go
// postCtrl.go

...

func PostShow(c *gin.Context){
	// https://gorm.io/docs/query.html#Retrieving-objects-with-primary-key
	// Get single post
	
	// variable to hold post
	var post models.Post

	// Query the database
	initializers.DB.First(&post, c.Param("id"))

	// Response
	c.JSON(200, gin.H{
		"post": post
	})
}
```

Add a show route into `main.go`

```go
// main.go

... 

func main() {

	// setup a gin router
	r := gin.Default()

	// Home Route
	r.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "Hello from the other side!",
		})
	})

	// Index Route
	r.GET("/posts", controllers.PostsIndex)

	// Create Route
	r.POST("/posts", controllers.PostCreate)

	// Show Route
	r.GET("/posts/:id", controllers.PostShow)

	// setup our server listener
	r.Run()

}
```

Test that out by sending a GET request to `localhost:3000/posts/1` and we should expect to see the post with `id=1` 
Almost there lets update a post now, back to `postCtrl.go`

```go
// postsCtrl.go

...

func PostUpdate(c *gin.Context){
	// Get the id from params
	id := c.Param("id")
	
	// Get the request body
	var body struct {
		Title string
		Body string
	}
	
	// Find post to be updated
	var post models.Post
	initializers.DB.Find(&post, id)
	
	// https://gorm.io/docs/update.html#Updates-multiple-columns
	// Update Post
	initializers.DB.Model(&post).Updates(models.Post{
		Title: body.Title,
		Body: body.Body,
	})
	
	// Response with updated post
	c.JSON(200, gin.H{
		"updatedPost": post
	})
}
```

Now to update `main.go`

```go
// main.go

...
	// Create Route
	r.POST("/posts", controllers.PostCreate)

	// Update Route
	r.PUT("/posts/:id", controllers.PostUpdate)

	// Show Route
	r.GET("/posts/:id", controllers.PostShow)
...
```

Test, and confirm by sending a JSON body as a PUT request to `localhost:3000/posts/1` 

```js
// Request body
{
	"title": "updated"
}
```

_Note_ you can update any number of properties defined in the request body, or just one.
Now lets DELETE a post to round out our C.R.U.D. Operations, back to `postCtrl.go`

```go
// postCtrl.go

...

func PostDestroy(c *gin.Context) {
	// Get id from params
	id := c.Param("id")

	// https://gorm.io/docs/delete.html#Delete-with-primary-key
	// Delete the post
	initializers.DB.Delete(&models.Post{}, id)

	// Response
	c.JSON(200, gin.H{
		"message": "Successful Destory",
	})

}
```

Add in our Route to `main.go`

```go
// main.go
...
	// Create Route
	r.POST("/posts", controllers.PostCreate)

	// Update Route
	r.PUT("/posts/:id", controllers.PostUpdate)
	
	// Delete Route
	r.DELETE("/posts/:id", controllers.PostDestroy)

	// Show Route
	r.GET("/posts/:id", controllers.PostShow)
...
```



# Deployment
1. Now that the app is built lets deploy it to render.com
2. Create a render.com account and choose the free teir
3. We are going to create a new web service
4.  If you are hosting your site on github.com we can import our repo from there, scroll through the options until you find your project name and click connect.
5.  Render will ask you to give your porject a name
6. Now let's chose the advanced setting to set up our environment variables
7. . https://render.com/docs/environment-variables
8. We need to define our enviornment variables, so in YOUR local `.env` we have these variables defined, however when we depoly using render, render need to also know what these variables values are. 
9. Select the `Add Environment Variable` for each of the following
| Key          | Value                                    |
| ------------ | ---------------------------------------- |
| DATABASE_URL | {use your connection string from bit.io} |
| ENV          | "production"                             |
| GO111MODULE  | on                                       |
| GOPATH       | /opt/render/project/go                   |
10. Finally we can click create Web Service and Render will build our project, provided we did everything correct we should have a successful deployment.
