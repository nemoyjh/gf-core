
The GF software system implements the GF programming language. Its
components are

 * the *compiler*,
translating `.gf` source files to `.gfo` object files, to
`.pgf` run-time grammars, and to various other formats
 * the *run-time system*,
performing parsing, generation, translation and other functions with
`.pgf` grammars
 * the *command interpreter*, also known as the *GF shell*,
executing user commands by calling the compiler and the run-time system

This page describes the commands of the GF shell,
as well as the use of the compiler in batch mode.

## The GF shell 

The GF shell is invoked by the command `gf`, which takes arguments and
options according to the following syntax:

      gf (OPTION | FLAG)* FILE*

The shell maintains a  *state*, to which belong

 * a multilingual PGF grammar
 * optionally, a set of compiled GF modules (retaining `oper` definitions)
 * a history of previous commands
 * a set of string, tree, and command macros

Unless file arguments are provided to the `gf` command, the shell starts in an
empty state, with no grammars and no history.

In the shell, a set of commands
is available. Some of these commands may change the grammars in the state. The general
syntax of commands is given by the following BNF grammar:

      COMMAND_LINE ::= COMMAND_PIPE
      COMMAND_LINE ::= COMMAND_PIPE ";" COMMAND_LINE
      COMMAND_PIPE ::= COMMAND
      COMMAND_PIPE ::= COMMAND "|" COMMAND_PIPE
      COMMAND      ::= COMMAND_ID (OPTION | FLAG)* ARGUMENT?
      OPTION       ::= "-"OPTION_ID
      FLAG         ::= "-"OPTION_ID "=" VALUE
      ARGUMENT     ::= QUOTED_STRING | TREE
      VALUE        ::= IDENT | QUOTED_STRING

A command pipe is a sequence of commands interpreted in such a way
that the output of each command
is send as input to the next. The option `-tr` causes GF to show a trace,
i.e. the intermediate result of the command to which it is attached.

A command line is a sequence of pipes separated by `;`. These pipes are
executed one by one, in the order of appearance.

### GF shell commands 

The full set of GF shell commands is listed below with explanations.
This list can also be obtained in the GF shell by the command `help -full`.

#VSPACE

#### ! 

#NOINDENT
`!`: *system command: escape to system shell.*

#TINY

 * Syntax: `! SYSTEMCOMMAND`

 * Examples:


|`! ls *.gf` |list all GF files in the working directory|

#NORMAL

#VSPACE

#### ? 

#NOINDENT
`?`: *system pipe: send value from previous command to a system command.*

#TINY

 * Syntax: `? SYSTEMCOMMAND`

 * Examples:


|`gt | l | ? wc` |generate, linearize, word-count|

#NORMAL

#VSPACE

#### ai = abstract_info 

#NOINDENT
`ai` = `abstract_info`: *Provides an information about a function, an expression or a category from the abstract syntax.*

#TINY

The command has one argument which is either function, expression or
a category defined in the abstract syntax of the current grammar. 
If the argument is a function then ?its type is printed out.
If it is a category then the category definition is printed.
If a whole expression is given it prints the expression with refined
metavariables and the type of the expression.

 * Syntax: `ai IDENTIFIER  or  ai EXPR`

#NORMAL

#VSPACE

#### aw = align_words 

#NOINDENT
`aw` = `align_words`: *show word alignments between languages graphically.*

#TINY

Prints a set of strings in the .dot format (the graphviz format).
The graph can be saved in a file by the wf command as usual.
If the -view flag is defined, the graph is saved in a temporary file
which is processed by 'dot' (graphviz) and displayed by the program indicated
by the view flag. The target format is png, unless overridden by the
flag -format. Results from multiple trees are combined to pdf with convert (ImageMagick).

 * Options:


|`-giza` |show alignments in the Giza format; the first two languages|

 * Flags:


|`-format` |format of the visualization file (default "png")|
|`-lang` |alignments for this list of languages (default: all)|
|`-view` |program to open the resulting file|

 * Examples:


|`gr | aw` |generate a tree and show word alignment as graph script|
|`gr | aw -view="open"` |generate a tree and display alignment on Mac|
|`gr | aw -view="eog"` |generate a tree and display alignment on Ubuntu|
|`gt | aw -giza | wf -file=aligns` |generate trees, send giza alignments to file|

#NORMAL

#VSPACE

#### ca = clitic_analyse 

#NOINDENT
`ca` = `clitic_analyse`: *print the analyses of all words into stems and clitics.*

#TINY

Analyses all words into all possible combinations of stem + clitics.
The analysis is returned in the format stem &+ clitic1 &+ clitic2 ...
which is hence the inverse of 'pt -bind'. The list of clitics is give
by the flag '-clitics'. The list of stems is given as the list of words
of the language given by the '-lang' flag.

 * Options:


|`-raw` |analyse each word separately (not suitable input for parser)|

 * Flags:


|`-clitics` |the list of possible clitics (comma-separated, no spaces)|
|`-lang` |the language of analysis|

 * Examples:


|`ca -lang=Fin -clitics=ko,ni "nukkuuko minun vaimoni" | p` |to parse Finnish|

#NORMAL

#VSPACE

#### cc = compute_concrete 

#NOINDENT
`cc` = `compute_concrete`: *computes concrete syntax term using a source grammar.*

#TINY

Compute TERM by concrete syntax definitions. Uses the topmost
module (the last one imported) to resolve constant names.
N.B.1 You need the flag -retain when importing the grammar, if you want
the definitions to be retained after compilation.
N.B.2 The resulting term is not a tree in the sense of abstract syntax
and hence not a valid input to a Tree-expecting command.
This command must be a line of its own, and thus cannot be a part
of a pipe.

 * Syntax: `cc (-all | -table | -unqual)? TERM`
 * Options:


