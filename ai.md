# LLMs and AI

Pegasus ships with multiple example applications that integrate with LLMs and image generating AI models.
This page summarizes the various options.

## LLMs and Chat

Pegasus ships with a highly flexible and customizable Chat UI for interacting with LLMs.
This section covers how it works and the various supported options.

### Choosing an LLM model

You can choose between two options for your LLM chat: OpenAI and LLM (generic).
The OpenAI option limits you to OpenAI models, but supports streaming and asynchronous API access.
The generic "LLM" option uses the [llm library](https://github.com/simonw/llm) and can be used with many different
models---including local ones. However, it does not yet support streaming responses.

We recommend choosing "OpenAI" unless you know you want to use a different model.

### Configuring OpenAI

If you're using OpenAI, you need to set `OPENAI_API_KEY` in your environment or settings file (`.env` in development).
You can also change the model used by setting `OPENAI_MODEL`, which defualts to `"gpt-3.5-turbo"`.

### Configuring LLM

If you built with generic LLM support, you can configure it by setting the `LLM_MODELS` and `DEFAULT_LLM_MODEL`
values in your `settings.py`. For example:

```python
LLM_MODELS = {
    "gpt4": {"key": env("OPENAI_API_KEY", default="")},
    "claude-3-opus": {"key": env("ANTHROPIC_API_KEY", default="")},
    "Meta-Llama-3-8B-Instruct": {},
}
DEFAULT_LLM_MODEL = "gpt4"
```

The chat UI will use whatever is set in `DEFAULT_LLM_MODEL` out-of-the-box, but you can quickly change it
to another model to try different options.

Any models that you add will need to be installed as [llm plugins](https://llm.datasette.io/en/stable/plugins/index.html).
You can do this by putting them in your requirements files, [as outlined here](./python.md#adding-or-removing-a-package).
For example, to use Claude 3 you need to add the [`llm-claude-3` plugin](https://github.com/simonw/llm-claude-3),
and to use local models like Llama 3, you need [`llm-gpt4all`](https://github.com/simonw/llm-gpt4all).

For further reading, see the documentation of the [llm Python API](https://llm.datasette.io/en/stable/python-api.html),
and [llm generally](https://llm.datasette.io/en/stable/index.html).

### The Chat UI

The Chat UI has multiple different implementations, and the one that is used for your project will be determined by your build configuration.

If you build with asynchronous functionality enabled *and* htmx then it will use a websocket-based Chat UI.
This Chat UI supports streaming responses for OpenAI models, and is the recommended option.

If you build without asynchronous functionality enabled, the chat UI will instead use Celery and polling.
The React version of the chat UI also uses Celery and polling.
This means that [Celery must be running](./celery.md) to get responses from the LLM.

## Image Models

Pegasus also includes an example app for generating images with multiple different models,
including [Dall-E-2](https://openai.com/index/dall-e-2) and [Dall-E-3](https://openai.com/index/dall-e-3)
and [Stability AI](https://stability.ai/) (Stable Diffusion 3).

To use the Dall-E models, you must set `OPENAI_API_KEY` in your environment,
and to use Stability AI, you must set `STABILITY_AI_API_KEY`.

You can choose which model you want to use from the dropdown on the image generation page.
