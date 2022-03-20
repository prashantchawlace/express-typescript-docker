Today we will Dockerize our NodeJS(Express) Application.

We will assume that we already have a NodeJS application up and running. If you are interested to see how we built the express application go to the following article.

https://www.mohammadfaisal.dev/blog/create-express-typescript-boilerplate

### Get the NodeJS boilerplate

First, clone the [express boilerplate repository](https://github.com/Mohammad-Faisal/express-typescript-skeleton) where we have a working Express application with Typescript, EsLint and Prettier already set up.

We will integrate Docker on top of this.

```sh
git clone https://github.com/Mohammad-Faisal/express-typescript-skeleton.git
```

### Why Docker?

I am glad you asked. Docker can abstract away most of the pain points of environment-related errors because Docker ensures that your application will be the same on your machine and any other machine in the world.

Let's say your machine has NodeJS 14 installed. But you got an application that used NodeJS 12. So what do you do? Install both versions of Node? Yes that's possible to do using some tools like NVM. Then

But if you dockerize your application, you don't need any extra burden on your machine. You can run any version of NodeJS inside the docker container.

I hope you get an idea of the benefits. Let's jump into the code!

### Install Docker

If you don't have Docker already installed, just go to the [the following link](https://docs.docker.com/engine/install/) and follow the steps to install it.

Then create our **Dockerfile**

```sh
touch Dockerfile
```

### Define environment

Let's open the **Dockerfile** and start building the configuration for Docker.

The first thing we will need is to define the environment. For our application, we will use NodeJS 16. You can see the available versions on [Docker Hub](https://hub.docker.com/_/node)

```
FROM node:16
```

But most of the time, we don't need the entire NodeJS environment. We can just use a lightweight alternative.

```YAML
FROM node:lts-alpine
```

Usually, alpine versions are lightweight, but it provides enough functionality for your application.

### Create a work directory

The docker image will have a separate directory structure. So we need someplace inside the image to hold our application code.

Let's create a directory for our application.

```YAML
WORKDIR /usr/src/app
```

### Install dependencies

The image that we defined at the top already comes with NodeJS and NPM installed, so we don't have to worry about that.
The next thing we will need is to copy our **package.json** file into our working directory that we just defined above.

```YAML
# we are using a wildcard to ensure that both package.json and package-lock.json file into our work directory
COPY package*.json ./
```

Then install the dependencies. We can take advantage of Docker's **RUN** command.

```YAML
RUN npm install
```

If you want to build the code for production, we can use the following command. [The explanation about the command npm ci ](https://blog.npmjs.org/post/171556855892/introducing-npm-ci-for-faster-more-reliable)can be found here.

```YAML
RUN npm ci --only=production
```

### Copy the application code

You might be wondering why we didn't copy the whole application code into the working directory in the previous step.
That's because Docker creates an intermediate image for each layer. And a layer is created for each command.

As we didn't need the application codes in the previous steps, it would be a waste of space, and the process would be less efficient if we copied the whole code in the previous steps.

If you are interested to learn more. [Go here](https://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/)

Let's get the code now.

```YAML
COPY . .
```

Now let's expose the internal application port. Because by default, Docker images don't expose any port for security reasons.

```YAML
EXPOSE 3000
```

### Running the application

Now we have all the codes inside the working directory. Let's build our application now.

```YAML
RUN npm build
```

Now we will define the command to run our server.

As We will build our bundle inside the **dist** folder (Which we defined inside the **tsconfig.json** file), we have to run the built image.

```
CMD [ "node", "dist/index.js" ]
```

Notice we didn't use **npm start** to run the application because running with Node makes it efficient. Secondly, it causes exit signals such as SIGTERM and SIGINT to be received by the Node.js process instead of npm swallowing them.

### Create a dockerignore file

We can further optimize our Docker image by avoiding copying some resources into the application. Create a **.dockerignore** file and paste the following code there

```yaml
Dockerfile
docker*.yml

.git
.gitignore
.config

.npm
.vscode
node_modules

README.md
```

### Build our image

Let's build our image by running the following command.

```sh
docker build . -t express-typescript-docker
```

Notice the **.** in the command. It is not accidental. and also we are tagging our application by passing the **-t** option

You can see the built image by running the following command.

```sh
docker images

REPOSITORY                                TAG                   IMAGE ID       CREATED          SIZE
express-typescript-docker                 latest                11b3fd2ab6d4   29 seconds ago   262MB
```

### Run our image

Let's run our image inside a container by running the following command.

```sh
docker run -p 3000:3000 -d express-typescript-docker:latest
```

Notice the **-p** option that maps the exposed docker port with our local machine port. In our **Dockerfile**, we exposed port 3000. And we mapped that same port to our machine.

If we want to run our application on port 4000 on our local machine, we can do this.

````sh
-p 4000:3000
```

You can also pass environment variables into the container by passing **-e "NODE_ENV=production"** into the command.

### Inspect the Docker Containers

You can see the container state by running the following command.

```sh
docker ps


CONTAINER ID   IMAGE                              COMMAND                  CREATED         STATUS         PORTS                              NAMES
9be24fc778bb   express-typescript-docker:latest   "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:3000->3000/tcp, 8080/tcp   eager_bardeen
````

### Test the endpoint

Now go to your browser or some HTTP client like Postman and hit the following URL.

```
http://localhost:3000
```

and see the result

```json
{ "message": "Hello World!" }
```

Also, you can see the logs inside the container by running the following command. Get the container_id from the **docker ps** command we ran earlier.

```
docker logs container_id
Connected successfully on port 3000
{ message: 'Hello World' }
```

### Resources

- [Docker Nodejs Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
- [Official NodeJS documentation to setup Docker](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)

## Github repo