|`-all` |pick all strings (forms and variants) from records and tables|
|`-list` |all strings, comma-separated on one line|
|`-one` |pick the first strings, if there is any, from records and tables|
|`-table` |show all strings labelled by parameters|
|`-unqual` |hide qualifying module names|
|`-trace` |trace computations|

#NORMAL

#VSPACE

#### dc = define_command 

#NOINDENT
`dc` = `define_command`: *define a command macro.*

#TINY

Defines IDENT as macro for COMMANDLINE, until IDENT gets redefined.
A call of the command has the form %IDENT. The command may take an
argument, which in COMMANDLINE is marked as ?0. Both strings and
trees can be arguments. Currently at most one argument is possible.
This command must be a line of its own, and thus cannot be a part
of a pipe.

 * Syntax: `dc IDENT COMMANDLINE`

#NORMAL

#VSPACE

#### dg = dependency_graph 

#NOINDENT
`dg` = `dependency_graph`: *print module dependency graph.*

#TINY

Prints the dependency graph of source modules.
Requires that import has been done with the -retain flag.
The graph is written in the file _gfdepgraph.dot
which can be further processed by Graphviz (the system command 'dot').
By default, all modules are shown, but the -only flag restricts them
by a comma-separated list of patterns, where 'name*' matches modules
whose name has prefix 'name', and other patterns match modules with
exactly the same name. The graphical conventions are:
  solid box = abstract, solid ellipse = concrete, dashed ellipse = other
  solid arrow empty head = of, solid arrow = **, dashed arrow = open
  dotted arrow = other dependency

 * Syntax: `dg (-only=MODULES)?`

 * Flags:


|`-only` |list of modules included (default: all), literally or by prefix*|

 * Examples:


|`dg -only=SyntaxEng,Food*` |shows only SyntaxEng, and those with prefix Food|

#NORMAL

#VSPACE

#### dt = define_tree 

#NOINDENT
`dt` = `define_tree`: *define a tree or string macro.*

#TINY

Defines IDENT as macro for TREE or STRING, until IDENT gets redefined.
The defining value can also come from a command, preceded by "<".
If the command gives many values, the first one is selected.
A use of the macro has the form %IDENT. Currently this use cannot be
a subtree of another tree. This command must be a line of its own
and thus cannot be a part of a pipe.

 * Syntax: `dt IDENT (TREE | STRING | "<" COMMANDLINE)`

 * Examples:


|`dt ex "hello world"` |define ex as string|
|`dt ex UseN man_N` |define ex as string|
|`dt ex < p -cat=NP "the man in the car"` |define ex as parse result|
|`l -lang=LangSwe %ex | ps -to_utf8` |linearize the tree ex|

#NORMAL

#VSPACE

#### e = empty 

#NOINDENT
`e` = `empty`: *empty the environment (except the command history).*

#TINY

#NORMAL

#VSPACE

#### eb = example_based 

#NOINDENT
`eb` = `example_based`: *converts .gfe files to .gf files by parsing examples to trees.*

#TINY

Reads FILE.gfe and writes FILE.gf. Each expression of form
'%ex CAT QUOTEDSTRING' in FILE.gfe is replaced by a syntax tree.
This tree is the first one returned by the parser; a biased ranking
can be used to regulate the order. If there are more than one parses
the rest are shown in comments, with probabilities if the order is biased.
The probabilities flag and configuration file is similar to the commands
gr and rt. Notice that the command doesn't change the environment,
but the resulting .gf file must be imported separately.

 * Syntax: `eb (-probs=FILE | -lang=LANG)* -file=FILE.gfe`
 * Options:


|`-api` |convert trees to overloaded API expressions (using Syntax not Lang)|

 * Flags:


|`-file` |the file to be converted (suffix .gfe must be given)|
|`-lang` |the language in which to parse|
|`-probs` |file with probabilities to rank the parses|

#NORMAL

#VSPACE

#### eh = execute_history 

#NOINDENT
`eh` = `execute_history`: *read commands from a file and execute them.*

#TINY

 * Syntax: `eh FILE`

#NORMAL

#VSPACE

#### gr = generate_random 

#NOINDENT
`gr` = `generate_random`: *generate random trees in the current abstract syntax.*

#TINY

Generates a list of random trees, by default one tree.
If a tree argument is given, the command completes the Tree with values to
all metavariables in the tree. The generation can be biased by probabilities,
given in a file in the -probs flag.

 * Syntax: `gr [-cat=CAT] [-number=INT]`

 * Flags:


|`-cat` |generation category|
|`-lang` |uses only functions that have linearizations in all these languages|
|`-number` |number of trees generated|
|`-depth` |the maximum generation depth|
|`-probs` |file with biased probabilities (format 'f 0.4' one by line)|

 * Examples:


|`gr` |one tree in the startcat of the current grammar|
|`gr -cat=NP -number=16` |16 trees in the category NP|
|`gr -lang=LangHin,LangTha -cat=Cl` |Cl, both in LangHin and LangTha|
|`gr -probs=FILE` |generate with bias|
|`gr (AdjCN ? (UseN ?))` |generate trees of form (AdjCN ? (UseN ?))|

#NORMAL

#VSPACE

#### gt = generate_trees 

#NOINDENT
`gt` = `generate_trees`: *generates a list of trees, by default exhaustive.*

#TINY

Generates all trees of a given category. By default, 
the depth is limited to 4, but this can be changed by a flag.
If a Tree argument is given, the command completes the Tree with values
to all metavariables in the tree.

 * Flags:


