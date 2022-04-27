# Getting started

Entropy has a web-browser based interface, and includes the whole stack you need
for writing, executing, managing and storing your experimental workflows that
are setup in default installation with a few commands. Advanced users on the
other hand might be interested in advanced modular installations that are also
supported.

## Installing Docker compose

Follow official Docker Compose installation documentation for your own operating
system:

[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

## Simple single-machine installation

For now, installation is directly from GitHub:

```
git clone --depth 1 https://github.com/entropy-lab/entropy-hub.git
cd entropy-hub
docker-compose up
```

When starting the first time, this will take some time since the system is being
downloaded and initialised. Once the terminal information stops changing, open
`http://localhost` in your web-browser.

To switch off entropy press ++ctrl+c++ in terminal once, and wait for the system
to switch off. To run again Entropy, it is enough to run `docker-compose up`
again.

## Running an example

Open in the web browser `http://localhost`. Open Experiments panel. Code editor
will open. In terminal window type

```
python3 -m entropylab.flame.example.simple
```

And follow up on commands there. Alternatively, you can try more complex example
by running

```
python3 -m entropylab.flame.example.complex
```

For final workflow execution, switch to workflow graphical view and press `Run`.

To start empty project start

```
python3 -m entropylab.flame.template.empty
```

## Advanced installations

### Central data store

### Runtime outside of container

### Runtime consisting of multiple machines

### Single front-end, multiple runtimes
