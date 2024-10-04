# ProjectMean 🚀 

## MEAN Stack Deployment to Ubuntu in AWS.

In this project, We are tasked to implement a web solution based on MEAN stack in AWS Cloud.

MEAN Web stack consists of following components:

Mongodb: A document-based, No-SQL database used to store application data in a form of documents.
Expressjs: A server side Web Application framework for Back-end application.
Angular (Front-end application framework) - Handles Client and Server Requests
Node.js (JavaScript runtime environment) - Accepts requests and displays results to end user.

## In this project, we will implement a simple Book Register web form using the MEAN stack.

## Prerequisites:

1. AWS account and a virtual server with Ubuntu server OS (instance of t2 micro).
2. Install Git Bash.
3. SSH key pair for connecting to ec2 instance vial GitBash terminal.


## Step 1 - Get your instance up and running

1. Launch a new instance.

2. Select a region close to you and launch an instance of t2.micro family with Linux ubuntu server.

3. Create a new .pem priavte key(If you don't have one already) and save it securely.

4. Configure your security groups by ensuring that the following ports are allowed:
  
   SSH (port 22) – for remote access
  
   HTTP (port 80) – for web traffic
  
   HTTPS (port 443) – for secure web traffic (optional)

   ![image 1](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%201.jpg)


## Step 2 - SSH into your instance

1. Open your Git Bash and navigate to your downloads directory

   `cd downloads`

2. Connect to the instance by running

   `ssh -i (your-key.pem) ubuntu@(your-ec2-public-ip)`

## Step 3 - Install NodeJS

Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js is used in this tutorial to set up Express routes and AngularJS controllers.

1. Update Ubuntu:

   `sudo apt update`

2. Upgrade Ubuntu:

   `sudo apt upgrade`

3. Add certificates:

   `sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`

   `curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -`

      ![image 2](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%202.jpg)


5. Install Node.js:

   `sudo apt install -y nodejs`



## Step 4 - Install MongoDB

MongoDB stores data in flexible, JSON-like documents. For this application, we will add book records containing book name, ISBN number, author, and the number of pages.

1. Install MongoDB:

   `sudo apt-get install -y gnupg curl`


   `curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor`


   `echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list`


   `sudo apt-get install -y mongodb-org`

   You should update the package list before installing mongodb by updating the package lists from the newly added MongoDB repository.

   `sudo apt-get update`

2. Start the MongoDB server:

   `sudo service mongod start`

3. Verify that the MongoDB service is up and running:

   `sudo systemctl status mongod`

      ![image 3](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%203.jpg)

4. Install npm (Node Package Manager) using nvm (Node Version Manager):

   `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash`

5. Then, activate nvm:


   `source ~/.bashrc`

   
6. Install Node.js Using nvm. Once nvm is installed, you can install Node.js (which will automatically install npm):
  
   `nvm install --lts`


      ![image 4](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%204.jpg)

8. Check node version

   `npm -v`


   `node -v`

      ![image 5](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%205.jpg)


      
7. Install the body-parser package (We need body parser to help us process JSON files passed in request to the server):

   `sudo npm install body-parser`


8. Create a folder for the project:

   `mkdir Books && cd Books`


9. Initialize the npm project:

   `npm init`

      ![image 6](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%206.jpg)

   
11. Create a file named server.js and add the following code:

   `nano server.js`

   ```
    const express = require('express');
    const bodyParser = require('body-parser');
    const mongoose = require('mongoose');
    const path = require('path');
    const app = express();
    const PORT = process.env.PORT || 3300;
    
    // MongoDB connection
    mongoose.connect('mongodb://localhost:27017/test', {
        useNewUrlParser: true,
        useUnifiedTopology: true,
    })
    .then(() => console.log('MongoDB connected'))
    .catch(err => console.error('MongoDB connection error:', err));
    
    app.use(express.static(path.join(__dirname, 'public')));
    app.use(bodyParser.json());
    
    require('./apps/routes')(app);
    
    app.listen(PORT, () => {
        console.log(`Server up: http://localhost:${PORT}`);
    });
   
   ```


## Step 5 - Install Express and Set Up Routes

1. Install Express and Mongoose:

   `sudo npm install express mongoose`


2. Create a folder named apps:

   `mkdir apps && cd apps`

   
3. Create a file named routes.js and add the following code:

   `nano routes.js`

   ```   
        const Book = require('./models/book');
        const path = require('path');
        
        module.exports = function(app) {
            app.get('/book', async (req, res) => {
                try {
                    const books = await Book.find();
                    res.json(books);
                } catch (err) {
                    res.status(500).json({ message: 'Error fetching books', error: err.message });
                }
            });
        
            app.post('/book', async (req, res) => {
                try {
                    const book = new Book({
                        name: req.body.name,
                        isbn: req.body.isbn,
                        author: req.body.author,
                        pages: req.body.pages
                    });
                    const savedBook = await book.save();
                    res.status(201).json({
                        message: 'Successfully added book',
                        book: savedBook
                    });
                } catch (err) {
                    res.status(400).json({ message: 'Error adding book', error: err.message });
                }
            });
        
            app.delete('/book/:isbn', async (req, res) => {
                try {
                    const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
                    if (!result) {
                        return res.status(404).json({ message: 'Book not found' });
                    }
                    res.json({ message: 'Successfully deleted the book', book: result });
                } catch (err) {
                    res.status(500).json({ message: 'Error deleting book', error: err.message });
                }
            });
        
            app.get('*', (req, res) => {
                res.sendFile(path.join(__dirname, '../public', 'index.html'));
            });
        };

   ```

4. Create a folder named models:

   `mkdir models && cd models`


5. Create a file named book.js and add the following code:

   `nano book.js`

   
    ```
    const mongoose = require('mongoose');
    
    const bookSchema = new mongoose.Schema({
        name: { type: String, required: true },
        isbn: { type: String, required: true, unique: true, index: true },
        author: { type: String, required: true },
        pages: { type: Number, required: true, min: 1 }
    }, {
        timestamps: true
    });
    
    module.exports = mongoose.model('Book', bookSchema);
    ```


## Step 6 - Access the Routes with AngularJS

1. Change the directory back to Books:

   `cd ../..`

   
2. Create a folder named public:

   `mkdir public && cd public`

   
3. Create a file named script.js and add the following code:

   `nano scripts.js`

    ```
    angular.module('myApp', [])
    .controller('myCtrl', function($scope, $http) {
        function fetchBooks() {
            $http.get('/book')
            .then(response => {
                $scope.books = response.data;
            })
            .catch(error => {
                console.error('Error fetching books:', error);
            });
        }
    
        fetchBooks();
    
        $scope.del_book = function(book) {
            $http.delete(`/book/${book.isbn}`)
            .then(() => {
                fetchBooks();
            })
            .catch(error => {
                console.error('Error deleting book:', error);
            });
        };
    
        $scope.add_book = function() {
            const newBook = {
                name: $scope.Name,
                isbn: $scope.Isbn,
                author: $scope.Author,
                pages: $scope.Pages
            };
    
            $http.post('/book', newBook)
            .then(() => {
                fetchBooks();
                // Clear form fields
                $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = '';
            })
            .catch(error => {
                console.error('Error adding book:', error);
            });
        };
    });
    
    ```

4. In the public folder, create a file named index.html and add the following code:

   `nano index.html`

   ```
            <!DOCTYPE html>
      <html ng-app="myApp" ng-controller="myCtrl">
      <head>
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <title>Book Management</title>
          <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
          <script src="script.js"></script>
          <style>
              body { font-family: Arial, sans-serif; margin: 20px; }
              table { border-collapse: collapse; width: 100%; }
              th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
              th { background-color: #f2f2f2; }
              input[type="text"], input[type="number"] { width: 100%; padding: 5px; }
              button { margin-top: 10px; padding: 5px 10px; }
          </style>
      </head>
      <body>
          <h1>Book Management Application</h1>
      
          <form ng-submit="add_book()">
              <label>Name:</label>
              <input type="text" ng-model="Name" required>
      
              <label>ISBN:</label>
              <input type="text" ng-model="Isbn" required>
      
              <label>Author:</label>
              <input type="text" ng-model="Author" required>
      
              <label>Pages:</label>
              <input type="number" ng-model="Pages" required>
      
              <button type="submit">Add Book</button>
          </form>
      
          <h2>Books</h2>
      
          <table>
              <thead>
                  <tr>
                      <th>Name</th>
                      <th>ISBN</th>
                      <th>Author</th>
                      <th>Pages</th>
                      <th>Action</th>
                  </tr>
              </thead>
              <tbody>
                  <tr ng-repeat="book in books">
                      <td>{{ book.name }}</td>
                      <td>{{ book.isbn }}</td>
                      <td>{{ book.author }}</td>
                      <td>{{ book.pages }}</td>
                      <td><button ng-click="del_book(book)">Delete</button></td>
                  </tr>
              </tbody>
          </table>
      </body>
      </html>
   ```


## Step 7 - Run the App

1. Go back to the project root directory and run the following command:

   `cd ..`

   
2. Run the code:

   `node server.js`


      ![image 7](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%207.jpg)
   

   To access it from the Internet, you need to open TCP port 3300 in your AWS Web Console for your EC2 Instance.


      ![image 8](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%208.jpg)



## You should now be able to access your Book Register Application by opening a browser and going to http://(your_server_public_ip):3300

Congratulations!!! You have completed your M.E.A.N web stack.

  ![image 9](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/image%209.jpg)




## Errors to likely encounter and Solutions to implement


1. The code `sudo service mongodb start` gave me an error code while starting mongodb server as seen below:

   
  ![error 1](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/error%201.jpg)

  
The code `sudo systemctl status mongod` worked instead while starting mongodb server

  ![solution 1](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/solution%201.jpg)


2. The code `sudo apt install npm` gave me an error when trying to install node package manager.
  
  ![error 2](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/error%202.jpg)

  
  The following code worked better when trying to install npm(node package manager) with nvm (node version manager):

  `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash`

  
  `source ~/.bashrc`

  
  `nvm install --lts`
  
  ![solution 2](https://github.com/Captnfresh/ProjectMean/blob/main/ProjectMean/solution%202.jpg)






















