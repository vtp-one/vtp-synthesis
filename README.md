# VTP Synthesis

Synthesis is an SOS-System to provide an LLM interface for function calling. 

Use Digital Intelligence to Synthesize Digital Intelligence.


It currently is not very functional since it has been put to the side, but it was the inspiration for SOS-Toolkit. 
One of the goals of SOS-Toolkit is to utilize LLM function calling, and this is a prototype interface for doing that.

SOS-Toolkit grew out of the experimentation done here. 
There where too many problems to get functions to do anything besides call very basic things such as using a web-api or solving simple math problems. 
Useful functions need to be more fundamental to the programming language used, and we needed to basically start from scratch to implement that. 
It made sense to start with the tools that are going to be used to make the tools; which brought about SOS-Toolkit.

Synthesis is in hibernation mode while SOS-Toolkit is built out. 
Once it is more functional, it will be plugged into Synthesis and development here will continue. 
Currently it functions as a testbed for SOS-Toolkit functionality.


# Requirements
 - [SOS-Toolkit](https://github.com/vtp-one/sos-toolkit)

Follow the instructions to get a working SOS-Toolkit virtual environment. 

Installation instructions assume you already have an activated virtual environment. 

All Commands in the environment are prefaced with: `(sos)$`


# Current Release Target
```
v0.0.1
```


# Install
These steps will pull, install, and run VTP-Synthesis. 

You should start from your SOS-Toolkit root directory: `/sos` or it's equivalent in your environment

:warning: WARNING :warning:
```
A SYSTEM CAN RUN ANY TERMINAL COMMAND AND ANY ARBITRARY PYTHON CODE INCLUDING DYNAMICALLY GENERATED CODE
DO NOT INTERACT WITH SYSTEMS THAT YOU DO NOT KNOW, TRUST, OR HAVE THE CAPABILITY TO MANUALLY VERIFY THEIR OPERATION
```

### 1. Pull the vtp-synthesis repository and enter it
```
(sos)$ sos-toolkit pull https://github.com/vtp-one/vtp-synthesis --remote-branch v0.0.1
(sos)$ cd vtp-synthesis
```

### 2. Setup the System
```
(sos)$ sos-toolkit setup
```

This step will also setup various required SOS-Toolkit Services: ollama, chromadb, and mongodb. 
If you don't have these services already available, they will require downloading or building a few local containers.


### 3. Configure the System

(See configuration section below)


### 4. Start Dev Mode

Currently Synthesis only works from dev mode.
```
(sos)$ sos-toolkit dev
```

This action will utilize the SOS-Toolkit Ollama service to install required LLM models from the Ollama Library. 
This is accomplished by using a plugin provided by the service. 
The plugin is loaded as an `on_context_load` hook, which provides a tool: `ollama.download_models`. 
This tool will start the Ollama Service container and then use the Ollama Python API to download the needed models.
Models may take a long time to download depending on your download speeds.
One of the required models is 8.54GB.


### 5. Build the System
```
(sos)$ sos-toolkit build
```
Build will be building containers locally and can require 5-10 minutes depending on what base containers are available.

There is a glitch with the npm installer in the frontend container that occasionally fails the build process with an error: 
```
can't extract file from archive
```

If the build process fails, run the command again.


### 6. Run the System
```
(sos)$ sos-toolkit up
```

This command will output a system status message with the address the Synthesis frontend is available on:
```
{'terminal.print_object': 'SYNTHESIS FRONTEND RUNNING ON: http://0.0.0.0:30000'}
```
Open the address in a web browser to access Synthesis.

Startup can take a long-time depending on the runtime state of the frontend and if it needs to compile resources or not. 
It is currently built using an older version of [Reflex](https://github.com/reflex-dev/reflex), which while functional, does not work very well the way it's being used here. 
It uses an external Docker volume to cache the build artifacts, but this currently needs to use the running container on start-up to do that.
Sometimes this step times out, if that occurs you can restart the system and it "should" work.
```
(sos)$ sos-toolkit restart
```

### 7. Usage
The interface itself is more or less undocumented for now: it works on my machine. 

Once we start integrating SOS-Toolkit, that will be fixed. 
The goal is to provide complete access to SOS-Toolkit commands, actions, systems, services, and tools. 
How that will work is going to require quite a bit more work, but the idea is to be able to create, manage, and use SOS-Systems from an LLM based interface.

In it's current state, it provides a few different methods for interacting and injecting information into an Ollama Chat request with no interaction with SOS-Toolkit at all.

There are three columns, one for User Input, one for System Output, and one for Global State.

User input allows you to input text that is sent using one of the available methods: 
 - ollama-chat: direct requests with the Ollama Python API
 - toolkit-ollama: use the Synthesis Toolkit for proxied requests to the Ollama Python API
 - langchain-chat: use the Synthesis Toolkit for calls through langchain to Ollama

An important note is that the Toolkit mentioned here is not the SOS-Toolkit, but a middleware API server that was named Toolkit because it was intended to provide annotated tools for function calling.
SOS-Toolkit grew out of this Toolkit API server.

The different methods will return chat output to the System Output column.

Both the User Input and System Output columns have a button for clearing their respective history. 
The input and output messages from these columns are bundled with new user prompts and used as chat history with a request.

The System Column is where the experimentation was occurring. It provides four tabs:
 - System Prompt: the system prompt text that is used with every request
 - Plan: Additional information injected into the System Prompt intended to provide planning information for a task
 - Context: Additional information injected into the System Prompt as key-value pairs
 - Session: Rudimentary session functionality for saving/loading state of the User Input, System Output, and Global State values. 
 - Import and Export currently do not work, but you can name a session and save it to MongoDB to load again. The database "should" persist across system restarts.

Function calling via langchain is currently disabled. 
When this was being actively worked on, Ollama function calling wasn't very functional. 
The work required to get around that problem resulted in the initial versions of SOS-Toolkit, and development is there for now. 

In the words of Carl Sagan: "If you wish to make an apple pie from scratch, you must first invent the universe."


### 8. Shutdown the System
```
(sos)$ sos-toolkit down
```


# Configuration
SOS-Synthesis bundles three configuration methods: changing the ollama data directory, changing the ChromaDB data directory, and changing the MongoDB data directory. 
All three options should be set before enabling dev mode `sos-toolkit dev` because the directory targets will be initialized at that time. 
It is possible to change them after or move a pre-existing install to a different location but this isn't recommended.
The default location is:
```
<system_path>/internal/local_data/<data type>
```

### Ollama Data Directory
To change the ollama data directory use:
```
(sos)$ sos-toolkit config --target=ollama_data

Set Ollama Data Directory: 

```
Enter the target directory you want to use to store your models. 
The directory must be valid, and there are currently no safety or validity checks run on the input value.


### ChromaDB Data Directory
To change the ChromaDB data directory:
```
(sos)$ sos-toolkit config --target=chromadb_data

Set ChromaDB Data Directory: 

```
Enter the target directory you want to use to store the ChromaDB database. 
The directory must be valid, and there are currently no safety or validity checks run on the input value.


### MongoDB Data Directory
To change the MongoDB data directory:
```
(sos)$ sos-toolkit config --target=mongodb_data

Set MongoDB Data Directory: 

```
Enter the target directory you want to use to store the Mongo database. 
The directory must be valid, and there are currently no safety or validity checks run on the input value.


# More Information
 - [SOS-Toolkit](https://github.com/vtp-one/sos-toolkit)
 - [SOS-Test_System](https://github.com/vtp-one/sos-test_system)



# License
LICENSE WILL CHANGE

CURRENT LICENSE: VTP-LICENSE-v0.0.1

FOR APPROVED USE: POLYFORM - STRICT