|`-cat` |the generation category|
|`-depth` |the maximum generation depth|
|`-lang` |excludes functions that have no linearization in this language|
|`-number` |the number of trees generated|

 * Examples:


|`gt` |all trees in the startcat, to depth 4|
|`gt -cat=NP -number=16` |16 trees in the category NP|
|`gt -cat=NP -depth=2` |trees in the category NP to depth 2|
|`gt (AdjCN ? (UseN ?))` |trees of form (AdjCN ? (UseN ?))|

#NORMAL

#VSPACE

#### h = help 

#NOINDENT
`h` = `help`: *get description of a command, or a the full list of commands.*

#TINY

Displays information concerning the COMMAND.
Without argument, shows the synopsis of all commands.

 * Syntax: `h (-full)? COMMAND?`
 * Options:


|`-changes` |give a summary of changes from GF 2.9|
|`-coding` |give advice on character encoding|
|`-full` |give full information of the commands|
|`-license` |show copyright and license information|
|`-t2t` |output help in txt2tags format|

#NORMAL

#VSPACE

#### i = import 

#NOINDENT
`i` = `import`: *import a grammar from source code or compiled .pgf file.*

#TINY

Reads a grammar from File and compiles it into a GF runtime grammar.
If its abstract is different from current state, old modules are discarded.
If its abstract is the same and a concrete with the same name is already in the state
it is overwritten - but only if compilation succeeds.
The grammar parser depends on the file name suffix:
  .cf    context-free (labelled BNF) source
  .ebnf  extended BNF source
  .gfm   multi-module GF source
  .gf    normal GF source
  .gfo   compiled GF source
  .pgf   precompiled grammar in Portable Grammar Format

 * Options:


|`-retain` |retain operations (used for cc command)|
|`-src` |force compilation from source|
|`-v` |be verbose - show intermediate status information|

 * Flags:


|`-probs` |file with biased probabilities for generation|

#NORMAL

#VSPACE

#### l = linearize 

#NOINDENT
`l` = `linearize`: *convert an abstract syntax expression to string.*

#TINY

Shows the linearization of a Tree by the grammars in scope.
The -lang flag can be used to restrict this to fewer languages.
A sequence of string operations (see command ps) can be given
as options, and works then like a pipe to the ps command, except
that it only affect the strings, not e.g. the table labels.
These can be given separately to each language with the unlexer flag
whose results are prepended to the other lexer flags. The value of the
unlexer flag is a space-separated list of comma-separated string operation
sequences; see example.

 * Options:


|`-all` |show all forms and variants, one by line (cf. l -list)|
|`-bracket` |show tree structure with brackets and paths to nodes|
|`-groups` |all languages, grouped by lang, remove duplicate strings|
|`-list` |show all forms and variants, comma-separated on one line (cf. l -all)|
|`-multi` |linearize to all languages (default)|
|`-table` |show all forms labelled by parameters|
|`-tabtreebank` |show the tree and its linearizations on a tab-separated line|
|`-treebank` |show the tree and tag linearizations with language names|
|`-bind` |bind tokens separated by Prelude.BIND, i.e. &+|
|`-chars` |lexer that makes every non-space character a token|
|`-from_amharic` |from unicode to GF Amharic transliteration|
|`-from_ancientgreek` |from unicode to GF ancient Greek transliteration|
|`-from_arabic` |from unicode to GF Arabic transliteration|
|`-from_arabic_unvocalized` |from unicode to GF unvocalized Arabic transliteration|
|`-from_cp1251` |decode from cp1251 (Cyrillic used in Bulgarian resource)|
|`-from_devanagari` |from unicode to GF Devanagari transliteration|
|`-from_greek` |from unicode to GF modern Greek transliteration|
|`-from_hebrew` |from unicode to GF unvocalized Hebrew transliteration|
|`-from_nepali` |from unicode to GF Nepali transliteration|
|`-from_persian` |from unicode to GF Persian/Farsi transliteration|
|`-from_sanskrit` |from unicode to GF Sanskrit transliteration|
|`-from_sindhi` |from unicode to GF Sindhi transliteration|
|`-from_telugu` |from unicode to GF Telugu transliteration|
|`-from_thai` |from unicode to GF Thai transliteration|
|`-from_urdu` |from unicode to GF Urdu transliteration|
|`-from_utf8` |decode from utf8 (default)|
|`-lexcode` |code-like lexer|
|`-lexgreek` |lexer normalizing ancient Greek accentuation|
|`-lexgreek2` |lexer normalizing ancient Greek accentuation for text with vowel length annotations|
|`-lexmixed` |mixture of text and code, as in LaTeX (code between $...$, \(...)\, \[...\])|
|`-lextext` |text-like lexer|
|`-to_amharic` |from GF Amharic transliteration to unicode|
|`-to_ancientgreek` |from GF ancient Greek transliteration to unicode|
|`-to_arabic` |from GF Arabic transliteration to unicode|
|`-to_arabic_unvocalized` |from GF unvocalized Arabic transliteration to unicode|
|`-to_cp1251` |encode to cp1251 (Cyrillic used in Bulgarian resource)|
|`-to_devanagari` |from GF Devanagari transliteration to unicode|
|`-to_greek` |from GF modern Greek transliteration to unicode|
|`-to_hebrew` |from GF unvocalized Hebrew transliteration to unicode|
|`-to_html` |wrap in a html file with linebreaks|
|`-to_nepali` |from GF Nepali transliteration to unicode|
|`-to_persian` |from GF Persian/Farsi transliteration to unicode|
|`-to_sanskrit` |from GF Sanskrit transliteration to unicode|
|`-to_sindhi` |from GF Sindhi transliteration to unicode|
|`-to_telugu` |from GF Telugu transliteration to unicode|
|`-to_thai` |from GF Thai transliteration to unicode|
|`-to_urdu` |from GF Urdu transliteration to unicode|
|`-to_utf8` |encode to utf8 (default)|
|`-unchars` |unlexer that puts no spaces between tokens|
|`-unlexcode` |code-like unlexer|
|`-unlexgreek` |unlexer de-normalizing ancient Greek accentuation|
|`-unlexmixed` |mixture of text and code (code between $...$, \(...)\, \[...\])|
|`-unlextext` |text-like unlexer|
|`-unwords` |unlexer that puts a single space between tokens (default)|
|`-words` |lexer that assumes tokens separated by spaces (default)|

 * Flags:


