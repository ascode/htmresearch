# Numenta Imbu

Imbu is a web application to demonstrate applications of Hierarchical
Temporal Memory (HTM) to Natural Language Processing (NLP) problems, as well as
support ongoing research.

## License

  AGPLv3. See [LICENSE.txt](LICENSE.txt)


## Repository

```shell
.
├── Dockerfile                     # Docker build file
├── LICENSE.txt                    # Dual: Commercial and AGPLv3
├── README.md                      # This file, a project overview
├── conf/
│   ├── mime.types                 # nGinx mime types configuration
│   ├── nginx-fluent.conf          # Main nGinx configuration file
│   ├── nginx-supervisord.conf     # nGinx-specific supervisor configuration
│   ├── nginx-uwsgi.conf           # nGinx reverse proxy configuration
│   └── supervisord.conf           # Main supervisor configuration file
├── engine/
│   ├── fluent_api.py              # Python web application
│   └── sample_reviews/				# Sample dataset and models
├── gui/
│   ├── browser/
│   │   ├── actions/
│   │   │   ├── dataset-list.js	
│   │   │   ├── dataset-select.js
│   │   │   ├── dialog-close.js
│   │   │   ├── dialog-open.js
│   │   │   ├── search-clear.js
│   │   │   ├── search-query.js
│   │   │   └── server-status.js
│   │   ├── app.js                 # Fluxible application entry point
│   │   ├── components/
│   │   │   ├── dialog.jsx         # Dialog component for viewing full documents
│   │   │   ├── main.jsx           # Main JSX template
│   │   │   ├── search-results.jsx
│   │   │   └── search.jsx
│   │   ├── css/
│   │   │   └── main.css           # Main CSS file
│   │   ├── index.html             # HTML Index
│   │   └── stores/
│   │       ├── search.js
│   │       └── server-status.js
│   ├── loader.js                  # Javascript initialization
│   └── main.js                    # Application entry point, initializes browser app
├── gulpfile.babel.js              # Babel.js ES6 Config file for the Gulp build tool
├── package.json                   # Node.js `npm` packages, dependencies, and App config
├── requirements.txt               # Python dependencies
├── setup.py                       # Python setuptools install entry point
└── start_imbu.sh                  # IMBU startup entry point
```

## Technology Stack

Imbu is implemented as a hybrid Python and Javascript application -- Javascript
on the front end, Python on the back end.

### Javascript:

- [npm](https://www.npmjs.com/), for Javascript package management
- [Gulp](http://gulpjs.com/), for building Javascript frontend
- [Fluxible](http://fluxible.io/), for frontend framework
- [Babel](https://babeljs.io/), for compiling JSX templates

### Python:

- [NuPIC](http://numenta.org/), Machine Intelligence research platform

### Infrastructure:

- [nGinx](https://www.nginx.com/), for serving content
- [uWSGI](https://uwsgi-docs.readthedocs.org/en/latest/), for running Python web app
- [Docker](https://www.docker.com/), for reproducible build and runtime environment

### Third-party Services:

- [Cortical.io](http://www.cortical.io/)

## API

Imbu implements a web service API at `/fluent`, supporting a `POST` HTTP
method for querying Imbu models.


## Running Imbu

### Cortical.io Setup

1. Get [cortical.io](http://www.cortical.io/) API key from http://www.cortical.io/resources_apikey.html
1. Create `CORTICAL_API_KEY` environment variable with your new API key.
1. Create `IMBU_RETINA_ID` environment variable with the name of the *retina* to use.
1. Create `IMBU_LOAD_PATH_PREFIX` environment variable with the root directory for pre-trained models

### Run Imbu in Docker

In the root of `nupic.research`:

```
# build
docker build -t imbu:latest -f projects/imbu/Dockerfile .

# run
docker run \
  --name imbu \
  -d \
  -p 8080:80 \
  -e IMBU_LOAD_PATH_PREFIX=${IMBU_LOAD_PATH_PREFIX} \
  -e CORTICAL_API_KEY=${CORTICAL_API_KEY} \
  -e IMBU_RETINA_ID=${IMBU_RETINA_ID} \
  imbu:latest

# get ip (local docker network, maybe 192.168.x.x)
docker-machine ls

# open in browser:
#   http://192.168.x.x:8080
```

A few salient points about the command(s) above:

- `docker build -t imbu:latest -f projects/imbu/Dockerfile .` builds a docker
   image from the `Dockerfile` in `projects/imbu`, specifying the root of
   `nupic.research` as the build context.  This is required for inclusion of
   `htmresearch` in addition to `projects/imbu/`.
- The repository is called `imbu`, and the image is tagged `latest`.
- `docker run` starts a container from the image tagged `latest` in the `imbu`
  repository.
- `--name imbu` names the container `imbu`
- `-d` causes the container to run in `daemon` mode in the background
- `-p 8080:80` maps port 80 in the container to port 8080 on the host.
- `-e IMBU_LOAD_PATH_PREFIX=${IMBU_LOAD_PATH_PREFIX}` specifies the location
  from which pre-trained models will be loaded.
- `-e CORTICAL_API_KEY=${CORTICAL_API_KEY}` defines the `CORTICAL_API_KEY`
  environment variable in the container.  For the command above to work as-is
  you must have `CORTICAL_API_KEY` set in your own environment!  You may specify
  the value inline.
- `-e IMBU_RETINA_ID=${IMBU_RETINA_ID}` defines the `IMBU_RETINA_ID`
  environment variable in the container.  For the command above to work as-is
  you must have `IMBU_RETINA_ID` set in your own environment!  You may specify
  the value inline.

Additionally, there are some optional `docker run` arguments:

- Map the `/opt/numenta/nupic.research/projects/imbu/cache` directory in the
  container to one on on the host external to the container to persist model
  state across containers.  For example:

  ```
  -v `pwd`/cache:/opt/numenta/nupic.research/projects/imbu/cache
  ```

A few helper utilities are included for your convenience:

- `destructive-build-and-refresh-container.sh` to clear persistent model cache,
  rebuild, and restart the container from scratch
- `graceful-build-and-refresh-container.sh` to rebuild and restart the
  container, preserving persistent model cache

You may also set additional options to the `docker` command by specifying the
`IMBU_DOCKER_OPTIONS` environment variable. Use `docker help` for a list of
available options.

Both of these helper utilities will pass along additional arguments to the
`docker run` command specified in the `IMBU_DOCKER_EXTRAS` environment
variable.  For example, to map the directories described above.  It can be
helpful to define all relevant environment variables in a file that you can
either `source` at-will, or in your bash profile.  For example:

```
export IMBU_DOCKER_EXTRAS="-v <path to cache>:/opt/numenta/nupic.research/projects/imbu/cache "
export CORTICAL_API_KEY="<cortical.io api key>"
export IMBU_RETINA_ID="en_associative"
export IMBU_LOAD_PATH_PREFIX="/opt/numenta/nupic.research/projects/imbu/engine"
export IMBU_DOCKER_OPTIONS="--tlsverify=false"
```

Once running, you can monitor progress of the container in real-time by running
`docker logs -f --tail=all imbu`.  Should you need to "log in" to to debug the
running container, you can run `docker exec -ti imbu /bin/bash` to run a bash
prompt in the container.
