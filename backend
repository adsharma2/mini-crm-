// backend/models/Customer.js
const mongoose = require('mongoose');

const customerSchema = new mongoose.Schema({
  name: String,
  email: String,
  totalSpends: Number,
  visits: Number,
  lastVisit: Date
});

module.exports = mongoose.model('Customer', customerSchema);

// backend/models/Order.js
const mongoose = require('mongoose');

const orderSchema = new mongoose.Schema({
  customerId: mongoose.Schema.Types.ObjectId,
  amount: Number,
  date: Date
});

module.exports = mongoose.model('Order', orderSchema);

// backend/models/CommunicationLog.js
const mongoose = require('mongoose');

const communicationLogSchema = new mongoose.Schema({
  customerId: mongoose.Schema.Types.ObjectId,
  message: String,
  status: String
});

module.exports = mongoose.model('CommunicationLog', communicationLogSchema);

// backend/controllers/customerController.js
const Customer = require('../models/Customer');

exports.createCustomer = async (req, res) => {
  try {
    const customer = new Customer(req.body);
    await customer.save();
    res.status(201).send(customer);
  } catch (error) {
    res.status(400).send(error);
  }
};

// backend/controllers/orderController.js
const Order = require('../models/Order');

exports.createOrder = async (req, res) => {
  try {
    const order = new Order(req.body);
    await order.save();
    res.status(201).send(order);
  } catch (error) {
    res.status(400).send(error);
  }
};

// backend/controllers/campaignController.js
const Customer = require('../models/Customer');
const CommunicationLog = require('../models/CommunicationLog');
const publisher = require('../pubsub/publisher');

exports.createAudience = async (req, res) => {
  try {
    const rules = req.body.rules;
    const query = buildQuery(rules);
    const audience = await Customer.find(query);
    
    const communications = audience.map(customer => ({
      customerId: customer._id,
      message: `Hi ${customer.name}, here is 10% off on your next order`
    }));

    await CommunicationLog.insertMany(communications);

    communications.forEach(comm => {
      publisher.publish('campaign', JSON.stringify(comm));
    });

    res.status(201).send(audience);
  } catch (error) {
    res.status(400).send(error);
  }
};

function buildQuery(rules) {
  let query = {};
  rules.forEach(rule => {
    if (rule.field === 'totalSpends' && rule.operator === '>') {
      query.totalSpends = { $gt: rule.value };
    }
    // Add more rules here
  });
  return query;
}

// backend/routes/customerRoutes.js
const express = require('express');
const { createCustomer } = require('../controllers/customerController');
const router = express.Router();

router.post('/customers', createCustomer);

module.exports = router;

// backend/routes/orderRoutes.js
const express = require('express');
const { createOrder } = require('../controllers/orderController');
const routerr = express.Router();

router.post('/orders', createOrder);

module.exports = router;

// backend/routes/campaignRoutes.js
const express = require('express');
const { createAudience } = require('../controllers/campaignController');
const routerrr = express.Router();

router.post('/campaigns', createAudience);

module.exports = router;

// backend/pubsub/publisher.js
const redis = require('redis');
const publisher = redis.createClient();

publisher.on('error', (err) => {
  console.error('Redis error:', err);
});

module.exports = publisher;

// backend/pubsub/subscriber.js
const redis = require('redis');
const subscriber = redis.createClient();
const CommunicationLog = require('../models/CommunicationLog');

subscriber.on('message', async (channel, message) => {
  if (channel === 'campaign') {
    const comm = JSON.parse(message);
    // Simulate hitting an external API
    setTimeout(async () => {
      const status = Math.random() > 0.1 ? 'SENT' : 'FAILED';
      await CommunicationLog.updateOne(
        { _id: comm._id },
        { status }
      );
    }, 1000);
  }
});

subscriber.subscribe('campaign');

module.exports = subscriber;

// backend/app.js
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const customerRoutes = require('./routes/customerRoutes');
const orderRoutes = require('./routes/orderRoutes');
const campaignRoutes = require('./routes/campaignRoutes');
require('./pubsub/subscriber');

const app = express();

app.use(bodyParser.json());
app.use('/api', customerRoutes);
app.use('/api', orderRoutes);
app.use('/api', campaignRoutes);

mongoose.connect('mongodb://localhost:27017/mini-crm', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

module.exports = app;

// backend/server.js
const app = require('./app');

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});

