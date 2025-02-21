# OpenTransit Metrics

Welcome to OpenTransit! We're a team of volunteers that use open data to improve public transit systems around the world.

If you'd like to work with us, get in touch on our Slack channel! [Join the Code for PDX Slack](https://join.slack.com/t/codeforpdx/shared_invite/zt-4msr5np3-n5qBye3GG~4hA_7XZkczgA) and find the `#opentransit-pdx` channel. We're excited to partner with transit agencies, journalists, and other data junkies across the world. See below for instructions on joining our team of contributors.

## About this repository

This repo is for [our flagship app](http://muni.opentransit.city/), which uses historical transit data to help riders, transit advocates, and transit planners understand how well -- or how poorly -- transit systems are doing and find ways to improve them.

The app currently supports San Francisco and Portland, but we're working to generalize it to work easily for other cities.

## Getting involved

[Our onboarding doc](https://bit.ly/opentransitpdx) is a great way to get started. It'll provide you instructions on joining our GitHub organization, our Slack, our Google Drive, etc.

### Contributing

Once you've followed the instructions on the onboarding doc, visit our Issues page and [identify good first issues](https://github.com/codeforpdx/opentransit-metrics/labels/Good%20First%20Issue) to find a good project to get started on.

Our Slack is very active, so don't hesitate to ask there if you need guidance or suggestions on picking a project!

If you're non-technical, ask on Slack -- there's a lot of product management, marketing, design, and other work that we don't track on GitHub.

## Getting started

First make a local clone of this repo.

Then get Docker for your local environment (to run the application from that local code): Install [Docker Desktop](https://www.docker.com/products/docker-desktop) or another Docker distribution for your platform.

Build and run the Docker containers -- run this on your local terminal from the root of your local repository clone:

```sh
docker-compose up
```

**Mac M1 users see 'Notes for Developers' below.**

This will run the React frontend in development mode at http://localhost:3000,
and the Flask backend in development mode at http://localhost:5000.

Your local directory will be shared within the Docker container at `/app`.
When you edit files in your local directory, the React and Flask containers should automatically update with the new code.

To start a shell within the Flask Docker container, run `./docker-shell.sh` (Linux/Mac) or `docker-shell` (Windows).

You can run command line scripts like `backend/compute_arrivals.py` and `backend/headways.py` from the shell in the Docker container.

If you need to install some new dependencies in the Docker images, you can rebuild them via `docker-compose build`.

### Troubleshooting

| Error message | Solution |
| --- | --- |
| `Module not found: can't resolve ...` | Run `docker-compose build` |
| `Failed to execute script docker-compose` | Open Docker Desktop app first |

## Your first pull request

Our usual workflow is for a GitHub contributor (once you're added to the GitHub organization; see the onboarding guide) to create a new branch for each pull request. Once you're ready, start a pull request to merge your branch back into master.

Our pull request template will request that you fill out some key fields. It'll also automatically tag some repo maintainers to review your PR.

### Code style

This repository uses eslint to enforce a consistent style for frontend JavaScript code.

Before committing, run `dev/docker-lint.sh` (Mac/Linux) or `dev\docker-lint.bat` (Windows) to check for style errors and automatically fix formatting issues. (You will need to run `docker-compose up` or `docker-compose build` at least once before the docker-lint script will work.)

GitHub automatically runs tests for each push to check for eslint errors. If eslint reports any style errors, pull requests will show a failing check.

### Deploying to Heroku

When you make a Pull Request, we would suggest you deploy your branch to Heroku so that other
team members can try out your feature.

First, create an account  at [heroku.com](https://heroku.com) and
[create an app](https://dashboard.heroku.com/apps). Follow the instructions to deploy
using Heroku Git with an existing Git repository.

The first time you deploy to Heroku, you'll need to tell it to build Docker
containers using heroku.yml:

```sh
heroku stack:set container
```

You then need to set up a remote called `heroku`. Then run this to deploy your local branch:

```sh
git push heroku local-branch-name:master
```

Then copy the link to this app and paste it in the PR.

#### How Deployment Works

(**TODO**) Once a PR is merged into master, Google Cloud Build  will automatically build
the latest code and deploy it to a cluster on Google Kubernetes Engine (GKE).
The build steps are defined in `cloudbuild.yaml`.

## Our tech stack

### Overall

- **Docker** - to ensure a consistent environment across machines.
- **Docker Compose** - to run multiple containers at once.

### Frontend

- **NPM** - for package management. We explicitly decided to *not* use Yarn, because both
package managers offer similar performance, we were already using NPM for backend
package management, and the Yarn roadmap did not offer compelling
improvements going forward.
- **React** - Selected for popularity, simple view, and speedy virtual DOM. Code lives in the `/frontend` directory.  It was built using
[Create React App](https://facebook.github.io/create-react-app/docs/folder-structure).
- **Material UI** - which we use over Bootstrap since MUI doesn't rely on jQuery. It has a
popular React framework and looks great on mobile.
- **Redux** - for state management and to simplify our application and component interaction.
- **Redux Thunk** - as middleware for Redux.
- **React Hooks** - to manage interactions with state management.
- **Functional Components** - We migrated away from ES6 React Components and toward React
[Functional Components](https://reactjs.org/docs/components-and-props.html) due to the simpler component logic and the ability to use React Hooks that Functional Components offer.
- **ESLint** - Linting set in the format of AirBNB Style.
- **Prettier** - Code formatter to maintain standard code format for the frontend code.
- **Husky** - Pre-commit hook to trigger Prettier auto formatting before pushing to Github.

### Backend

- **Flask** - provides API endpoints used by the frontend.
- **GraphQL/Graphene** - a flexible API for returning various metrics requested by the frontend.
- **Pandas** - much of the data processing logic is implemented using Pandas data frames, e.g. when computing arrival times from raw GPS data.
- **NumPy** - algorithms involving arrays are implemented using Numpy for better performance, e.g. computing wait times and trip times.
- **Amazon S3** - the backend stores various data files (including route configuration, timetables, historical arrival times, and precomputed stats) as publicly-readable gzipped JSON files in S3, allowing the frontend to fetch data directly from S3 without hitting the Flask backend, and allowing multiple developers to share the same data without having to compute it themselves.
- **opentransit-collector** - A node.js app in a separate repo (https://github.com/codeforpdx/opentransit-collector) which fetches the raw GPS location data for all vehicles in a transit agency every 15 seconds and stores the data in S3.
- **Unittest** - Framework for testing the backend Python code.

## Notes for developers

### Python

If you ever need to use a new pip library, make sure you run `pip freeze > requirements.txt`
so other contributors have the latest versions of required packages.

### Running JupyterLab on Docker

0. Prerequisite - make sure you can run `docker-compose up` and navigate to `localhost:3000/` before making any of the below changes.
1. Add port 9999:9999 binding to your `docker-compose.override.yml` file (create this file if you don't have one). Your file should look something like:
```
version: "3.7"
services:
  flask-dev:
    ports:
      - "9999:9999"
    environment:
      AWS_ACCESS_KEY_ID: "XXX"
      AWS_SECRET_ACCESS_KEY: "XXX"
```
2. Add `jupyterlab` to the end of the `backend/requirements.txt` file.
3. Open a Terminal window and make sure you're inside the repo root directory (you should see the `docker-composexxx.yml` files if you run `ls` in the Terminal).
4. Run `docker-compose build` to rebuild the image.
5. Run `docker-compose up` to start up the containers.
6. Once the flask-dev container is running, start a new Terminal in the root directory and run `sh docker-shell.sh`.
7. You should see:
```
+ docker exec -it metrics-flask-dev bash
root@2dd2f4d0e170:/app/backend#
```
8. Now run:
```
jupyter-lab --ip=0.0.0.0 --port=9999 --allow-root --no-browser --NotebookApp.token='' --NotebookApp.password=''
```
This will start a new jupyter notebook server running on port 9999.

9. Go to `localhost:9999/` in your favorite browser.

### Windows

If you're developing within Docker on Windows, by default, React does not automatically recompile the frontend code when you make changes.
In order to allow React to automatically recompile the frontend code within the Docker container when you edit files shared from your
Windows host computer, you can create a `docker-compose.override.yml` to enable CHOKIDAR_USEPOLLING like this:

```yml
version: "3.7"
services:
  react-dev:
    environment:
      CHOKIDAR_USEPOLLING: "true"
      CHOKIDAR_INTERVAL: "2500"
```

This setting is not in the main docker-compose.yml file because CHOKIDAR_USEPOLLING causes high CPU/battery usage for developers using Mac OS X,
and CHOKIDAR_USEPOLLING is not necessary on Mac OS X to automatically recompile the frontend code when it changes.

### Mac M1 
If you're developing within Docker on a Mac with an M1 chip, you need to change the docker-compose platform. Create a `docker-compose.override.yml` to enable specify the platform like this:

```yml
version: "3.7"
services:
  flask-dev:
    platform: linux/amd64
  react-dev:
    platform: linux/amd64
```

### Configuring the displayed transit agency

By default, the app shows statistics for TriMet in Portland, Oregon. You can configure the transit agency displayed in the web app by setting the OPENTRANSIT_AGENCY_IDS environment variable.

Other available agency IDs include:
- `trimet` (TriMet in Portland, Oregon)
- `portland-sc` (Portland Streetcar)
- `muni` (San Francisco Muni)

To set this environment variable in development when using Docker, create a file named docker-compose.override.yml file in the root directory of this repository, like so:

```
version: "3.7"
services:
  flask-dev:
    environment:
      OPENTRANSIT_AGENCY_IDS: portland-sc
```

After changing docker-compose.override.yml, you will need to re-run `docker-compose up` for the changes to take effect.

### Advanced Concepts

Please see [ADVANCED_DEV.md](docs/ADVANCED_DEV.md) for even more advanced information like computing arrival times and deploying to AWS.

See [agencies.md](docs/agencies.md) for configuring for different agencies, and how the front end gets the configuration information.  Important for testing with other devices against your dev machine.
