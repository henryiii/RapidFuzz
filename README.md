<h1 align="center">
<img src="https://raw.githubusercontent.com/maxbachmann/RapidFuzz/main/docs/img/RapidFuzz.svg?sanitize=true" alt="RapidFuzz" width="400">
</h1>
<h4 align="center">Rapid fuzzy string matching in Python and C++ using the Levenshtein Distance</h4>

<p align="center">
  <a href="https://github.com/maxbachmann/RapidFuzz/actions">
    <img src="https://github.com/maxbachmann/RapidFuzz/workflows/Test%20Build/badge.svg"
         alt="Continuous Integration">
  </a>
  <a href="https://pypi.org/project/rapidfuzz/">
    <img src="https://img.shields.io/pypi/v/rapidfuzz"
         alt="PyPI package version">
  </a>
  <a href="https://anaconda.org/conda-forge/rapidfuzz">
    <img src="https://img.shields.io/conda/vn/conda-forge/rapidfuzz.svg"
         alt="Conda Version">
  </a>
  <a href="https://www.python.org">
    <img src="https://img.shields.io/pypi/pyversions/rapidfuzz"
         alt="Python versions">
  </a><br/>
  <a href="https://maxbachmann.github.io/RapidFuzz">
    <img src="https://img.shields.io/badge/-documentation-blue"
         alt="Documentation">
  </a>
  <a href="https://github.com/maxbachmann/RapidFuzz/blob/main/LICENSE">
    <img src="https://img.shields.io/github/license/maxbachmann/rapidfuzz"
         alt="GitHub license">
  </a>
</p>

<p align="center">
  <a href="#description">Description</a> •
  <a href="#installation">Installation</a> •
  <a href="#usage">Usage</a> •
  <a href="#license">License</a>
</p>

---

## Description
RapidFuzz is a fast string matching library for Python and C++, which is using the string similarity calculations from [FuzzyWuzzy](https://github.com/seatgeek/fuzzywuzzy). However there are a couple of aspects that set RapidFuzz apart from FuzzyWuzzy:
1) It is MIT licensed so it can be used whichever License you might want to choose for your project, while you're forced to adopt the GPL license when using FuzzyWuzzy
2) It provides many string_metrics like hamming or jaro_winkler, which are not included in FuzzyWuzzy
3) It is mostly written in C++ and on top of this comes with a lot of Algorithmic improvements to make string matching even faster, while still providing the same results. For detailed benchmarks check the [documentation](https://maxbachmann.github.io/RapidFuzz/fuzz.html)
4) Fixes multiple bugs in the `partial_ratio` implementation

## Requirements

- Python 3.6 or later
- On Windows the [Visual C++ 2019 redistributable](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads) is required

## Installation

There are several ways to install RapidFuzz, the recommended methods
are to either use `pip`(the Python package manager) or
`conda` (an open-source, cross-platform, package manager)

### with pip

RapidFuzz can be installed with `pip` the following way:

```bash
pip install rapidfuzz
```

