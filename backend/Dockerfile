# Use the official Node.js image
FROM node:18

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm init -y && npm install express mongoose cors body-parser

# Copy application code
COPY . .

# Create index.js with Express API and MongoDB connection
RUN echo "const express = require('express'); \
const mongoose = require('mongoose'); \
const app = express(); \
const PORT = 5000:80; \
mongoose.connect('mongodb://db:27017/mydb', { useNewUrlParser: true, useUnifiedTopology: true }) \
.then(() => console.log('Connected to MongoDB')) \
.catch(err => console.log('MongoDB connection error:', err)); \
app.get('/', (req, res) => res.send('Hello from Backend')); \
app.listen(PORT, () => console.log('Backend running on port ' + PORT));" > index.js

# Expose port 5000
EXPOSE 5000:80

# Run the application
CMD ["node", "index.js"]