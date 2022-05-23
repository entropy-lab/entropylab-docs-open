# Entropy pipeline: an experiment graph

An experiment is a set of steps which may or may not be directly dependent on each other. 
You may want to run steps (or not run them) based on the result of a previous step. 
You may want to run steps using a result from a previous step. And you will almost always need to have access to the
results each previous step had generated.

This model is well described by a Directed Acyclic Graph (DAG). Directed means there is a direction to the 
flow of the graph, from root to leaves. Acyclic means there shouldn't be closed loops in which you can get "stuck"
when flowing through the graph. DAG based authoring of experiments is perhaps the defining feature of Entropy. 
It's very much like running a data processing pipeline, which is why this section is called the entropy "pipeline".

A simple DAG is shown below. Each experimental stage can be placed in a separate node and nodes pass data to descendant 
nodes. This is marked as "res" on vertices in the image. Note how you can also set the node run order without passing 
data between nodes. For example, noded b and d have a vertex connecting them but no data is passed.

 ![DAG](../assets/DAG.svg)

The data passed into, and out of nodes is automatically saved to the result handling backend (see below).
But we can also take advantage of the structure to do some pretty nifty things. A graph run can have branching logic 
(conditional execution of branches), can be parameterized and there can be more than one way to traverse a graph from 
one end to another. This last point is particularly important when you have a graph with one target node, 
including the algorithm you really want to run and many calibration nodes with the all the experimental prerequisites.
Solving the problem of complex calibration is actually the first task we had in mind for this project.

## Building your first graph

The PyNode is the basic element in an Entropy grpah. It runs arbitrary python code defined in the `program` parameter.
In the example below, the `program` is defined by the function `node_operation` which doesn't do anything but returns a 
dictionary with the key:value pair `{`res`:1}`. Programs used in PyNodes must always return a dictionary. 
The node also has a name, set by `label` and defined the names of the variables it (optionally) passes on to other nodes.

```python
def node_operation():
    return {'res':1}

node1 = PyNode(label="first_node", program=node_operation,output_vars={'res'})
```

You can now attach the node to a graph (containing just one node)

```python
experiment = Graph(resources=None, graph={node1}, story="run_a") 
```

There are some explanations in order here. You can see that this graph
has the `resources` parameter set to `None`, but we have not defined
resources yet. We also didn't say anything about the `story` associated
with a graph. But let's ignore that for now.

We can now run the graph, saving the results to `handle`

```.python
def node_operation():
    return {'res':1}

node1 = PyNode(label="first_node", program=node_operation,output_vars={'res'})
experiment = Graph(resources=None, graph={node1}, story="run_a") 
handle = experiment.run()
```

```
    2021-09-29 12:53:56,805 - entropy - WARNING - Results of current execution None will be permanently lost on session close
    2021-09-29 12:53:56,807 - entropy - INFO - Running node <PyNode> first_node
    2021-09-29 12:53:56,807 - entropy - INFO - Finished entropy experiment execution successfully
```

"What's with the `WARNING`?" you may ask. Because we didn't define
any `resources`, and we don\'t have a backend for result saving defined,
Entropy warns us we will not have any persistence (nothing will be saved
for posterity). We will address this issue soon. Regardless, while in
the python session, we can access the (wholly unspectacular) experiment
results as follows:

```.python
handle.results.get_results_from_node("first_node")[0].results[0].data

1

```

You can see the returned result is what was returned by
`node_operation`. There is clearly some structure here: results are
returned in a list, that list is made of a certain type of object and so
on. We will go into more detail later but will gloss over this for now.

