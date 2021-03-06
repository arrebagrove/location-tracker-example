# Location Tracker

A sample agent showcasing a zero-knowledge proof of location.

We simulate farmers harvesting coffee beans and selling them to merchants with a zero-knowledge proof of authorized location.
The merchants advertise a list of authorized areas for the coffee beans and the verifies that incoming beans come from an authorized area, without revealing anything about the exact area of the beans.

It can be useful when farmers need to comply with the merchants' requirements without revealing their exact location.

## Zero-knowledge proof setup

The setup was done with [Stratumn's proof of location ZKP app](https://github.com/stratumn/pequin/tree/master/pepper/proof_of_location) by running the `setup.sh` script.
It generated two executables and two keys (one for the prover and one for the verifier).
We assume that the following ceremony happened in a secure way:

* Merchants received the verifier executable and verifier key
* Farmers received the prover executable and prover key

For this demo, those executables and keys are located in the `proof-of-location` directory.

### Merchant actions

Each merchant will initialize a map with its merchant name.
The merchant can then add authorized areas to the state with the `addArea` action.
When farmers send coffee beans via the `sendBeans` action, the agent will verify the zero-knowledge proof and automatically reject coffee beans from unauthorized locations.

**Note**: for the purpose of this demo, we restricted the ZKP circuit to a number of three authorized areas. That means each map should ideally use the `addArea` action three times otherwise you might see some issues with the proofs. This is of course something we'll change in the future as we productize the system.

### Farmer actions

The farmers will use the `sendBeans` action to send coffee beans with a zero-knowledge proof of location.
First, the farmers need to fetch the latest list of allowed areas from the state of the merchant they wish to send coffee beans to.
This means they choose the segment to which they want to append.
Another solution would be that they could choose to append all segments to a chain with no forks and reference the segment containing the list of authorized areas.
It's up to the specific usecase to define how they want to organize the chainscript architecture.

To compute the proof, the farmer simply runs the `proof_of_location/prove.sh` script.
Alternatively, the farmer can use the provided docker image: `docker run -it stratumn/location-tracker-prover:latest`.
It will ask for the farmer's location and will generate the proof.
The list of authorized areas is in `proof_of_location/prover_verifier_shared/proof_of_location.inputs` and can be updated to match the state in the map.

The farmer directly sends the proof bytes (in hexadecimal) to the `sendBeans` action.

### Next steps

Nothing currently prevents a farmer from computing a proof of location with fake GPS data (faking his location).
Nothing prevents farmers to re-use an old proof either.
This system isn't secure until we add secure hardware in the farmer's hands and cryptographic signatures.
This is what we'll want to focus on to build this system in real-life scenario.

## Requirements

* [Docker >= 1.10](https://www.docker.com/products/docker)
* [Docker Compose >= 1.6](https://docs.docker.com/compose/install)

Docker Compose is already included in some distributions of Docker.

You can check versions by running:

```
$ docker version
$ docker-compose version
```

Your installation of Docker must bind containers to `127.0.0.1` in order for the
agent user interface to work properly (which should already be the case if you
are using a recent release of Docker).

To make sure that it is the case, check that the value of `$DOCKER_HOST`
(or `%DOCKER_HOST%` on Windows) is not set:

```
$ echo $DOCKER_HOST # or echo %DOCKER_HOST% on Windows
```

Also make sure Docker is running properly:

```
$ docker run hello-world
```

If your installation of Docker requires `root` permissions, you can add your
username to the `docker` group (you might need to restart your terminal after
doing so).

## Launch development server

To launch all the services locally, go to the root folder and run:

```
$ strat up
```

The agent is bound to `http://localhost:3000`. It will automatically restart
when a file is changed.

A web user interface for the agent is available on `http://localhost:4000`.
You can use it to test and visualize your workflows.

Press `Ctrl^C` to stop the services.

**Note:** While the agent will automatically restart if a file changes, you will
have to run `strat up` again if you add NodeJS packages to `package.json`.
During development, the segments will be saved to the `./segments` directory.
Make sure Docker is configured to allow mounting that directory.
Note that the file storage adapter is very slow and only suited for development and
testing.

## Run tests

```
$ strat test
```

## Project structure

The agent's files are in the `./agent` directory.

### Workflow actions

The actions are defined in `./agent/lib/actions.js`.
You can arrange your actions in different files then `require()` them if you
want.

### Tests

The tests are defined in `./agent/test/actions`. You can also arrange them in
different files if you prefer.

During tests, the same store and fossilizer types used in development are
launched. They are started in a different namespace so that they don't conflict
with the development Docker containers.

### Configuration

There are some configuration files in `./config` that make it possible to define
environment variables for the services, including your agent.

The variables defined in `common.env` are accessible by all services during
development and testing.

The variables defined in `common.secret.env` work similarly, but will not be
included in a Git repository so it is a good place to define more sensitive
variables.

The variables defined in `dev.env` and `dev.secret.env` are only accessible
during development.

The variables defined in `test.env` and `test.secret.env` are only accessible
during testing.

### Validation

There is a json file in `./validation` named `rules.json`.
It contains json schema validation rules that are executed for each action your processes handle.

## License

See `LICENSE` file.
