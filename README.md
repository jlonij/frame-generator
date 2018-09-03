# Frame Generator

Tool for extracting topics, keywords and their co-occurence patterns (forming so-called frames) from a Dutch corpus.

## Background

The Frame Generator was created in collaboration with prof. dr. Joris van Eijnatten (UU) during his KB Digital Humanities Fellowship in 2016. Its aim is to meaningfully reducing a set of texts to word patterns that cut across the distributions generated by topic modelling, thus providing additional insight into the content of the data set.

## Features
 
- Generate topics with the [Mallet](http://mallet.cs.umass.edu) or [Gensim](https://radimrehurek.com/gensim/) topic modelling library
- Extract a single, ranked list of keywords based on either topics or tf-idf scores
- Find co-occurrence patterns for the keywords in the texts from which they were originally extracted
- Optionally lemmatize and pos-tag the input texts with NLP suite [Frog](https://languagemachines.github.io/frog/) and restrict the keywords and collocates to specific part-of-speech tags
- Optionally correct OCR errors and spelling variations with user-provided lists of regular expressions
- Access and reuse the results of each processing stage as comma-separated values files
- Use from the command line, as a Python library or web application

## Requirements

- Python 2.7
- Mallet 2.0.7

## Installation

Clone or download the GitHub repository:

```
$ git clone https://github.com/jlonij/frame-generator
```

Install the required Python packages:

```
$ cd frame-generator
$ pip install -r requirements.txt
```
Install Frog + Dependencies:
See here: https://github.com/LanguageMachines/frog

Start frog: 
```
	$ frog -S 4096
```
Install Frog-wrapper:
Place the directory frogger in your www-root (/var/www/frogger/),
See if the frog-wrapper can contact your local Frog service:

```
$ python frog.py
```

This should ouput some test-text if all went well, now try the wrapper with browser:
```
$ curl -s http://localhost/frogger/?text="Dit is een test"
```
This sould return some text-text.

## Usage

Basic command line execution with the default values for all options:

```
$ cd frame-generator
$ ./generator.py
```

This will generate keywords and frames for the sample documents provided and print the results:

```
Keywords and frames generated:
(1) zotheid/N [0.153417875946]
prijzen/WW (1.81959197914), verkondigen/WW (0.778800783071), kwaden/ADJ (0.472366552741), gepast/ADJ (0.367879441171), staan/WW (0.28650479686), aanstonds/ADJ (0.28650479686)
(2) god/N [0.0681857686941]
goddelijk/ADJ (0.606530659713), stellen/WW (0.606530659713), vervroolijk/ADJ (0.472366552741), uitzonderen/WW (0.367879441171)
(3) mensch/N [0.0271176554179]
vervroolijk/ADJ (0.778800783071), toeschrijven/WW (0.778800783071), gewoonlijk/ADJ (0.606530659713), verjagen/WW (0.367879441171), goddelijk/ADJ (0.367879441171), spreken/WW (0.28650479686)
...
```

## Command line interface

Input files are expected to be either utf-8 or iso-8859-1 encoded and have to be placed in the appropriate `frame-generator/input` subdirectories: 

- `docs` contains plain text files with `.txt` extension of the documents to be processed.
- `stop` contains optional stop word lists to be applied when creating the vocabulary. The stop word lists should be plain text files with `.txt` extension in which each word occupies a single line.
- `regex` contains optional lists of regular expressions to be replaced in the input documents. These lists should have a `.tsv` extension and consist of two tab-separated columns, the first containing the regular expression and the second its intended replacement.

The Frame Generator command line interface accepts a number of options to control the process:

- `--gtype`: the type of results to be generated. The user can choose between generating `topics`, `keywords` or `frames`. The default value is `frames`.
- `--dlen`: the number of sentences the subdocuments used as units of analysis are to contain. Default value is `0`, in which case the original, unsplit documents will be used.
- `--nopos`: when this option is entered (no value required) the part-of-speech tagging functionality of the Frame Generator is turned off. This saves a lot of processing time and allows the generator to run offline.
- `--tcount`:  the number of topics to be generated. Default value is `10`.
- `--tsize`: the number of words to be contained in each topic. Default value is `10`.
- `--mallet`: full path to the Mallet executable; if not provided, Gensim's LDA implementation will be used to generate topics.
- `--kmodel`: model to be used for scoring keywords, either `lda` or `tf-idf`. By default `lda` is used, meaning keywords are extracted on the basis of a topic model.
- `--kcount`: the number of keywords to be generated. Default value is `10`.
- `--ktags`: the part-of-speech tags to be included in the keyword list separated by spaces, e.g. `ADJ N WW`. 
- `--wdir`: the direction, `left` or `right`, of the keyword in which frame words are searched for. When omitted both directions are taken into account.
- `--wsize`: the maximum word distance of a frame word to the keyword. Default value is `5`.
- `--fsize`: the maximum number of frame words to be generated with each keyword. Default value is `10`.
- `--ftags`: the part-of-speech tags to be included in the keyword list, e.g. `ADJ N WW`.

Values accepted as part-of-speech tags with the `--ktags` and `--ftags` options are the following main tags from the [CGN tag set](http://lands.let.ru.nl/cgn/doc_Dutch/topics/version_1.0/annot/pos_tagging/tg_prot.pdf):

- `ADJ` Adjective
- `BW` Adverb
- `N` Noun
- `SPEC` Names and unknown
- `TSW` Interjection
- `TW` Numerator
- `VNW` Pronoun
- `WW` Verb
	
## Application programming interface

The Frame Generator can be used from another Python script or the Python interpreter by calling the `generate()` function in the `generator.py` script:

```
>>> import generator
>>> _, keyword_list, frame_list = generator.generate()
>>> keyword_list.print_keywords()
Keywords generated:
(1) zotheid/N [0.169311243818]
(2) god/N [0.0725594158983]
(3) mensch/N [0.034531287966]
...
```

The arguments of the function correspond to the command line options listed above, the function signature being:

```
generate(gtype='frames', dlen=0, pos=True, tcount=10, tsize=10, mallet=None, kmodel='lda', kcount=10,
	ktags=[], wdir=None, wsize=5, fsize=10, ftags=[], input_dir='input', output_dir='output')
```

## Web application

The Frame generator can also be run as a simple Bottle web application accepting post requests:

```
$ ./web.py
```

By default the service is started at `http://localhost:8091/`.

## Demo

An online demo providing a graphical user interface to the Frame Generator’s main functionality and a basic visualization of the results is available at [http://www.kbresearch.nl/frames/](http://www.kbresearch.nl/frames/). The source code of the demo can be found [here](https://github.com/jlonij/frame-generator-gui).
