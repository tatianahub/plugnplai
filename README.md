# 🎸 Plug and Plai

Plug and Plai is an open source library aiming to simplify the integration of AI plugins into open-source language models (LLMs).

It provides utility functions to get a list of active plugins from [plugnplai.com](https://plugnplai.com/) directory, get plugin manifests, and extract OpenAPI specifications and load plugins.

## Installation

You can install Plug and Plai using pip:

```python
pip install plugnplai
```

## Quick Start Example

**Plugins Retrieval API - https://www.plugnplai.com/_functions/retrieve?text={user_message_here}:** [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edreisMD/plugnplai/blob/main/examples/retrieve_plugins_api.ipynb)

**Use an Specific Plugin - Step by Step:** [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edreisMD/plugnplai/blob/main/examples/plugins_step_by_step.ipynb)


## More Examples

**Generate Prompt with Plugins Description:** [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edreisMD/plugnplai/blob/main/examples/create_prompt_plugins.ipynb)

**Plugins Retrieval:** [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edreisMD/plugnplai/blob/main/examples/plugin_retriever_with_langchain_agent.ipynb)

**OAuth Plugins** Example for setting up plugins with OAuth: [examples/oauth_example](https://github.com/edreisMD/plugnplai/tree/master/examples/oauth_example)

## Usage

### Get a list of plugins

- `urls = get_plugins()`: Get a list of available plugins from a [plugins repository](https://www.plugnplai.com/).

- `urls = get_plugins(filter = 'ChatGPT', category='dev')`: Use 'filter' or 'category' variables to query specific plugins

Example:

```python
import plugnplai

# Get all plugins from plugnplai.com
urls = plugnplai.get_plugins()

#  Get ChatGPT plugins - only ChatGPT verified plugins
urls = plugnplai.get_plugins(filter = 'ChatGPT')

#  Get working plugins - only tested plugins (in progress)
urls = plugnplai.get_plugins(filter = 'working')

#  Get plugins by category - only tested plugins (in progress)
urls = plugnplai.get_plugins(category = 'travel')

#  Get the names list of categories
urls = plugnplai.get_category_names()
```

### Utility Functions

Help to load the plugins manifest and OpenAPI specification

- `manifest = get_plugin_manifest(url)`: Get the AI plugin manifest from the specified plugin URL.
- `specUrl = get_openapi_url(url, manifest)`: Get the OpenAPI URL from the plugin manifest.
- `spec = get_openapi_spec(openapi_url)`: Get the OpenAPI specification from the specified OpenAPI URL.
- `manifest, spec = spec_from_url(url)`: Returns the Manifest and OpenAPI specification from the plugin URL.

Example:

```python
import plugnplai

# Get the Manifest and the OpenAPI specification from the plugin URL
manifest, openapi_spec = plugnplai.spec_from_url(urls[0])
```

### Load Plugins
**Example:** [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edreisMD/plugnplai/blob/main/examples/plugins_step_by_step.ipynb)

```python
from plugnplai import Plugins

###### ACTIVATE A MAX OF 3 PLUGINS ######
# Context length limits the number of plugins you can activate,
# you need to make sure the prompt fits in your context lenght,
# still leaving space for the user message

# Initialize 'Plugins' by passing a list of urls, this function will
# load the plugins and build a default description to be used as prefix prompt
plugins = Plugins.install_and_activate(urls)

#  Print the deafult prompt for the activated plugins
print(plugins.prompt)

#  Print the number of tokens of the prefix prompt
print(plugins.tokens)
```

Example on installing (loading) all plugins, and activating a few later:

```python
from plugnplai import Plugins

# If you just want to load the plugins, but activate only
# some of them later use Plugins(urls) instead
plugins = Plugins(urls)

# Print the names of installed plugins
print(plugins.list_installed)

# Activate the plugins you want
plugins.activate(name1)
plugins.activate(name2)
plugins.activate(name3)

# Deactivate the last plugin
plugins.deactivate(name3)
```

### Prompt and Tokens Counting

The `plugins.prompt` attribute contains a prompt with descriptions of the active plugins.
The `plugins.tokens` attribute contains the number of tokens in the prompt.

For example:
```python
plugins = Plugins.install_and_activate(urls)
print(plugins.prompt)
print(plugins.tokens)
```
This will print the prompt with plugin descriptions and the number of tokens.


### Parse LLM Response for API Tag

The `parse_llm_response()` function parses an LLM response searching for API calls. It looks for the `<API>` pattern defined in the `plugins.prompt` and extracts the plugin name, operation ID, and parameters.


### Call API

The `call_api()` function calls an operation in an active plugin. It takes the plugin name, operation ID, and parameters extracted by `parse_llm_response()` and makes a request to the plugin API. 

**Plugin Authentication:** Plugins with Oauth, user or service level authentication pass api_key(string) as parameter. For oauth, use the access_token as the api_key parameter. For more detail about oauth authentication please see [oauth_example/run_plugin_with_oauth.py](https://github.com/edreisMD/plugnplai/tree/master/examples/oauth_example/run_plugin_with_auth.py) and [oauth_example/oauth_server.py](https://github.com/edreisMD/plugnplai/tree/master/examples/oauth_example/oauth_server.py) files.


### Apply Plugins

The `@plugins.apply_plugins` decorator can be used to easily apply active plugins to an LLM function. 

This method is suboptimal for GPT chat models because it doesn't make use of the different available roles on the chat api (system, user, or assistant roles). But it may be useful for other LLM providers or open-source models.

1. Import the Plugins class and decorator:

```python
from plugnplai import Plugins, plugins.apply_plugins
```

2. Define your LLM function, that necessarily takes a string (the user input) as the first argument and returns a string (the response):

```python
@plugins.apply_plugins
def call_llm(user_input):
  ...
  return response
```

3. The decorator will handle the following:

- Prepending the prompt (with plugin descriptions) to the user input 
- Checking the LLM response for API calls (the <API>...</API> pattern)
- Calling the specified plugins 
- Summarizing the API calls in the LLM response
- Calling the LLM function again with the summary to get a final response

4. If no API calls are detected, the original LLM response is returned.

To more details on the implementation of these steps, see example "Step by Step": [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edreisMD/plugnplai/blob/main/examples/plugins_step_by_step.ipynb)

### Plugins Retrieval

**PlugnPlai Retrieval API - https://www.plugnplai.com/_functions/retrieve?text={user_message_here} :** [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edreisMD/plugnplai/blob/main/examples/retrieve_plugins_api.ipynb)

**Build our own Plugins Vector Database:** [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edreisMD/plugnplai/blob/main/examples/plugin_retriever_with_langchain_agent.ipynb)


```python
from plugnplai import PluginRetriever

# Initialize the plugins retriever vector database and index the plugins descriptions.
# Loading the plugins from plugnplai.com directory
plugin_retriever = PluginRetriever.from_directory()

#  Retrieve the names of the plugins given a user's message
plugin_retriever.retrieve_names("what shirts can i buy?")
```


## Contributing

Plug and Plai is an open source library, and we welcome contributions from the entire community. If you're interested in contributing to the project, please feel free to fork, submit pull requests, report issues, or suggest new features.

#### To dos
- [x] [Load] Define a default object to read plugins - use LangChain standard? (for now using only manifest and specs jsons)  
- [ ] [Load] Fix breaking on reading certain plugins specs
- [x] [Load] Accept different specs methods and versions (param, query, body)  
- [x] [Prompt] Build a utility function to return a default prompts for a plugin  
- [x] [Prompt] Fix prompt building for body (e.g. "Speak")   
- [x] [Prompt] Build a utility function to return a default prompts for a set of plugins  
- [x] [Prompt] Build a utility function to count tokens of the plugins prompt  
- [ ] [Prompt] Use the prompt tokens number to short or expand a plugin prompt, use LLM to summarize the prefix prompt  
- [x] [CallAPI] Build a function to call API given a dictionary of parameters  
- [x] [CallAPI] Add example for calling API  
- [ ] [Embeddings] Add filter option (e.g. "working", "ChatGPT") to "PluginRetriever.from_directory()"  
- [x] [Docs] Add Sphynx docs  
- [ ] [Verification] Build automated tests to verify new plugins  
- [x] [Verification] Build automated monitoring for working plugins for plugnplai
- [ ] [Verification] Build automated monitoring for working plugins for langchain
- [ ] [Website] Build an open-source website  
- [x] [Authentication] Add support for OAuth, user and service level authentication
- [x] [Functions] Automaticly create OpenAI function dictionary for plugins (plugins.functions)
- [ ] [Model] Create a dataset for finetuning open-source models to call Plugins / APIs. Use the descriptions on plugins.prompt (eventually plugins.functions) as input
- [ ] [Model] Finetune an open-source model to call Plugins / APIs

#### Project Roadmap
1. Build auxiliary functions that helps everyone to use plugins as defined by [OpenAI](https://platform.openai.com/docs/plugins/introduction)  
2. Build in compatibility with different open-source formats (e.g. LangChain, BabyAGI, etc)  
3. Find a best prompt format for plugins, optimizing for token number and description completness  
4. Help with authentication  
5. Build a dataset to finetune open-source models to call plugins  
6. Finetune an open-source model to call plugins
7. Etc.

## Links
- Plugins directory: [https://plugnplai.com/](https://plugnplai.com/)  
- API reference: [https://plugnplai.github.io/](https://plugnplai.github.io/)
