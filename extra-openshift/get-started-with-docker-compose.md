# Get started with Docker Compose

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

This exercise demonstrates the original Docker Compose workflow. **You can run these same steps with Podman** using the `podman compose` command (see the Podman Compose chapter), but this exercise uses the Docker toolchain.

{% hint style="info" %}
**Using with Podman:** If you prefer to use Podman instead of Docker, replace `docker-compose` with `podman compose` in all commands below. You'll need `podman-compose` or `docker-compose` installed as a compose provider.
{% endhint %}

### Step 1: Setup <a href="#step-1-setup" id="step-1-setup"></a>

Define the application dependencies.

1.  Create a directory for the project:

    ```
    $ mkdir composetest
    $ cd composetest
    ```
2.  Create a file called `app.py` in your project directory and paste this in:

    ```python
    import time
    import redis
    from flask import Flask

    app = Flask(__name__)
    cache = redis.Redis(host='redis', port=6379)

    def get_hit_count():
        retries = 5
        while True:
            try:
                return cache.incr('hits')
            except redis.exceptions.ConnectionError as exc:
                if retries == 0:
                    raise exc
                retries -= 1
                time.sleep(0.5)

    @app.route('/')
    def hello():
        count = get_hit_count()
        return 'Hello World! I have been seen {} times.\n'.format(count)
    ```

    In this example, `redis` is the hostname of the redis container on the application's network. We use the default port for Redis, `6379`.

3.  Create another file called `requirements.txt` in your project directory and paste this in:

    ```
    flask
    redis
    ```

### Step 2: Create a Containerfile <a href="#step-2-create-a-dockerfile" id="step-2-create-a-dockerfile"></a>

In your project directory, create a file named `Containerfile` (or `Dockerfile`) and paste the following:

```
FROM docker.io/python:3.12-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

This tells the container engine to:

* Build an image starting with the Python 3.12 Alpine image.
* Set the working directory to `/code`.
* Set environment variables used by the `flask` command.
* Install gcc and other dependencies
* Copy `requirements.txt` and install the Python dependencies.
* Add metadata to the image to describe that the container is listening on port 5000
* Copy the current directory `.` in the project to the workdir `.` in the image.
* Set the default command for the container to `flask run`.

### Step 3: Define services in a Compose file <a href="#step-3-define-services-in-a-compose-file" id="step-3-define-services-in-a-compose-file"></a>

Create a file called `compose.yaml` in your project directory and paste the following:

```yaml
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "docker.io/redis:alpine"
```

This Compose file defines two services: `web` and `redis`.

#### Web service <a href="#web-service" id="web-service"></a>

The `web` service uses an image that's built from the Containerfile in the current directory. It then binds the container and the host machine to the exposed port, `5000`.

#### Redis service <a href="#redis-service" id="redis-service"></a>

The `redis` service uses a public Redis image pulled from Docker Hub.

### Step 4: Build and run your app with Compose <a href="#step-4-build-and-run-your-app-with-compose" id="step-4-build-and-run-your-app-with-compose"></a>

1.  From your project directory, start up your application:

    ```
    $ podman compose up
    ```

    Compose pulls a Redis image, builds an image for your code, and starts the services you defined.
2.  Enter http://localhost:5000/ in a browser to see the application running.

    You should see a message in your browser saying:

    ```
    Hello World! I have been seen 1 times.
    ```
3.  Refresh the page. The number should increment.

### Step 5: Edit the Compose file to add a bind mount <a href="#step-5-edit-the-compose-file-to-add-a-bind-mount" id="step-5-edit-the-compose-file-to-add-a-bind-mount"></a>

Edit `compose.yaml` in your project directory to add a bind mount for the `web` service:

```yaml
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_DEBUG: "1"
  redis:
    image: "docker.io/redis:alpine"
```

The new `volumes` key mounts the project directory on the host to `/code` inside the container, allowing you to modify the code on the fly without having to rebuild the image.

### Step 6: Re-build and run the app <a href="#step-6-re-build-and-run-the-app-with-compose" id="step-6-re-build-and-run-the-app-with-compose"></a>

From your project directory, run `podman compose up` to build the app with the updated Compose file, and run it.

```
$ podman compose up
```

### Step 7: Update the application <a href="#step-7-update-the-application" id="step-7-update-the-application"></a>

Because the application code is now mounted into the container using a volume, you can make changes to its code and see the changes instantly, without having to rebuild the image.

Change the greeting in `app.py` and save it. For example, change the `Hello World!` message to `Hello from Podman!`:

```python
return 'Hello from Podman! I have been seen {} times.\n'.format(count)
```

Refresh the app in your browser. The greeting should be updated, and the counter should still be incrementing.

### Step 8: Clean up <a href="#step-8-experiment-with-some-other-commands" id="step-8-experiment-with-some-other-commands"></a>

You can bring everything down, removing the containers entirely, with the `down` command. Pass `--volumes` to also remove the data volume used by the Redis container:

```
$ podman compose down --volumes
```
