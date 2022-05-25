# QUAM API

## Definition

Quantum Abstract Machine (QuAM)  is an abstraction layer to help describe the possible operations in a lab, author experiments and manage experimental parameters. 

### Terminology

**ParamStore** -  The QuAM manages parameters in a ParamStore, a database where users can save (commit) the values of all parameters associated with the QuAM. A commit operation can be labeled with a name identifying it. The ParamStore state can be reverted to any previously committed state, similar to a GIT project. Unlike a GIT project, forking is not supported. 

**ConfigBuilder** - The [config builder](config_builder.md) is an abstraction above the QUA configuration. It enables describing elements in the QUA configuration in an object oriented way and generates the QUA config dictionary. QuAM uses ConfigBuilder when elements are added to it.

**Components** - Lab instruments 


### Personas

The QuAM API is split for 3 different personas: 👷 **Admin**, **🥼 User** and **🤖 Oracle**.
 
**👷 Admin** - This is the person who sets up the lab. They know about all the instruments in the lab and have connected things together. They builder knows which output port of a voltage source is connected to the qubit flux line. The builder knows which input port of the OPX is connected to the downconversion mixer. The builder probably knows how to write a QUA config manually.

**🥼 User** - This is the person who is running experiments, knows the physics but doesn't want to spend their time on learning how the lab is hooked up (unless where necessary). They want to write experiments where they measure some physical parameter as a function or one or more other physical parameters. For example, they want to measure resonant frequency of a transmon as a function of flux line set point and XY drive power. 

**🤖 Oracle** - This is an external product or tool which needs to interact with the lab and needs a service for discovering which operations are supported and allowed for users. the "user" can also behave as a robot. A robot, unlike a user, will probably not write QUA code directly, but generate it automatically. 

All three personas have access to a ParamStore database to save and retrieve experimental parameters or data.

## Example


### Admin

An administrator initializes and defines the QUA configuration to run the experiments:

```python 
    from qualang_tools.config.components import *
    from qualang_tools.config.primitive_components import *

    class MyAdmin(Admin):
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

    admin = MyAdmin(path=db_file_path)
    admin.set(lo=5e5)
    
    def voltage_setter(val: float):
        ##implement what you want to do with the value
        print(val)

    voltage = admin.parameter("voltage", setter=voltage_setter)
    voltage(12)

    admin.params.save_temp()
    print(admin.config)
```

The administrator class is initialized simply by passing the path to a persistent DB to which all the parameters needed to run the experiment are saved. All objects needed to generate QUA configuration can be added to the `ConfigBuilder` object, config can be generated, once the `Parameter`s are set (or reset) by the administrator. Administrator implements the `prepare_config` method. In this specific example, a `controller` and a `Transmon` are added. The administrator fills in information about, for example, the physical ports that connect the OPX to the IQ mixer. The adminstrator parametrizes the local oscillator frequency of the transmon with the parameter `lo`.


# Oracle

The oracle object is obtained from Admin with the `get_oracle` method
```python
    oracle = admin.get_oracle()
    print(oracle.elements)
    print(oracle.parameters)
    print(oracle.pulses)
```
here we can query the list of elements, pulses, waveforms available in the QUA configuration.

# User

The user object is obtained from Admin with the `get_user` method,
```python
    user = admin.get_user() # requires a local instance of gateway server to initialize User
    print(user.elements.qb)

    with program() as prog:
        play(user.pulses.cw, user.elements.qb)

    res = user.simulate(prog)
```
the user writes the QUA program using the attributes: pulses, elements etc. In an interactive session, the list of available options can be seen by pressing a tab once the user object is initialized. The QUA program can then be eventually run with `simulate` (on simulator) or `execute` (on real hardware) methods.
