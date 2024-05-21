https://rvohealth.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/29411942#overview

Trying out docker with a react app.

created a new dir called try-docker
create a new app called frontend using
npx create-react-app frontend

created a new Dockerfile.dev

delete the node_modules folder completely to avoid duplicate dependencies.

from terminal :
docker build -f Dockerfile.dev .

the above gives us an image id. Grab it, and :
docker run <imageid>

The above does not expose localhost:3000 , so :

docker run -p 3000:3000 <imageid>

Every time we make change to the source code, we want it to automatically be propogated to our container. (like hot reloading)
For that, we can use Docker Volume so it can act as a reference to the code folder outside the container :

docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <imageid>

In the above, the -v /app/node_modules (with the :), we are just saying this is a placeholder for the node_modules inside app in the container.
Don't try to map it to any folder.
Whereas, when we say -v ${pwd}:/app, it maps /app to the current present working directory (pwd) folder pointing to frontend folder.

---

Using Docker Compose:
Instead of using the long docker run command of docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <imageid>
we can use docker compose by creating docker-compose.yml, and then just running this from terminal :
docker-compose up

To run the tests, we overide our default npm start command from the terminal :
docker build -f Dockerfile.dev .
docker run -it <imageid> npm run test

Now same way, for hot reloading tests , we can setup a second service (container) in docker-compose for test files
Added the second service in docker-compose.yml :
tests:
build:
context: .
dockerfile: Dockerfile.dev
volumes: - /app/node_modules - .:/app
command: ['npm', 'run', 'test']

Then from terminal :
docker-compose up --build

Now this run the web app on port 3000, as well as runs the tests.

---

Production environment :
Now for production environment, we need something that will take incoming request and responds to it.
For that, we use Nginx, which will take incoming traffic and respond to it with routing.

We do a multi step build process (build phase, and run phase )

Added the Dockerfile with multistep which finally copies the build folder to nginx

To build :
docker build -t ajaythackerrvo/dockerreactapp .

To run, need to open port 8080 for nginx :
docker run -p 8080:80 ajaythackerrvo/dockerreactapp

Then we can go to http://localhost:8080

To push the container to docker hub :
docker login
Once logged in to ajaythackerrvo
docker push ajaythackerrvo/dockerreactapp

Once the container is pushed to public repository, from any other system, i can :
docker run -p 8080:80 ajaythackerrvo/dockerreactapp
