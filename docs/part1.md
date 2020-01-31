
# Table of Contents

-   [Part 1](#org598592d)
-   [About This Document](#org9ad22aa)
-   [The Story of an African Chef](#org2edc59b)
-   [The First CSV Statement](#org4eda956)



<a id="org598592d"></a>

# Part 1

This is the first part in a a series of step-by-step instructions.

They are intended to be read in sequence. Head over to the [docs README](README.md) to see all parts.


<a id="org9ad22aa"></a>

# About This Document

This document is a [literate](https://www.offerzen.com/blog/literate-programming-empower-your-writing-with-emacs-org-mode) [program](https://orgmode.org/worg/org-contrib/babel/intro.html).
You can read it like a normal article, either [on the web](https://github.com/apauley/hledger-flow/blob/master/docs/part1.org) or [as a slide show](https://pauley.org.za/hledger-flow/).

But you can also [clone the repository](https://github.com/apauley/hledger-flow) and open [this org-mode file](https://raw.githubusercontent.com/apauley/hledger-flow/master/docs/part1.org) in emacs.
Then you can execute each code snippet by pressing `C-c C-c` with your cursor on the relevant code block.


<a id="org2edc59b"></a>

# The Story of an African Chef

Meet Gawie de Groot. Gawie is a successful [mopane worm](https://en.wikipedia.org/wiki/Gonimbrasia_belina#As_food) chef, based in the northernmost section of the [Kruger National Park](https://en.wikipedia.org/wiki/Kruger_National_Park).

![img](./img/mopane-worm-meal.jpg)

In times gone by he used to work in a big city as an IT professional.
But after a few years he decided to choose a quieter life in the bushveld.

He now works at a renowned bush restaurant aptly named the "`Grillerige Groen Goggatjie`".

Demand for his traditional African cuisine has sky-rocketed, with visitors from all over Zimbabwe, Mozambique and South Africa
coming to enjoy his dishes.

Business has been good the last few years, but Gawie wants to make sure that he'll also be OK when business is down.

It is time for Gawie to take charge of his finances.


<a id="org4eda956"></a>

# The First CSV Statement

Gawie followed the [build instructions](https://github.com/apauley/hledger-flow#build-instructions) and ended up with his very own `hledger-flow` executable.

Let's just run this `hledger-flow` command and see what happens&#x2026;

    hledger-flow

    An hledger workflow focusing on automated statement import and classification:
    https://github.com/apauley/hledger-flow#readme
    
    Usage: hledger-flow ([-v|--verbose] [-H|--hledger-path HLEDGER-PATH]
                        [--show-options] [--sequential] (import | report) |
                        (-V|--version))
    
    Available options:
      -h,--help                Show this help text
      -v,--verbose             Print more verbose output
      -H,--hledger-path HLEDGER-PATH
                               The full path to an hledger executable
      --show-options           Print the options this program will run with
      --sequential             Disable parallel processing
      -V,--version             Display version information
    
    Available commands:
      import                   Converts electronic transactions into categorised journal files
      report                   Generate Reports

Well OK, at least it prints some helpful output.

Gawie wants to import his CSV statements.

Luckily the `import` subcommand has a bit of help text as well:

    hledger-flow import --help

    Usage: hledger-flow import [BASEDIR] [-H|--hledger-path HLEDGER-PATH]
                               [-v|--verbose] [--show-options] [--sequential]
      Converts CSV transactions into categorised journal files
    
    Available options:
      BASEDIR                  The hledger-flow base directory
      -H,--hledger-path HLEDGER-PATH
                               The full path to an hledger executable
      -v,--verbose             Print more verbose output
      --show-options           Print the options this program will run with
      --sequential             Disable parallel processing
      -h,--help                Show this help text

Gawie starts by creating a new directory specifically for his hledger journals:

    mkdir my-finances

Now he can point the import to his new finances directory:

    hledger-flow import ./my-finances

Hmmm, an error:

    I couldn't find any input files underneath my-finances/import/
    
    hledger-makitso expects to find its input files in specifically
    named directories.
    
    Have a look at the documentation for a detailed explanation:
    https://github.com/apauley/hledger-flow#input-files

Gawie carefully interprets the error message using the skills he obtained during his years as an IT professional.

He concludes that `hledger-flow` expects to find its input files in specifically named directories.

Looking at the [documentation](https://github.com/apauley/hledger-flow#input-files) he sees there should be several account and bank-specific directories
under the `import` directory.

Gawie's salary is deposited into his cheque account at `Bogart Bank`, so this seems like a good account to start with:

    mkdir -p my-finances/import/gawie/bogart/cheque/1-in/2016/
    cp Downloads/Bogart/123456789_2016-03-30.csv \
      my-finances/import/gawie/bogart/cheque/1-in/2016/

Let's see what our tree structure looks like now:

    tree my-finances/

    my-finances/
    └── import
        └── gawie
            └── bogart
                └── cheque
                    └── 1-in
                        └── 2016
                            └── 123456789_2016-03-30.csv
    
    6 directories, 1 file

It is time to add what we have to source control.

    cd my-finances/
    git init .
    git add .
    git commit -m 'Initial commit'
    cd ..

Let's try the import again:

    hledger-flow import ./my-finances

    I couldn't find an hledger rules file while trying to import
    import/gawie/bogart/cheque/1-in/2016/123456789_2016-03-30.csv
    
    I will happily use the first rules file I can find from any one of these 2 files:
    import/gawie/bogart/cheque/bogart-cheque.rules
    import/bogart.rules
    
    Here is a bit of documentation about rules files that you may find helpful:
    https://github.com/apauley/hledger-flow#rules-files

Another cryptic error.

This one is caused by a missing [rules file](https://github.com/apauley/hledger-flow#the-rules-file).

After looking through the [hledger documentation on CSV rules files](http://hledger.org/csv.html),
Gawie concludes that the dates in Bogart Bank's CSV statement is incompatible with basic logic, reason and decency.

Luckily he isn't the only one suffering at the hands of bureaucratic incompetence: someone else has already written [a script](https://github.com/apauley/fnb-csv-demoronizer) to
fix stupid dates like those used by Bogart Bank.

This looks like a job for a [preprocess script](https://github.com/apauley/hledger-flow#the-preprocess-script).

Gawie adds the CSV transformation script as a submodule to his repository:

    cd my-finances/
    git submodule add https://github.com/apauley/fnb-csv-demoronizer.git
    git commit -m 'Added submodule: fnb-csv-demoronizer'
    cd ..

`hledger-flow` looks for a file named [preprocess](https://github.com/apauley/hledger-flow#the-preprocess-script) in the account directory.

Gawie just creates a symbolic link named `preprocess`.
This works because the downloaded script takes an input file and an output file as the first two positional arguments,
very much as the `preprocess` script would expect.
And luckily it ignores the other parameters that `hledger-flow` sends through.

    cd my-finances/import/gawie/bogart/cheque
    ln -s ../../../../fnb-csv-demoronizer/fnb-csv-demoronizer preprocess

Now when we try the import again, it still displays an error due to our missing rules file:

    hledger-flow import ./my-finances

    I couldn't find an hledger rules file while trying to import
    import/gawie/bogart/cheque/2-preprocessed/2016/123456789_2016-03-30.csv
    
    I will happily use the first rules file I can find from any one of these 2 files:
    import/gawie/bogart/cheque/bogart-cheque.rules
    import/bogart.rules
    
    Here is a bit of documentation about rules files that you may find helpful:
    https://github.com/apauley/hledger-flow#rules-files

This time we can see that our statement was preprocessed despite the rules file error:

    head -n 2 my-finances/import/gawie/bogart/cheque/2-preprocessed/2016/123456789_2016-03-30.csv

    "5","'Nommer'","'Datum'","'Beskrywing1'","'Beskrywing2'","'Beskrywing3'","'Bedrag'","'Saldo'","'Opgeloopte Koste'"
    "5","1","2016-03-01","#Monthly Bank Fee","","","-500.00","40000.00",""

Time for another git checkpoint.

    cd my-finances/
    git add .
    git commit -m 'The preprocessed CSV now has dates we can work with!'
    cd ..

Now that we have sane dates in a CSV file, let's try to create a [rules file](http://hledger.org/manual.html#csv-rules):

    skip 1
    
    fields _, _, date, desc1, desc2, desc3, amount, balance, _
    
    currency R
    status *
    
    account1 Assets:Current:Gawie:Bogart:Cheque
    description %desc1/%desc2/%desc3

Gawie saves this file as `my-finances/import/gawie/bogart/cheque/bogart-cheque.rules`.

Time for another git checkpoint.

    cd my-finances/
    git add .
    git commit -m 'A CSV rules file'
    cd ..

This time the import is successful, and we see a number of newly generated files:

    hledger-flow import ./my-finances
    tree my-finances

    Collecting input files...
    Found 1 input files in 0.011479186s. Proceeding with import...
    Imported 1 journals in 0.208365781s
    my-finances
    ├── all-years.journal
    ├── fnb-csv-demoronizer
    │   ├── fnb-csv-demoronizer
    │   └── README.org
    └── import
        ├── 2016-include.journal
        ├── all-years.journal
        └── gawie
            ├── 2016-include.journal
            ├── all-years.journal
            └── bogart
                ├── 2016-include.journal
                ├── all-years.journal
                └── cheque
                    ├── 1-in
                    │   └── 2016
                    │       └── 123456789_2016-03-30.csv
                    ├── 2016-include.journal
                    ├── 2-preprocessed
                    │   └── 2016
                    │       └── 123456789_2016-03-30.csv
                    ├── 3-journal
                    │   └── 2016
                    │       └── 123456789_2016-03-30.journal
                    ├── all-years.journal
                    ├── bogart-cheque.rules
                    └── preprocess -> ../../../../fnb-csv-demoronizer/fnb-csv-demoronizer
    
    11 directories, 16 files

Bogart Bank's CSV file has been transformed into an `hledger` journal file.

This is the first transaction in the file:

    head -n 3 my-finances/import/gawie/bogart/cheque/3-journal/2016/123456789_2016-03-30.journal

    2016/03/01 * #Monthly Bank Fee//
        Assets:Current:Gawie:Bogart:Cheque        R-500.00 = R40000.00
        expenses:unknown                           R500.00

A final checkpoint and we're done with part 1.

    cd my-finances/
    git add .
    git commit -m 'My first imported journal'
    cd ..

The story continues with [part 2](part2.md).