|`-lang` |the languages of linearization (comma-separated, no spaces)|
|`-unlexer` |set unlexers separately to each language (space-separated)|

 * Examples:


|`l -lang=LangSwe,LangNor no_Utt` |linearize tree to LangSwe and LangNor|
|`gr -lang=LangHin -cat=Cl | l -table -to_devanagari` |hindi table|
|`l -unlexer="LangAra=to_arabic LangHin=to_devanagari"` |different unlexers|

#NORMAL

#VSPACE

#### lc = linearize_chunks 

#NOINDENT
`lc` = `linearize_chunks`: *linearize a tree that has metavariables in maximal chunks without them.*

#TINY

A hopefully temporary command, intended to work around the type checker that fails
trees where a function node is a metavariable.

 * Options:


|`-treebank` |show the tree and tag linearizations with language names|
|`-bind` |bind tokens separated by Prelude.BIND, i.e. &+|
|`-chars` |lexer that makes every non-space character a token|
|`-from_amharic` |from unicode to GF Amharic transliteration|
|`-from_ancientgreek` |from unicode to GF ancient Greek transliteration|
|`-from_arabic` |from unicode to GF Arabic transliteration|
|`-from_arabic_unvocalized` |from unicode to GF unvocalized Arabic transliteration|
|`-from_cp1251` |decode from cp1251 (Cyrillic used in Bulgarian resource)|
|`-from_devanagari` |from unicode to GF Devanagari transliteration|
|`-from_greek` |from unicode to GF modern Greek transliteration|
|`-from_hebrew` |from unicode to GF unvocalized Hebrew transliteration|
|`-from_nepali` |from unicode to GF Nepali transliteration|
|`-from_persian` |from unicode to GF Persian/Farsi transliteration|
|`-from_sanskrit` |from unicode to GF Sanskrit transliteration|
|`-from_sindhi` |from unicode to GF Sindhi transliteration|
|`-from_telugu` |from unicode to GF Telugu transliteration|
|`-from_thai` |from unicode to GF Thai transliteration|
|`-from_urdu` |from unicode to GF Urdu transliteration|
|`-from_utf8` |decode from utf8 (default)|
|`-lexcode` |code-like lexer|
|`-lexgreek` |lexer normalizing ancient Greek accentuation|
|`-lexgreek2` |lexer normalizing ancient Greek accentuation for text with vowel length annotations|
|`-lexmixed` |mixture of text and code, as in LaTeX (code between $...$, \(...)\, \[...\])|
|`-lextext` |text-like lexer|
|`-to_amharic` |from GF Amharic transliteration to unicode|
|`-to_ancientgreek` |from GF ancient Greek transliteration to unicode|
|`-to_arabic` |from GF Arabic transliteration to unicode|
|`-to_arabic_unvocalized` |from GF unvocalized Arabic transliteration to unicode|
|`-to_cp1251` |encode to cp1251 (Cyrillic used in Bulgarian resource)|
|`-to_devanagari` |from GF Devanagari transliteration to unicode|
|`-to_greek` |from GF modern Greek transliteration to unicode|
|`-to_hebrew` |from GF unvocalized Hebrew transliteration to unicode|
|`-to_html` |wrap in a html file with linebreaks|
|`-to_nepali` |from GF Nepali transliteration to unicode|
|`-to_persian` |from GF Persian/Farsi transliteration to unicode|
|`-to_sanskrit` |from GF Sanskrit transliteration to unicode|
|`-to_sindhi` |from GF Sindhi transliteration to unicode|
|`-to_telugu` |from GF Telugu transliteration to unicode|
|`-to_thai` |from GF Thai transliteration to unicode|
|`-to_urdu` |from GF Urdu transliteration to unicode|
|`-to_utf8` |encode to utf8 (default)|
|`-unchars` |unlexer that puts no spaces between tokens|
|`-unlexcode` |code-like unlexer|
|`-unlexgreek` |unlexer de-normalizing ancient Greek accentuation|
|`-unlexmixed` |mixture of text and code (code between $...$, \(...)\, \[...\])|
|`-unlextext` |text-like unlexer|
|`-unwords` |unlexer that puts a single space between tokens (default)|
|`-words` |lexer that assumes tokens separated by spaces (default)|

 * Flags:


|`-lang` |the languages of linearization (comma-separated, no spaces)|

 * Examples:


|`l -lang=LangSwe,LangNor -chunks ? a b (? c d)`|

#NORMAL

#VSPACE

#### ma = morpho_analyse 

#NOINDENT
`ma` = `morpho_analyse`: *print the morphological analyses of all words in the string.*

#TINY

Prints all the analyses of space-separated words in the input string,
using the morphological analyser of the actual grammar (see command pg)

 * Options:


|`-known` |return only the known words, in order of appearance|
|`-missing` |show the list of unknown words, in order of appearance|

 * Flags:


|`-lang` |the languages of analysis (comma-separated, no spaces)|

#NORMAL

#VSPACE

#### mq = morpho_quiz 

