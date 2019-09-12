# Syllabifiers for language learning models
A Python package for syllabifying text data in English or Polish

## Description

This Python package contains programs that can extract syllabic cues from text data in English or Polish, whether the data consist of a word, a sentence or a whole corpus. The resulting syllabic cues can be used as inputs to train language learning models that accepts data in an 'associative learning' style (i.e. each training example that is fed to the model is an event made up of some cues that trigger some outcomes) such as the naive discriminative learning (NDL; https://pyndl.readthedocs.io/en/latest/index.html) or the models provided in the Deep text modelling package (DTM; https://github.com/Adnane017/Deep_text_modelling). One motivation behind the developement of this package is a study that aims to compare how well orthographic and phonetic methods can explain response time in reading.   

Our syllabication method consists of two stages: 

1) We transcribe the text in IPA (International Phonetic Alphabet). This is done using the 'Epitran' package (https://github.com/dmort27/epitran/)
2) We extract all the syllables from each transcribed word seperately. We assume that a syllable is made of one vowel along with the consonants that surround it. For example, if a word is of the form #cvccvcv# ('c' and 'v' refer to a consonant and a vowel respectively; # marks the beginning or the ending of the word), then the syllabic cues will be #cvcc_ccvc_cv#.

## Usage

This section presents the main functions that are offered in the package along with some basic examples illustrating their usage. For more detailed examples, see the 'examples' section below. 

### Syllabify a sentence

To syllabify a sentence, use the following function: 

```python
syllabify_line(line, ipa_converter, syllable_pattern, not_symbol_pattern, add_boundaries, as_event)
```

* `line` - Line to syllabify.
* `ipa_converter` - Function that converts a string to ipa. For English, use `english_to_ipa` and for Polish, use `polish_to_ipa`.
* `syllable_pattern` - Tuple of compiled regular expressions that matche the syllables in a word.
* `not_symbol_pattern` - Compiled regular expression that matches disallowed characters like punctuation.
* `add_boundaries` - Whether or not to add hashtags to the outermost syllables to mark the begining and end of the word.
* `as_event` - Whether or not to return a (cues, outcomes)-tuple,
        where the cues are all the syllables and the outcomes are all the words, separated by underscores. If false, a list of the syllabic cues is a returned.

For English, the workflow would be: 

```python
import syllabifiers.ipa as sy 
import regex
sent_en = "Imagination is more important than knowledge"
ipa_converter_en = sy.english_to_ipa
syllable_pattern_en = sy.regex_en_ipa 
ENGLISH = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ" # list of allowed English characters
not_symbol_pattern_en = regex.compile(f"[^{ENGLISH}]") # Matches disallowed English characters
sy.syllabify_line(line = sent_en, 
                  ipa_converter = ipa_converter_en, 
                  syllable_pattern = syllable_pattern_en, 
                  not_symbol_pattern = not_symbol_pattern_en, 
                  add_boundaries = True, 
                  as_event = True)
>>> ('#ɪm_mæd͡ʒ_d͡ʒən_nejʃ_ʃən#_#ɪz#_#mɔɹ#_#ɪmp_mpɔɹt_ɹtənt#_#ðæn#_#nɑl_ləd͡ʒ#', 'imagination_is_more_important_than_knowledge')
```

For Polish:

```python
import syllabifiers.ipa as sy 
import regex
sent_pol = "Miło cię  spotkać!"
ipa_converter_pol = sy.polish_to_ipa
syllable_pattern_pol = sy.regex_pol_ipa
POLISH = 'aąbcćdeęfghijklłmnńoóprsśtuwyzźżqvxAĄBCĆDEĘFGHIJKLŁMNŃOÓPRSŚTUWYZŹŻQVX' # list of allowed Polish characters
not_symbol_pattern_pol = regex.compile(f"[^{POLISH}]") # Matches disallowed Polish characters
sy.syllabify_line(line = sent_pol, 
                  ipa_converter = ipa_converter_pol, 
                  syllable_pattern = syllable_pattern_pol, 
                  not_symbol_pattern = not_symbol_pattern_pol, 
                  add_boundaries = True, 
                  as_event = True)
>>> ('#miw_wɔ#_#t͡ɕɛ#_#spɔtk_tkat͡ɕ#', 'miło_cię_spotkać')
```

### Syllabify a corpus

There is also a syllabifier that accepts a path to a corpus file as input, and return an event file that contains both the syllabic cues and outcomes (all the words in each sentence). The function is defined as follows:

```python
syllabify_corpus(corpus_path, event_file_path, ipa_converter, syllable_pattern, not_symbol_pattern, add_boundaries, numcores, chunksize)
```

* `corpus_path` - Path to the corpus to syllabify.
* `event_file_path` - Path to the event file that will be generated, which contains the syllabic cues and outcomes.
* `ipa_converter` - Function that converts a string to ipa. For English, use `english_to_ipa` and for Polish, use `polish_to_ipa`.
* `syllable_pattern` - Tuple of compiled regular expressions that match the syllables in a word.
* `not_symbol_pattern` - Compiled regular expression that matches disallowed characters like punctuation.
* `add_boundaries` - Whether or not to add hashtags to the outermost syllables to mark the begining and end of the word.
* `numcores` - Number of cores to use.
* `chunksize` - Number of lines each process will work on in parallel.

