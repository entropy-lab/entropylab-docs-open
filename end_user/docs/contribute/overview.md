# Entropy in computer science language

End-user documentation is for users of Entropy - i.e. scientists. It aims to
explain capabilities in general language, staying away from domain specific
jargon. All distributed computing language should be abstracted away and
interface should be made for general scientists, without requiring them to learn
a lengthy glossary of terms.

However, since you are now looking at a guide for (software) contributors, we
will be using all computer science terminology to ease developer onboarding.
Entropy is

- **actor framework** for **distributed computing** Importantly, it is not just
  data-pipeline workflow (although you can write one if you want), or
  asynchronous framework (although you can use it as such). Instead all actors
  run in parallel, in so much as the operating system and a number of cores on
  underlying hardware enables real parallelism, and does not do just
  asynchronous process interleaving.
- can be seen as sort of **distributed operating system** from one perspective
  (one workflow runs on heterogeneous hardware), that acts as virtualisation
  level on top of the existing operating system (check Plan9 and Inferno)
- actors use **asynchronous, broker-less communication** done with ZeroMQ (0MQ)
  library, with MessagePack used for schema-less communication serialisation.
  ZeroMQ is broker-less communication that allows for all sorts of communication
  topologies (not only client-server), and allows use of different underlying
  communication mechanisms (in process shared memory, inter-process via
  sockets/pipes/TCP, broadcasts via UDP and more).
- Entropy aims to be **language-agnostic** in terms of the language for writing
  actors. NodeIO library uses just ZeroMQ and MessagePack that have
  implementations for a larga number of programming languages
- Entropy **does not do isolation of individual processes** and this is by
  design: since these processes are controlling experimental hardware, that is
  other devices connected to the machine, we need to **provide full bare-metal**
  access to the machine. Containers and isolation can be built on top of the
  Entropy framework if needed, e.g. by running some nodes as containers. Lack of
  isolation also allows easy in-memory passing of large data sets within the
  same machine to be done by users if needed for advanced use.

Now in addition to authoring the nodes and workflows, you might be interested in
technical details of the workflow execution.

- Entropy executor (Flame) starts individual nodes from the command line,
  passing required information about the workflow as two command-line arguments.
  This works on all types of hardwares and OS, and should