#NOINDENT
`mq` = `morpho_quiz`: *start a morphology quiz.*

#TINY

 * Syntax: `mq (-cat=CAT)? (-probs=FILE)? TREE?`

 * Flags:


|`-lang` |language of the quiz|
|`-cat` |category of the quiz|
|`-number` |maximum number of questions|
|`-probs` |file with biased probabilities for generation|

#NORMAL

#VSPACE

#### p = parse 

#NOINDENT
`p` = `parse`: *parse a string to abstract syntax expression.*

#TINY

Shows all trees returned by parsing a string in the grammars in scope.
The -lang flag can be used to restrict this to fewer languages.
The default start category can be overridden by the -cat flag.
See also the ps command for lexing and character encoding.

The -openclass flag is experimental and allows some robustness in 
the parser. For example if -openclass="A,N,V" is given, the parser
will accept unknown adjectives, nouns and verbs with the resource grammar.

 * Options:


|`-bracket` |prints the bracketed string from the parser|

 * Flags:


|`-cat` |target category of parsing|
|`-lang` |the languages of parsing (comma-separated, no spaces)|
|`-openclass` |list of open-class categories for robust parsing|
|`-depth` |maximal depth for proof search if the abstract syntax tree has meta variables|

#NORMAL

#VSPACE

#### pg = print_grammar 

#NOINDENT
`pg` = `print_grammar`: *print the actual grammar with the given printer.*

#TINY

Prints the actual grammar, with all involved languages.
In some printers, this can be restricted to a subset of languages
with the -lang=X,Y flag (comma-separated, no spaces).
The -printer=P flag sets the format in which the grammar is printed.
N.B.1 Since grammars are compiled when imported, this command
generally shows a grammar that looks rather different from the source.
N.B.2 Another way to produce different formats is to use 'gf -make',
the batch compiler. The following values are available both for
the batch compiler (flag -output-format) and the print_grammar
command (flag -printer):

 bnf		BNF (context-free grammar)
 ebnf		Extended BNF
 fa		finite automaton in graphviz format
 gsl		Nuance speech recognition format
 haskell		Haskell (abstract syntax)
 java		Java (abstract syntax)
 js		JavaScript (whole grammar)
 jsgf		JSGF speech recognition format
 pgf_pretty		human-readable pgf
 prolog		Prolog (whole grammar)
 python		Python (whole grammar)
 regexp		regular expression
 slf		SLF speech recognition format
 srgs_abnf		SRGS speech recognition format in ABNF
 srgs_abnf_nonrec		SRGS ABNF, recursion eliminated
 srgs_xml		SRGS speech recognition format in XML
 srgs_xml_nonrec		SRGS XML, recursion eliminated
 vxml		Voice XML based on abstract syntax

 * Options:


|`-cats` |show just the names of abstract syntax categories|
|`-fullform` |print the fullform lexicon|
|`-funs` |show just the names and types of abstract syntax functions|
|`-langs` |show just the names of top concrete syntax modules|
|`-lexc` |print the lexicon in Xerox LEXC format|
|`-missing` |show just the names of functions that have no linearization|
|`-opt` |optimize the generated pgf|
|`-pgf` |write current pgf image in file|
|`-words` |print the list of words|

 * Flags:


|`-file` |set the file name when printing with -pgf option|
|`-lang` |select languages for the some options (default all languages)|
|`-printer` |select the printing format (see flag values above)|

 * Examples:


|`pg -funs | ? grep " S ;"` |show functions with value cat S|

#NORMAL

#VSPACE

#### ph = print_history 

#NOINDENT
`ph` = `print_history`: *print command history.*

#TINY

Prints the commands issued during the GF session.
The result is readable by the eh command.
The result can be used as a script when starting GF.

 * Examples:


|`ph | wf -file=foo.gfs` |save the history into a file|

#NORMAL

#VSPACE

#### ps = put_string 

#NOINDENT
`ps` = `put_string`: *return a string, possibly processed with a function.*

#TINY

Returns a string obtained from its argument string by applying
string processing functions in the order given in the command line
option list. Thus 'ps -f -g s' returns g (f s). Typical string processors
are lexers and unlexers, but also character encoding conversions are possible.
The unlexers preserve the division of their input to lines.
To see transliteration tables, use command ut.

 * Syntax: `ps OPT? STRING`
 * Options:


