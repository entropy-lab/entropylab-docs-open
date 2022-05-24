# Parameter Store

Storing and recalling experimental parameters is a crucially important task when running or analyzing experiments. 
The ParameterStore is a tool for storing parameter key-value pairs, very much like a python dictionary. 
Unlike a python dictionary, the paramstore has a mechanism for saving its state and reverting to previous states. 
This is a git-like versioning for your experimental parameters, because it allows you to commit a snapshot. That snapshot 
receives a unique identified, but in addition can be labeled with a non-unique text string. 

## InProcessParamStore
The paramStore interface is defined such that we can eventually have different implementations of varying complexity. 
We may wish to implement a paramstore with some powerful DB like mongoDB or PostgreSQL but for now, we have a simple text
based implementation. This first implementation is the `InProcessParamStore`. You can use it as follows
```python
from entropylab.api.in_process_param_store import InProcessParamStore
params =InProcessParamStore()
```
If no path is supplied to the constructor of this class, no persistence will occur. Once the process dies, the param store dies. 
However, if a path is supplied a persistent text file containing the DB will be generated.  

```python
from entropylab.api.in_process_param_store import InProcessParamStore
params =InProcessParamStore('.')
```

## Storing and retrieving keys

Accessing keys in the paramstore can be done in one of two styles. The first is a dictionary style access:

```
params['a']=1
```

The second is a field (JavaScript-like) style
```
params.a=1
```

The nice thing about the second style is that by implementing this, we are able to provide autocompletion and by simply typing 
the name of the paramstore object followed by a period (.) all key names are suggested (when using a modern IDE).

## Tags
A key can be associated with a tag as follows 

```python
    target.add_tag("tag", "foo")
```

You can then list the keys associated with a specific tag. This allows for the paramstore to have more strucutre. 
Instead of a flat key-value collection, multiple keys can be combined. An example would be to have all keys associated 
with a specific qubit to share a tag e.g. `qubit1`. 

## ParamStore and the entropy pipeline

A param store can be saved as a resource for an entropy graph. This allows usage of the paramstore to set parameters in different 
nodes of the experiment. 
Checkout of a commit that is appropriate to a specific node can be used to set parameters suitable for that node. 



