# ICT-06_LB01_Name1_Name2

## Get started with the Full Stack Web App Using Vue.Js, Express Js and MySQL

### Introduction
This full stack web application has been developed with the following technology-stack:
#### Backend
* Express.js (middleware, model-view-controller)
* Node.js (webbased backend infrastructure)
* MySQL (database)

#### Frontend
* Vue.js (component-based user-interfaces)
* babel (transpiler, compatibility of new ECMAScript 6 and higher to ECMAScript 5)

>This step-by-step installation handles the backend!  

#### Prerequisite
You need to install [nodejs](https://nodejs.org/en/) in order to build the required infrastructure. With the installation of nodejs the tool `npm` (Node package manager) is included.
Checkpoint: Type in the following command to ensure you have correctly installed Node and npm.
```
node -v && npm -v
```

### Part 1: Create database (if not already exists)
#### Prerequisite
1. MySQL is installed and running
2. DataGrip (by Jetbrains) is installed

If one of these prerequisite are not met, then go to the missing lessons on 
http://ict.bzzlab.ch

#### Steps
Create a database with the name “pos_db”.
If “pos_db” does not already exists then ...

Step 1: Open Datagrip and connect to MySQL

Step 2: Execute the following SQL-Scripts

To create a database with MySQL, it can be done by executing the following query:

```
CREATE DATABASE pos_db;
use pos_db;
``` 
Next, create a table in the “pos_db” database.
To create a "product" table, it can be done by executing the following SQL command:

```
CREATE TABLE product(
product_id INT(11) PRIMARY KEY AUTO_INCREMENT,
product_name VARCHAR(200),
product_price DOUBLE
);
``` 

### Part 2: Cloning server and installing backend dependencies - Express, MySQL2, and Cors 
Step 1: Create a folder on your computer, here I give the name "fullstack-app".

Step 2: Then open the folder using a code editor, here I am using Visual Studio Code.
Next, open a terminal in Visual Studio Code on the terminal menu bar.

Then, create a "backend" folder in the "fullstack-app" folder by typing the following command in the terminal:
```
mkdir backend
```
Then go to the "backend" folder by typing the following command:
```
cd backend
```
After that, type the following command to create a "package.json" file in the "backend" folder:
```
npm init -y
```

##### Remarks for MacOS-users
When using  `npm` on the command line the usage differs on Windows and MacOS based platforms.

##### Windows 
On Windows you can use the `npm ...` straight away. Example: 
```
npm install
```
##### MacOS
On MacOS - if you open the terminal as non-administrator - then use it with the `sudo`-command 
Example: 
```
sudo npm install
``` 
and then type  in the administrator password. 
In the following installation guide the `sudo`-command is omitted. But be aware of the if you're an Apple user without default administrator privileges.

Next, install app dependencies express, mysql2, and cors by typing the following commands in the terminal:
```
npm install express mysql2 cors
```
Next, add the following code to the "package.json" file:
```
"type": "module",
```
So that the "package.json" file looks like this:

```
{
  "name": "backend",
  "version": "1.0.0",
  "description": "",
  "type": "module",
  "main": "index.js",
  "scripts": {
    "test": "echo "Error: no test specified" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.17.1",
    "mysql2": "^2.2.5"
  }
}
```
In this way we can use the ES6 Module Syntax to export and import modules.


### Part 3: Application Structure - MVC (Model-View-Controller)
#### Steps 
To make the application more neatly structured, we will apply the [MVC (Model-View-Controller)](https://de.wikipedia.org/wiki/Model_View_Controller) pattern.

Create folders "config", "models", "controllers" and "routes" inside the "backend" folder.

Then create a file "database.js" in the "config" folder, create a file "productModel.js" in the "models" folder, create a file "product.js" in the "controllers" folder, create a file "routes.js" inside the “routes” folder, and create a “index.js” file inside the “backend” folder.

Look at the following picture for more details:

![App Structure](https://mfikri.com/assets/images/files/en/fullstack/app-strucktur.png)

### Part 4: Server - Add program code for database connection
#### Steps

Step 1: Connect to Database - Open the "database.js" file in the "config" folder, then add the code below and type in your root password.
* It imports mysql as module
* It defines a connection and connects to the database
* It exports the db-object for further use in other modules 
```
import mysql from "mysql2";
  
// create the connection to database
const db = mysql.createConnection({
  host: 'localhost',
  user: 'root',
  password: 'TypeYourPasswordHere',
  database: 'pos_db'
});

db.connect(function(err) {
    if (err) throw err;
    console.log("Database connected!");
});
 
export default db;
```

Step 2: Models

Open the "productModel.js" file in the "models" folder, then type the following code:
* It imports the defined database connection from step 1

```        
// import connection
import db from "../config/database.js";
 
// Get All Products
export const getProducts = (result) => {
    db.query("SELECT * FROM product", (err, results) => {             
        if(err) {
            console.log(err);
            result(err, null);
        } else {
            result(null, results);
        }
    });   
}
 
// Get Single Product
export const getProductById = (id, result) => {
    db.query("SELECT * FROM product WHERE product_id = ?", [id], (err, results) => {             
        if(err) {
            console.log(err);
            result(err, null);
        } else {
            result(null, results[0]);
        }
    });   
}
 
// Insert Product to Database
export const insertProduct = (data, result) => {
    db.query("INSERT INTO product SET ?", [data], (err, results) => {             
        if(err) {
            console.log(err);
            result(err, null);
        } else {
            result(null, results);
        }
    });   
}
 
// Update Product to Database
export const updateProductById = (data, id, result) => {
    db.query("UPDATE product SET product_name = ?, product_price = ? WHERE product_id = ?", [data.product_name, data.product_price, id], (err, results) => {             
        if(err) {
            console.log(err);
            result(err, null);
        } else {
            result(null, results);
        }
    });   
}
 
// Delete Product to Database
export const deleteProductById = (id, result) => {
    db.query("DELETE FROM product WHERE product_id = ?", [id], (err, results) => {             
        if(err) {
            console.log(err);
            result(err, null);
        } else {
            result(null, results);
        }
    });   
}
```

Step 3: Controllers

Open the "product.js" file in the "controllers" folder, then type the code below.

```
// Import function from Product Model
import { getProducts, getProductById, insertProduct, updateProductById, deleteProductById } from "../models/productModel.js";
 
// Get All Products
export const showProducts = (req, res) => {
    getProducts((err, results) => {
        if (err){
            res.send(err);
        }else{
            res.json(results);
        }
    });
}
 
// Get Single Product 
export const showProductById = (req, res) => {
    getProductById(req.params.id, (err, results) => {
        if (err){
            res.send(err);
        }else{
            res.json(results);
        }
    });
}
 
// Create New Product
export const createProduct = (req, res) => {
    const data = req.body;
    insertProduct(data, (err, results) => {
        if (err){
            res.send(err);
        }else{
            res.json(results);
        }
    });
}
 
// Update Product
export const updateProduct = (req, res) => {
    const data  = req.body;
    const id    = req.params.id;
    updateProductById(data, id, (err, results) => {
        if (err){
            res.send(err);
        }else{
            res.json(results);
        }
    });
}
 
// Delete Product
export const deleteProduct = (req, res) => {
    const id = req.params.id;
    deleteProductById(id, (err, results) => {
        if (err){
            res.send(err);
        }else{
            res.json(results);
        }
    });
}
```

Step 4: Routes

Open the "routes.js" file in the "routes" folder, then type the following code:
```
// import express
import express from "express";

// import function from controller
import { showProducts, showProductById, createProduct, updateProduct, deleteProduct } from "../controllers/product.js";

// init express router
const router = express.Router();

// Get All Product
router.get('/products', showProducts);

// Get Single Product
router.get('/products/:id', showProductById);

// Create New Product
router.post('/products', createProduct);

// Update Product
router.put('/products/:id', updateProduct);

// Delete Product
router.delete('/products/:id', deleteProduct);

// export default router
export default router;
```

Step 5: Index.js

Open the "index.js" file in the "backend" folder, then type the following code:
```        
// import express
import express from "express";
// import cors
import cors from "cors";
// import routes
import Router from "./routes/routes.js";

// init express
const app = express();

// use express json
app.use(express.json());

// use cors
app.use(cors());

// use router
app.use(Router);

app.listen(5000, () => console.log('Server running at http://localhost:5000'));
```

Step 5: Next, type the following command in the terminal:
```        
node index
```

At this point you have successfully created the backend.

To make sure the backend is running well, you can use POSTMAN to do some testing.


### Part 5: Frontend

#### Prerequisite
You need to install Vue CLI in order to build the frontend. 

To make sure Vue CLI is installed on your computer, type the following command in the terminal:
```
vue --version
```
To install Vue CLI, open a new terminal in Visual Studio Code and enter the command below:
```
npm install –g @vue/cli
```

#### Steps
Step 1: If Vue CLI has been installed on your computer, please create a Vue project by typing the following command in the terminal:
```
vue create frontend
```
If the installation is successful, there will be a new "frontend" folder inside the "fullstack-app" folder. 

In the "fullstack-app" folder there are now two folders, namely: "backend" and "frontend". The "backend" folder is the application folder that was built before using node.js express, while the "frontend" is the application folder created using vue.js.


Step 2: Next, go to the "frontend" folder and install the following dependencies.
* Vue-router to create routes in the vue js application
* Axios is a promise based http client for browsers and node.js
* Bulma is a CSS framework to beautify the user interface (UI)

First open the folder by typing the following code:
```
cd frontend
```
After that, install vue-router, axios, and bulma by typing the following commands in the terminal:
```
npm install vue-router axios bulma
```

After that, to make sure everything is going well, type the following command to run the vue js application
```
npm run serve
```

Open your browser, then visit the following URL:
```
http://localhost:8080/
```

If it goes well, it will look like this:
![Vue.js Start Template](https://mfikri.com/assets/images/files/en/fullstack/welcome-vue.png)


Step 3: Components

Create the components files "ProductList.vue", "AddProduct.vue", and "EditProduct.vue" in the "frontend/src/components" folder.

Open the file "ProductList.vue", then type the following code:
```
<template>
  <div>
    <router-link :to="{ name: 'Create' }" class="button is-success mt-5"
      >Add New</router-link
    >
    <table class="table is-striped is-bordered mt-2 is-fullwidth">
      <thead>
        <tr>
          <th>Product Name</th>
          <th>Price</th>
          <th class="has-text-centered">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in items" :key="item.product_id">
          <td>{{ item.product_name }}</td>
          <td>{{ item.product_price }}</td>
          <td class="has-text-centered">
            <router-link
              :to="{ name: 'Edit', params: { id: item.product_id } }"
              class="button is-info is-small"
              >Edit</router-link
            >
            <a
              class="button is-danger is-small"
              @click="deleteProduct(item.product_id)"
              >Delete</a
            >
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
 
<script>
// import axios
import axios from "axios";
 
export default {
  name: "ProductList",
  data() {
    return {
      items: [],
    };
  },
 
  created() {
    this.getProducts();
  },
 
  methods: {
    // Get All Products
    async getProducts() {
      try {
        const response = await axios.get("http://localhost:5000/products");
        this.items = response.data;
      } catch (err) {
        console.log(err);
      }
    },
 
    // Delete Product
    async deleteProduct(id) {
      try {
        await axios.delete(`http://localhost:5000/products/${id}`);
        this.getProducts();
      } catch (err) {
        console.log(err);
      }
    },
  },
};
</script>
 
