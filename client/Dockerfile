#Start with base node image version 24, (latest LTS @April 2026)
FROM node:24

#Create a working directory in the container if it doesn't exist for app files
WORKDIR /app

#Copy package.json file (dependencies) from project source dir to container app dir
COPY package.json /app

#Install the dependencies using node package manager
RUN npm install

#Copy the client source code to the container
COPY . /app

#Expose network port number
EXPOSE 3000

#Start React development server when container starts
CMD ["npm", "start"]