|`-lines` |apply the operation separately to each input line, returning a list of lines|
|`-bind` |bind tokens separated by Prelude.BIND, i.e. &+|
|`-chars` |lexer that makes every non-space character a token|
|`-from_amharic` |from unicode to GF Amharic transliteration|
|`-from_ancientgreek` |from unicode to GF ancient Greek transliteration|
|`-from_arabic` |from unicode to GF Arabic transliteration|
|`-from_arabic_unvocalized` |from unicode to GF unvocalized Arabic transliteration|
|`-from_cp1251` |decode from cp1251 (Cyrillic used in Bulgarian resource)|
|`-from_devanagari` |from unicode to GF Devanagari transliteration|
|`-from_greek` |from unicode to GF modern Greek transliteration|
|`-from_hebrew` |from unicode to GF unvocalized Hebrew transliteration|
|`-from_nepali` |from unicode to GF Nepali transliteration|
|`-from_persian` |from unicode to GF Persian/Farsi transliteration|
|`-from_sanskrit` |from unicode to GF Sanskrit transliteration|
|`-from_sindhi` |from unicode to GF Sindhi transliteration|
|`-from_telugu` |from unicode to GF Telugu transliteration|
|`-from_thai` |from unicode to GF Thai transliteration|
|`-from_urdu` |from unicode to GF Urdu transliteration|
|`-from_utf8` |decode from utf8 (default)|
|`-lexcode` |code-like lexer|
|`-lexgreek` |lexer normalizing ancient Greek accentuation|
|`-lexgreek2` |lexer normalizing ancient Greek accentuation for text with vowel length annotations|
|`-lexmixed` |mixture of text and code, as in LaTeX (code between $...$, \(...)\, \[...\])|
|`-lextext` |text-like lexer|
|`-to_amharic` |from GF Amharic transliteration to unicode|
|`-to_ancientgreek` |from GF ancient Greek transliteration to unicode|
|`-to_arabic` |from GF Arabic transliteration to unicode|
|`-to_arabic_unvocalized` |from GF unvocalized Arabic transliteration to unicode|
|`-to_cp1251` |encode to cp1251 (Cyrillic used in Bulgarian resource)|
|`-to_devanagari` |from GF Devanagari transliteration to unicode|
|`-to_greek` |from GF modern Greek transliteration to unicode|
|`-to_hebrew` |from GF unvocalized Hebrew transliteration to unicode|
|`-to_html` |wrap in a html file with linebreaks|
|`-to_nepali` |from GF Nepali transliteration to unicode|
|`-to_persian` |from GF Persian/Farsi transliteration to unicode|
|`-to_sanskrit` |from GF Sanskrit transliteration to unicode|
|`-to_sindhi` |from GF Sindhi transliteration to unicode|
|`-to_telugu` |from GF Telugu transliteration to unicode|
|`-to_thai` |from GF Thai transliteration to unicode|
|`-to_urdu` |from GF Urdu transliteration to unicode|
|`-to_utf8` |encode to utf8 (default)|
|`-unchars` |unlexer that puts no spaces between tokens|
|`-unlexcode` |code-like unlexer|
|`-unlexgreek` |unlexer de-normalizing ancient Greek accentuation|
|`-unlexmixed` |mixture of text and code (code between $...$, \(...)\, \[...\])|
|`-unlextext` |text-like unlexer|
|`-unwords` |unlexer that puts a single space between tokens (default)|
|`-words` |lexer that assumes tokens separated by spaces (default)|

 * Flags:


|`-env` |apply in this environment only|
|`-from` |backward-apply transliteration defined in this file (format 'unicode translit' per line)|
|`-to` |forward-apply transliteration defined in this file|

 * Examples:


|`l (EAdd 3 4) | ps -unlexcode` |linearize code-like output|
|`ps -lexcode | p -cat=Exp` |parse code-like input|
|`gr -cat=QCl | l | ps -bind` |linearization output from LangFin|
|`ps -to_devanagari "A-p"` |show Devanagari in UTF8 terminal|
|`rf -file=Hin.gf | ps -env=quotes -to_devanagari` |convert translit to UTF8|
|`rf -file=Ara.gf | ps -from_utf8 -env=quotes -from_arabic` |convert UTF8 to transliteration|
|`ps -to=chinese.trans "abc"` |apply transliteration defined in file chinese.trans|
|`ps -lexgreek "a)gavoi` a)'nvrwpoi' tines*"` |normalize ancient greek accentuation|

#NORMAL

#VSPACE

#### pt = put_tree 

#NOINDENT
`pt` = `put_tree`: *return a tree, possibly processed with a function.*

#TINY

Returns a tree obtained from its argument tree by applying
tree processing functions in the order given in the command line
option list. Thus 'pt -f -g s' returns g (f s). Typical tree processors
are type checking and semantic computation.

 * Syntax: `pt OPT? TREE`
 * Options:


|`-compute` |compute by using semantic definitions (def)|
|`-largest` |sort trees from largest to smallest, in number of nodes|
|`-nub` |remove duplicate trees|
|`-smallest` |sort trees from smallest to largest, in number of nodes|
|`-subtrees` |return all fully applied subtrees (stopping at abstractions), by default sorted from the largest|
|`-funs` |return all fun functions appearing in the tree, with duplications|

 * Flags:


|`-number` |take at most this many trees|

 * Examples:


|`pt -compute (plus one two)` |compute value|

#NORMAL

#VSPACE

#### q = quit 

#NOINDENT
`q` = `quit`: *exit GF interpreter.*

#TINY

#NORMAL

#VSPACE

#### r = reload 

#NOINDENT
`r` = `reload`: *repeat the latest import command.*

#TINY

#NORMAL

#VSPACE

#### rf = read_file 

#NOINDENT
`rf` = `read_file`: *read string or tree input from a file.*

#TINY

Reads input from file. The filename must be in double quotes.
The input is interpreted as a string by default, and can hence be
piped e.g. to the parse command. The option -tree interprets the
input as a tree, which can be given e.g. to the linearize command.
The option -lines will result in a list of strings or trees, one by line.

 * Options:


|`-lines` |return the list of lines, instead of the singleton of all contents|
|`-tree` |convert strings into trees|

 * Flags:


|`-file` |the input file name|

#NORMAL

#VSPACE

#### rt = rank_trees 

#NOINDENT
`rt` = `rank_trees`: *show trees in an order of decreasing probability.*

#TINY

Order trees from the most to the least probable, using either
even distribution in each category (default) or biased as specified
by the file given by flag -probs=FILE, where each line has the form
'function probability', e.g. 'youPol_Pron  0.01'.

 * Options:


|`-v` |show all trees with their probability scores|

 * Flags:


|`-probs` |probabilities from this file (format 'f 0.6' per line)|

 * Examples:


|`p "you are here" | rt -probs=probs | pt -number=1` |most probable result|

#NORMAL

#VSPACE

#### sd = show_dependencies 

#NOINDENT
`sd` = `show_dependencies`: *show all constants that the given constants depend on.*

#TINY

