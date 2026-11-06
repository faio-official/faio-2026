There are $3n$ numbers on the table, where $n$ is odd. You and your opponent take them apart in turns until each of you holds exactly $n$ numbers. Let $S$ be the sum of all $3n$ numbers. You want the sum of your own numbers to end up as close to $S/3$ as possible.

Both sides know the whole array from the start. The only thing you do not know is the order in which the numbers will show up in the game.

A game lasts $n$ rounds. Each round puts one more triple of numbers from the array on the table. One of the players is in charge of that triple: you in odd rounds, your opponent in even ones. Whoever is in charge splits the triple completely: takes one number, gives one to the other player and discards one. A discarded number never comes back. The other player makes no decision that round.

Since $n$ is odd, the first and the last rounds of a game are yours.

After $n$ rounds each side holds exactly $n$ numbers. Let your sum be $A$ and your opponent’s sum be $B$. You have **not lost** the game if

$$\left|A - \frac{S}{3}\right| \;\le\; \left|B - \frac{S}{3}\right|.$$

The strict inequality the other way round means the game is lost.

Note that no strategy guarantees you never lose: there are deals where the opponent ends up closer to $S/3$ no matter how you play. Your task is to lose as rarely as possible.

**This is a function-writing problem.** You do no input or output: the judge runs the games and calls your functions itself. Pick **Make** in the compiler list.

Download [template.py](https://drive.google.com/file/d/12eaPUnwqdqLAOXCKRVeWWc4UaVLh4AWK/view?usp=sharing) (a solution skeleton) and [local_test.py](https://drive.google.com/file/d/1sSrMorhF8Ns6I6g0G2whbgl26jQ8PS0U/view?usp=sharing) (a local check), put your strategy into `template.py` and submit the resulting file.

## Input format

Your solution reads nothing — it receives the data as arguments of three functions the judge calls:

```
def init(n, a):                       # start of a game
def my_move(t):                       # your round, an odd one
def opp_move(t, take, give, drop):    # opponent's round, an even one
```

**`init(n, a)`** is called before every game: $n$ is the number of rounds, `a` is the list of $3n$ integers ($1 \le a_i \le 10^9$), the whole array of the game. The values of $n$ per file are listed in the notes. A file holds 50 deals, that is 50 games, so `init` is called 50 times per file — reset all your variables there. The state of your module is wiped between games, so nothing can be carried from one game to the next.

**`my_move(t)`** is called in odd rounds: `t` is the list of three numbers, the revealed triple. What to return is described in the output format.

**`opp_move(t, take, give, drop)`** is called in even rounds: `t` is the revealed triple, and `take`, `give`, `drop` are indices from 1 to 3 — which element the opponent took, which it gave you, which it discarded. Return nothing.

Numbers inside a triple may repeat, so the split is passed as element indices, not as values.

## Output format

Your solution prints nothing. The only thing required of it is the return value of `my_move`: a tuple of three distinct integers from 1 to 3.

```
return (take, give, drop)
```

`take` is the index of the element you take, `give` the one you hand to the opponent, `drop` the one you discard. For example `return (2, 3, 1)` means: the second element of the triple to yourself, the third to the opponent, the first to the discard pile.

If the function returns anything else, raises an exception or hangs, the current game counts as lost; the remaining games of the file go on.

## Scoring

The tests are split into 10 files. A file holds 50 deals: each deal has its own array of $3n$ numbers and its own order in which the triples show up. In every deal you play one game against the jury’s opponent.

Let $W$ be the number of deals you won in a file. The score for the file is

$$10 \cdot \frac{\min(W,\ 40)}{40},$$

that is, 40 or more deals won give the full score of the file, fewer give a proportional share: every win is worth a quarter of a point.

The files differ:

| **File** | **$n$** | **Points** | **What is in it** |
| --- | --- | --- | --- |
| 1 | every odd | 10 | the array $1, 2, 3, \ldots, 3n$; triples come in order: $(1,2,3)$, then $(4,5,6)$ and so on; one deal per $n$ from 1 to 99 |
| 2 | 99 | 10 | the same values $1 \ldots 3n$ but shuffled, triples revealed at random |
| 3 | 99 | 10 | arbitrary values in increasing order; triples follow the array |
| 4 | 99 | 10 | an arbitrary array; triples follow the array |
| 5 | 99 | 10 | the same, other arrays |
| 6 | 99 | 10 | a uniformly random array, triples revealed at random |
| 7 | 99 | 10 | the same, other arrays |
| 8 | 99 | 10 | an array of a special kind (many equal numbers, wildly different magnitudes), triples revealed at random |
| 9 | 99 | 10 | the same, other arrays |
| 10 | 99 | 10 | the deal is arranged to make play hard |
|  |  | **100** |  |

## Notes

**An example game.** Here $n = 3$ for brevity. Let the array be `5 1 8 4 3 9 2 7 6`, so $S = 45$ and the target is a sum close to $15$. The order in which triples show up is not known in advance and in general does not follow the array; say in this game they came as $(9,1,6)$, then $(5,7,4)$, then $(3,8,2)$.

| **Call** | **What happens** |
| --- | --- |
| init(3, [5, 1, 8, 4, 3, 9, 2, 7, 6]) | the game starts |
| my_move([9, 1, 6]) $\to$ (3, 1, 2) | you take 6, give 9 to the opponent, discard 1 |
| opp_move([5, 7, 4], 2, 1, 3) | the opponent took 7, gave you 5, discarded 4 |
| my_move([3, 8, 2]) $\to$ (1, 2, 3) | you take 3, give 8 to the opponent, discard 2 |

The result: $A = 6 + 5 + 3 = 14$, $B = 9 + 7 + 8 = 24$. The distances to the target are $1$ and $9$, so the game is not lost.

**How to check your solution locally.** Run `python3 local_test.py template.py` — the program reads $n$ and $3n$ numbers, plays one game against a simple opponent and prints every round. This is only a sample: there the triples follow the array you typed and the opponent is deliberately primitive, while the real tests reveal the triples in an order you cannot predict and use a stronger strategy. So it checks that your interface works, not how well you play.
