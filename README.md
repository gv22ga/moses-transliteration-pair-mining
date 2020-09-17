# moses-transliteration-pair-mining

Extract transliteration pairs from a parallel corpus using Moses

## Setup

### Basic setup
```
git clone https://github.com/gv22ga/moses-transliteration-pair-mining.git
cd moses-transliteration-pair-mining
```
For the next steps, please use `moses-transliteration-pair-mining` as the intial working directory

### Moses setup
```
export LC_ALL=C
sudo apt-get install g++ git subversion automake libtool zlib1g-dev libicu-dev libboost-all-dev libbz2-dev liblzma-dev python-dev graphviz imagemagick make cmake libgoogle-perftools-dev autoconf doxygen 
git clone https://github.com/moses-smt/mosesdecoder.git
cd mosesdecoder
./bjam -j4
```
Moses compilation takes ~10 minutes.

### Mgiza setup
```
git clone https://github.com/moses-smt/mgiza.git
cd mgiza/mgizapp
cmake .
make
make install
```

### Create external bin dir
```
mkdir external_bin
cp mgiza/mgizapp/bin/* external_bin/
cp mgiza/mgizapp/scripts/merge_alignment.py external_bin/
```

## Usage

I have put a small parallel corpus in `sample/corpus/`. `raw.en` contains 5K English sentences and `raw.hi` contains 5K corresponding Hindi sentences. There is also a sense of direction in the process, so here we will be generating transliteration pairs from `en->hi` side.

### Clean data for moses
```
export LC_ALL=C
./mosesdecoder/scripts/tokenizer/lowercase.perl < sample/corpus/raw.en > sample/corpus/raw_lowercase.en
./mosesdecoder/scripts/tokenizer/lowercase.perl < sample/corpus/raw.hi > sample/corpus/raw_lowercase.hi
./mosesdecoder/scripts/training/clean-corpus-n.perl sample/corpus/raw_lowercase en hi sample/corpus/clean 1 50
```

### Run train-model.pl
```
./mosesdecoder/scripts/training/train-model.perl -root-dir sample/ --corpus sample/corpus/clean --f en --e hi -external-bin-dir external_bin/ -mgiza --last-step=3
```
This step will be quick for the provided sample. But it will take over ~1 hour for a corpus with ~500K sentence pairs.

### Run train-transliteration-module.pl
```
./mosesdecoder/scripts/Transliteration/train-transliteration-module.pl --corpus-f /home/gaurav.g/moses/sample/corpus/clean.en --corpus-e /home/gaurav.g/moses/sample/corpus/clean.hi --alignment /home/gaurav.g/moses/sample/model/aligned.grow-diag-final --moses-src-dir mosesdecoder/ --external-bin-dir external_bin/ --input-extension en --output-extension hi  --out-dir sample/transliteration_model --srilm-dir .
```
For some parameters, you need to provide the full paths, otherwise it will throw errors. This command will actually end with an error message `sh: 1: ./ngram-count: not found` since we haven't properly configured `srilm`, but it doesn't affect this use case.

## Phew! That's it
You will find the transliteration pairs here `sample/transliteration_model/1-1.en-hi.pair-probs`. This file also has a score value associated with each pair. <b> Score of 0 is good, 1 is bad </b>. A threshold value of `0.001` works good to filter the correct transliteration pairs.

