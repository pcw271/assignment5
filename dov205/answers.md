
## Questions

**Describe the changes you made to make the game work with Python 3.**

The following changes were made:

```
File: adventure.py
Line: 37
From:
	line = raw_input('> ')
To:
	line = input('> ')
Reason:
	The raw_input() builtin function from Python 2 was deprecated 
	in favor of input() in Python 3.
```

```
File: test_walks.py
Line: 9 - 10
From:
#	tests.addTests(doctest.DocFileSuite('walkthrough1.txt'))
#	tests.addTests(doctest.DocFileSuite('walkthrough2.txt'))
To:
	tests.addTests(doctest.DocFileSuite('walkthrough1.txt'))
	tests.addTests(doctest.DocFileSuite('walkthrough2.txt'))
Reason:
	We would like to run all tests in our test suite.
```

**Describe three main techniques that the author used to structure the program.**

The author utilizes three main techniques to structure the program: modules, classes, and unit tests.

Modules allow the author to delegate concerns and single responsibilities to each method as defined within said modules.

Classes allow the author to group together different actions and actions associated with particular objects together for cohesion (e.g. Pirates, Dwarves, Rooms, etc.).

Finally, unit tests allow the author to systematically test their code and emphasizes test-driven development.

**Has the author used meaningful names? Give some examples of meaningful names used and what you think they mean. Give some examples of where the author has not used meaningful names.**

In many cases, the author *has* used meaningful names. For example, in `adventure.py`, we see the following code:

```python
if args.savefile is None:
	game = Game()
	load_advent_dat(game)
	game.start()
	baudout(game.output)
else:
	game = Game.resume(args.savefile)
	baudout('GAME RESTORED\n')

while not game.is_finished:
	line = raw_input('> ')
	words = re.findall(r'\w+', line)
	if words:
		baudout(game.do_command(words))
```

The code is highly literate: as a result of a cursory scan, we understand that if the user does not have a savefile a new game is created and started, whereas if they do it is resumed. Furthermore, while the game has not finished, the prompts continue so long as input is valid.

On the other hand, the author also has cases where the names are not meaningful. Consider the following code, also from `adventure.py`:

```python
def baudout(s):
	for c in s:
		sleep(9. / BAUD)  # 8 bits + 1 stop bit @ the given baud rate
		stdout.write(c)
		stdout.flush()
```

The variable name `s` does not help us reason about the function of the `baudout` function. Furthermore, the iterator `c` gives us no insight as to what kind of iterable object `s` is, aside from the nominal `stdout.write()` function.

Likewise, method names like `section1`, `section2`, etc. from `data.py` are only meaningful to the original implementor and do not serve others well.

**Do the functions used in the code do one thing? Give some examples of functions that only do one thing. Give some examples of functions that do more than one thing.**

No, not all functions used in the code do one thing. For example, consider the following code from `adventure.py`:

```python
def loop():
    parser = argparse.ArgumentParser(
        description='Adventure into the Colossal Caves.',
        prog='{} -m adventure'.format(os.path.basename(executable)))
    parser.add_argument(
        'savefile', nargs='?', help='The filename of game you have saved.')
    args = parser.parse_args()

    if args.savefile is None:
        game = Game()
        load_advent_dat(game)
        game.start()
        baudout(game.output)
    else:
        game = Game.resume(args.savefile)
        baudout('GAME RESTORED\n')

    while not game.is_finished:
        line = input('> ')
        words = re.findall(r'\w+', line)
        if words:
            baudout(game.do_command(words))         
```

Our method header implies a simple looping mechanism --- likely the while-loop defined at the tail of the method. That said, `loop` has distinct portions of the method reserved for defining and initializing the argument parser (`adventure.py` lines 20 - 25) and creating a new game if no save file is provided (`adventure.py` lines 27 - 34). The author may want to consider splitting this function into `start_game` and `loop`.

Many functions in the repository are good examples of single-responsibility functions, including `accumulate_message` in `adventure.py`