There are pre-built binaries (wheels) of RapidFuzz for MacOS (10.9 and later), Linux x86_64 and Windows. Wheels for armv6l (Raspberry Pi Zero) and armv7l (Raspberry Pi) are available on [piwheels](https://www.piwheels.org/project/rapidfuzz/).

> :heavy_multiplication_x: &nbsp;&nbsp;**failure "ImportError: DLL load failed"**
>
> If you run into this error on Windows the reason is most likely, that the [Visual C++ 2019 redistributable](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads) is not installed, which is required to find C++ Libraries (The C++ 2019 version includes the 2015, 2017 and 2019 version).

### with conda

RapidFuzz can be installed with `conda`:

```bash
conda install -c conda-forge rapidfuzz
```

### from git
RapidFuzz can be installed directly from the source distribution by cloning the repository. This requires a C++17 capable compiler.

```bash
git clone --recursive https://github.com/maxbachmann/rapidfuzz.git
cd rapidfuzz
pip install .
```

## Usage
Some simple functions are shown below. A complete documentation of all functions can be found [here](https://maxbachmann.github.io/RapidFuzz/Usage/index.html).

### Scorers
Scorers in RapidFuzz can be found in the modules `fuzz` and `string_metric`.

#### Simple Ratio
```console
> fuzz.ratio("this is a test", "this is a test!")
96.55171966552734
```

#### Partial Ratio
```console
> fuzz.partial_ratio("this is a test", "this is a test!")
100.0
```

#### Token Sort Ratio
```console
> fuzz.ratio("fuzzy wuzzy was a bear", "wuzzy fuzzy was a bear")
90.90908813476562
> fuzz.token_sort_ratio("fuzzy wuzzy was a bear", "wuzzy fuzzy was a bear")
100.0
```

#### Token Set Ratio
```console
> fuzz.token_sort_ratio("fuzzy was a bear", "fuzzy fuzzy was a bear")
83.8709716796875
> fuzz.token_set_ratio("fuzzy was a bear", "fuzzy fuzzy was a bear")
100.0
```

### Process
The process module makes it compare strings to lists of strings. This is generally more
performant than using the scorers directly from Python.
Here are some examples on the usage of processors in RapidFuzz:

```console
> from rapidfuzz import process, fuzz
> choices = ["Atlanta Falcons", "New York Jets", "New York Giants", "Dallas Cowboys"]
> process.extract("new york jets", choices, scorer=fuzz.WRatio, limit=2)
[('New York Jets', 100, 1), ('New York Giants', 78.57142639160156, 2)]
> process.extractOne("cowboys", choices, scorer=fuzz.WRatio)
("Dallas Cowboys", 90, 3)
```

The full documentation of processors can be found [here](https://maxbachmann.github.io/RapidFuzz/Usage/process.html)

## Benchmark

The following benchmark gives a quick performance comparison between RapidFuzz and FuzzyWuzzy.
More detailed benchmarks for the string metrics can be found in the [documentation](https://maxbachmann.github.io/RapidFuzz/fuzz.html). For this simple comparison I generated a list of 10.000 strings with length 10, that is compared to a sample of 100 elements from this list:
```python
words = [
    "".join(random.choice(string.ascii_letters + string.digits) for _ in range(10))
    for _ in range(10_000)
]
samples = words[:: len(words) // 100]
```

The first benchmark compares the performance of the scorers in FuzzyWuzzy and RapidFuzz when they are used directly
from Python in the following way:
```python3
for sample in samples:
  for word in words:
    scorer(sample, word)
```
The following graph shows how many elements are processed per second with each of the scorers. There are big performance differences between the different scorers. However each of the scorers is faster in RapidFuzz

<img src="https://raw.githubusercontent.com/maxbachmann/RapidFuzz/main/docs/img/scorer.svg?sanitize=true" alt="Benchmark Scorer">

The second benchmark compares the performance when the scorers are used in combination with cdist in the following
way:
```python3
cdist(samples, words, scorer=scorer)
```
The following graph shows how many elements are processed per second with each of the scorers. In RapidFuzz the usage of scorers through processors like `cdist` is a lot faster than directly using it. That's why they should be used whenever possible.

<img src="https://raw.githubusercontent.com/maxbachmann/RapidFuzz/main/docs/img/cdist.svg?sanitize=true" alt="Benchmark cdist">


## Support the project

If you are using RapidFuzz for your work and feel like giving a bit of your own benefit back to support the project, consider sending us money through GitHub Sponsors or PayPal that we can use to buy us free time for the maintenance of this great library, to fix bugs in the software, review and integrate code contributions, to improve its features and documentation, or to just take a deep breath and have a cup of tea every once in a while. Thank you for your support.

Support the project through [GitHub Sponsors]() or via [PayPal](https://www.paypal.com/donate/?hosted_button_id=VGWQBBD5CTWJU):

[![](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/donate/?hosted_button_id=VGWQBBD5CTWJU).

## License
RapidFuzz is licensed under the MIT license since I believe that everyone should be able to use it without being forced to adopt the GPL license. That's why the library is based on an older version of fuzzywuzzy that was MIT licensed as well.
This old version of fuzzywuzzy can be found [here](https://github.com/seatgeek/fuzzywuzzy/tree/4bf28161f7005f3aa9d4d931455ac55126918df7).
