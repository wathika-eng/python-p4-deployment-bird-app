```markdown
# Introduction

In the previous lesson, we deployed a small Flask API application to Render to learn about the deployment process and the necessary steps to get code from our machine to a server.

In this lesson, we'll tackle a more complex app with a modern Flask-React stack and explore some of the challenges of getting these two apps to run together on a single server.

# Setup

To follow along, fork this repository from GitHub.

After downloading the code, set up the repository locally:

```sh
$ npm install --prefix client
$ pipenv install && pipenv shell
```

Create a `.env` file at the root and add the following variable:

```
DATABASE_URI=postgresql://{retrieve this from render}
```

We've installed a new package in this repository called `python-dotenv`. It allows us to set environment variables using `.env` files. This is a nice midway point between setting invisible environment variables from the command line and writing hidden values into our code.

To generate these environment variables, add the following to the beginning of the module:

```python
# server/app.py

from dotenv import load_dotenv
load_dotenv()
```

After this, we can import any of our `.env` variables with `os.environ.get()`.

Run the following commands to upgrade and seed our database:

```sh
$ cd server
$ flask db upgrade
$ python seed.py
```

This application has a RESTful Flask API, a React frontend using React Router for client-side routing, and PostgreSQL for the database.

You can now run the app locally with:

```sh
$ honcho start -f Procfile.dev
```

Spend some time familiarizing yourself with the code for the demo app before proceeding. We'll walk through its setup and why certain choices were made during this lesson.

# React Production Build

One of the great features of Create React App is the ability to build different versions of a React application for different environments.

When working in development, a typical workflow involves:

1. Running `npm start` to start a development server.
2. Making changes to the app by editing files.
3. Viewing those changes in the browser.

Create React App uses webpack under the hood to create a development server with hot module reloading. This allows changes to files to be instantly visible in the browser. It also shows good error and warning messages via the console.

Create React App can also build a production version of the application, thanks to webpack. For production, we need to:

1. Build static HTML, JavaScript, and CSS files to run the app in the browser, keeping them as small as possible.
2. Serve these files from a server hosted online.
3. Avoid showing error messages meant for developers.

To build a production version of our React app and serve it from Flask:

1. Build the production version of our React app:

    ```sh
    $ npm run build --prefix client
    ```

    This command generates a bundled and minified version of our React app in the `client/build` folder.

2. Add static routes to Flask:

    In `app.py`, configure Flask to search for static and template files in the `client/build/` directory:

    ```python
    app = Flask(
        __name__,
        static_url_path='',
        static_folder='../client/build',
        template_folder='../client/build'
    )

    ...

    @app.errorhandler(404)
    def not_found(e):
        return render_template("index.html")
    ```

    This setup catches all routes not defined on the server and serves `index.html` to enable client-side routing through React Router.

3. Run the Flask server:

    ```sh
    $ gunicorn --chdir server app:app
    ```

    Visit [http://localhost:8000](http://localhost:8000) in the browser to see the production version of the React application.

# Render Build Process

To deploy the application to Render:

1. Commit all work to your fork on GitHub and copy the project URL.
2. Navigate to your Render dashboard and create a new web service. Find your forked repository or use the copied URL.
3. Change the "Build Command" to:

    ```sh
    $ pip install -r requirements.txt && npm install --prefix client && npm run build --prefix client
    ```

4. Change the "Start Command" to:

    ```sh
    $ gunicorn --chdir server app:app
    ```

    These commands will:
    - Install Python dependencies with pip.
    - Install Node dependencies with npm.
    - Build the static site with npm.
    - Run the Flask server.

5. Set the following environment variables under the "Environment" tab:

    ```sh
    DATABASE_URI=postgresql://{retrieve this from render}
    PYTHON_VERSION=3.8.13
    ```

6. Click "Save Changes" and wait for the deployment to complete. When Render indicates the site is "Live," navigate to your site URL.

Congratulations! You've successfully deployed your Flask-React application to Render.
```
