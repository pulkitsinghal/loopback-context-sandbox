### Project Layout
1. `0.10.44` - if node_modules is installed when the npm version is 2.x and node version is 0.10.x ... userContext can grab a valid value for loopback.getCurrentContext() to operate on when there is no accessToken and when there is an accessToken present
1. if node_modules is ALREADY installed when the npm version was 2.x and node version was 0.10.x ... then node version is changed to 5.x and npm version is changed to 3.x ... again it works
1. `npm2-node5` if npm-shrinkwrap.json from an npm-v2.x and node-v0.10.x based install is available ... and then versions are changed to npm-v3.x and node-v5.x ... and then the install is run but it is based off the pre-existing npm-shrinkwrap.json file ... then it also works. This is a better workaround than 2a in my opinion.
1. `5.12.0` - if node_modules is installed when the npm version is 3.x and node version is 5.x ... userContext can grab a valid value for loopback.getCurrentContext() to operate on when there is no accessToken ... BUT not when an accessToken is present!!! ... this to me seems like a bug and exactly what kamal is experiencing a. is this a bug because of 5.x ... doesn't seem like it if you look at step 2 b. is this a bug because of how npm3 traverses and installs dependencies ... maybe! c. quick workaround? install dependencies with npm version 2.x and node version 0.10.x ... then switch over to using node5.x and npm3.x d. painful workaround? figure out why the dependency tree for step 3 is so screwed up and fix it ... this is tough because the structure of npm2 and npm3 shrinkwrap files is very very different

### Notes
1. npm3 is too smart, it does not rely on the package.json being present in present working directory so its a bit unpredictable to setup with ... Instead its better to move npm-shrinkwrap.json file, run isntall and then move it back:
  1. cd npm2-node5 && \
     mv ./npm-shrinkwrap.json ./.. && \
     cd .. && \
     npm install && \
     mv ./npm-shrinkwrap.json ./npm2-node5 && \
     mv ./node_modules ./npm2-node5/   && \
  2. cd 5.12.0 && npm install

### FAQ
* Why was this project created?
 * To provide a quick way to experiment with loopback and mongo together.
* Why use docker?
 * Its not just `docker` but rather `docker-compose` alongside it that gives `a quick way` to bring servers up and get going.
* Are there other such projects?
 * Sure, searching on GitHub with [loopback mongo filename:docker-compose.yml](https://github.com/search?utf8=%E2%9C%93&q=mongo+filename%3Adocker-compose.yml+loopback&type=Code&ref=searchresults), yields close to 6 results when this line in the README was last edited.
* How can I access loopback once its running?
  * Open your browser to `http://localhost:3000/explorer` and play around.
* How can I access mongo once its running?
  * Use `mongo shell` or `RoboMongo` or `MongoChef` or any client you best see fit .. to connect to `mongodb://localhost:3001/loopback-mongo-sandbox` from your host machine.
* Why is loopback published on its default port `3000` but mongo is published to a non-default port `3001`?
  * Background: Loopback refers to mongo via the url `mongodb://mongo:27017/loopback-mongo-sandbox` which leverages the mapping created by `docker-compose` for the `mongo` service.
  * Answer: If mongo was published on its default port then `mongodb://localhost:27017/loopback-mongo-sandbox` would also start working as a valid URL within `datasources.json` and users wouldn't notice a difference. I wanted to discourage that and bring attention to the benefits of referencing a service by name. You won't live on localhost forever, as your code will reach the world one day!

### Run it

```
cd ~/dev
git clone https://github.com/ShoppinPal/loopback-mongo-sandbox.git
cd ~/dev/loopback-mongo-sandbox
npm install
docker-compose up
```

### Testing for loopbackContext

1. Create a user. Since all loopback servers in thei project  share the same mongo backend, only need to do this once:

  ```
  curl -XPOST http://localhost:3002/api/Users \
  -H "Content-Type: application/json" \
  -d '{
    "email": "one@one.com",
    "password": "111111"
  }'
  ```
2. Login with the new user to get the accessToken. Since all loopback servers in thei project  share the same mongo backend, only need to do this once:
  ```
  curl -XPOST http://localhost:3002/api/Users/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "one@one.com",
    "password": "111111"
  }'
  ```
3. Take the accessToken value and use it with each server's `localhost:[PORT]/explorer` to test if loopbackContext is working

