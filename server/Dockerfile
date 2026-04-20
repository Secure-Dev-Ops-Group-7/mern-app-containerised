#Start with base node image version 24, (latest LTS @April 2026)
FROM node:24

#Create a working directory in the container if it doesn't exist for app files
WORKDIR /app

#Copy package.json file (dependencies) from project source dir to container app dir
COPY package.json /app

#Install the the dependencies to the container
RUN npm install

#Copy server source code to container dir
COPY . /app

#Expose network port number
EXPOSE 5000

#Start node.js when container starts
CMD ["node", "server.js"]


