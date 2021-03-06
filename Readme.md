This package contains a full analysis pipeline for Spanish that takes text (one sentence per line) as input and does PoS tagging, named entity recognition and classification, parsing and co-reference resolution. Output format is CoNLL annotated with entities.

Please reference this paper if you use the pipeline: 
```
@inproceedings{zora136447,
       booktitle = {15th Conference of the European Chapter of the Association for Computational Linguistics},
           month = {April},
           title = {Co-reference Resolution of Elided Subjects and Possessive Pronouns in Spanish-English Statistical Machine Translation},
          author = {Annette Rios and Don Tuggener},
       publisher = {Association for Computational Linguistics},
            year = {2017},
           pages = {657--662},
             url = {http://dx.doi.org/10.5167/uzh-136447}
}
```

An annotated version of the news commentary 2011 corpus used in the paper is available from https://github.com/a-rios/CorefMT

## Installation 

### FreeLing

`git clone https://github.com/TALP-UPC/freeling`

Installation (make sure to install from sources, headers are needed),  see: https://talp-upc.gitbooks.io/freeling-user-manual/content/installation.html

compile Freeling analyzer with crf output format for wapiti:
```
export $FREELING_INSTALLATION_DIR= path to you installation of FreeLing
export $PARSING_PIPELINE_DIR= path to this package
g++ -c -o output_crf.o output_crf.cc -I$FREELING_INSTALLATION_DIR/include -I$PARSING_PIPELINE_DIR/FreeLingModules/config_squoia
g++ -c -o analyzer_client.o analyzer_client.cc -I$FREELING_INSTALLATION_DIR/include -I$PARSING_PIPELINE_DIR/FreeLingModules/config_squoia
g++ -std=gnu++11 -c  -o server_squoia.o server_squoia.cc -I$FREELING_INSTALLATION_DIR/include -I$PARSING_PIPELINE_DIR/FreeLingModules/config_squoia
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$FREELING_INSTALLATION_DIR/lib
g++ -O3 -Wall -o server_squoia server_squoia.o output_crf.o -L$FREELING_INSTALLATION_DIR/lib -lfreeling -lboost_program_options -lboost_system -lboost_filesystem -lpthread
```

named entity classification:
```
g++ -std=gnu++11 -o nec nec.cc -I$FREELING_INSTALLATION_DIR/include -I$PARSING_PIPELINE_DIR/FreeLingModules/config_squoia -L$FREELING_INSTALLATION_DIR/lib -lfreeling -lboost_program_options -lboost_system -lboost_filesystem -lpthread
```

analyzer_client:

```
g++ -O3 -Wall -o analyzer_client analyzer_client.o -L$FREELING_INSTALLATION_DIR/local/lib -lfreeling
export FREELINGSHARE=$FREELING_INSTALLATION_DIR/share/freeling
```

once compiled, you can test the server:
```
./server_squoia -f $PARSING_PIPELINE_DIR/FreeLingModules/es_squoia.cfg  --server --port=$PORT 2> logtagging &
echo "eso es una prueba" |./analyzer_client $PORT
```

Link server_squoia, analyzer_client and nec to the /bin folder (optional, if you do not link them, change the paths in es.cfg):

```
cd $PARSING_PIPELINE_DIR/bin
ln -s ../FreeLingModules/server_squoia .
ln -s ../FreeLingModules/analyzer_client .
ln -s ../FreeLingModules/nec .
```

For system wide use, either link client and server to somewhere in your $PATH (e.g. in `/usr/local/bin`), or add their location to $PATH


### Wapiti

https://wapiti.limsi.fr/

follow installation instructions, then adapt path to wapiti in es.cfg


### MaltParser

http://www.maltparser.org/download.html

follow installation instructions, see http://www.maltparser.org/install.html

set maltPath in es.cfg to your installation of maltparser

compile server-client modules ($MALTPARSER_DIR= path to your maltparser installation):

```
cd $PARSING_PIPELINE_DIR/maltparser_tools/src 
javac -cp $MALTPARSER_DIR/maltparser-1.8/maltparser-1.8.jar MPClient.java
javac -cp $MALTPARSER_DIR/maltparser-1.8/maltparser-1.8.jar MaltParserServer.java
```

move binaries to ../bin:
`mv MaltParserServer.class MPClient.class ../bin/`

### Perl modules required: 
```
Getopt::Long;
Storable;
File::Basename;
File::Spec::Functions
```

parse with parse.pm:

```
cd $PARSING_PIPELINE_DIR
./parse.pm -c es.cfg 
```

use `./parse.pm --help` to see input/output format options

As an example for how to add co-reference annotations to your conll with corzu, see coref_example.sh


