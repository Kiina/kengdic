kengdic
=======

kengdic is a large Korean/English dictionary database created by Joe
Speigle. It was originally hosted at ezkorean.com, and was made
available by Garfield Nate <https://github.com/garfieldnate/kengdic>
because it was no longer available anywhere else. This implementation
provides serverless access via a convenient SQLite API. The dictionary is
released under MPL 2.0.

Details
-------

From Garfield Nate <https://github.com/garfieldnate>:

> "I found a note from Joe Speigle at http://www.perapera.org/korean/
> last year (2013):
> 
>> the lowdown is that my dictionary has about 90,000 words in it of
>> which 70,000 have been human-verified by me, and has about 20,000
>> hanja which are linked to the correct definition though the word is
>> the same (and the hanja are different). This is more than either
>> English wiktionary (which has K as a definition) or the K-K
>> wiktionary. I have never found a more complete dictionary online or
>> anywhere for korean-English that is opensource as mine. That is why
>> I made it, people. I'm not into reduplicating the wheel as they
>> say. I have spent thousands of hours on this project. was it worth
>> it? ... Anybody can contact me about this, I am extremely willing
>> to share all knowledge I have about it. My email address is at
>> the bottom of the page for ezcorean.com
> 
> Unfortunately, it was my sad finding that Joe Speigle passed away last
> year. The Wayback machine indexed ezkorean.com, but did not index
> kengdic, his amazing dictionary. After quite a bit of searching, I
> found that Maran Emil Cristian had obtained and stored a copy. He was
> kind enough to copy it for me, and so now I am able to provide it here."

Installation
------------

Install the Python repository from GitHub:

```
pip install --user git+https://github.com/scottgigante/kengdic.git#subdir=python
```

Usage
-----

This repository provides a simple SQLite API to the kengdic dictionary. FIrst, create the dictionary object.

```
>>> import kengdic
>>> db = kengdic.Kengdic()
```

You can search for exact text matches in any of the dictionary fields: `korean`, `english`, `hanja` are the most useful. The others mostly aren't used.

```
>>> db.search(english="January")
[<class 'kengdic.kengdic.KengdicResult'>:
{'word_id': 100223, 'korean': '1월', 'synonym': None, 'english': 'January', 'part_of_speech_number': 1.0, 'part_of_speech': '1', 'submitter': 'engdic', 'date_of_entry': '2006-01-16 00:52:46', 'word_size': 4.0, 'hanja': None, 'word_id2': 100223, 'extra_data': 't'}, <class 'kengdic.kengdic.KengdicResult'>:
{'word_id': 209320, 'korean': '일월', 'synonym': None, 'english': 'January', 'part_of_speech_number': 1.0, 'part_of_speech': '1', 'submitter': 'joe', 'date_of_entry': '2009-01-01 11:23:14', 'word_size': 6.0, 'hanja': '一月', 'word_id2': 201725, 'extra_data': 'mtA'}]
>>> db.search(korean="일월")
[<class 'kengdic.kengdic.KengdicResult'>:
{'word_id': 209320, 'korean': '일월', 'synonym': None, 'english': 'January', 'part_of_speech_number': 1.0, 'part_of_speech': '1', 'submitter': 'joe', 'date_of_entry': '2009-01-01 11:23:14', 'word_size': 6.0, 'hanja': '一月', 'word_id2': 201725, 'extra_data': 'mtA'}]
>>> db.search(hanja="一月")
[<class 'kengdic.kengdic.KengdicResult'>:
{'word_id': 209320, 'korean': '일월', 'synonym': None, 'english': 'January', 'part_of_speech_number': 1.0, 'part_of_speech': '1', 'submitter': 'joe', 'date_of_entry': '2009-01-01 11:23:14', 'word_size': 6.0, 'hanja': '一月', 'word_id2': 201725, 'extra_data': 'mtA'}]
```

Kengdic also supports three partial match functions: 
* `search_like`: SQL LIKE function
* `search_glob`: Unix glob matching
* `search_regex`: Python regular expressions