To view the graph, you need to have `graphviz` installed ([installation
guide](https://graphviz.org/download/)). If it\'s properly installed on
your environment, you can view the graph as follows:

```.python
handle.dot_graph()
```


![one_node](../assets/one_node.svg)

If not, you can use a textual representation of the graph. This is
called `dot` notation and is a standard way to serialize a graph. To
access the dot representation, run the following command

```.python
print(str(handle.dot_graph()))
```

```
    digraph {
    	graph [rankdir=LR]
    	first_node [shape=box]
    }
```

This string can then be pasted in a service such as
[GraphVizOnline](https://dreampuf.github.io/GraphvizOnline/) or
[Edotor](https://edotor.net/) to get the graphical representation.

## A graph with two nodes

As a graph with a single node is hardly a graph, here's an example
containing two nodes which pass a result between them.

For simplicity, both nodes are running the same operation (same
`node_operation` as before)

```.python
node1 = PyNode(label="first_node", program=node_operation,output_vars={'res'})
node2 = PyNode(label="first_node", program=node_operation,output_vars={'res'},input_vars={'ina':node1.outputs['res']})
experiment = Graph(resources=None, graph={node1,node2}, story="run_a") 
handle = experiment.run()
graph=handle.dot_graph()
graph.graph_attr['rankdir'] = 'LR'
graph
```

![two_nodes](../assets/two_nodes.svg)


By inspecting the results object, we can see we have two `ResultRecord`s
coming from each of the nodes, and an additional one which is
automatically added. The additional record is the result of the final
node, and it is always included.

```.python
handle.results.get_results()
```

```
    [ResultRecord(experiment_id=1632909245982750200, id=0, label='res', story=None, stage=0, data=1, time=datetime.datetime(2021, 9, 29, 12, 54, 5, 984892)),
     ResultRecord(experiment_id=1632909245982750200, id=1, label='res', story=None, stage=1, data=1, time=datetime.datetime(2021, 9, 29, 12, 54, 5, 985904)),
     ResultRecord(experiment_id=1632909245982750200, id=2, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}, time=datetime.datetime(2021, 9, 29, 12, 54, 5, 985904))]
```


# Results in Entropy: Setting up a DB

Collecting and arranging experimental data is a difficult task. It's
often managed in files and folders with some structure users agree upon,
but in many cases that is not enforced. To solve this problem we provide
a way to automate data saving as an integral part of structuring the
experimental sequence. Each node can save the data you specify, but also
saves additional data on its own.

Make sure you have an entropy project set up. You can do that by running the `init` CLI command. 
You can then create you db connection object as follows:
```.python
db = SqlAlchemyDB('.')

experiment_resources = ExperimentResources(db)
```
Running an experiment that saves something to the DB is now easy

```.python
node1 = PyNode("node_with_DB", node_operation,output_vars={'res'})
experiment_with_db = Graph(resources=experiment_resources, graph={node1},story="run with DB")
handle2 = experiment_with_db.run()
```

```
    2021-05-09 18:58:13,827 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:13,884 - entropy - INFO - Finished entropy experiment execution successfully
```


If you now try to observe the saved results, what do you expect to see?

```.python
handle2.results.get_results()
```

```
    [ResultRecord(experiment_id=1, id=1, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=1, id=2, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1})]
```


Only the results of the last run are here. Even if you run this
experiments a few more times, you will only see the last run.

This is a bit disappointing as we promised persistence. So, what gives?

Well, the `handle` is just for a specific experimental run, so it makes
sense you only see the results of *that run*. To see everything in the
database you can query it directly

```.python
db.get_results()
```

```
    [ResultRecord(experiment_id=1, id=1, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=1, id=2, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1})]
```


And if we run the experiment a few more times, see how the DB is
populated

```.python
[experiment_with_db.run() for ind in range(10)]
db.get_results()
```

```
    2021-05-09 18:58:24,634 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:24,709 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:24,725 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:24,799 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:24,820 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:24,872 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:24,886 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:24,928 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:24,942 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:24,988 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:25,000 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:25,049 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:25,061 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:25,108 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:25,120 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:25,163 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:25,174 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:25,221 - entropy - INFO - Finished entropy experiment execution successfully
    2021-05-09 18:58:25,234 - entropy - INFO - Running node <PyNode> node_with_DB
    2021-05-09 18:58:25,280 - entropy - INFO - Finished entropy experiment execution successfully
```
```
    [ResultRecord(experiment_id=1, id=1, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=1, id=2, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=2, id=3, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=2, id=4, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=3, id=5, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=3, id=6, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=4, id=7, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=4, id=8, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=5, id=9, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=5, id=10, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=6, id=11, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=6, id=12, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=7, id=13, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=7, id=14, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=8, id=15, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=8, id=16, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=9, id=17, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=9, id=18, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=10, id=19, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=10, id=20, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1}),
     ResultRecord(experiment_id=11, id=21, label='res', story=None, stage=0, data=1),
     ResultRecord(experiment_id=11, id=22, label='experiment_result', story='Final output of the experiment', stage=-1, data={'res': 1})]
```

# The Entropy Environment

By \"device management\" what is meant is more than just the set of all
instruments you use in the lab. A *device* in this context can be many
things. For example, it could be some calculation that is run on a
remote compute resource, it may be the object representing a web-based
GUI you are working with or, indeed, it can be your DC voltage source.
Sharing these resources between nodes in Entropy is easy. The nodes in
an Entropy graph have access to resources automatically, and there is a
centralized store in which you can register the resources that are
always present in the lab or temporary resources which only exist for a
specific run of an experiment.

## Lab and Experiment Resoruces

The mental image you need to keep to understand resources management in
Entropy is as follows: Resources belong to a lab and persist in the DB.
Every time we set up an experiment, we can use a lab resource by
importing it.

As an example, we first build a class representing some device, say an
oscilloscope.

``` {.python}
class my_scope:
    def __init__(self,name, ip):
        self.name=name
        self.ip=ip      
```

Typically you would start from an existing driver you already have, or a
contributed one. You can use exising drivers from the QCodes library
using our wrapper class, more details on this in seperate tutorial.

We start our example by deleting our DB file from before and setting up
a new lab.

Warning: This is destructive to any work we\'ve done so far so if you
want to keep the existing DB please copy the file over.

Note that setting up a lab is only done once under normal circumstances.
:::

::: {.cell .code}
``` {.python}
db_file='docs_cache/tutorial.db'
if os.path.exists(db_file):
  os.remove(db_file)
db = SqlAlchemyDB(db_file)
lab = LabResources(db)

```
:::

::: {.cell .markdown}
### Registering resources

We can now **register** our resources (our oscilloscope) to the lab. The
act of registration adds a resource to the ones available to us in the
lab.

There are many options here we will go into in later tutorials but for
now, we can point out just a few. The `register_resource` method takes a
`resource_class` argument, note that this is the **name** of the class
and not an instance. There are then `args` and `kwargs` we pass, which
are the *positional* arguments passed to the constructor of our resource
and the *keyword* (named) arguments we pass. We don\'t have do use both,
we can use just one or the other or (as in the example below) a
combination of both.
:::

::: {.cell .code}
``` {.python}
class my_scope:
    def __init__(self,name, ip):
        self.name=name
        self.ip=ip      
    def identify(self):
        print(f'This is scope:{self.name}')

lab.register_resource(name="my_scope", resource_class=my_scope,args=["gals scope"],kwargs={'ip':0})

```

::: {.output .stream .stdout}
    2021-05-09 22:13:20,016 - entropy - WARNING - instrument my_scope is not an entropylab Resource and additional metadata won't be saved
    2021-05-09 22:13:20,017 - entropy - WARNING - could not save source of my_scope
:::
:::

::: {.cell .markdown}
If you now try to run the cell from above again, this will fail. Why is
that? Because you are trying to register an existing device to the DB.
The `LabResources` API supports adding and removing resources from the
lab and this failing behaviour is intentional. It is meant to help
preventing an accidental overwrite of a device.

### Running an experiment with a resource

Let\'s now run an experiment the uses a registered resource. We need to
start an `experiment_resource` instance (and specify the db to which it
writes), and then import the resource we want to use from the lab.
:::

::: {.cell .code}
``` {.python}
experiment_resources = ExperimentResources(db)
experiment_resources.import_lab_resource('my_scope')

def node_action_with_scope(context: EntropyContext):
   
    scp=context.get_resource("my_scope")
    scp.identify()
    return {"a_out": 10}

node1 = PyNode("first_node", node_action_with_scope,output_vars={'a_out'})
experiment_with_instrument = Graph(experiment_resources, {node1}, "run_a") #No resources used here
handle_with_instrument = experiment_with_instrument.run()
```

::: {.output .stream .stdout}
    2021-05-09 22:49:55,134 - entropy - INFO - Running node <PyNode> first_node
    This is scope:gals scope
    2021-05-09 22:49:55,205 - entropy - INFO - Finished entropy experiment execution successfully
:::
:::

::: {.cell .markdown}
As you can see the `identify` method was called as expected.
:::

::: {.cell .markdown}
# Entropy with QUA

Entropy is developed to be a general purpose lab manager. It is however
developed primarily by [Quantum Machines](www.quantum-machines.co),
creators of the Quantum Orchastration Platform and the QUA programming
language.

Support of QUA is going to be extended in coming versions, primarily via
Entropy extensions. However, you can already run QUA code by using the
general purpose PyNode. This is shown in the following basic exmaple
:::

::: {.cell .code}
``` {.python}
#first we delete the DB file again
db_file='docs_cache/tutorial.db'
if os.path.exists(db_file):
  os.remove(db_file)
db = SqlAlchemyDB(db_file)
lab = LabResources(db)

#we now import the QUA package
from qm.QuantumMachinesManager import QuantumMachinesManager
from qm.qua import *
from qm import SimulationConfig

#now register the quntum machines mnager and import it to the experiment resources
lab.register_resource('qmm',QuantumMachinesManager)
experiment_resources = ExperimentResources(db)
experiment_resources.import_lab_resource('qmm')
```
:::

::: {.cell .markdown}
After some intro house keeping, we set up a two-node graph. A first node
to generate the config and pass it on, and a second node to run the QUA
code. We write the operation of these two PyNodes in the two following
functions:
:::

::: {.cell .code}
``` {.python}
def get_config():
    config = {
        "version": 1,
        "controllers": {
            "con1": {
                "type": "opx1",
                "analog_outputs": {
                    1: {"offset": +0.0},
                },
            }
        },
        "elements": {
            "qe1": {
                "singleInput": {"port": ("con1", 1)},
                "intermediate_frequency": 5e6,
                "operations": {
                    "playOp": "constPulse",
                },
            },
        },
        "pulses": {
            "constPulse": {
                "operation": "control",
                "length": 1000,  # in ns
                "waveforms": {"single": "const_wf"},
            },
        },
        "waveforms": {
            "const_wf": {"type": "constant", "sample": 0.2},
        },
    }

    return {'config': config}


def QUA_node_action(context: EntropyContext,config):
    qmm = context.get_resource('qmm')
    QM1 = qmm.open_qm(config)
    with program() as prog:
        play("playOp", "qe1")
    job = QM1.simulate(prog, SimulationConfig(int(1000)))  
    samples = job.get_simulated_samples()
    samples1 = samples.con1.analog['1']
    return {"samples": samples1}
```
:::

::: {.cell .markdown}
Finally, we can set up the nodes and graph and run our experiment:
:::

::: {.cell .code}
``` {.python}
config_node = PyNode("config_node", get_config,output_vars={'config'})
qua_node = PyNode("QUA_node", QUA_node_action,input_vars={'config':config_node.outputs['config']},output_vars={'samples'})
experiment_with_QUA = Graph(experiment_resources, {config_node,qua_node}, "a QUA run")
handle_with_QUA = experiment_with_QUA.run()
```

::: {.output .stream .stdout}
    2021-05-09 23:47:03,095 - qm - INFO - Performing health check
    2021-05-09 23:47:03,103 - qm - INFO - Health check passed
    2021-05-09 23:47:03,122 - entropy - INFO - Running node <PyNode> config_node
    2021-05-09 23:47:03,167 - entropy - INFO - Running node <PyNode> QUA_node
    2021-05-09 23:47:03,218 - qm - INFO - to simulate a program, use QuantumMachinesManager.simulate(..)
    2021-05-09 23:47:03,246 - qm - INFO - Flags: 
    2021-05-09 23:47:03,248 - qm - INFO - Simulating Qua program
    2021-05-09 23:47:03,502 - entropy - INFO - Finished entropy experiment execution successfully
:::
:::

We can plot the result we obtained from running our experiment. Graph
generation will be the first this the next Entropy version will
automate.

```python
plt.figure()
plt.plot(handle_with_QUA.results.get_results()[1].data)
plt.show()
```




