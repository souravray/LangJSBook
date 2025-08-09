# LangJSBook - Dockerized Jupyter notebook for langchain.js

A ready-to-use Docker environment for developing with LangChain.js. It runs a JupyterLab instance with a Node.js 20.x (_LTS_) kernel, pre-loaded with Langchain _@latest_ moodules and utilities for rapid experimentation.

## Quick Start

### Prerequisites
* Docker & Docker Compose
*	NPM (optional)


### Setup
* Clone the Repository
```
git clone <your-repository-url>
cd your-project
```

* Create Notebooks Directory. This folder will sync with JupyterLab and store your work.
```
mkdir notebooks
```

* Copy the example environment file and add your secret keys.
```
cp .env.example .env
```
* Build the Docker Image.
```
npm run docker:build
```

* Starts the JupyterLab container in the background.
```
npm run docker:up
```

* Stops and removes the container.
```
npm run docker:down
```
## Useful Tips
### Accessing JupyterLab
To access the running JupyterLab session, you need a URL with a security token.
#### Get the container logs:
```
docker logs langchain_js_jupyter
```
#### Find and copy the URL:
 Look for a line in the logs that starts with ` http://127.0.0.1:8888/lab?token=.. ` and paste it into your browser.


### Using Environment Variables
The `docker-compose.yml` file automatically loads variables from your `.env` file. You do not need to import the `dotenv` package in your notebooks. Access your keys directly:
```js
const apiKey = process.env.OPENAI_API_KEY;
```
## Making _async_ Calls
The `ijavascript` kernel does not support top-level `await`. To run asynchronous code, wrap it in an immediately-invoked async function:
```js
(async () => {
  const response = await model.invoke("Why is the sky blue?");
  console.log(response.content);
})();
```