Show recursively all qualified constant names, by tracing back the types and definitions
of each constant encountered, but just listing every name once.
This command requires a source grammar to be in scope, imported with 'import -retain'.
Notice that the accuracy is better if the modules are compiled with the flag -optimize=noexpand.
This command must be a line of its own, and thus cannot be a part of a pipe.

 * Syntax: `sd QUALIFIED_CONSTANT+`
 * Options:


|`-size` |show the size of the source code for each constants (number of constructors)|

 * Examples:


|`sd ParadigmsEng.mkV ParadigmsEng.mkN` |show all constants on which mkV and mkN depend|
|`sd -size ParadigmsEng.mkV` |show all constants on which mkV depends, together with size|

#NORMAL

#VSPACE

#### se = set_encoding 

#NOINDENT
`se` = `set_encoding`: *set the encoding used in current terminal.*

#TINY

 * Syntax: `se ID`

 * Examples:


|`se cp1251` |set encoding to cp1521|
|`se utf8` |set encoding to utf8 (default)|

#NORMAL

#VSPACE

#### so = show_operations 

#NOINDENT
`so` = `show_operations`: *show all operations in scope, possibly restricted to a value type.*

#TINY

Show the names and type signatures of all operations available in the current resource.
This command requires a source grammar to be in scope, imported with 'import -retain'.
The operations include the parameter constructors that are in scope.
The optional TYPE filters according to the value type.
The grep STRINGs filter according to other substrings of the type signatures.

 * Syntax: `so (-grep=STRING)* TYPE?`
 * Options:


|`-raw` |show the types in computed forms (instead of category names)|

 * Flags:


|`-grep` |substring used for filtering (the command can have many of these)|

 * Examples:


|`so Det` |show all opers that create a Det|
|`so -grep=Prep` |find opers relating to Prep|
|`so | wf -file=/tmp/opers` |write the list of opers to a file|

#NORMAL

#VSPACE

#### sp = system_pipe 

#NOINDENT
`sp` = `system_pipe`: *send argument to a system command.*

#TINY

 * Syntax: `sp -command="SYSTEMCOMMAND", alt. ? SYSTEMCOMMAND`

 * Flags:


|`-command` |the system command applied to the argument|

 * Examples:


|`gt | l | ? wc` |generate trees, linearize, and count words|

#NORMAL

#VSPACE

#### ss = show_source 

#NOINDENT
`ss` = `show_source`: *show the source code of modules in scope, possibly just headers.*

#TINY

Show compiled source code, i.e. as it is included in GF object files.
This command requires a source grammar to be in scope, imported with 'import -retain'.
The optional MODULE arguments cause just these modules to be shown.
The -size and -detailedsize options show code size as the number of constructor nodes.
This command must be a line of its own, and thus cannot be a part of a pipe.

 * Syntax: `ss (-strip)? (-save)? MODULE*`
 * Options:


|`-detailedsize` |instead of code, show the sizes of all judgements and modules|
|`-save` |save each MODULE in file MODULE.gfh instead of printing it on terminal|
|`-size` |instead of code, show the sizes of all modules|
|`-strip` |show only type signatures of oper's and lin's, not their definitions|

 * Examples:


|`ss` |print complete current source grammar on terminal|
|`ss -strip -save MorphoFin` |print the headers in file MorphoFin.gfh|

#NORMAL

#VSPACE

#### tq = translation_quiz 

#NOINDENT
`tq` = `translation_quiz`: *start a translation quiz.*

#TINY

 * Syntax: `tq -from=LANG -to=LANG (-cat=CAT)? (-probs=FILE)? TREE?`

 * Flags:


|`-from` |translate from this language|
|`-to` |translate to this language|
|`-cat` |translate in this category|
|`-number` |the maximum number of questions|
|`-probs` |file with biased probabilities for generation|

 * Examples:


|`tq -from=Eng -to=Swe` |any trees in startcat|
|`tq -from=Eng -to=Swe (AdjCN (PositA ?2) (UseN ?))` |only trees of this form|

#NORMAL

#VSPACE

#### tt = to_trie 

#NOINDENT
`tt` = `to_trie`: *combine a list of trees into a trie.*

#TINY

 * Syntax: `to_trie`

#NORMAL

#VSPACE

#### ut = unicode_table 

#NOINDENT
`ut` = `unicode_table`: *show a transliteration table for a unicode character set.*

#TINY

 * Options:


|`-amharic` |Amharic|
|`-ancientgreek` |ancient Greek|
|`-arabic` |Arabic|
|`-arabic_unvocalized` |unvocalized Arabic|
|`-devanagari` |Devanagari|
|`-greek` |modern Greek|
|`-hebrew` |unvocalized Hebrew|
|`-nepali` |Nepali|
|`-persian` |Persian/Farsi|
|`-sanskrit` |Sanskrit|
|`-sindhi` |Sindhi|
|`-telugu` |Telugu|
|`-thai` |Thai|
|`-urdu` |Urdu|

#NORMAL

#VSPACE

#### vd = visualize_dependency 

#NOINDENT
`vd` = `visualize_dependency`: *show word dependency tree graphically.*

#TINY

Prints a dependency tree in the .dot format (the graphviz format, default)
or LaTeX (flag -output=latex)
or the CoNLL/MaltParser format (flag -output=conll for training, malt_input
for unanalysed input).
By default, the last argument is the head of every abstract syntax
function; moreover, the head depends on the head of the function above.
The graph can be saved in a file by the wf command as usual.
If the -view flag is defined, the graph is saved in a temporary file
which is processed by dot (graphviz) and displayed by the program indicated
by the view flag. The target format is png, unless overridden by the
flag -format. Results from multiple trees are combined to pdf with convert (ImageMagick).
See also 'vp -showdep' for another visualization of dependencies.

 * Options:


