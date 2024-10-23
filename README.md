# Cloud Based CRM(Customer Relationship Management) System

Building a Cloud-Based Customer Relationship Management (CRM) System involves creating a platform where businesses can manage their interactions with customers. In this project, we will use the MERN (MongoDB, Express.js, React, Node.js) stack for development, and we will deploy it to a cloud service like AWS, Heroku, or DigitalOcean.

Here's a step-by-step guide to creating the CRM system, along with code snippets.

**Project Structure**

```go
cloud-crm-system/
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   ├── models/
│   │   └── Customer.js
│   ├── routes/
│   │   └── customers.js
│   └── .env
```
```
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   │   ├── components/
│   │   │   ├── CustomerForm.js
│   │   │   └── CustomerList.js
│   │   ├── App.js
│   │   └── index.js
└── docker-compose.yml
```

**Step 1: Backend Setup**

**A. Initialize Backend Project**

**Create backend directory and navigate to it:**

```bash
mkdir backend
cd backend
```

**Initialize a Node.js project:**

```bash
npm init -y
```

**Install necessary dependencies:**

```bash
npm install express mongoose dotenv cors body-parser
```

**Create server.js:**

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const bodyParser = require('body-parser');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(bodyParser.json());

mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));

const customerRoutes = require('./routes/customers');
app.use('/api/customers', customerRoutes);

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

**Create the MongoDB model for customers:**

```javascript

const mongoose = require('mongoose');

const customerSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    phone: { type: String, required: true },
    company: { type: String },
    status: { type: String, enum: ['Active', 'Inactive'], default: 'Active' },
    createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Customer', customerSchema);
Create customer routes:
javascript
Copy code

const express = require('express');
const router = express.Router();
const Customer = require('../models/Customer');

router.get('/', async (req, res) => {
    try {
        const customers = await Customer.find();
        res.json(customers);
    } catch (error) {
        res.status(500).json({ message: error.message });
    }
});

router.post('/', async (req, res) => {
    const customer = new Customer({
        name: req.body.name,
        email: req.body.email,
        phone: req.body.phone,
        company: req.body.company
    });

    try {
        const newCustomer = await customer.save();
        res.status(201).json(newCustomer);
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
});

router.get('/:id', async (req, res) => {
    try {
        const customer = await Customer.findById(req.params.id);
        res.json(customer);
    } catch (error) {
        res.status(404).json({ message: 'Customer not found' });
    }
});

router.put('/:id', async (req, res) => {
    try {
        const updatedCustomer = await Customer.findByIdAndUpdate(req.params.id, req.body, { new: true });
        res.json(updatedCustomer);
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
});

router.delete('/:id', async (req, res) => {
    try {
        await Customer.findByIdAndDelete(req.params.id);
        res.json({ message: 'Customer deleted' });
    } catch (error) {
        res.status(500).json({ message: error.message });
    }
});
module.exports = router;
```

**Create a .env file to store environment variables:**

```bash
MONGO_URI=mongodb://mongo:27017/crm
```

**Create Dockerfile for Backend:**

```dockerfile
backend/Dockerfile
FROM node:14

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5000

CMD ["node", "server.js"]
```

**Step 2: Frontend Setup**

**A. Initialize Frontend**

**Create frontend directory and navigate to it:**

```bash
mkdir ../frontend
cd ../frontend
```

**Create a React app:**

```bash
npx create-react-app .
```

**Install Axios for API requests:**

```bash
npm install axios
```

**Create CustomerForm and CustomerList components:**

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const CustomerForm = () => {
    const [customer, setCustomer] = useState({
        name: '',
        email: '',
        phone: '',
        company: ''
    });

    const handleChange = (e) => {
        setCustomer({
            ...customer,
            [e.target.name]: e.target.value
        });
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        await axios.post('/api/customers', customer);
        setCustomer({
            name: '',
            email: '',
            phone: '',
            company: ''
        });
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                type="text"
                name="name"
                placeholder="Customer Name"
                value={customer.name}
                onChange={handleChange}
                required
            />
            <input
                type="email"
                name="email"
                placeholder="Customer Email"
                value={customer.email}
                onChange={handleChange}
                required
            />
            <input
                type="text"
                name="phone"
                placeholder="Customer Phone"
                value={customer.phone}
                onChange={handleChange}
                required
            />
            <input
                type="text"
                name="company"
                placeholder="Company Name"
                value={customer.company}
                onChange={handleChange}
            />
            <button type="submit">Add Customer</button>
        </form>
    );
};

export default CustomerForm;
```

```javascript
import React, { useEffect, useState } from 'react';
import axios from 'axios';

const CustomerList = () => {
    const [customers, setCustomers] = useState([]);

    useEffect(() => {
        const fetchCustomers = async () => {
            const response = await axios.get('/api/customers');
            setCustomers(response.data);
        };

        fetchCustomers();
    }, []);

    return (
        <div>
            <h2>Customer List</h2>
            <ul>
                {customers.map((customer) => (
                    <li key={customer._id}>
                        {customer.name} ({customer.email})
                    </li>
                ))}
            </ul>
        </div>
    );
};

export default CustomerList;
```

**Update App.js to include components:**

```javascript
import React from 'react';
import CustomerForm from './components/CustomerForm';
import CustomerList from './components/CustomerList';

function App() {
    return (
        <div>
            <h1>Cloud CRM System</h1>
            <CustomerForm />
            <CustomerList />
        </div>
    );
}

export default App;
```

**Create Dockerfile for Frontend:**

```dockerfile
FROM node:14 as build

WORKDIR /app

COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Step 3: Docker Compose Setup**

**A. Create docker-compose.yml**

```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
    ports:
      - "5000:5000"
    env_file:
      - ./backend/.env
    depends_on:
      - mongo

  frontend:
    build:
      context: ./frontend
    ports:
      - "3000:80"

  mongo:
    image: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

**Step 4: Running the Application**

**Navigate to the root directory of your project:**

```bash
cd cloud-crm-system
```

**Run Docker Compose:**

```bash
docker-compose up --build
```

**Access the application:**

```Frontend: http://localhost:3000
Backend: http://localhost:5000/api/customers
```

**Step 5: Deploying to the Cloud**
You can deploy the application to a cloud platform like Heroku or AWS. Below are simplified deployment steps using Heroku.

**Log in to Heroku:**

```bash
heroku login
```

**Create a Heroku app:**

```bash
heroku create my-crm-system
```

**Push the code to Heroku:**

```bash
heroku container:push web
heroku container:release web**
```

**Set environment variables:**

```bash
heroku config:set MONGO_URI=<your-mongo-uri>
```

**Open your application:**

```bash
heroku open
```

**Conclusion**

This CRM system allows businesses to manage customers by adding, viewing, updating, and deleting customer records. You can extend this project by adding features like customer segmentation, reporting, user authentication, and third-party API integrations (email, SMS).
