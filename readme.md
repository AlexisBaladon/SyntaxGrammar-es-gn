# Rule-Based Machine Translation System from Spanish to Guarani

## About
This project develops a rule-based machine translation system focused on translating Spanish to Guarani. Utilizing syntactic transfer, it is capable of generating synthetic bilingual sentence pairs or parsing and translating existing Spanish text into Guarani. This process relies on creating feature grammars, context-free grammars (CFG), and applying weighted sentence generation and translation based on a set of predefined rules and lexicons.

## Getting Started

### Prerequisites
- Python 3.9 or higher
- Access to the project's repository containing grammar definitions, translation rules, and the necessary Python scripts.

### Installation
Clone the repository to your local machine:

`git@github.com:baladon-lucas-pardinas/SyntaxGrammar-es-gn.git`

Navigate to the project directory:

`cd SyntaxGrammar-es-gn`


## Usage Instructions - Creating a new, synthetic parallel corpus

### 1. Create the Feature Grammar
Generate a feature grammar based on the specified subject type. This is the first step in preparing the system for sentence generation or translation. Other configurations, such as input and output file paths, are defined in the config file.

First navigate to `grammar/grammars`, then run:

`python ninth-grammar/create-featgram.py -c ninth-grammar/config.yaml -s <subject-type>`
* `<subject-type>` can be one of the following: `pronoun`, `np`, `adj`, or `all`.

### 2. Create the CFG
Create a context-free grammar (CFG) from the feature grammar. This incorporates word frequency (based on the Jojajovai corpus) to assist in the generation process.

Navigate to `grammar`, then run:

`python cfg-grammar.py grammar-files/g11-<subject-type>/feature-grammar.txt grammar-files/g11-<subject-type>/cfg-grammar.txt -w ../spanish/jojajovai/word_occurrences.csv`


### 3. Weighted Sentence Generation
Generate synthetic Spanish sentences using the CFG (with word frequencies) and feature grammar, annotating each with its syntax tree. `<number-of-sentences>` refers to the number of sentences the CFG will generate, where each one will then be parsed using the feature grammar. If successful, the sentence and its syntax tree(s) are added to the output and trees files respectively. Since only a small minority of sentences generated by the CFG will actually comply with the feature grammar, we suggest specifying a number at least 100 times higher than the number of sentences you expect to obtain. 

Navigate to `grammar`, then run:

`python weighted-generate-sentences.py -n <number-of-sentences> -o grammar-files/g11-<subject-type>/output.txt grammar-files/g11-<subject-type>/cfg-grammar.txt grammar-files/g11-<subject-type>/feature-grammar.txt -t grammar-files/g11-<subject-type>/trees.txt`

### 4. Translation
Translate the generated sentences from Spanish to Guarani, applying syntactic transfer based on predefined rules and lexicon files, and a custom unification algorithm. Remember to place your trees file, generated in the previous step, inside the trees folder and pass its location as the first argument.

Navigate to `grammar/guarani`, then run:

`python -m translate.translate_trees trees/<tree-file>.txt rules.json -o translations/<output-directory>/<output-file>.csv --max-translations <max-translations>`

* `<max-translations>` is the maximum number of translations per Spanish syntax tree, to accommodate multiple interpretations.

## Usage Instructions - Translating an existing monolingual corpus (Ancora)

### 1. Extract Sentences from Ancora

Extract sentences from the Ancora corpus, performing minor preprocessing to prepare them for parsing and translation. This step converts the corpus from a text file into a more manageable CSV format.

Navigate to `grammar/ancora`, then run:

`python ../parsing-subtrees/extract-ancora-sentences.py ../../ancora/ancora-sentences/ancora_all.txt extracted-3.csv`

### 2. Parse Subsentences

Parse subsentences from the extracted sentences using a predefined feature grammar. This step identifies possible syntax trees within each sentence and isolates subsentences that cannot be parsed for further analysis &mdash; the approach taken here is described in the associated paper. The program then stores the syntax trees obtained, the indices for each tree (meaning what sentence it belongs to, and the specific initial and final positions within that sentence), and the subsentences that could not be parsed.

Navigate to `grammar/ancora/v9`, then run:

`python ../../parsing-subtrees/parse-subtrees-3.py --grammar feature-grammar.txt --input ../extracted-3.csv --output trees.txt --indices indices.csv --nonparsed unparsed.txt`

* Make sure you have the feature grammar located in the current directory. This grammar is a more forgiving version of the one used for synthetic corpora generation, as we're working under the assumption that the monolingual corpus is gramatically correct.

### 3. Translate Parsed Trees

Translate the parsed syntax trees from Spanish to Guarani, applying syntactic transfer rules. This step utilizes a custom unification algorithm, generating translated sentence pairs and tracking the original sentence indices.

Navigate to `grammar/guarani`, then run:

`python -m translate.translate_ancora ../ancora/v9/trees.txt rules-ancora.json -o ../ancora/v9/translations.csv --indices ../ancora/v9/indices.csv > ../ancora/v9/untranslated.csv`

* The indices for the successfully translated pairs will be stored in <indices-filename>_out.csv, in the original indices file's directory.

### 4. Embed Translations into Original Sentences

Finally, embed the translated Guarani subsentences back into their original positions within the monolingual corpus' sentences. This final step integrates the translations to produce a partially translated corpus, akin to a sort of articial code-switching scenario.

Navigate to `grammar/ancora`, then run:

`python embed-guarani.py --indices v9/indices_out.csv --extracted extracted-3.csv --translations v9/translations.csv --output v9/output.csv`

