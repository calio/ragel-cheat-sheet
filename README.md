Most of this document is excepted from Ragel 6.8 document, copyright belongs to the original author Adrian Thurston.

# Ragel Cheat Sheet

> Ragel compiles executable finite state machines from a high level **regular language** notation to C, C++, Objective-C, D, Go, Java and Ruby code.

## Construction State Machines

A multi-line FSM spec starts with %%{ and ends with }%%. A single-line FSM spec starts with %% and ends at the first newline.

	#include <string.h>
	#include <stdio.h>
	%%{
		machine foo;
		main :=
			( ’foo’ | ’bar’ )
			0 @{ res = 1; };
	}%%
	
	%% write data;
	
	int main( int argc, char **argv )
	{
		int cs, res = 0;
		if ( argc > 1 ) {
			char *p = argv[1];
			char *pe = p + strlen(p) + 1;
			%% write init;
			%% write exec;
		}
		printf("result = %i\n", res );
		return 0;
	}
	
#### Machine Definition

	<name> = <expression>;

#### Machine Instantiation

	<name> := <expression>;

## Ragel Block Lexical Analysis

#### Basic Machines

* ’hello’ -  Concatenation Literal
* "hello" - Identical to the single quoted version
* [hello] - Produces a union of characters
* ’’, "", and [] - Zero Length Machine
* 42 - Numerical Literal
* /simple_regex/ - Regular Expression. This notation also supports the i trailing option. Use it to produce case-insensitive machines, as in /GET/i.
* ’a’ .. ’z’ - Range
* variable_name - – Lookup the machine definition assigned to the variable name given and use an instance of it

#### builtin_machine