|`-v` |show extra information|
|`-conll2latex` |convert conll to latex|

 * Flags:


|`-abslabels` |abstract configuration file for labels, format per line 'fun label*'|
|`-cnclabels` |concrete configuration file for labels, format per line 'fun {words|*} pos label head'|
|`-file` |same as abslabels (abstract configuration file)|
|`-format` |format of the visualization file using dot (default "png")|
|`-output` |output format of graph source (latex, conll, dot (default but deprecated))|
|`-view` |program to open the resulting graph file (default "open")|
|`-lang` |the language of analysis|

 * Examples:


|`gr | vd` |generate a tree and show dependency tree in .dot|
|`gr | vd -view=open` |generate a tree and display dependency tree on with Mac's 'open'|
|`gr | vd -view=open -output=latex` |generate a tree and display latex dependency tree with Mac's 'open'|
|`gr -number=1000 | vd -abslabels=Lang.labels -cnclabels=LangSwe.labels -output=conll` |generate a random treebank|
|`rf -file=ex.conll | vd -conll2latex | wf -file=ex.tex` |convert conll file to latex|

#NORMAL

#VSPACE

#### vp = visualize_parse 

#NOINDENT
`vp` = `visualize_parse`: *show parse tree graphically.*

#TINY

Prints a parse tree in the .dot format (the graphviz format).
The graph can be saved in a file by the wf command as usual.
If the -view flag is defined, the graph is saved in a temporary file
which is processed by dot (graphviz) and displayed by the program indicated
by the view flag. The target format is png, unless overridden by the
flag -format. Results from multiple trees are combined to pdf with convert (ImageMagick).

 * Options:


|`-showcat` |show categories in the tree nodes (default)|
|`-nocat` |don't show categories|
|`-showdep` |show dependency labels|
|`-showfun` |show function names in the tree nodes|
|`-nofun` |don't show function names (default)|
|`-showleaves` |show the leaves of the tree (default)|
|`-noleaves` |don't show the leaves of the tree (i.e., only the abstract tree)|

 * Flags:


|`-lang` |the language to visualize|
|`-file` |configuration file for dependency labels with -deps, format per line 'fun label*'|
|`-format` |format of the visualization file (default "png")|
|`-view` |program to open the resulting file (default "open")|
|`-nodefont` |font for tree nodes (default: Times -- graphviz standard font)|
|`-leaffont` |font for tree leaves (default: nodefont)|
|`-nodecolor` |color for tree nodes (default: black -- graphviz standard color)|
|`-leafcolor` |color for tree leaves (default: nodecolor)|
|`-nodeedgestyle` |edge style between tree nodes (solid/dashed/dotted/bold, default: solid)|
|`-leafedgestyle` |edge style for links to leaves (solid/dashed/dotted/bold, default: dashed)|

 * Examples:


|`p "John walks" | vp` |generate a tree and show parse tree as .dot script|
|`gr | vp -view=open` |generate a tree and display parse tree on a Mac|
|`p "she loves us" | vp -view=open -showdep -file=uddeps.labels -nocat` |show a visual variant of a dependency tree|

#NORMAL

#VSPACE

#### vt = visualize_tree 

#NOINDENT
`vt` = `visualize_tree`: *show a set of trees graphically.*

#TINY

Prints a set of trees in the .dot format (the graphviz format).
The graph can be saved in a file by the wf command as usual.
If the -view flag is defined, the graph is saved in a temporary file
which is processed by dot (graphviz) and displayed by the command indicated
by the view flag. The target format is postscript, unless overridden by the
flag -format. Results from multiple trees are combined to pdf with convert (ImageMagick).
With option -mk, use for showing library style function names of form 'mkC'.

 * Options:


|`-api` |show the tree with function names converted to 'mkC' with value cats C|
|`-mk` |similar to -api, deprecated|
|`-nofun` |don't show functions but only categories|
|`-nocat` |don't show categories but only functions|

 * Flags:


|`-format` |format of the visualization file (default "png")|
|`-view` |program to open the resulting file (default "open")|

 * Examples:


|`p "hello" | vt` |parse a string and show trees as graph script|
|`p "hello" | vt -view="open"` |parse a string and display trees on a Mac|

#NORMAL

#VSPACE

#### wf = write_file 

#NOINDENT
`wf` = `write_file`: *send string or tree to a file.*

#TINY

 * Options:


|`-append` |append to file, instead of overwriting it|

 * Flags:


|`-file` |the output filename|

#NORMAL

## The GF batch compiler 

With the option `-batch`, GF can be invoked in batch mode, i.e.
without opening the shell, to compile files from `.gf` to `.gfo`.
The `-s` option ("silent") eliminates all messages except errors.

      $ gf -batch -s LangIta.gf

With the option `-make`, and as a set of
top-level grammar files (with the same abstract syntax) as arguments,
GF produces a `.pgf` file. The flag `-optimize-pgf` minimizes
the size of the `.pgf` file, and is recommended for grammars to be shipped.

      $ gf -make -optimize-pgf LangIta.gf LangEng.gf LangGer.gf

The flag `-output-format` changes the output format from `.pgf` to
some other format. For instance

      $ gf -make -output-format=js LangEng.pgf LangGer.pgf

Notice that the arguments can be `.pgf` files, which in this case
are merged and written into a JavaScript grammar file.

More options and instructions are obtained with

      $ gf -help

To run GF from a *script*, redirection of standard input can be used:

      $ gf <script.gfs

The file `script.gfs` should then contain a sequence of GF commands, one per line.
Unrecognized command lines are skipped without terminating GF.

