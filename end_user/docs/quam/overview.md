# QUAM API

## Definition

Quantum Abstract Machine (QuAM)  is an abstraction layer to help describe the possible operations in a lab, author experiments and manage experimental parameters. 

### Terminology

**ParamStore** -  The QuAM manages parameters in a [ParamStore](../paramstore/overview.md), a database where users can save (commit) the values of all parameters associated with the QuAM. A commit operation can be labeled with a name identifying it. The ParamStore state can be reverted to any previously committed state, similar to a GIT project. Unlike a GIT project, forking is not supported.

**ConfigBuilder** - The [config builder](config_builder.md) is an abstraction above the QUA configuration. It enables describing elements in the QUA configuration in an object oriented way and generates the QUA config dictionary. QuAM uses ConfigBuilder when elements are added to it.

**Components** - Lab instruments 


### Personas

The QuAM API is split between 2 different personas: 👷 **QuAMManager**, **🥼 QuAM**.
 
**👷 QuAMManager** - This is used by the person who sets up the lab. They know about all the instruments in the lab and have connected things together. The builder knows which output port of a voltage source is connected to the qubit flux line. The builder knows which input port of the OPX is connected to the downconversion mixer. The builder probably knows how to write a QUA config manually.

**🥼 QuAM** - This is used by the person who is running experiments, knows the physics but doesn't want to spend their time on learning how the lab is hooked up (unless where necessary). They want to write experiments where they measure some physical parameters as a function of one or more other physical parameters. For example, they want to measure resonant frequency of a transmon as a function of flux line set point and XY drive power.

Both personas have access to a ParamStore database to save and retrieve experimental parameters or data.

## Example


### QuAMManager

A lab manager initializes and defines the QUA configuration to run the experiments:

```python 
    from qualang_tools.config.components import *
    from qualang_tools.config.primitive_components import *
    from entropylab.quam.core import QuAMManager

    class MyManager(QuAMManager):
        def __init__(self, path:str):
            super().__init__(path=path)

        def prepare_config(self, cb: ConfigBuilder):
            con1 = Controller("con1")
            cb.add(con1)
            qb = Transmon(
                "qb",
                I=con1.analog_output(1),
                Q=con1.analog_output(2),
                intermediate_frequency=1e7,
            )
            cb.add(qb)
            qb.lo_frequency = self.parameter("lo")
            qb.add(
                ControlPulse(
                    "cw",
                    [
                        ConstantWaveform("const_wf", 0.1),
                        ConstantWaveform("zero", 0),
                    ],
                    20,
                )
            )

            
    db_file_path = "path to entropy db"

    manager = MyManager(path=db_file_path)
    manager.param_store["lo"] = 5e5
    
    def voltage_setter(val: float):
        ##implement what you want to do with the value
        print(val)

    voltage = manager.parameter("voltage", setter=voltage_setter)
    voltage(12)

    manager.param_store.save_temp()
    print(manager.generate_config())
```

The manager class is initialized simply by passing the path to a persistent DB to which all the parameters needed to run the experiment are saved. All objects needed to generate QUA configuration can be added to the `ConfigBuilder` object, config can be generated, once the `Parameter`s are set (or reset) by the administrator. Administrator implements the `prepare_config` method. In this specific example, a `controller` and a `Transmon` are added. The administrator fills in information about, for example, the physical ports that connect the OPX to the IQ mixer. The adminstrator parametrizes the local oscillator frequency of the transmon with the parameter `lo`.


# QuAM

The QuAM object is obtained from `QuAMManager` with the `open_quam` method,
```python
    quam = manager.open_quam() # requires a local instance of gateway server to initialize User
    print(quam.elements.qb)

    with program() as prog:
        play(quam.pulses.cw, quam.elements.qb)

    config = quam.config
    res = quam.qmm.open_qm(quam.config).simulate(prog, simulate=SimulationConfig(duration=1000))
```
quam can be used to write the QUA program using the attributes: pulses, elements etc. In an interactive session, the list of available options can be seen by pressing a tab once the quam object is initialized. 


# Combining QuAM with pipeline

QuAM is intended to be used as a module that the lab manager can version and distribute to the users. Users can then import from this module (for e.g. quam object) and use it in the PyNode programs.

## Example

Let us consider a sample calibration graph involving two nodes: 1) play a constant waveform and update a parameter in the config dictiornary and 2) print the latest config. The lab manager implements a sample module (`module.py`) as follows

```python
    from entropylab.quam.core import QuAMManager
    from qualang_tools.config import ConfigBuilder
    from qualang_tools.config.components import *

    class MyManager(QuAMManager):
        def __init__(self, path: str):
            super().__init__(path=path)
        
        def prepare_config(self, cb: ConfigBuilder):
            con1 = Controller('con1')
            cb.add(con1)
            
            qb = Transmon(
                'qb',
                I=con1.analog_output(1),
                Q=con1.analog_output(2),
                intermediate_frequency=1e7
            )
            cb.add(qb)
            
            qb.lo_frequency = self.parameter('lo')
            qb.add(
                ControlPulse(
                    'cw', 
                    [
                        ConstantWaveform('const_wf', 0.1),
                        ConstantWaveform('zero', 0.0),
                    ],
                    20,
                )
            )
    db_file_path = 'params.db'

    manager = MyManager(path=db_file_path)
    manager.param_store["lo"] = 5e5
    manager.param_store.save_temp()

    def voltage_setter(val: float):
        # implement what you want to do with the value
        print(val)
        
    voltage = manager.parameter('voltage', setter=voltage_setter)
    voltage(12)

    print(manager.parameter("voltage"))
    print(manager.parameter("lo"))

    quam = manager.open_quam()
```

The user can then import `quam` object from the module and use it in any regular `PyNode` as follows,


```python
    from entropylab import *
    from qm.qua import *
    from qm.simulate import SimulationConfig

    from module import quam

    def play_cw():

        quam.param_store.load_temp()
        print(quam.elements.qb)
        config = quam.config

        with program() as prog:
            play(quam.pulses.cw, quam.elements.qb)

        res = quam.qmm.open_qm(config).simulate(prog, simulate=SimulationConfig(duration=1000))
        quam.config["elements"]["qb"]["intermediate_frequency"] = 2e5

        return {'result':1}


    def print_config():

        print(quam.config)

        return {'result':1}


    node1 = PyNode(label="PlayCW", program=play_cw, output_vars={'result'})

    node2 = PyNode(label="PrintConfig", program=print_config, 
                input_vars = {'result':node1.outputs['result']},
                output_vars={'result'})

    experiment = Graph(resources=None, graph={node1, node2})
    handle = experiment.run()
```