* `any` - Any character in the alphabet.
* `ascii` - Ascii characters. 0..127
* `extend` - Ascii extended characters. This is the range -128..127 for signed alphabets and the range 0..255 for unsigned alphabets.
* `alpha` - Alphabetic characters. [A-Za-z]
* `digit` - Digits. [0-9]
* `alnum` - Alpha numerics. [0-9A-Za-z]
* `lower` - Lowercase characters. [a-z]
* `upper` - Uppercase characters. [A-Z]
* `xdigit` - Hexadecimal digits. [0-9A-Fa-f]
* `cntrl` - Control characters. 0..31
* `graph` - Graphical characters. [!-~]
* `print` - Printable characters. [ -~]
* `punct` - Punctuation. Graphical characters that are not alphanumerics. [!-/:-@[-‘{-~]
* `space` - Whitespace. [\t\v\f\n\r ]
* `zlen` - Zero length string. ""
* `empty` - Empty set. Matches nothing. ^any

#### Regular Language Operators

###### Union

`expr | expr`

The union operation produces a machine that matches any string in machine one or machine two

###### Intersection

`expr & expr`

Intersection produces a machine that matches any string that is in both machine one and machine two
	
###### Difference

`expr - expr`

The difference operation produces a machine that matches strings that are in machine one but are not in machine two

For example: `(any - space)*`
	
###### Strong Difference

`expr -- expr`

Strong difference produces a machine that matches any string of the first machine that does not have any string of the second machine as a substring

###### Concatenation

`expr . expr`

Concatenation produces a machine that matches all the strings in machine one followed by all the strings in machine two

###### Kleene Star

`expr*`

The machine resulting from the Kleene Star operator will match zero or more repetitions of the machine it is applied to

###### One Or More Repetition

`expr+`

###### Optional `expr?`

###### Repetition

* `expr {n}` – Exactly N copies of expr.
* `expr {,n}` – Zero to N copies of expr.
* `expr {n,}` – N or more copies of expr.
* `expr {n,m}` – N to M copies of expr.

###### Negation

`!expr`

Negation produces a machine that matches any string not matched by the given machine. Negation is equivalent to (any* - expr).

######  Character-Level Negation

`^expr`

Character-Level Negation is equivalent to (any - expr)

## User Actions

#### Two ways of using action

1.		main := ( lower* >{ printf("action lower"); }) . ' ';
2.		action A { printf("action lower"); }
		main := ( lower* >A) . ' ';
		
#### Transition actions

* `expr > action`	Entering Action
* `expr @ action`	Finishing Action
* `expr $ action`	All Transition Action
* `expr % action`	Leaving Actions

#### State actions

###### The different classes of states are:
* `\>` – the start state
* `<` – any state except the start state
* `$` – all states
* `%` – final states
* `@` – any state except final states
* `<>` – any except start and final (middle)

###### The different kinds of embeddings are:
* `~` – to-state actions (to)
* `\*` – from-state actions (from)
* `/` – EOF actions (eof)
* `!` – error actions (err)
* `^` – local error actions (lerr)

###### To-State Actions
* `\>~action  >to(name)  >to{...}` – the start state
* `<~action  <to(name)  <to{...}` – any state except the start state
* `$~action  $to(name)  $to{...}` – all states
* `%~action  %to(name)  %to{...}` – final states
* `@~action  @to(name)  @to{...}` – any state except final states
* `<>~action  <>to(name)  <>to{...}` – any except start and final (middle)

###### From-State Actions
* `\>*action  >from(name)  >from{...}` – the start state
* `<*action  <from(name)  <from{...}` – any state except the start state
* `$*action  $from(name)  $from{...}` – all states
* `%*action  %from(name)  %from{...}` – final states
* `@*action  @from(name)  @from{...}` – any state except final states
* `<>*action  <>from(name)  <>from{...}` – any except start and final (middle)

###### EOF Actions
* `\>/action  >eof(name)  >eof{...}` – the start state
* `</action  <eof(name)  <eof{...}` – any state except the start state
* `$/action  $eof(name)  $eof{...}` – all states
* `%/action  %eof(name)  %eof{...}` – final states
* `@/action  @eof(name)  @eof{...}` – any state except final states
* `<>/action  <>eof(name)  <>eof{...}` – any except start and final (middle)

###### Global Error Actions
* `\>!action  >err(name)  >err{...}` – the start state
* `<!action  <err(name)  <err{...}` – any state except the start state
* `$!action  $err(name)  $err{...}` – all states
* `%!action  %err(name)  %err{...}` – final states
* `@!action  @err(name)  @err{...}` – any state except final states
* `<>!action  <>err(name)  <>err{...}` – any except start and final (middle)

###### Local Error Actions
* `\>^action  >lerr(name)  >lerr{...}` – the start state
* `<^action  <lerr(name)  <lerr{...}` – any state except the start state
* `$^action  $lerr(name)  $lerr{...}` – all states
* `%^action  %lerr(name)  %lerr{...}` – final states
* `@^action  @lerr(name)  @lerr{...}` – any state except final states
* `<>^action  <>lerr(name)  <>lerr{...}` – any except start and final (middle)

#### Values and Statements Available in Code Blocks

Value name  | Explanation
------------|-------------
fpc         | A pointer to the current character. This is equivalent to accessing the p variable
fc          | The current character. This is equivalent to the expression (*p).
fcurs       |  An integer value representing the current state. This value should only be read from. To move to a different place in the machine from action code use the fgoto, fnext or fcall statements. Outside of the machine execution code the cs variable may be modified.
ftargs      | An integer value representing the target state. This value should only be read from. Again, fgoto, fnext and fcall can be used to move to a specific entry point.
fentry(<label>) | Retrieve an integer value representing the entry point label. E.g. `fgoto *((ctx->done) ? fentry(html_default) : fentry(html_special));`


Statement name  | Explanation
----------------|-------------
fhold;          | Do not advance over the current character
fexec <expr>;   | Set the next character to process. This can be used to backtrack to previous input or advance ahead. Unlike fhold, which can be used anywhere, fexec requires the user to ensure that the target of the backtrack is in the current buffer block or is known to be somewhere ahead of it. The machine will continue iterating forward until pe is arrived at, fbreak is called or the machine moves into the error state. In actions embedded into transitions, the fexec statement is equivalent to setting p to one position ahead of the next character to process. If the user also modifies pe, it is possible to change the buffer block entirely.
fgoto <label>;  | Jump to an entry point defined by <label>.
fgoto *<expr>;  | Jump to an entry point given by <expr>. Use together with `fentry`.
fnext <label>;  | Set the next state to be the entry point defined by label.
fnext *<expr>;  | Set the next state to be the entry point given by <expr>. Use together with `fentry`.
fcall <label>;  | Push the target state and jump to the entry point defined by <label>. The next fret will jump to the target of the transition on which the call was made. Use of fcall requires the declaration of a call stack. An array of integers named stack and a single integer named top must be declared
fcall *<expr>;  | Push the current state and jump to the entry point given by <expr>. Use together with `fentry`.
fret;           | Return to the target state of the transition on which the last fcall was made.
fbreak;         | fbreak; – Advance p, save the target state to cs and immediately break out of the execute loop. After an fbreak statement the p variable will point to the next character in the input. The current state will be the target of the current transition. Note that fbreak causes the target state’s to-state actions to be skipped.


## Controlling Nondeterminism

#### Guarded Operators

######  Entry-Guarded Concatenation

`expr :> expr`

###### Finish-Guarded Concatenation

`expr :>> expr`

###### Left-Guarded Concatenation

`expr <: expr`

###### Longest-Match Kleene Star

`expr**`

This version of kleene star puts a higher priority on staying in the machine versus wrapping around and starting over.

#### Operator Precedence

Precedence      | Operator                      | Comments
----------------|-------------------------------|---------
1               | ,                             | Join
2               | \| & - --                     | Union, Intersection and Subtraction
3               | . <: :> :>>                   | Concatenation
4               | :                             | Label
5               | ->                            | Epsilon Transition
6               | > @ $ %                       | Transitions Actions and Priorities
"               | >/ $/ %/ &lt;/ @/ &lt;>/      | EOF Actions
"               | >! $! %! &lt;! @! &lt;>!      | Global Error Actions
"               | >^ $^ %^ &lt;^ @^ &lt;>^      | Local Error Actions
"               | >~ $~ %~ &lt;~ @~ &lt;>~      | To-State Actions
"               | >* $* %* &lt;* @* &lt;>*      | From-State Action
7               | * ** ? + {n} {,n} {n,} {n,m}  | Repetition
8               | ! ^                           | Negation and Character-Level Negation
9               | ( <expr> )                    | Grouping

## Interface to Host Program

#### Write

    %%{
    machine foo;
    write data;
    }%%

    void init(...) {
        ...
        %% write init;
        ...
    }

    int parse(...) {
        ...
        %% write exec;
        ...
    }

###### Access Statement

`access fsm->;`

The access statement specifies how the generated code should access the machine data that is persistent across processing buffer blocks. This applies to all variables except p, pe and eof. This includes cs, top, stack, ts, te and act. The access statement is useful if a machine is to be encapsulated inside a structure in C code.

######  Alphtype Statement

`alphtype unsigned char;`

The alphtype statement specifies the alphabet data type that the machine operates on. During the compilation of the machine, integer literals are expected to be in the range of possible values of the alphtype. The default is char for all languages except Go where the default is byte.

#### Code generation

Code Output Style Options       | Meaning			    | Languages
--------------------------------|-----------------------------------|-------------------
-T0                             | binary search table-driven        | C/D/Java/Ruby/C#/Go
-T1 				| binary search, expanded actions   | C/D/Ruby/C#/Go
-F0 				| flat table-driven                 | C/D/Ruby/C#/Go
-F1                             | flat table, expanded actions      | C/D/Ruby/C#/Go
-G0                             | goto-driven                       | C/D/C#/Go
-G1                             | goto, expanded actions            | C/D/C#/Go
-G2                             | goto, in-place actions            | C/D/Go

## Debugging

The most effective way to debug Ragel grammar is to generate a transition graph from Ragel file using Ragel and Graphviz.

    ragel -Vp a.rl -o a.dot
    dot a.dot -Tpng -o a.png

`-V`: to generate a dot file

`-p`: use character labels instead of number labels

###### Get better graphs

If the final graph is too large to be meaningful, the use is able to inspect portions of the parser by naming particular regular expression definition with -S and -M options.

_Using machine name:_

    ragel -Vp -S [machine_name] a.rl -o a.dot

_Or using rule name:_

    ragel -Vp -M [rule_name] a.rl -o a.dot

## Misc
* Ragel vim grammar highlight: <https://github.com/jneen/ragel.vim>
* State minimization is on by default. It can be turned off with the -n option. The algorithm implemented is similar to Hopcroft’s state minimization algorithm.
* Ragel 6.8 document <http://www.colm.net/files/ragel/ragel-guide-6.8.pdf>