<style>
</style>
```
Next, open the file "AddProduct.vue", then type the following code:
```
<template>
  <div>
    <div class="field">
      <label class="label">Product Name</label>
      <div class="control">
        <input
          class="input"
          type="text"
          placeholder="Product Name"
          v-model="productName"
        />
      </div>
    </div>
 
    <div class="field">
      <label class="label">Price</label>
      <div class="control">
        <input
          class="input"
          type="text"
          placeholder="Price"
          v-model="productPrice"
        />
      </div>
    </div>
 
    <div class="control">
      <button class="button is-success" @click="saveProduct">SAVE</button>
    </div>
  </div>
</template>
 
<script>
// import axios
import axios from "axios";
 
export default {
  name: "AddProduct",
  data() {
    return {
      productName: "",
      productPrice: "",
    };
  },
  methods: {
    // Create New product
    async saveProduct() {
      try {
        await axios.post("http://localhost:5000/products", {
          product_name: this.productName,
          product_price: this.productPrice,
        });
        this.productName = "";
        this.productPrice = "";
        this.$router.push("/");
      } catch (err) {
        console.log(err);
      }
    },
  },
};
</script>
 
<style>
</style>
```

After that, open the file "EditProduct.vue", then type the following code:
```
<template>
  <div>
    <div class="field">
      <label class="label">Product Name</label>
      <div class="control">
        <input
          class="input"
          type="text"
          placeholder="Product Name"
          v-model="productName"
        />
      </div>
    </div>
 
    <div class="field">
      <label class="label">Price</label>
      <div class="control">
        <input
          class="input"
          type="text"
          placeholder="Price"
          v-model="productPrice"
        />
      </div>
    </div>
    <div class="control">
      <button class="button is-success" @click="updateProduct">UPDATE</button>
    </div>
  </div>
