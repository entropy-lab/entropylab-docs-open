# Entropy with QUA

Entropy is developed to be a general purpose lab manager. It is however
developed primarily by [Quantum Machines](www.quantum-machines.co),
creators of the Quantum Orchestration Platform and the QUA programming
language.

Support of QUA is going to be extended in coming versions, primarily via
Entropy extensions. However, you can already run QUA code by using the
general purpose PyNode. This is shown in the following basic example

```.python
from qm.QuantumMachinesManager import QuantumMachinesManager
from qm.qua import *
from qm import SimulationConfig

#now register the quntum machines mnager and import it to the experiment resources
lab.register_resource('qmm',QuantumMachinesManager)
experiment_resources = ExperimentResources(db)
experiment_resources.import_lab_resource('qmm')
```

After some initial housekeeping, we set up a two-node graph. A first node
to generate the config and pass it on, and a second node to run the QUA
code. We write the operation of these two PyNodes in the two following
functions:

```.python
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

Finally, we can set up the nodes and graph and run our experiment:

```.python
config_node = PyNode("config_node", get_config,output_vars={'config'})
qua_node = PyNode("QUA_node", QUA_node_action,input_vars={'config':config_node.outputs['config']},output_vars={'samples'})
experiment_with_QUA = Graph(experiment_resources, {config_node,qua_node}, "a QUA run")
handle_with_QUA = experiment_with_QUA.run()
```

```
    2021-05-09 23:47:03,095 - qm - INFO - Performing health check
    2021-05-09 23:47:03,103 - qm - INFO - Health check passed
    2021-05-09 23:47:03,122 - entropy - INFO - Running node <PyNode> config_node
    2021-05-09 23:47:03,167 - entropy - INFO - Running node <PyNode> QUA_node
    2021-05-09 23:47:03,218 - qm - INFO - to simulate a program, use QuantumMachinesManager.simulate(..)
    2021-05-09 23:47:03,246 - qm - INFO - Flags: 
    2021-05-09 23:47:03,248 - qm - INFO - Simulating Qua program
    2021-05-09 23:47:03,502 - entropy - INFO - Finished entropy experiment execution successfully
```

We can plot the result we obtained from running our experiment. This is done automatically in the web GUI,
but you can manipulate the data manually.

```python
plt.figure()
plt.plot(handle_with_QUA.results.get_results()[1].data)
plt.show()
```