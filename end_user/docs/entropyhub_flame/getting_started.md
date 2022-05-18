# Getting started

Entropy has a web-browser based interface, and includes the whole stack you need
for writing, executing, managing and storing your experimental workflows that
are setup in default installation with a few commands. Advanced users on the
other hand might be interested in advanced modular installations that are also
supported.

## Simple installation

This is for method that installs all components of EntropyHub on a single-machine.
Choose installation depending if you are using EntropyHub or 
developing it:

=== "User of EntropyHab and Flame"
    In the folder where you want to setup entropyhub and project files do from terminal

    ```
    pip install entropyhub
    python -m entropyhub
    ```

=== "Developer/Contributor"
    ```bash
    git clone --depth 1 https://github.com/entropy-lab/entropy-hub.git
    cd entropy-hub
    pip install --upgrade --force-reinstall ./entropylab ./entropyhub
    python -m entropyhub --build
    ```

When starting the first time, this will take some time since the system is being
downloaded and initialised. Once the terminal information stops changing, open
`http://localhost` in your web-browser.

To switch off entropy press ++ctrl+c++ in terminal once, and wait for the system
to switch off. To run again EntropyHub, it is enough to run `python -m entropyhub`
in the same folder again.

## Running an example

Open in the web browser `http://localhost`. Open Experiments panel. Code editor
will open. Left-Click on `main_pc` on the side menu and
select `Open remote SSH terminal` to open terminal.

![How to open terminal](../assets/open_terminal.png)

In terminal window type

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