</template>
 
<script>
// import axios
import axios from "axios";
 
export default {
  name: "EditProduct",
  data() {
    return {
      productName: "",
      productPrice: "",
    };
  },
  created: function () {
    this.getProductById();
  },
  methods: {
    // Get Product By Id
    async getProductById() {
      try {
        const response = await axios.get(
          `http://localhost:5000/products/${this.$route.params.id}`
        );
        this.productName = response.data.product_name;
        this.productPrice = response.data.product_price;
      } catch (err) {
        console.log(err);
      }
    },
 
    // Update product
    async updateProduct() {
      try {
        await axios.put(
          `http://localhost:5000/products/${this.$route.params.id}`,
          {
            product_name: this.productName,
            product_price: this.productPrice,
          }
        );
        this.productName = "";
        this.productPrice = "";
        this.$router.push("/");
      } catch (err) {
        console.log(err);
      }
    },
  },
};
</script>
 
<style>
</style>
```

Step 3: Main.js

Open the file "main.js" in the "frontend/src" folder, then change it to be like this:
```
import Vue from 'vue'
import VueRouter from 'vue-router'
 
import App from './App.vue'
import Create from './components/AddProduct.vue'
import Edit from './components/EditProduct.vue'
import Index from './components/ProductList.vue'
 
Vue.use(VueRouter)
 