```python
def accumulate_message(dictionary, n, line):
    dictionary[n] = dictionary.get(n, '') + line + '\n'
```

as well as `play` and `resume` in `play.py`:

```python

def play(seed=None):
    """Turn the Python prompt into an Adventure game.
    With optional the `seed` argument the caller can supply an integer
    to start the Python random number generator at a known state.
    """
    global _game

    from game import Game
    from prompt import install_words

    _game = Game(seed)
    load_advent_dat(_game)
    install_words(_game)
    _game.start()
    print(_game.output[:-1])

def resume(savefile, quiet=False):
    global _game

    from game import Game
    from prompt import install_words

    _game = Game.resume(savefile)
    install_words(_game)
    if not quiet:
        print('GAME RESTORED\n')
```

All of the above functions are responsible for a single task, which makes unit testing much easier and modular for future developers. Note: I do not necessarily agree with the function contents (`global` definitions, lack of docstrings, etc.) but the functions themselves are concise and do what they are expected to do.

**Do any of the functions cause side effects? If so, which ones?**

Yes, some of the defined functions *do* cause side-effects. For example, consider the previously-mentioned function `loop` from `adventure.py`:

```python
def loop():
    parser = argparse.ArgumentParser(
        description='Adventure into the Colossal Caves.',
        prog='{} -m adventure'.format(os.path.basename(executable)))
    parser.add_argument(
        'savefile', nargs='?', help='The filename of game you have saved.')
    args = parser.parse_args()

    if args.savefile is None:
        game = Game()
        load_advent_dat(game)
        game.start()
        baudout(game.output)
    else:
        game = Game.resume(args.savefile)
        baudout('GAME RESTORED\n')

    while not game.is_finished:
        line = input('> ')
        words = re.findall(r'\w+', line)
        if words:
            baudout(game.do_command(words))         
```

As a user, we would expect `loop` to perform the final logical code block: while the current game has not finished, take an input, and evaluate the game's response before asking the user for more input. Instead, `loop` will create a new game instance or load the most recent save file before asking for more input.

**Can you find any repeated code that could be made into a function?**

We note that, within `game.py`, we have multiple instances of the following code

```python
	i_walk = ask_verb_what
	i_drop = ask_verb_what
	i_say = ask_verb_what
	i_nothing = say_okay_and_finish
	i_wave = ask_verb_what
	i_calm = ask_verb_what
	i_rub = ask_verb_what
	i_throw = ask_verb_what
	i_find = ask_verb_what
	i_feed = ask_verb_what
	i_break = ask_verb_what
	i_wake = ask_verb_what
```

if we instead had a function `i_do` that took a list of what the user does (e.g. 'walk', 'drop', 'say') and produced a key-value pair, this would significantly reduce the complexity and density of our code.

Not to bore with repeated examples, but the final logical code block within `adventure.py`'s `loop` method can be binned into a `receive_input` method.

**Does the program use exception handling? Can you find any input that causes the program to terminate abnormally? Hint: run the program from the terminal prompt. The invalid input may not be normal text.**

Yes, the program does use exception handling. In the driver `adventure.py` file, we see the author catch end-of-file errors should the user enter Control-D.

Likewise, should the user attempt to pass in an invalid save file -- any random text file or a file not defined in the local directory --while running the program from the terminal we see an abnormal exit case.

**Do any of the classes have responsibility over more than one piece of functionality. If so, which ones?**

The `Game` class as defined in `Game.py` shares the responsibilities of initializing a game object, keeping track of a large collection of attributes, receiving user input, and generally interfacing between the user and the defined classes. Relatively, classes like `Dwarves` and `Pirates` are split much more appropriately.

**Are all the classes cohesive? List any that aren’t.**

