# jenkins-hookshot
jenkins-hookshot is an experimental Python app (using the [Tornado][1]
framework) that listens for and processes GitHub webhooks using [Jenkins][2]
and the Jenkins [Job DSL plugin][3].

When a webhook is received, jenkins-hookshot creates a seed job on a random
Jenkins master running on [Marathon][4], passing various parameters such as
the username or organization, repo URL, and Git SHA. The original webhook
payload is stored in a database for further use.

For more information on setting up Jenkins masters on Marathon, see this blog
post: [Scaling Jenkins with Mesos and Marathon][5].


## Prerequisites
  * Python 3.3 (developed and tested with Python 3.3.6)
  * Marathon 0.7.5 (API v2)
  * Working installations of Mesos, Marathon, and Redis
  * One or more Jenkins masters running in Marathon under a single app ID
  * pyenv and pyenv-virtualenv (recommended)


## Caveats
  * Only GitHub `ping` and `push` event actions are currently implemented.
  * See [TODO.txt][6] for a more detailed list of tasks remaining.


## Usage
### Getting the code
_As of this writing, jenkins-hookshot is not available on PyPI._

Clone the repo from GitHub:

```
$ git clone https://github.com/rji/jenkins-hookshot
```

### Running the app
To run the app using pyenv and pyenv-virtualenv:

```
$ python --version              # ensure Python 3.3.x
$ pyenv virtualenv hookshot     # name is arbitrary, but should be unique
$ pyenv activate hookshot
(hookshot) $ pip install -r requirements.txt
(hookshot) $ python run.py [OPTIONS]
```

Set the configuration parameters as they apply to your environment. For example,
let's say Marathon isn't running on the same host as this app. You should use
something like:

```
$ python run.py --marathon_host=10.141.141.10:8080
```

For a detailed list of CLI arguments (and their defaults), run
`python run.py --help`.

If you want to test this project locally, you might want to try out [ngrok][7]
to receive hooks from GitHub.

Lastly, configure a GitHub repo to send "push" events (type `application/json`)
to the publicly-available hostname (or IP address) and port.


## Writing Jenkins Job DSL Scripts
The following parameters are passed to the seed job, and are available to use
in Job DSL scripts:
  * `REPO_NAMESPACE` – the GitHub username or organization that owns the repo
  * `REPO_NAME` – the actual name of the Git repo
  * `REPO_DESCRIPTION` – the description of the GitHub repo
  * `REPO_URL` – URL to the GitHub repo
  * `GIT_SHA` – the HEAD SHA (also known as `after`) in the webhook payload
  * `UUID` – unique identifier generated by this app

For full documentation on how to write Jenkins Job DSL scripts, see the
[Jenkins Job DSL plugin wiki][8].


## Endpoints
### `POST /v1/create`
When a GitHub webhook POSTs JSON to this endpoint, a seed job is created on a
random Jenkins instance running on Marathon. The seed job is then triggered
with various parameters in the payload, and the original payload is stored in
Redis for later consumption.

### `GET /ping`
Health check; returns `pong`.

**Request:**

```
HTTP/1.1 200 OK
Content-Length: 4
Content-Type: text/plain
Date: Wed, 31 Dec 2014 01:01:16 GMT
Etag: "0e514a0662bcb69dc863953d1ce26e3d40e81a87"
Server: TornadoServer/4.0.2
```

**Response:**

```
pong
```

## Author
Roger Ignazio, me@rogerignazio.com

## License
Apache License, Version 2.0


[1]: http://www.tornadoweb.org/en/stable/
[2]: http://jenkins-ci.org
[3]: https://wiki.jenkins-ci.org/display/JENKINS/Job+DSL+Plugin
[4]: https://mesosphere.github.io/marathon/
[5]: http://rogerignazio.com/blog/scaling-jenkins-mesos-marathon/
[6]: TODO.txt
[7]: https://ngrok.com/
[8]: https://github.com/jenkinsci/job-dsl-plugin/wiki
