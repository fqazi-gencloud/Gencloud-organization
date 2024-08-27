
# DOCKERFILE FOR NODE.JS APPLICATION

 creating a lightweight container for a Node.js application

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --production

COPY . .

EXPOSE 3000

CMD ["npm", "start"]



````
# 1.Specify the base image
FROM node:18-alpine

# 2. Set the working directory inside the container
WORKDIR /app

# 3. Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# 4. Install dependencies
RUN npm install --production

# 5. Copy the rest of the application code to the working directory
COPY . .

# 6. Expose the port the app runs on (e.g., 3000)
EXPOSE 3000

# 7. Define the command to run the app
CMD ["npm", "start"]
