# Settings and Configuration

## Project Metadata

Pegasus uses the `PROJECT_METADATA` setting in `settings.py` to generate various things
like page titles and social sharing information.

These should be changed to match the desired values for your own project.

```python
PROJECT_METADATA = {
    'NAME': 'Your Project Name',
    'URL': 'http://www.example.com',
    'DESCRIPTION': 'My Amazing SaaS Application',
    'IMAGE': 'https://upload.wikimedia.org/wikipedia/commons/2/20/PEO-pegasus_black.svg',
    'KEYWORDS': 'SaaS, django',
}
```

Of particular importance is the `URL` setting, which is used to generate absolute 
URLs in your application.

## User Sign Up


## Google Analytics


## Sending Email
