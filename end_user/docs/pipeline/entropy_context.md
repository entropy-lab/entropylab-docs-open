# The Entropy Context

By "resource management" what is meant is more than just the set of all
instruments you use in the lab. A *resource* in this context can be many
things. For example, it could be some calculation that is run on a
remote compute resource or, indeed, it can be your DC voltage source.
Sharing these resources between nodes in Entropy is easy. The nodes in
an Entropy graph have access to resources automatically, and there is a
centralized store in which you can register the resources that are
always present in the lab or temporary resources which only exist for a
specific run of an experiment.

## Lab and Experiment Resources

The mental image you need to keep to understand resources management in
Entropy is as follows: Resources belong to a lab and persist in the DB.
Every time we set up an experiment, we can use a lab resource by
importing it.

As an example, we first build a class representing some device, say an
oscilloscope.

```.python
class my_scope:
    def __init__(self,name, ip):
        self.name=name
        self.ip=ip      
```

Typically, you would start from an existing driver you already have, or a
contributed one. 

## Registering resources

We can **register** our resources (our oscilloscope) to the lab. The
act of registration adds a resource to the ones available to us in the
lab.

There are many options here we will go into in later tutorials but for
now, we can point out just a few. The `register_resource` method takes a
`resource_class` argument, note that this is the **name** of the class
and not an instance. There are then `args` and `kwargs` we pass, which
are the *positional* arguments passed to the constructor of our resource
and the *keyword* (named) arguments we pass. We don't have to use both,
we can use just one or the other or (as in the example below) a
combination of both.

```.python
class my_scope:
    def __init__(self,name, ip):
        self.name=name
        self.ip=ip      
    def identify(self):
        print(f'This is scope:{self.name}')

lab.register_resource(name="my_scope", resource_class=my_scope,args=["gals scope"],kwargs={'ip':0})

```

If you now try to run the cell from above again, this will fail. Why is
that? Because you are trying to register an existing device to the DB.
The `LabResources` API supports adding and removing resources from the
lab and this failing behaviour is intentional. It is meant to help
to prevent an accidental overwrite of a device.

## Running an experiment with a resource

Let's now run an experiment the uses a registered resource. We need to
start an `experiment_resource` instance (and specify the db to which it
writes), and then import the resource we want to use from the lab.

```.python
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

```
    2021-05-09 22:49:55,134 - entropy - INFO - Running node <PyNode> first_node
    This is scope:gals scope
    2021-05-09 22:49:55,205 - entropy - INFO - Finished entropy experiment execution successfully
```

As you can see the `identify` method was called as expected.
