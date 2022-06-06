# The Entropy Project and entropy CLI

Entropylab is a python package for lab workflow management.
It is built for streamlining running quantum information processing experiments. 

Our goal is to address several hurdles in experiment design: 

1. Building, maintaining and executing complex experiments
2. Data collection
3. Device management
4. Calibration automation

To tackle these problems, Entropy revolves one central structure: an execution graph. 
The nodes of a graph give us a convenient way
to brake down experiments into stages and to automate some repetitive tasks.

For example data collection is automated, at least in part, 
by saving node data and code to a database. 

Device management is the challenge of managing the state and control
of a variety of different resources. These include, but are not limited to,
lab instruments. 
They can also be computational resources, software resources or others. 
Entropy is built with tools to save such resources to a shared database 
and give nodes access to the resources needed during an experiment. 

Automating the calibration process is an important reason why we built
Entropy.
This could be though of as the use-case most clearly benefiting from shared
resources, persistent storage of different pieces of information
and the graph structure. If the final node in a graph is the target
experiment, then most nodes between the root node and the target are likely to be
calibration steps.

## Versioning and the Alpha release 

Entropy is still in its Alpha release stages, you can learn more about the Entropy versioning scheme in the `VERSION.md`
document of the entropylab repository. 
This means this version is a work in progress in several important ways: 

1. It is not fully tested
2. There are important features missing.
3. There will more than likely be breaking changes to the API for a while until we learn how things should be done. 

Keep this in mind as you start your journey. 

## Installation

Installation is done from pypi using the following command

```shell
pip install entropylab
```

## Testing your installation

Entropy ships with a Command Line Interface (CLI). 

To test your installation go to the command line, type `entropy` and press ↩. 
The following usage description should appear:
```
usage: entropy [-h] [-v] {init,upgrade,serve} ...

positional arguments:
  {init,upgrade,serve}
    init                initialize a new Entropy project
    upgrade             upgrade an Entropy project to the latest version
    serve               serve & launch the results dashboard app in a browser

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit

```
## Hello world
In addition to the previous test you can run a simple "hello-world" example by running the following code.

```python
from entropylab import *

def my_func():
    return {'res': 1}

node1 = PyNode("first_node", my_func, output_vars={'res'})
experiment = Graph(None, {node1}, "run_a")  # No resources used here
handle = experiment.run()
```