For English, the workflow is as follows: 

```python
import syllabifiers.ipa as sy 
import regex
CORPUS_EN =  'Corpus_sample_en.txt' 
S2L_EVENT_EN = 'S2L_events_en.gz' 
ipa_converter_en = sy.english_to_ipa
syllable_pattern_en = sy.regex_en_ipa
ENGLISH = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ" # list of allowed English characters
not_symbol_pattern_en = regex.compile(f"[^{ENGLISH}]") # Matches disallowed English characters
sy.syllabify_corpus(corpus_path = CORPUS_EN,
                    event_file_path = S2L_EVENT_EN,
                    ipa_converter = ipa_converter_en,
                    syllable_pattern = syllable_pattern_en,
                    not_symbol_pattern = not_symbol_pattern_en,
                    add_boundaries = True,
                    numcores = 8,
                    chunksize = 125)
### Print the first 3 lines from the S2L event file 
with gzip.open(S2L_EVENTS_POL, 'rt', encoding='utf-8') as e:   
    for x in range(3):
        print(next(e))
>>> '#ðə#_#mejd͡ʒ_d͡ʒəɹ#_#ɪmp_mpækt#_#ɪz#_#jɛt#_#tə#_#kʌm#	the_major_impact_is_yet_to_come'
    '#ɹejz_zɪŋ#_#mʌn_ni#_#fɔɹ#_#jɔɹ#_#fejv_vəɹ_ɹɪt#_#t͡ʃɛɹ_ɹɪt_ti#_#kæn#_#bi#_#fʌn#	raising_money_for_your_favourite_charity_can_be_fun'
    '#dɪd#_#ju#_#now#	did_you_know'
```

For Polish:

```python
import syllabifiers.ipa as sy 
import regex
CORPUS_POL =  'Corpus_sample_pol.txt' 
S2L_EVENTS_POL = 'S2L_events_pol.gz' 
ipa_converter_pol = sy.polish_to_ipa
syllable_pattern_pol = sy.regex_pol_ipa
POLISH = 'aąbcćdeęfghijklłmnńoóprsśtuwyzźżqvxAĄBCĆDEĘFGHIJKLŁMNŃOÓPRSŚTUWYZŹŻQVX' # list of allowed Polish characters
not_symbol_pattern_pol = regex.compile(f"[^{POLISH}]") # Matches disallowed Polish characters
sy.syllabify_corpus(corpus_path = CORPUS_POL,
                    event_file_path = S2L_EVENT_POL,
                    ipa_converter = sy.polish_to_ipa,
                    syllable_pattern = syllable_pattern_pol,
                    not_symbol_pattern = not_symbol_pattern_pol,
                    add_boundaries = True,
                    numcores = 8,
                    chunksize = 125)
### Print the first 3 lines from the S2L event file 
with gzip.open(S2L_EVENTS_POL, 'rt', encoding='utf-8') as e:   
    for x in range(3):
        print(next(e))
>>> '#nadmj_dmjɛrn_rnɛ#_#napj_pjɛnt͡ɕ_nt͡ɕɛ#_#ɡrup_pɨ#_#mjɛ̃ɕɲ_ɕɲi#_#pɔv_vɔd_duj_jɛ#_#ɲɛpr_prav_vidw_dwɔv_vɛ#_#ust_stavj_vjɛɲ_ɲɛ#_#stavj_vjɛ#	nadmierne_napięcie_grupy_mięśni_powoduje_nieprawidłowe_ustawienie_w_stawie'
    '#mjɛ̃ɕɲ_ɕɲɛ#_#spast_stɨt͡ʂn_t͡ʂnɛ#_#ɲɛ#_#sɔ̃#_#mjɛ̃ɕɲ_ɕɲam_mi#_#pɔr_raʐ_ʐɔn_nɨm_mi#_#i#_#mɔɡ_ɡɔ̃#_#ɕɛ#_#kurt͡ʂ_rt͡ʂɨt͡ɕ#	mięśnie_spastyczne_nie_są_mięśniami_porażonymi_i_mogą_się_kurczyć'
    '#t͡sɔ#_#vjɛnt͡s_nt͡sɛj#_#t͡ʂɛ̃st_stɔ#_#ɲɛ#_#sɔ̃#_#mjɛ̃ɕɲ_ɕɲam_mi#_#ɕiln_lnɨm_mi#	co_więcej_często_nie_są_mięśniami_silnymi'
```

## Examples

- [Syllabification of English texts](https://nbviewer.jupyter.org/github/Adnane017/ipa_syllabifiers/blob/master/illustrative_examples/English_syllabification_examples.ipynb)

- [Syllabification of Polish texts](https://nbviewer.jupyter.org/github/Adnane017/ipa_syllabifiers/blob/master/illustrative_examples/Polish_syllabification_examples.ipynb)

## Installation

All you need to start using the package is to copy the folder inside 'package' in your computer and make it as your working directory in Python. You will also need to install the following packages:

- epitran
- regex

## Authors

Adnane Ez-zizi (I am very grateful to Christian Adam for optimising an earlier version of the code)



