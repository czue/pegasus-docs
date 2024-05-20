# LLMs and AI

Pegasus ships with multiple example applications that integrate with LLMs and image generating AI models.
This page summarizes the various options.

## LLMs and Chat

Pegasus comes with an optional Chat UI for interacting with LLMs.
This section covers how it works and the various supported options.

### Choosing an LLM model

You can choose between two options for your LLM chat: OpenAI and LLM (generic).
The OpenAI option limits you to OpenAI models, but supports streaming and asynchronous API access.
The generic "LLM" option uses the [litellm library](https://docs.litellm.ai/docs/) and can be used with many different
models---including local ones.

We recommend choosing "OpenAI" unless you know you want to use a different model.

### Configuring OpenAI

If you're using OpenAI, you need to set `OPENAI_API_KEY` in your environment or settings file (`.env` in development).
You can also change the model used by setting `OPENAI_MODEL`, which defaults to `"gpt-3.5-turbo"`.

See [this page](https://help.openai.com/en/articles/4936850-where-do-i-find-my-secret-api-key) for help
finding your OpenAI API key.

### Configuring LLM

If you built with generic LLM support, you can configure it by setting the `LLM_MODELS` and `DEFAULT_LLM_MODEL`
values in your `settings.py`. For example:

```python
LLM_MODELS = {
    "gpt-3.5-turbo": {"api_key": env("OPENAI_API_KEY", default="")},
    "gpt4": {"api_key": env("OPENAI_API_KEY", default="")},
    "claude-3-opus-20240229": {"api_key": env("ANTHROPIC_API_KEY", default="")},
    "ollama_chat/llama3": {"api_base": env("OLLAMA_API_BASE", default="http://localhost:11434")},  # requires a running ollama instance
}
DEFAULT_LLM_MODEL = env("DEFAULT_LLM_MODEL", default="gpt4")
```

The chat UI will use whatever is set in `DEFAULT_LLM_MODEL` out-of-the-box, but you can quickly change it
to another model to try different options.

For further reading, see the documentation of the [litellm Python API](https://docs.litellm.ai/docs/completion),
and [litellm providers](https://docs.litellm.ai/docs/providers).

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

Pegasus also includes an optional example app for generating images with multiple different models,
including [Dall-E-2](https://openai.com/index/dall-e-2) and [Dall-E-3](https://openai.com/index/dall-e-3)
and [Stability AI](https://stability.ai/) (Stable Diffusion 3).

To use the Dall-E models, you must set `OPENAI_API_KEY` in your environment,
and to use Stability AI, you must set `STABILITY_AI_API_KEY`.

You can choose which model you want to use from the dropdown on the image generation page.
