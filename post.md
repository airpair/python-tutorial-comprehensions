## 1 Comprehension and Generator Syntax

<!--code lang=python linenums=true-->

    [transform(datum) for datum in iterable if valid(datum)]
    {transform(datum) for datum in iterable if valid(datum)}
    {keyFun(key): valFun(val) for key,val in iterable.items() if valid(key,val)}
    (transform(datum) for datum in iterable if valid(datum))

One everyday task in programming is to construct a container having target data generated with valid source data from an iterable. Python comprehensions and syntactically supported generators have a convenient and efficient uniform syntax for performing this task; they make code easier to read when thoughtfully constructed.

These single-line code fragments enhance development and maintenance by containing loop complexity in an enclosed form.

These forms contain a four step process:

* Acquiring data from a source;
* Validating the data;
* Transforming the data; and
* Collecting the transformed data into a target container.

Typically, each step is done on a separate line of code;
but this sequence of steps is so common that Python has a special syntax for performing this task in a single line of code with protection from simple loop construction errors. In this article you will learn how to simplify this task.

* In Archetypes you will learn what the basic pattern is.
* In Defaults you will learn shortcuts.
* In Advantages you will learn how complexity is reduced.
* In Examples you will learn see a variety of use cases.
* In References you will see some other resources to study.