```
>>> db.search_like(korean="일확_금")
[<class 'kengdic.kengdic.KengdicResult'>:
{'word_id': 224664, 'korean': '일확천금', 'synonym': None, 'english': 'to become rich without having made an effort. .  벼락부자  (a lightning rich person)', 'part_of_speech_number': 1.0, 'part_of_speech': '1', 'submitter': 'mr.hanja', 'date_of_entry': '2009-01-01 11:23:14', 'word_size': 12.0, 'hanja': '一攫千金', 'word_id2': 600168, 'extra_data': 'gssot2'}]
>>> db.search_glob(korean="일확?금")
[<class 'kengdic.kengdic.KengdicResult'>:
{'word_id': 224664, 'korean': '일확천금', 'synonym': None, 'english': 'to become rich without having made an effort. .  벼락부자  (a lightning rich person)', 'part_of_speech_number': 1.0, 'part_of_speech': '1', 'submitter': 'mr.hanja', 'date_of_entry': '2009-01-01 11:23:14', 'word_size': 12.0, 'hanja': '一攫千金', 'word_id2': 600168, 'extra_data': 'gssot2'}]
>>> db.search_regex(korean="일확.금")
[<class 'kengdic.kengdic.KengdicResult'>:
{'word_id': 224664, 'korean': '일확천금', 'synonym': None, 'english': 'to become rich without having made an effort. .  벼락부자  (a lightning rich person)', 'part_of_speech_number': 1.0, 'part_of_speech': '1', 'submitter': 'mr.hanja', 'date_of_entry': '2009-01-01 11:23:14', 'word_size': 12.0, 'hanja': '一攫千金', 'word_id2': 600168, 'extra_data': 'gssot2'}]
```

FIles
-----

* `kengdic.sql` is a PostgreSQL database dump of a the Korean/English dictionary as of 2009. It assumes the existence of a `modpgwebuser` schema, as well as a `korean_english_wordid_seq` sequence. It contains one table, `korean_english` with 143,795 rows. There is no primary key, but indexes are created on `doe`, `wordid`, and `word`. This table contains the following information:

    * Korean/English spelling
    * Hanja spellings, comma-separated
    * English definition
    * part of speech
    * source of the entry

* `kengdic_2011.tsv` is a tab-separated file containing a database dump of the same Korean/English dictionary, but instead of being a SQL file it is simply a tab-separated file containing all of the data. It contains 133,876 rows. As this is probably cleaner, newer, better data than `kengdic.sql`, it may not be worth cleaning up that file at all.

* `create_kengdic.sql` creates the kengdic database and sources the data in `kengdic_2011.tsv`. To run it:
    * Make sure postgres is running on your machine and run this command in the terminal: `psql < create_table.sql`
        * You may need to provide a user with the `-U` switch, like so: `psql -U username < create_table.sql`
    * This should create a database called 'kengdic' with one table called 'korean_english'
    * Run 'psql kengdic' and make sure everything imported correctly: "SELECT COUNT(\*) FROM korean_english;" should tell you there are 133,876 rows in the table.

* `ezcorean_6000.sql` is a database dump of 6000 common Korean words, along
with the definitions and hanja.

TODOs for each file:

* `kengdic.sql`:

    * fix entries where the entries are HTML containing both hangeul and hanja in text body
    * normalize Hanja (have relation table and hanja table separate instead of comma-separating them)
    * find the difference between `pos` and `posn`
    * find which numbers indicate which part of speech
    * add comments for all columns and tables
    * replace 'see 6000' in the definitions with the definition from the 6000 list from ezkorean.com
    * add primary key (I'm pretty sure that would be good; there are already indices, though)
    * export to sqlite

* `kengdic_2011.tsv`:

    * Do something about entries with no definition

Information may be incomplete, as we are still exploring and documenting the contents of this repository. Any contributions to information about this content would be much appreciated.
