<template>
  <article>
    <p>
      Contents:
    </p>
    <ol>
      <li><a href="#TOC_1">Introduction</a>
      </li>
      <li><a href="#TOC_2">Recursive Definitions</a>
      </li>
      <li><a href="#TOC_3">Self-application</a>
      </li>
      <li><a href="#TOC_4">Partially explicit recursion</a>
      </li>
      <li><a href="#TOC_5">A nested recursion combinator</a>
      </li>
    </ol>
    <a name="TOC_1">
      <h2><strong>Introduction</strong></h2>
    </a>
    <p>
      This note was prompted by Nick Forde's version of a Joy operator which computes the
      Ackermann function which is one of the few well-known functions that use nested
      recursion. He was trying to avoid explicit recursion in the definition, but found
      that none of the recursion combinators of Joy <em>at the time</em> were up to
      the task. His finding prompted the inclusion of a new Joy combinator
      <em>condnestrec</em> into the language.
    </p>
    <p>
      This note gives definitions of (the common simplification of) Ackermann's function,
      of McCarthy's 91-function, of a function producing a Hamiltonian path over a hypercube,
      of a function producing Gray sequences, and of the Hanoi problem. All involve nested
      recursion. Several possible definition methods are used.
    </p>
    <p>
      The definitions and the output of the examples in this note were obtained by running
      the file
      <a href="joy/jp-nestrec.joy">
      jp-nestrec.joy</a>. Only the descriptive text in
      this note was added later.
      <a name="TOC_2"></a>
    </p>
    <h2><a name="TOC_2"><strong>Recursive Definitions</strong></a></h2>
    <p>
      The following is the more or less standard definition of the factorial function:
      <br>
    </p>
    <pre>     fact(n) = 
        IF n = 0 
        THEN 1 
        ELSE  n * fact(n-1) 
    </pre>
    Here is the corresponding recursive definition in Joy:
    <br>
    <pre> DEFINE
          r-fact == 
        [ null ] 
        [ pop 1] 
        [ dup pred r-fact *] 
        ifte. 
    </pre>
    The following tests this definition - redundantly, of course. Note that the
    response from Joy is indented to the left. The same style will be used in all later
    tests.
    <pre> [ 0 1 2 3 4 5 6 ]  [r-fact]  map. 
    [1 1 2 6 24 120 720]
    </pre>
    <p>
      The next definition is of a not so familiar function originally due to McCarthy,
      his intriguing "91 function":
    </p>
    <pre>     mcc91(i) = 
        IF i &gt; 100 
        THEN i - 10 
        ELSE mcc91(mcc91(i + 11)) 
    </pre>
    Note one very important feature not shared with the factorial function: In
    the ELSE-part the recursive call to itself relies on the result of another recursive
    call to itself. There are very few function definitions in which this pattern appears.
    The following is the corresponding definition in Joy, note the double recursive call
    in the second last line:
    <pre> DEFINE 
          r-mcc91 == 
        [ 100 &gt; ] 
        [ 10 - ] 
        [ 11 + r-mcc91 r-mcc91 ] 
        ifte. 
    </pre>
    Here is a simple test for some representative values:
    <br>
    <pre> [ -7 42 99 100 101 102 345 ]  [r-mcc91]  map. 
    [91 91 91 91 91 92 335]
    </pre>
    <p>
      The Ackermann function widely quoted in the literature is actually a simplification
      of the function originally defined by Ackermann to show that there are terminating
      recursive functions that are not primitive recursive. Both the original and the
      descendent due to Peters grow at an extraordinary rate. Here is the now common
      definition:
    </p>
    <pre>     ack(m, n) = 
        IF m = 0  THEN n + 1 
        ELSEIF n = 0  THEN ack(m - 1, 1) 
        ELSE ack(m - 1, ack(m, n - 1)) 
    </pre>
    and its counterpart in Joy:
    <br>
    <pre> DEFINE 
          r-ack == 
      # stack putln 
        [ [ [pop null]  popd succ ] 
          [ [null]  pop pred 1 r-ack ] 
          [ [dup pred swap] dip pred r-ack r-ack ] ] 
        cond. 
    </pre>
    The line that has been commented out with the hash character is useful if
    one wants to see a trce of what is sitting on the stack. The following tests are
    complete from (0,0) up to (3,5), and give some indication of the behaviour. The final
    line only gives the value for (4,0) because computation for (4,1) crashes the Joy
    stack (and also, it seems, calculations in other languages).
    <pre> [ [0 0] [0 1] [0 2] [0 3] [0 4] [0 5] ]   [i r-ack]  map  putln 
      [ [1 0] [1 1] [1 2] [1 3] [1 4] [1 5] ]   [i r-ack]  map  putln 
      [ [2 0] [2 1] [2 2] [2 3] [2 4] [2 5] ]   [i r-ack]  map  putln 
      [ [3 0] [3 1] [3 2] [3 3] [3 4] [3 5] ]   [i r-ack]  map  putln 
      [ [4 0]                               ]   [i r-ack]  map. 
    [1 2 3 4 5 6] 
    [2 3 4 5 6 7] 
    [3 5 7 9 11 13] 
    [5 13 29 61 125 253] 
    [13]
    </pre>
    <p>
      In the Towers of Hanoi puzzle the disks have to be moved in a particular order. Ignoring
      what the target peg is, for three disks the order is 1 2 1 3 1 2 1. In general
      for n disks it is a sequence of (2^n)-1 steps. In the usual implementation of the
      Hanoi program the disks that have to be moved are not mentioned at all. The sequence
      of steps is also one that performs a Hamiltonian path on an n-dimensional hypercube.
      The following is the Joy program:
    </p>
    <pre>  
      DEFINE 
          r-hamilhyp ==   #  [] n  =&gt;  [...] 
        [ null ] 
        [ pop ] 
        [ dup rollup pred       r-hamilhyp 
          dupd cons swap pred   r-hamilhyp ] 
        ifte. 
    </pre>
    Note that unlike the program for the 91-function and the Ackermann function,
    the two recursive calls are not consecutive. Since the sequences are less known than
    the Hanoi program, here are several test outputs:
    <br>
    <pre> []  3  r-hamilhyp. 
    [1 2 1 3 1 2 1]
      []  4  r-hamilhyp. 
    [1 2 1 3 1 2 1 4 1 2 1 3 1 2 1]
      []  5  r-hamilhyp. 
    [1 2 1 3 1 2 1 4 1 2 1 3 1 2 1 5 1 2 1 3 1 2 1 4 1 2 1 3 1 2 1]
    </pre>
    <a name="TOC_3">
      <h2><strong>Self-application</strong></h2>
    </a>
    <p>
      One way to get by without recursive definitions is by using Curry's "paradoxical"
      Y combinator which can also be defined in Joy. Another way would use what is known
      as "self-application", giving a function a parameter which is also a function -
      e.g. itself. In Joy there is an inbuilt combinator to this effect, but it might
      have been defined by
    </p>
    <pre>        DEFINE
                x == dup i.
    </pre>
    Using this combinator a non-recursive definition of the factorial function
    is this:
    <br>
    <pre> DEFINE 
          x-fact == 
        [ [ pop null ] 
          [ pop pop 1] 
          [ [dup pred] dip x *] 
          ifte ] 
        x. 
    </pre>
    Here is a test, and keep in mind that because the definition of <kbd>x-fact</kbd>is not recursive, as a parameter to the <kbd>map</kbd> combinator instead of using
    the quotation <kbd>[x-fact]</kbd> it would have been possible to use quotation of
    the body of the definition.
    <pre> [ 0 1 2 3 4 5 6 ]  [x-fact]  map. 
    [1 1 2 6 24 120 720]
    </pre>
    <p>
      In the recursive definition of the 91-function the recursive branch used a double
      recursion. For a non-recursive version it is necessary to have a double self-call
      with the x-combinator. Such a double call is useful elsewhere, so here we have
      a definition:
    </p>
    <pre> DEFINE 
          twice-x == 
        dup [x] dip x. 
    </pre>
    Below is a non-recursive definition of the 91-function and the same test set
    as before:
    <br>
    <pre> DEFINE 
          x-mcc91 == 
        [ [ pop 100 &gt; ] 
          [ pop 10 - ] 
          [ [11 +] dip twice-x ] 
          ifte ] 
        x. 
       
      [ -7 42 99 100 101 102 345 ]  [x-mcc91]  map. 
    [91 91 91 91 91 92 335]
    </pre>
    <p>
      Exactly the same technique can be used to give a non-recursive definition of the
      Ackermann function using th x-combinator. Only one row of the test matrix is used
      here:
    </p>
    <pre> DEFINE 
          x-ack == 
        [ [ [ [pop pop null]  pop popd succ ] 
            [ [pop null]  [pop pred 1] dip x ] 
            [ [[dup pred swap] dip pred] dip twice-x ] ] 
        cond ] 
        x. 
       
      [ [3 0] [3 1] [3 2] [3 3] [3 4] [3 5] ]   [i x-ack]  map. 
    [5 13 29 61 125 253]
    </pre>
    Whatever may be achieved with the x-combinator can also be achieved with the
    less efficient but more familiar y-combinator, which needs to be defined first:
    <pre> DEFINE
          y ==
        [dup cons] swoncat dup cons i;
          twice-i ==
        dup [i] dip i.
      DEFINE
          y-ack ==
        [ [ [ [pop pop null]  pop popd succ ]
            [ [pop null]  [pop pred 1] dip i ]
            [ [[dup pred swap] dip pred] dip twice-i ] ]
        cond ]
        y.
      
      [ [3 0] [3 1] [3 2] [3 3] [3 4] [3 5] ]   [i y-ack]  map.
    [5 13 29 61 125 253]
    </pre>
    <p>
      Here is the non-recursive definition, using the x-combinator, of the function that
      generates the hamiltonian path sequence on a hypercube, together with one of the
      tests:
      <br>
    </p>
    <pre> DEFINE 
          x-hamilhyp == 
        [ [ pop null ] 
          [ pop pop ] 
          [ dup [ [dup rollup pred] dip  x ] dip 
            [dupd cons] dip 
            [swap pred] dip  x ] 
          ifte ] 
        x. 
       
      []  5  x-hamilhyp. 
    [1 2 1 3 1 2 1 4 1 2 1 3 1 2 1 5 1 2 1 3 1 2 1 4 1 2 1 3 1 2 1]
    </pre>
    <a name="TOC_4">
      <h2><strong>Partially explicit recursion</strong></h2>
    </a>
    <p>
      Nick Forde suggested writing the Ackermann function by using the linrec combinator
      to achieve one recursion, and to use explicit recursion for the other. For reasons
      that do not concern us here, his version computes the <em>converse</em> of the
      text book definition. For that reason, in the tests below the call is to
      <kbd>swap ack</kbd>:
    </p>
    <pre>  
      DEFINE ack == (* I:n I:m -&gt; I:a *) 
          [ [ [0 =] [pop 1 +] ] 
            [ [swap 0 =] [popd 1 - 1 swap] [] ] 
            [ [dup rollup [1 -] dip] [swap 1 - ack] ] ] 
          condlinrec. 
       
      [ [0 0] [0 1] [0 2] [0 3] [0 4] [0 5] ]   [i swap ack]  map  putln 
      [ [1 0] [1 1] [1 2] [1 3] [1 4] [1 5] ]   [i swap ack]  map  putln 
      [ [2 0] [2 1] [2 2] [2 3] [2 4] [2 5] ]   [i swap ack]  map  putln 
      [ [3 0] [3 1] [3 2] [3 3] [3 4] [3 5] ]   [i swap ack]  map  putln 
      [ [4 0]                               ]   [i swap ack]  map. 
    [1 2 3 4 5 6] 
    [2 3 4 5 6 7] 
    [3 5 7 9 11 13] 
    [5 13 29 61 125 253] 
    [13]
    </pre>
    <p>
      In the same style we may define the 91-function using only a partially recursive
      definition:
      <br>
    </p>
    <pre> DEFINE 
          l-mcc91 == 
        [ 100 &gt; ] 
        [ 10 - ] 
        [ 11 + ] 
        [ l-mcc91 ] 
        linrec. 
       
      [ -7 42 99 100 101 102 345 ]  [l-mcc91]  map. 
    [91 91 91 91 91 92 335]
    </pre>
    <p>
      The following partially recursive definition of the Ackermann function also uses
      just the explicitly recursive call in the quotation for the double recursion:
    </p>
    <pre> DEFINE 
          clr-ack == 
        [ [ [pop null]  [popd succ] ] 
          [ [null]  [pop pred 1]  [] ] 
          [ [[dup pred swap] dip pred]  [clr-ack] ] ] 
        condlinrec. 
       
      [ [3 0] [3 1] [3 2] [3 3] [3 4] [3 5] ]   [i clr-ack]  map. 
    [5 13 29 61 125 253]
    </pre>
    <p>
      In the same style here is the partially recursive definition of the function for
      the Hamiltonian path on a hypercube. But since there is only a two way branching,
      the linrec combinator is used rather than the slightly more complex (and hence
      more flexible) condlinrec combinator. Remember that the implicit recursion occures
      between the third and fourth quotation.
    </p>
    <pre> DEFINE  
          lr-hamilhyp == 
        [ null ] 
        [ pop ] 
        [ dup rollup pred ] 
        [ dupd cons swap pred lr-hamilhyp ] 
        linrec. 
       
      []  5  lr-hamilhyp. 
    [1 2 1 3 1 2 1 4 1 2 1 3 1 2 1 5 1 2 1 3 1 2 1 4 1 2 1 3 1 2 1]
    </pre>
    <p>
      Using almost the same definition with only a small change we obtain a definition
      for something seemingly quite different. In the following only the commented line
      has been inserted. The definition generates Gray sequences of subsets of small
      numbers. Each subset is generated from its predecessor by having just one number
      added or removed. This step is here done by an operation called <em>toggle</em>,
      which takes a set of small numbers and another small number as parameters and returns
      a set. If the number was in the original set then it is removed, if the number
      was not in the original set then it is added. Here is the definition and some examples:
    </p>
    <pre> DEFINE 
          toggle ==   #   {..} i  -&gt;  {..] 
        [has]  [[not] dip swons not]  [swons]  ifte. 
       
      {}  4  toggle. 
    {4}
      {1 2 7}  2 toggle. 
    {1 7}
    </pre>
    Here is the function which constructs Gray sequences, and three examples:
    <br>
    <pre> DEFINE 
          lr-grayseq == 
        [ null ] 
        [ pop ] 
        [ dup rollup pred ] 
        [ dupd 
          dup [first swap toggle] dip  # inserted line 
          cons swap pred lr-grayseq ] 
        linrec. 
       
      [{}]  3  lr-grayseq. 
    [{3} {1 3} {1 2 3} {2 3} {2} {1 2} {1} {}]
      [{3}]  3  lr-grayseq. 
    [{} {1} {1 2} {2} {2 3} {1 2 3} {1 3} {3}]
      [{1 2 3}]  3 lr-grayseq. 
    [{1 2} {2} {} {1} {1 3} {3} {2 3} {1 2 3}]
    </pre>
    <a name="TOC_5">
      <h2><strong>A nested recursion combinator</strong></h2>
    </a>
    <p>
      Nested recursion occurs not only in contrived functions like the Ackermann function
      and the 91-function, but also in some others. Since one of the aims of Joy is to
      emphasise common patterns, a new recursion combinator called <kbd>condnestrec</kbd>  was added to Joy. It resembles the <kbd>condlinrec</kbd> combinator in that it
      takes just one parameter which is a list of cases. Each case is a list, all but
      the last have a quoted test predicate as their first element. All other elements
      in a case list are again quotations. But whereas <kbd>condlinrec</kbd> allows at
      most two other quotations, <kbd>condnestrec</kbd> allows any number. For both combinators
      an implicit recursion occurs between any consecutive other quotations.
    </p>
    <p>
      Here is the example for the Hamiltonian path on a hypercube. Note that the parameter
      to the combinator is a list of just two cases. The first case consists of a test
      quotation and another quotation, so there is no recursion here. The second case
      is also the last, so it is the default, and it has no test quotation. There are
      three quotations, so the implicit recursion occurs between them, so it occurs twice.
    </p>
    <pre> DEFINE 
          cnr-hamilhyp == 
        [ [ [null] [pop] ] 
          [ [dup rollup pred] 
            [dupd cons swap pred] 
            [] ] ] 
        condnestrec. 
       
      [] 4 cnr-hamilhyp. 
    [1 2 1 3 1 2 1 4 1 2 1 3 1 2 1]
    </pre>
    <p>
      In the following definition of the Ackermann function implicit recursion occurs once
      in the second case, and twice in the third, default case. In each of the three
      places the quotation following the implicit recursion is actually empty.
    </p>
    <pre> DEFINE 
          cnr-ack == 
        [ [ [pop null]  [popd succ] ] 
          [ [null]  [pop pred 1]  [] ] 
          [ [[dup pred swap] dip pred]  []  [] ] ] 
        condnestrec. 
       
      3 4 cnr-ack. 
    125
    </pre>
    <p>
      The program for Gray sequences is again obtained from the one for the Hamiltonian
      path on a hypercube by inserting one line as indicated:
    </p>
    <pre> DEFINE 
          cnr-grayseq == 
        [ [ [null]  [pop] ] 
          [ [dup rollup pred] 
            [dupd 
             dup [first swap toggle] dip # inserted 
             cons swap pred] 
            [] ] ] 
        condnestrec. 
       
      [{}] 3 cnr-grayseq. 
    [{3} {1 3} {1 2 3} {2 3} {2} {1 2} {1} {}]
    </pre>
    <p>
      Since the Hamiltonian path sequence is also the sequence of the disk numbers in the
      Hanoi problem, it is perhaps not too surprising that the solution to the Hamoi
      problem is so similar. The solution of course has to specify the pegs: each move
      has to specify from which peg the top disk is to be removed and to which peg it
      is to be moved. However, the number of the disk is not normally mentioned. In the
      following program the number of the disk directs the recursion just as in the Hamiltonian
      problem, but the number of the disk does not appear in the output.
    </p>
    <p>
      In this particular version of the program the user specifies a list of the names
      of the three pegs, and the number of disks on the first peg. The disks on the first
      peg are to be moved to the peg whose name is the second on the list, and the peg
      whose name is the third item can be used as a temporary. The user chose to call
      the pegs "source" "destination" and "temp" in the first example, and "S" "D" and
      "T" in the second example.
    </p>
    <pre> DEFINE 
          cnr-hanoi == 
        [[rolldown] infra] dip 
        [ [ [null] [pop pop] ] 
          [ [dup2 [[rotate] infra] dip pred] 
            [ [dup rest put] dip 
              [[swap] infra] dip pred ] 
            [] ] ] 
        condnestrec. 
       
      [source destination temp]  2  cnr-hanoi. 
    [source temp] [source destination] [temp destination]
      [S D T]  5  cnr-hanoi. 
    [S D] [S T] [D T] [S D] [T S] [T D] [S D] [S T] [D T] [D S] [T S] [D T] [S D] [S T] [D T] [S D] [T S] [T D] [S D] [T S] [D T] [D S] [T S] [T D] [S D] [S T] [D T] [S D] [T S] [T D] [S D]      
    </pre>
    It is interesting to note that although the Gray sequence program and the
    Hanoi program are based on the same program, the Hamiltonian path program, the Gray
    sequence produced consists of 2^n members and the Hanoi move sequence consists of
    (2^n)-1 members.
    <p>
      Since this note started with the recursive definitions of the factorial function
      and the 91-function, This section would not be complete without the corresponding
      definitions using the <em>condnestrec</em> combinator:
      <br>
    </p>
    <pre> DEFINE 
          cnr-fact == 
        [ [ [null] [pop 1] ] 
          [ [dup pred] [*] ] ] 
        condnestrec; 
       
          cnr-mcc91 == 
        [ [ [100 &gt;] [10 -] ] 
          [ [11 +] [] [] ] ] 
        condnestrec. 
       
      [ 0 1 2 3 4 5 6 ]  [cnr-fact]  map. 
    [1 1 2 6 24 120 720]
      [ -7 42 99 100 101 102 345 ]  [cnr-mcc91]  map. 
    [91 91 91 91 91 92 335]
    </pre>
    <p>
      In fact, <kbd>condnestrec</kbd> is a veritable swiss army knife combinator. In
      the following definitions of the <kbd>even</kbd> predicate and the <kbd>abs</kbd>  (absolute value) operator,
      <kbd>condlinrec</kbd> is used just for conditionals without making use of the possibility
      of recursion. For the <kbd>abs</kbd> operator the default is actually empty.
    </p>
    <pre> DEFINE 
          cnr-even == 
        [ [ [2 rem null] [pop true] ] 
          [ [pop false] ] ] 
        condnestrec; 
       
          cnr-abs == 
        [ [ [0 &lt;] [0 swap -] ] 
          [ [] ] ] 
        condnestrec. 
       
      3 cnr-even. 
    false
      4 cnr-even. 
    true
      -5 cnr-abs. 
    5
      6 cnr-abs. 
    6
    </pre>
    <p>
      The use of nested recursion in Joy has not yet been explored beyond these examples.
      All the examples took a numeric parameter to guide the recursion, and one obvious
      alternative to investigate is the use of lists.
    </p>
    <p>
      MYNOTE: Bauer and Woessner, functions ble and morris. Manna, Ackermann for strings.
    </p>
  </article>
</template>

<style src="~/assets/articles.css" scoped>
</style>