Vue.config.productionTip = false
 
const routes = [
  {
    name: 'Create',
    path: '/create',
    component: Create
  },
  {
    name: 'Edit',
    path: '/edit/:id',
    component: Edit
  },
  {
    name: 'Index',
    path: '/',
    component: Index
  },
];
 
const router = new VueRouter({ mode: 'history', routes: routes })
 
new Vue({
  // init router
  router,
  render: h => h(App),
}).$mount('#app')
```

Step 4: App.vue

Open the file “App.vue" in the "frontend/src" folder, then change it to be like this:
```
<template>
  <div id="app" class="container is-max-desktop">
    <router-view />
  </div>
</template>
 
<script>
export default {
  name: "App",
};
</script>
 
<style>
/* import style bulma */
@import "~bulma/css/bulma.css";
</style>
```


## Part 6: Testing

#### READ, CREATE, UPDATE, DELETE
Back to browser and visit the following URL:
```
http://localhost:8080/
```
Try it out: Click the "Add New" button, then enter the Product Name and Price and click the "SAVE" button.

If it goes well, after adding a new product it will look like the following image:
![Vue.js Example App](https://mfikri.com/assets/images/files/en/fullstack/product-list-added.png)


Based on: [mfikri.com](https://mfikri.com/en/blog/nodejs-express-mysql-vue#)
