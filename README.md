<h1 align="center">
  ✨ YouTube Transcript API ✨
</h1>

<p align="center">
  <a href="https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BAENLEW8VUJ6G&source=url">
    <img src="https://img.shields.io/badge/Donate-PayPal-green.svg" alt="Donate">
  </a>
  <a href="https://github.com/jdepoix/youtube-transcript-api/actions">
    <img src="https://github.com/jdepoix/youtube-transcript-api/actions/workflows/ci.yml/badge.svg?branch=master" alt="Build Status">
  </a>
  <a href="https://coveralls.io/github/jdepoix/youtube-transcript-api?branch=master">
    <img src="https://coveralls.io/repos/github/jdepoix/youtube-transcript-api/badge.svg?branch=master" alt="Coverage Status">
  </a>
  <a href="http://opensource.org/licenses/MIT">
    <img src="http://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat" alt="MIT license">
  </a>
  <a href="https://pypi.org/project/youtube-transcript-api/">
    <img src="https://img.shields.io/pypi/v/youtube-transcript-api.svg" alt="Current Version">
  </a>
  <a href="https://pypi.org/project/youtube-transcript-api/">
    <img src="https://img.shields.io/pypi/pyversions/youtube-transcript-api.svg" alt="Supported Python Versions">
  </a>
</p>

<p align="center">
  <b>This is a python API which allows you to retrieve the transcript/subtitles for a given YouTube video. It also works for automatically generated subtitles, supports translating subtitles and it does not require a headless browser, like other selenium based solutions do!</b>
</p>
<p align="center">
 Maintenance of this project is made possible by all the <a href="https://github.com/jdepoix/youtube-transcript-api/graphs/contributors">contributors</a> and <a href="https://github.com/sponsors/jdepoix">sponsors</a>. If you'd like to sponsor this project and have your avatar or company logo appear below <a href="https://github.com/sponsors/jdepoix">click here</a>. 💖
</p>

<p align="center">
  <a href="https://www.searchapi.io">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://www.searchapi.io/press/v1/svg/searchapi_logo_white_h.svg">
      <source media="(prefers-color-scheme: light)" srcset="https://www.searchapi.io/press/v1/svg/searchapi_logo_black_h.svg">
      <img alt="SearchAPI" src="https://www.searchapi.io/press/v1/svg/searchapi_logo_black_h.svg" height="40px" style="vertical-align: middle;">
    </picture>
  </a>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <a href="https://supadata.ai">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://supadata.ai/logo-dark.svg">
      <source media="(prefers-color-scheme: light)" srcset="https://supadata.ai/logo-light.svg">
      <img alt="supadata" src="https://supadata.ai/logo-light.svg" height="40px">
    </picture>
  </a>
</p>

## Install