No, not all the classes are cohesive. For example, the `Room` class defines, upon initialization, numerous fields that are never utilized within its defined class methods (save for attribute `is_light`, which is "used" in the `is_dark` method.

**Describe the author’s approach to commenting the code. Provide examples of good and bad comments that have been used in the code.**

The author sparsely comments their code -- despite not having fully-literate code -- and when they do it often comes at no additional clarity or insight into the reasoning behind the particular implementation. There exists little trace of method docstrings, legal comments, explanation of intent, warnings, etc.

An example of a bad comment is seen on line 765 of `game.py`, where the author ponders life and says `# Death and reincarnation` before defining the code for the `die_here` and `die` methods.

By contrast, a good comment is seen on line 117 of `game.py` where the author clarifies the backwards-compatible, 5-letter-only truncated input options. This comment gives us better insight as to how the program will eventually evaluate user input.

**Provide an example of where vertical formatting has been used to make the code clearer.**

An example where vertical formatting has been utilized to make the code clearer can be seen in `data.py` from lines 67 - 105. 

```python
def section3(data, x, y, *verbs):
    last_travel = data._last_travel
    if last_travel[0] == x and last_travel[1][0] == verbs[0]:
        verbs = last_travel[1]  # same first verb implies use whole list
    else:
        data._last_travel = [x, verbs]

    m, n = divmod(y, 1000)
    mh, mm = divmod(m, 100)

    if m == 0:
        condition = (None,)
    elif 0 < m < 100:
        condition = ('%', m)
    elif m == 100:
        condition = ('not_dwarf',)
    elif 100 < m <= 200:
        condition = ('carrying', mm)
    elif 200 < m <= 300:
        condition = ('carrying_or_in_room_with', mm)
    elif 300 < m:
        condition = ('prop!=', mm, mh - 3)

    if n <= 300:
        action = make_object(data.rooms, Room, n)
    elif 300 < n <= 500:
        action = n  # special computed goto
    else:
        action = make_object(data.messages, Message, n - 500)

    move = Move()
    if len(verbs) == 1 and verbs[0] == 1:
        move.is_forced = True
    else:
        move.verbs = [ make_object(data.vocabulary, Word, verb_n)
                       for verb_n in verbs if verb_n < 100 ] # skip bad "109"
    move.condition = condition
    move.action = action
    data.rooms[x].travel_table.append(move)
```

Despite being rather messy, the `section3` method utilizes blank spaces between different logical portions of the code: when attempting to check `last_travel`, initializing variables, determining what interval `m` and `n` fall into, and finally performing the move command.

**Run the tests provided with the program. Do they pass or fail? Do you consider the tests meet the F.I.R.S.T. criteria? Provide details of why they do or do not meet the criteria.**

After fixing the issue in `adventure.py`, we note that the tests (as provided by `$ python -m unittest discover`) *do*, in fact, pass.

**Fast**: We don't have an exact idea of how quickly one test should take, but it seems reasonable that (locally) 14 tests run in approximately 4 seconds.

**Independent**: No, we do not hold the tests to be independent since in `test_commands.py` our test first attempts `walkthrough1.txt` before proceeding to `walkthrough2.txt`.

**Repeatable**: Assuming the entire directory is on the foreign devlopment/staging/production environment, we would expect the tests to be repeatable.

**Self-validating**: All tests either pass or fail -- determined by `True` or `False` return values -- and so we determine that the testing is close to objective.

**Timely**: We don't have an exact idea of how the author developed the code and how they developed the tests, but can infer that the tests were written side-by-side due to being grouped in the same commit.

Overall, we would say that the author's unit tests would not pass the F.I.R.S.T. tests. We further note two concerns regarding the tests:

1. Evidently, none of the tests properly replicate the user's experience---if we change the correct `input('> ')` to `raw_input('> ')` in `adventure.py`, we note that none of the tests fail --- almost as though nothing had changed.

2. Furthermore, the states testing in the `test_data.py` method `test_move_repr_look_good` are seemingly arbitrary and not extensive by any measure. This leaves much coverage to be desired, seeing as how the game can have many attributes and different states.