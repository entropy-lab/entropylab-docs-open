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