>  The difference between a comprehension and a generator is important for efficiency. A comprehension fully populates its container when it is executed with all the memory consumption required to hold the container and all its contents. A generator does not populate its container, but generates one item each time a value is requested from the object. _(From the [Python wiki](https://wiki.python.org/moin/Generators))_

## 2 Archetype Syntax

These are the maximum form expressions. Their full complexity is illustrated by these forms:

**List comprehension (full immediate generation):**

`[transform(datum) for datum in iterable if valid(datum)]`

**Set comprehension (full immediate generation):**

`{transform(datum) for datum in iterable if valid(datum)}`

**Dict comprehension (full immediate generation):**

`{keyFun(key): valFun(val) for key,val in iterable.items() if valid(key,val)}`

**Tuple generator (generation on demand):**

`(transform(datum) for datum in iterable if valid(datum))`

Where `transform` returns a value; `datum` (or `key,val`) are produced by an `iterable`; `iterable` is any object that produces values on repetitive calls; and `valid` returns a value that can be used as True or False.

> In Python, any code that produces a particular value without side effects is equivalent; whether it is a manifest constant,
a variable having a value, a function returning a value,
a class member having a value, or a class method returning a value. In these archetypes, convenient named functions are used for transform and valid.

## 3 Default Transforms and Validation

Transform and valid need not be functions, and valid need not even be used. For instance these two code fragments are list comprehensions:

<!--code lang=python linenums=true-->

    [random() for _ in range(10)] # a list of 10 pseudorandom floats between 0.0 and 1.0.

    [_ for _ in range(10) if _&1] # a list of odd integers between 0 and 9 inclusive.

Notice how the first code fragment has no function for valid (assumed True) and how the second code fragment has no transform function. The '_' (underscore) variable is used as a throwaway variable name.

## 4 Advantages over Standard Style

The form is simple: a container, a transform, a generator expression, and a validator. This never varies except for simplifications such as eliminating the transform and/or eliminating the validator.

The form is recognizable, even when exercised heavily such as when it is used recursively. This constraint is rigid, lending itself to easy readability.

The entire form is expressed in its own microcosm without the heaviness of a function definition. There is no chance of logical interference with surrounding code as long as one does not share variable names.

If one were to share variable names the impact is as follows:

1. For a comprehension, the variable value becomes the last value used.

2. For a generator, the variable value will be shared (potential bugs).

It is best to avoid risking such bugs and use anonymous variables such as '_' underscore, or unshared names.

The form implements an enforcement. Any loop to be expressed in the form of a comprehension or generator of this type
resembles any other of its form without concern for source formatting.

## 5 Examples

### 5.1 Generic file reader with whitespace stripping

This code fragment illustrates how the task is usually performed:

<!--code lang=python linenums=true-->

    container = list()
    with open(filename) as source:
        for line in source:
            if valid(line):
                container.append(line.strip())

This code fragment illustrates how the task is performed with
a comprehension preserving the logic:

<!--code lang=python linenums=true-->

    with open(filename) as source:
        container = [line.strip() for line in source if valid(line)]

You could use the following form if you don't anticipate problems from waiting for garbage collection to close the file or opening a large number of files simultaneously:

<!--code lang=python linenums=true-->

        container = [line.strip() for line in open(filename) if valid(line)]

### 5.2 Random number of random odd numbers

Suppose you want a list of all the odd numbers from a list of random numbers. The traditional programming approach is to use
a loop to make a source list of random numbers, then a loop to copy the odd members to the target list. 

The comprehension approach uses identical logic but appears much simplified:

<!--code lang=python linenums=true-->

    from random import *

    def Test_random_odd(source):
        # Here is the usual code to generate 10 random integers between 1 and 100.
        iterable = []
        for count in range(10):
            iterable.append(randint(1,100))

        # Here is the equivalent comprehension.
        iterable = [randint(1, 100) for _ in range(10)]

        """Traditional loop to extract odd values from an iterable"""
        target1 = []
        for number in source:
            if number & 1:
                target1.append(number)

        """Comprehension equivalent to extract odd values from an iterable"""
        target2 = [number for number in source if number & 1]

        result = 'random_odd.py produced %s results for loops and comprehensions.'
        print result % ['different', 'identical'][target1 == target2]

        # Here is how to do the entire task even more compactly with nested comprehensions.
        target3 = [N for N in [randint(1, 100) for _ in range(10)] if N&1]

### 5.3 Generate complete set of (x,y,z) integer coordinates within limits

<!--code lang=python linenums=true-->

    from itertools import (product)

    def Test_xyz(E=7):
        N = -(E/2)              # Negative maximum extent
        P = E+N                 # Positive maximum extent
        V = range(N,P)          # A list of all integer values along edge

        X = sorted(V*E*E)       # list of E*E copies of sequential values
        Y = sorted(V*E)*E       # list of E copies of sequential values E times
        Z = sorted(V)*E*E       # list of sequential values E*E times

        comprehension = [(x,y,z) for (x,y,z) in zip(X,Y,Z)]
        library = list(product(V, repeat=3))

        result = 'xy.py produced %s tuples with comprehensions and itertools.'
        print result % ['different', 'identical'][library == comprehension]

### 5.4 List comprehensions

Here is a function which fully exposes the use of a list comprehension. If the conditional argument is absent, it is assumed to yield True:

<!--code lang=python linenums=true-->

    def listComprehension(source, transform, conditional):
        """
        Canonical use of a list comprehension produces a completely generated
        list of values derived from elements in a source container
        for elements that obey a condition.
        """
        return [
                transform(element)       # An operation on an element
                for element in source    # applied to elements from the source
                if conditional(element)  # for elements meeting a condition.
               ]

The first example from the introduction could be reprogrammed as follows. Note the simple syntax used in `container3` in the function `Test3`:

<!--code lang=python linenums=true-->

    def stripped(x):
        return x.strip()

    def conditional(x):
        return stripped(x)

    def Test3(filename='test.txt'):
        # Three ways of doing the same thing: two cumbersome and one elegant.
        container1 = []
        with open(filename) as source:
            for line in source:
                if conditional(line.strip()):
                    container1.append(line.strip())
        container2 = listComprehension(open(filename), stripped, conditional)
        container3 = [stripped(line) for line in open(filename) if conditional(line)]
        # Proof
        assert container1 == container2 == container3

    Test3()

### 5.5 Set and Dictionary comprehensions

#### 5.5.1 Set

Canonical use of a set comprehension produces a completely generated set of keys derived from elements in a source container for elements that obey a condition.

<!--code lang=python linenums=true-->

    names = ['Smith', 'Jones', 'Smith', 'Doe', 'Jones', 'Smith']

    # Loop style production
    loop = set()
    for name in names:
        if name != 'Jones':
            loop.add(name)

    # Set comprehension production
    comprehension = {name for name in names if name != 'Jones'}

    result = 'name_set.py produced %s sets with a comprehension and a loop.'
    print result % ['different', 'identical'][loop == comprehension]

#### 5.5.2 Dictionary

Here is a function which fully exposes the use of a dictionary comprehension. If the conditional argument is absent, it is assumed to yield True.

<!--code lang=python linenums=true-->

    def dictionaryComprehension(source, keyTransform, valueTransform, conditional):
        """
        Canonical use of a dictionary comprehension produces a completely generated
        dictionary of key:value pairs derived from elements in a source container
        for elements that obey a condition.
        """
        return {
                keyTransform(key): valueTransform(value)    # operations
                for key, value in source.iteritems()        # applied to values
                if conditional(key, value)                  # for allowed values
               }

Here is a demonstration of building a dictionary assigning random number to people's names.

<!--code lang=python linenums=true-->

    def Test6():
        seed()
        names = ['Ann', 'Ben', 'Cal', 'Deb']
        folks = {name: random() for name in names}
        noBen = {name: random() for name in names if name is not 'Ben'}
        print folks
        print noBen

    Test6()

### 5.6 Tuple generator

Here is a function which fully exposes the use of a tuple generator. If the conditional argument is absent, it is assumed to yield True. The generator syntax resembles the list comprehension syntax:

<!--code lang=python linenums=true-->

    def tupleGenerator(source, transform, conditional):
        """
        Canonical use of a tuple generator produces
        values derived from elements in a source iterable for
        elements that obey a condition.
        This is useful for handling data too large to fit into memory.
        """
        return (
                transform(element)       # An operation on an element
                for element in source    # applied to elements from the source
                if conditional(element)  # for elements meeting a condition.
               )

Tuple generators "yield" one data element for each request.
They do not validate or transform source data until a request for an element is made.

<!--code lang=python linenums=true-->

    def Test7():
        # This generator does not read the file in immediately.
        gen = (line.strip() for line in open('test.txt') if line.strip())

        # This loop causes the generator to read the next non-blank line.
        for text in gen:  # This extracts non-blank lines on-the-fly
            print text

    Test7()

## 6 Discussion

Opinion: comprehensions eliminate some of what makes loop production of containers error-prone. I prefer comprehensions over loops anywhere I can use them.

* They can be used for weightless threading:
    * http://www.ibm.com/developerworks/library/l-pythrd/
    * https://pypi.python.org/pypi/weightless-core
* They can be used to generate data fields such as Example 3 above.
* They can be used as ordinary looping constructs (just discard produced values).
* They are part of the base language
    * https://docs.python.org/2/tutorial/datastructures.html.

Comprehensions are a lot of fun.

For no other reason than fun, I decided to generate syntax trees for the first sentences of 100 famous novels. _Tree.py_ makes recursive trees out of such sentences by splitting them using recursive comprehensions. This is nothing more than an exercise, much like practicing the piano to limber up one's fingers.

### 6.1 Tree.py: Making syntax trees

<!--code lang=python linenums=true-->

    #!/usr/bin/env python

    """Tree.py

    Usage:
        Tree.py [(-v | --verbose)] FILE
        Tree.py (-h | --help)
        Tree.py (--version)

    Options:
        -v, --verbose                   Show execution details [default: False]
        -h --help                       Show this screen
        --version                       Show version

    Author: Jonathan D. Lettvin
    LinkedIn: jlettvin
    Date: 20141020
    """

    class Tree(dict):

        def __init__(self, **kw):
            "Fetch the sentences into a numbered sentence dictionary."
            with open(kw['FILE']) as src:
                self.sentences = {
                        int(number): sentence.strip() for number, sentence in
                        [(left,right.partition('|')[0]) for left,after,right in
                            [line.partition('.') for line in src.readlines()]] }

        def __call__(self, **kw):
            "Generate branching tree of sentences from the root."
            for number, sentence in self.sentences.items():
                word = sentence.split()
                if not word:
                    continue
                if not word[0] in self.keys():
                    self[word[0]] = dict()
                self.recurse(self[word[0]], word[1:])
            return self

        def recurse(self, parent, word):
            "Generate a higher branch."
            if not word:
                return
            if not word[0] in parent.keys():
                parent[word[0]] = dict()
            self.recurse(parent[word[0]], word[1:])

    if __name__ == "__main__":
        from docopt import (docopt)
        kw = docopt(__doc__, version="0.0.1")

        tree = Tree(**kw)()
        def ascend(tree, word, n=0):
            if not tree[word]:
                print ' '*n, word
            for more in tree[word]:
                print ' '*n, word
                ascend(tree[word], more, n+1)

        for word in ['The', 'I', 'A']:
            print '-'*79
            ascend(tree, word)
        print '-'*79

**sentence.txt:** to accompany the above Tree.py
<!--code lang=python linenums=true-->

    1. 124 was spiteful.| - Toni Morrison, [Beloved](/od/belovedtonimorrison/a/aa_belovedquote.htm) (1987)    
    2. A screaming comes across the sky.| - Thomas Pynchon, Gravity's Rainbow (1973)    
    3. All this happened, more or less.| - Kurt Vonnegut, [Slaughterhouse-Five](/od/slaughterhousefive/a/aa_slaughterqu.htm) (1969)    
    4. Call me Ishmael.| - Herman Melville, [Moby Dick](/od/mobydickhermanmelville/a/aa_mobydickqu.htm) (1851)    
    5. Happy families are all alike; every unhappy family is unhappy in its own way.| - Leo Tolstoy, [Anna Karenina](/od/annakarenina/a/aa_annakareninaquote.htm) (1877; trans. Constance Garnett)    
    6. He was an old man who fished alone in a skiff in the Gulf Stream and he had gone eighty-four days now without taking a fish.| - Ernest Hemingway, [The Old Man and the Sea](/od/oldmanthesea/fr/aa_oldman.htm) (1952)    
    7. I am a sick man... I am a spiteful man.| - Fyodor Dostoyevsky, Notes from Underground (1864; trans. Michael R. Katz)    
    8. I am an invisible man. No, I am not a spook like those who haunted Edgar Allan Poe; nor am I one of your Hollywood-movie ectoplasms. I am a man of substance, of flesh and bone, fiber and liquids -- and I might even be said to possess a mind. I am invisible, understand, simply because people refuse to see me.| - Ralph Ellison, [Invisible Man](/od/invisiblemanrellison/fr/aa_invisibleman.htm) (1952)    
    9. I had the story, bit by bit, from various people, and, as generally happens in such cases, each time it was a different story.| - Edith Wharton, [Ethan Frome](/od/ethanfromeedithwharton/a/aa_ethanfrome1.htm) (1911)    
    10. I was the shadow of the waxwing slain By the false azure in the windowpane| - [Vladimir Nabokov](/od/nabokovvladimir/p/Vladimir-Nabokov-Biography.htm), Pale Fire (1962)    
    11. If you really want to hear about it, the first thing you'll probably want to know is where I was born, and what my lousy childhood was like, and how my parents were occupied and all before they had me, and all that [David Copperfield](/od/davidcopperfielddickens/fr/aa_dcopperfield.htm) kind of crap, but I don't feel like going into it, if you want to know the truth.| - J. D. Salinger, [The Catcher in the Rye](/od/catcherintherye/fr/aa_catcherinrye.htm) (1951)    
    12. It is a truth universally acknowledged, that a single man in possession of a good fortune, must be in want of a wife.| - [Jane Austen](/cs/profileswriters/p/aa_jausten.htm), [Pride and Prejudice](/od/prideprejudice/fr/aa_prideprej.htm) (1813),    
    13. It was a bright cold day in April, and the clocks were striking thirteen.| - George Orwell, [1984 (Nineteen Eighty-Four)](/od/nineteeneightyfour/fr/aa_nineteen.htm) (1949)    
    14. It was a dark and stormy night; the rain fell in torrents, except at occasional intervals, when it was checked by a violent gust of wind which swept up the streets (for it is in London that our scene lies), rattling along the house-tops, and fiercely agitating the scanty flame of the lamps that struggled against the darkness.| - Edward George Bulwer-Lytton, Paul Clifford (1830)    
    15. It was the best of times, it was the worst of times, it was the age of wisdom, it was the age of foolishness, it was the epoch of belief, it was the epoch of incredulity, it was the season of Light, it was the season of Darkness, it was the spring of hope, it was the winter of despair.| - [Charles Dickens](/od/dickenscharles2/a/Charles-Dickens.htm), [A Tale of Two Cities](/od/soundandthefurywf/fr/aa_tale2cities.htm) (1859)    
    16. Many years later, as he faced the firing squad, Colonel Aureliano Buendia was to remember that distant afternoon when his father took him to discover ice.| - [Gabriel Garcia Marquez](/od/marquezgabrielgarcia/p/Gabriel-Garcia-Marquez-Biography.htm), [One Hundred Years of Solitude](/od/onehundredyears/fr/aa_100years.htm) (1967; trans. Gregory Rabassa)    
    17. Mrs. Dalloway said she would buy the flowers herself.| - Virginia Woolf, [Mrs. Dalloway](/od/mrsdalloway/fr/aa_mrsdalloway.htm) (1925)    
    18. Once an angry man dragged his father along the ground through his own orchard. &quot;Stop!&quot; cried the groaning old man at last, &quot;Stop! I did not drag my father beyond this tree.| - Gertrude Stein, The Making of Americans (1925)    
    19. Once upon a time and a very good time it was there was a moocow coming down along the road and this moocow that was coming down along the road met a nicens little boy named baby tuckoo.| - James Joyce, [A Portrait of the Artist as a Young Man](/od/portraitofanartist/fr/aa_portrait.htm) (1916)    
    20. One summer afternoon Mrs. Oedipa Maas came home from a Tupperware party whose hostess had put perhaps too much kirsch in the fondue to find that she, Oedipa, had been named executor, or she supposed executrix, of the estate of one Pierce Inverarity, a California real estate mogul who had once lost two million dollars in his spare time but still had assets numerous and tangled enough to make the job of sorting it all out more than honorary.| - Thomas Pynchon, The Crying of Lot 49 (1966)    
    21. Ships at a distance have every man's wish on board.| - Zora Neale Hurston, [Their Eyes Were Watching God](/od/theireyeswerewatching/fr/aa_their_eyes.htm) (1937)    
    22. Someone must have slandered Josef K., for one morning, without having done anything truly wrong, he was arrested.| - Franz Kafka, The Trial (1925; trans. Breon Mitchell)    
    23. Somewhere in la Mancha, in a place whose name I do not care to remember, a gentleman lived not long ago, one of those who has a lance and ancient shield on a shelf and keeps a skinny nag and a greyhound for racing.| - Miguel de Cervantes, [Don Quixote](/od/donquixotecervantes/fr/aa_donquixote.htm) (1605; trans. Edith Grossman)    
    24. Stately, plump Buck Mulligan came from the stairhead, bearing a bowl of lather on which a mirror and a razor lay crossed.| - James Joyce, [Ulysses](/od/joycejames9/fr/aa_ulysses.htm) (1922)    
    25. The sun shone, having no alternative, on the nothing new.| - Samuel Beckett, Murphy (1938),    
    26. There is a lovely road that runs from Ixopo into the hills. These hills are grass-covered and rolling, and they are lovely beyond any singing of it.| - Alan Paton, [Cry, the Beloved Country](/cs/productreviews/fr/aafpr_crythecty.htm) (1948)    
    27. There was a boy called Eustace Clarence Scrubb, and he almost deserved it.| - C. S. Lewis, The Voyage of the Dawn Treader (1952)    
    28. This is the saddest story I have ever heard.| - Ford Madox Ford, The Good Soldier (1915)    
    29. Through the fence, between the curling flower spaces, I could see them hitting.| - William Faulkner, [The Sound and the Fury](/od/soundandthefurywf/fr/aa_sound.htm) (1929)    
    30. When he was nearly thirteen, my brother Jem got his arm badly broken at the elbow.| - [Harper Lee](/od/leeharper/p/aa_harperlee.htm), [To Kill a Mockingbird](/od/tokillamockingbird/fr/aa_tokill.htm) (1960) 
    31. When Mr Bilbo Baggins of Bag End announced that he would shortly be celebrating his eleventy-first birthday with a party of special magnificence, there was much talk and excitement in Hobbiton.| - J.R.R. Tolkien (John Ronald Reuel Tolkien), The Lord of the Rings (1954-1955)    
    32. Whether I shall turn out to be the hero of my own life, or whether that station will be held by anybody else, these pages must show.| - [Charles Dickens](/od/dickenscharles2/a/Charles-Dickens.htm), [David Copperfield](/od/davidcopperfielddickens/fr/aa_dcopperfield.htm) (1850)    
    33. You are about to begin reading Italo Calvino's new novel, If on a winter's night a traveler.| - Italo Calvino, If on a winter's night a traveler (1979; trans. William Weaver)    
    34. You don't know about me without you have read a book by the name of The Adventures of Tom Sawyer; but that ain't no matter.| - Mark Twain, [Adventures of Huckleberry Finn](/od/adventuresofhuckleberry/fr/aa_huckfinn.htm) (1885)    
    35. Mother died today. Or, maybe, yesterday; I can't be sure. The telegram from the Home says: Your mother passed away. Funeral tomorrow. Deep sympathy. Which leaves the matter doubtful; it could have been yesterday.| - [Albert Camus](/cs/profileswriters/p/aa_acamus.htm), The Stranger, or The Outsider (1942; trans. Stuart Gilbert)    
    36. The sky above the port was the color of television, tuned to a dead channel.| - William Gibson, Neuromancer (1984)    
    37. Where now? Who now? When now?| - Samuel Beckett, The Unnamable (1953; trans. Patrick Bowles)    
    38. Dr. Weiss, at forty, knew that her life had been ruined by literature.| - Anita Brookner, The Debut (1981)    
    39. Ages ago, Alex, Allen and Alva arrived at Antibes, and Alva allowing all, allowing anyone, against Alex's admonition, against Allen's angry assertion: another African amusement . . . anyhow, as all argued, an awesome African army assembled and arduously advanced against an African anthill, assiduously annihilating ant after ant, and afterward, Alex astonishingly accuses Albert as also accepting Africa's antipodal ant annexation.| - Walter Abish, Alphabetical Africa (1974)

## 7 Additional references 
* [Python for beginners](http://www.pythonforbeginners.com/basics/list-comprehensions-in-python)
* [Python 2.7 core doc](https://docs.python.org/2/tutorial/datastructures.html)
* [Sage math](http://www.sagemath.org/doc/thematic_tutorials/tutorial-programming-python.html)