It is recommended to [install this module by using pip](https://pypi.org/project/youtube-transcript-api/):

```
pip install youtube-transcript-api
```

You can either integrate this module [into an existing application](#api) or just use it via a [CLI](#cli).

## API

The easiest way to get a transcript for a given video is to execute:

```python
from youtube_transcript_api import YouTubeTranscriptApi

ytt_api = YouTubeTranscriptApi()
ytt_api.fetch(video_id)
```

> **Note:** By default, this will try to access the English transcript of the video. If your video has a different 
> language, or you are interested in fetching a transcript in a different language, please read the section below.

> **Note:** Pass in the video ID, NOT the video URL. For a video with the URL `https://www.youtube.com/watch?v=12345` 
> the ID is `12345`.

This will return a `FetchedTranscript` object looking somewhat like this:

```python
FetchedTranscript(
    snippets=[
        FetchedTranscriptSnippet(
            text="Hey there",
            start=0.0,
            duration=1.54,
        ),
        FetchedTranscriptSnippet(
            text="how are you",
            start=1.54,
            duration=4.16,
        ),
        # ...
    ],
    video_id="12345",
    language="English",
    language_code="en",
    is_generated=False,
)
```

This object implements most interfaces of a `List`:

```python
ytt_api = YouTubeTranscriptApi()
fetched_transcript = ytt_api.fetch(video_id)

# is iterable
for snippet in fetched_transcript:
    print(snippet.text)

# indexable
last_snippet = fetched_transcript[-1]

# provides a length
snippet_count = len(fetched_transcript)
```

If you prefer to handle the raw transcript data you can call `fetched_transcript.to_raw_data()`, which will return 
a list of dictionaries:

```python
[
    {
        'text': 'Hey there',
        'start': 0.0,
        'duration': 1.54
    },
    {
        'text': 'how are you',
        'start': 1.54
        'duration': 4.16
    },
    # ...
]
```
### Retrieve different languages

You can add the `languages` param if you want to make sure the transcripts are retrieved in your desired language 
(it defaults to english).

```python
YouTubeTranscriptApi().fetch(video_id, languages=['de', 'en'])
```

It's a list of language codes in a descending priority. In this example it will first try to fetch the german 
transcript (`'de'`) and then fetch the english transcript (`'en'`) if it fails to do so. If you want to find out 
which languages are available first, [have a look at `list_transcripts()`](#list-available-transcripts).

If you only want one language, you still need to format the `languages` argument as a list

```python
YouTubeTranscriptApi().fetch(video_id, languages=['de'])
```

### Preserve formatting

You can also add `preserve_formatting=True` if you'd like to keep HTML formatting elements such as `<i>` (italics) 
and `<b>` (bold).

```python
YouTubeTranscriptApi().fetch(video_ids, languages=['de', 'en'], preserve_formatting=True)
```

### List available transcripts

If you want to list all transcripts which are available for a given video you can call:

```python
ytt_api = YouTubeTranscriptApi()
transcript_list = ytt_api.list_transcripts(video_id)
```

This will return a `TranscriptList` object which is iterable and provides methods to filter the list of transcripts for 
specific languages and types, like:

```python
transcript = transcript_list.find_transcript(['de', 'en'])
```

By default this module always chooses manually created transcripts over automatically created ones, if a transcript in 
the requested language is available both manually created and generated. The `TranscriptList` allows you to bypass this 
default behaviour by searching for specific transcript types:

```python
# filter for manually created transcripts
transcript = transcript_list.find_manually_created_transcript(['de', 'en'])

# or automatically generated ones
transcript = transcript_list.find_generated_transcript(['de', 'en'])
```

The methods `find_generated_transcript`, `find_manually_created_transcript`, `find_transcript` return `Transcript` 
objects. They contain metadata regarding the transcript:

```python
print(
    transcript.video_id,
    transcript.language,
    transcript.language_code,
    # whether it has been manually created or generated by YouTube
    transcript.is_generated,
    # whether this transcript can be translated or not
    transcript.is_translatable,
    # a list of languages the transcript can be translated to
    transcript.translation_languages,
)
```

and provide the method, which allows you to fetch the actual transcript data:

```python
transcript.fetch()
```

This returns a `FetchedTranscript` object, just like `YouTubeTranscriptApi().fetch() does.

### Translate transcript

YouTube has a feature which allows you to automatically translate subtitles. This module also makes it possible to 
access this feature. To do so `Transcript` objects provide a `translate()` method, which returns a new translated 
`Transcript` object:

```python
transcript = transcript_list.find_transcript(['en'])
translated_transcript = transcript.translate('de')
print(translated_transcript.fetch())
```

### By example
```python
from youtube_transcript_api import YouTubeTranscriptApi

ytt_api = YouTubeTranscriptApi()

# retrieve the available transcripts
transcript_list = ytt_api.list('video_id')

# iterate over all available transcripts
for transcript in transcript_list:

    # the Transcript object provides metadata properties
    print(
        transcript.video_id,
        transcript.language,
        transcript.language_code,
        # whether it has been manually created or generated by YouTube
        transcript.is_generated,
        # whether this transcript can be translated or not
        transcript.is_translatable,
        # a list of languages the transcript can be translated to
        transcript.translation_languages,
    )

    # fetch the actual transcript data
    print(transcript.fetch())

    # translating the transcript will return another transcript object
    print(transcript.translate('en').fetch())

# you can also directly filter for the language you are looking for, using the transcript list
transcript = transcript_list.find_transcript(['de', 'en'])  

# or just filter for manually created transcripts  
transcript = transcript_list.find_manually_created_transcript(['de', 'en'])  

# or automatically generated ones  
transcript = transcript_list.find_generated_transcript(['de', 'en'])
```

### Using Formatters
Formatters are meant to be an additional layer of processing of the transcript you pass it. The goal is to convert a
`FetchedTranscript` object into a consistent string of a given "format". Such as a basic text (`.txt`) or even formats 
that have a defined specification such as JSON (`.json`), WebVTT (`.vtt`), SRT (`.srt`), Comma-separated format 
(`.csv`), etc...

The `formatters` submodule provides a few basic formatters, which can be used as is, or extended to your needs:

- JSONFormatter
- PrettyPrintFormatter
- TextFormatter
- WebVTTFormatter
- SRTFormatter

Here is how to import from the `formatters` module.

```python
# the base class to inherit from when creating your own formatter.
from youtube_transcript_api.formatters import Formatter

# some provided subclasses, each outputs a different string format.
from youtube_transcript_api.formatters import JSONFormatter
from youtube_transcript_api.formatters import TextFormatter
from youtube_transcript_api.formatters import WebVTTFormatter
from youtube_transcript_api.formatters import SRTFormatter
```

### Formatter Example
Let's say we wanted to retrieve a transcript and store it to a JSON file. That would look something like this:

```python
# your_custom_script.py

from youtube_transcript_api import YouTubeTranscriptApi
from youtube_transcript_api.formatters import JSONFormatter

ytt_api = YouTubeTranscriptApi()
transcript = ytt_api.fetch(video_id)

formatter = JSONFormatter()

# .format_transcript(transcript) turns the transcript into a JSON string.
json_formatted = formatter.format_transcript(transcript)

# Now we can write it out to a file.
with open('your_filename.json', 'w', encoding='utf-8') as json_file:
    json_file.write(json_formatted)

# Now should have a new JSON file that you can easily read back into Python.
```

**Passing extra keyword arguments**

Since JSONFormatter leverages `json.dumps()` you can also forward keyword arguments into 
`.format_transcript(transcript)` such as making your file output prettier by forwarding the `indent=2` keyword argument.

```python
json_formatted = JSONFormatter().format_transcript(transcript, indent=2)
```

### Custom Formatter Example
You can implement your own formatter class. Just inherit from the `Formatter` base class and ensure you implement the 
`format_transcript(self, transcript: FetchedTranscript, **kwargs) -> str` and 
`format_transcripts(self, transcripts: List[FetchedTranscript], **kwargs) -> str` methods which should ultimately 
return a string when called on your formatter instance.

```python
class MyCustomFormatter(Formatter):
    def format_transcript(self, transcript: FetchedTranscript, **kwargs) -> str:
        # Do your custom work in here, but return a string.
        return 'your processed output data as a string.'

    def format_transcripts(self, transcripts: List[FetchedTranscript], **kwargs) -> str:
        # Do your custom work in here to format a list of transcripts, but return a string.
        return 'your processed output data as a string.'
```

## CLI

Execute the CLI script using the video ids as parameters and the results will be printed out to the command line:  

```  
youtube_transcript_api <first_video_id> <second_video_id> ...  
```  

The CLI also gives you the option to provide a list of preferred languages:  

```  
youtube_transcript_api <first_video_id> <second_video_id> ... --languages de en  
```

You can also specify if you want to exclude automatically generated or manually created subtitles:

```  
youtube_transcript_api <first_video_id> <second_video_id> ... --languages de en --exclude-generated
youtube_transcript_api <first_video_id> <second_video_id> ... --languages de en --exclude-manually-created
```

If you would prefer to write it into a file or pipe it into another application, you can also output the results as 
json using the following line:  

```  
youtube_transcript_api <first_video_id> <second_video_id> ... --languages de en --format json > transcripts.json
```  

Translating transcripts using the CLI is also possible:

```  
youtube_transcript_api <first_video_id> <second_video_id> ... --languages en --translate de
```  

If you are not sure which languages are available for a given video you can call, to list all available transcripts:

```  
youtube_transcript_api --list-transcripts <first_video_id>
```

If a video's ID starts with a hyphen you'll have to mask the hyphen using `\` to prevent the CLI from mistaking it for 
a argument name. For example to get the transcript for the video with the ID `-abc123` run:

```
youtube_transcript_api "\-abc123"
```

[//]: # (## Proxy  )

[//]: # ()
[//]: # (You can specify a https proxy, which will be used during the requests to YouTube:)

[//]: # ()
[//]: # (```python  )

[//]: # (from youtube_transcript_api import YouTubeTranscriptApi  )

[//]: # ()
[//]: # (YouTubeTranscriptApi.get_transcript&#40;video_id, proxies={"https": "https://user:pass@domain:port"}&#41;)

[//]: # (```  )

[//]: # ()
[//]: # (As the `proxies` dict is passed on to the `requests.get&#40;...&#41;` call, it follows the [format used by the requests library]&#40;https://requests.readthedocs.io/en/latest/user/advanced/#proxies&#41;.  )

[//]: # ()
[//]: # (Using the CLI:  )

[//]: # ()
[//]: # (```  )

[//]: # (youtube_transcript_api <first_video_id> <second_video_id> --https-proxy https://user:pass@domain:port)

[//]: # (```)

## Work around IP bans
TODO

## Cookies

Some videos are age restricted, so this module won't be able to access those videos without some sort of 
authentication. To do this, you will need to have access to the desired video in a browser. Then, you will need to 
download that pages cookies into a text file. You can use the Chrome extension 
[Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm?hl=en) and 
select "Netscape" during export, or the Firefox extension [cookies.txt](https://addons.mozilla.org/en-US/firefox/addon/cookies-txt/).

Once you have that, you can use it with the module to access age-restricted videos' captions like so.

```python  
ytt_api = YouTubeTranscriptApi(cookie_path='/path/to/your/cookies.txt')
ytt_api.fetch(video_id)
```

Using the CLI:

```
youtube_transcript_api <first_video_id> <second_video_id> --cookies /path/to/your/cookies.txt
```

## Overwriting request defaults

When initializing a `YouTubeTranscriptApi` object, it will create a `requests.Session` which will be used for all 
HTTP(S) request. This allows for caching cookies when retrieving multiple requests. However, you can optionally pass a
`requests.Session` object into its constructor, if you manually want to share cookies between different instances of 
`YouTubeTranscriptApi`, overwrite defaults, set custom headers, specify SSL certificates, etc.

```python
from requests import Session

http_client = Session()

# set custom header
http_client.headers.update({'x-test': 'true'})

# set path to CA_BUNDLE file
http_client.verify = '/path/to/certfile'

ytt_api = YouTubeTranscriptApi(http_client=session)
ytt_api.fetch(video_id)

# share same Session between two instances of YouTubeTranscriptApi
ytt_api_2 = YouTubeTranscriptApi(http_client=session)
# now shares cookies with ytt_api
ytt_api_2.fetch(video_id)
```

## Warning  

This code uses an undocumented part of the YouTube API, which is called by the YouTube web-client. So there is no 
guarantee that it won't stop working tomorrow, if they change how things work. I will however do my best to make things 
working again as soon as possible if that happens. So if it stops working, let me know!  

## Contributing

To setup the project locally run (requires [poetry](https://python-poetry.org/docs/) to be installed):
```shell
poetry install --with test,dev
```

There's [poe](https://github.com/nat-n/poethepoet?tab=readme-ov-file#quick-start) tasks to run tests, coverage, the 
linter and formatter (you'll need to pass all of those for the build to pass):
```shell
poe test
poe coverage
poe format
poe lint
```

If you just want to make sure that your code passes all the necessary checks to get a green build, you can simply run:
```shell
poe precommit
```

## Donations

If this project makes you happy by reducing your development time, you can make me happy by treating me to a cup of 
coffee, or become a [Sponsor of this project](https://github.com/sponsors/jdepoix) :)  

[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BAENLEW8VUJ6G&source=url)
