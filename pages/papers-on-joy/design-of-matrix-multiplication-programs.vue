<template>
  <article>
    <p>
      NOTE: Some of the definitions in here have been included in the matrix/vector library,
      but under quite different systematic names.
    </p>
    <h1>1. A numerical matrix addition operator</h1>
    <p>
      A matrix is a rectangular collection of items, most often numbers. Two matrices M1
      and M2 can be added just in case both have the same number of rows and both have
      the same number of columns. In that case their sum M3 has that same number of row
      and that same number of columns. Each element in the result matrix M3 is just the
      arithmetic sum of the corresponding elements of the other two matrices M1 and M2.
      Two matrices M1 and M2 can be multiplied just in case M1 has as many columns as
      M2 has rows. In that case their product M3 has as many rows as M1 and as many columns
      as M2. In detail, if M1 has I rows and J columns, and M2 has J rows and K columns,
      then the element in row i and column k of M3 is computed by multiplying the J pairs
      of corresponding elements of row i of M1 and column k of M2 and then adding the
      products.
    </p>
    <p>
      In Joy a matrix of I rows and J columns can be represented as a list of I lists each
      of J elements. The remainder of this note deals with matrix addition and matrix
      multiplication in Joy. The first sections deal with such operators just for numerical
      matrices. In Joy0 this means just integer, in Joy1 this means integer or float.
      The sections which then follow deal with general addition and multiplication combinators
      suitable for matrices built from other datatypes.
    </p>
    <p>
      How can we write a program <em>num-matadd</em> to add two matrices of numbers giving
      a third? We want
    </p>
    <pre>M1  M2  num-matadd   ==&gt;   M3
    </pre>
    As a first step consider element-wise addition of two vectors of the same
    length, represented in Joy just as a list. The result vector has that same length.
    Some languages have a zipwith combinator, but in Joy it is just called map2. Whereas
    map takes one list parameter, map2 takes two. Both combinators also take a quotation
    parameter which determines how the elements of the parameter list(s) are to be used
    to form the elements of the result list. For vector addition the elements have to
    be added, so the required program could be defined by
    <pre>num-vectadd   ==   [+]  map2
    </pre>
    Now to add two matrices of numbers we just have to use map2 again to add the
    rows as vectors:
    <pre>M1  M2  [num-vectadd]  map2   ==&gt;   M3
    </pre>
    Hence the following definition:
    <br>
    <pre>num-matadd   ==   [num-vectadd]  map2
    </pre>
    For later purposes it is useful to "inline" the definition of num-vectadd,
    giving
    <pre>num-matadd   ==   [[+] map2]  map2
    </pre>
    <p>
    </p>
    <h1>2. A numerical matrix multiplication operator</h1>
    <p>
      How can we write a program <em>matmul</em> to multiply two matrices to give a third?
      This is a little harder. We want:
    </p>
    <pre>M1  M2  num-matmull   ==&gt;   M3
    </pre>
    Notice that M3 has exactly as many rows as M1, and hence the Joy lists M1
    and M3 are of the same length. Each of the I component lists of M1 is mapped to a
    corresponding list of M3. The details of the mapping of course also depend on M2.
    So the mapping function will make use of M2. As a first step we may write
    <pre>M1  [M2 P]  map   ==&gt;   M3
    </pre>
    where P is to be determined later. Even then, M2 is not really supposed to
    be part of the program, because M2 will already be on the stack. But it is easy enough
    to take it off the stack and put it in front of P:
    <pre>M1  M2  [P]  cons  map   ==&gt;   M3
    </pre>
    P must be program which encounters M2 on the stack and below that a list of
    J numbers left behind from the map. P must use the K columns of M2 to produce a list
    of K numbers for M3. This looks like another case of map, with Q to be determined
    later: P == [Q] map. By substitution we have
    <pre>M1  M2  [[Q] map]  cons  map   ==&gt;  M3
    </pre>
    Two things are not quite right. Firstly, M2 is a list of J rows of K numbers,
    but the new map just introduced needs a list of K columns of J numbers. So it needs
    the transpose of M2:
    <pre>M1  M2  transpose  [[Q] map]  cons  map   ==&gt;  M3
    </pre>
    Secondly, the new map will consume each of the K columns from the transposed
    M2 to produce a list of K numbers for M3, but it will also leave behind the list
    of J numbers from M1 which the earlier, outer map left behind. This list of J numbers
    needs to be removed after the new map has finished. But the list to be removed is
    not on top of the stack but just below. So it must be removed by popd, which could
    be defined by popd == [pop] dip.
    <pre>M1  M2  transpose  [[Q] map popd]  cons  map   ==&gt;  M3
    </pre>
    Program Q will find two lists of length J on the stack: a row list from M1
    and a column list from M2. It must multiply corresponding numbers from the two lists
    and then add their products. This operattion is useful elsewhere, it is called scalar
    product. One way to compute it is to produce a list of products first, and then add
    the elements of that list. The list of products is similar to vector addition defined
    earlier, but it uses multiplication. So the list of products is produced by [*] map2.
    The resulting list can then be added with the sum operator. The sum operator works
    like this: starting with 0, add to that all the elements of the list. This is called
    "folding" the list, using the binary + operator. These are the definitions:
    <pre>sum   ==   0  [+]  fold
    scalar-product   ==   [*] map  sum
    </pre>
    So the entire matrix multiplication problem now is simply
    <br>
    <pre>M1  M2  transpose  [[scalar-product] map popd]  cons  map   ==&gt;   M3
    </pre>
    So we may define:
    <br>
    <pre>num-matmul   ==   transpose  [[scalar-product] map popd]  cons  map
    </pre>
    For later purposes it is again useful to inline scalar-product and then sum.
    The new definition is here written over several lines for comparison with later variations:
    <pre>num-matmul   ==   transpose
                      [[[*] map2 0 [+] fold]  map  popd]  cons
                      map
    </pre>
    <h1>3. General matrix combinators</h1>
    <p>
      The addition and multiplication operators of the preceding sections only work for
      numerical matrices. But the only four parts in the definitions that depend on that
      are [+] inside num-matadd, and [*], 0 and [+] inside num-matmul. However, matrices
      could have other kinds of things as elements. For example, one might have logical
      matrices in which the elements are the truth values true and false. In that case
      one would want two further definitions with just the four parts replaced by [or],
      [and], false and [or]. Or one might want two operators for different datatypes
      again. In fact, for matrix addition and multiplication the three matrices M1 M2
      M3 might have up to three different kinds of elements. It would be useful to have
      general matrix manipulation combinator that abstract from details.
    </p>
    <p>
      How can we write a general matrix addition combinator? It should satisfy
      <br>
    </p>
    <pre>M1  M2  [+]  gen-matadd   ==&gt;  M3
    </pre>
    which behaves just like the numerical addition operator, and with [or] instead
    of [+] behaves like a logical addition operator? In either case it has to transform
    the quotation [+] into [[+] map2] map2, or the quotation [or] into [[or] map2] map2.
    This is easy enough,in the numerical case it is
    <pre>M1  M2  [+]  [map2]  cons  map2   ==&gt;   M3
    </pre>
    So we may define a general matrix addition combinator
    <em>gen-matadd</em> by
    <br>
    <pre>gen-matadd   ==   [map2]  cons  map2
    </pre>
    Now matrix addition for numbers or for truth values can be defined as
    <br>
    <pre>num-matadd   ==    [+]  gen-matadd
    log-matadd   ==   [or]  gen-matadd
    </pre>
    In Joy truth values and sets are treated in a very similar way, and collectively
    they are called Boolean values. For truth values the or-operator produces the disjunction,
    and for sets the or-operator produces the union. So the above logical matrix addition
    operator would work equally well for matrices in which the element are sets. Because
    of that it is better to call it a Boolean matrix addition operator, defined as:
    <pre>bool-matadd   ==   [or]  gen-matadd
    </pre>
    As always, instead of actually having the definition one could just as easily
    use the right hand side directly.
    <p>
      A general matrix multiplication combinator is a little harder. Let Q be any program
      for combining a row from M1 and a column from M2 to form an element for M3. In
      the nnumerical case the Q would have been the scalar product program. Then a combinator
      <b>gen-matmul</b> would be used like this:
    </p>
    <pre>M1  M2  [Q]  gen-matmul   ==&gt;   M3
    </pre>
    Now M2 needs to be transposed as before, but it is below the [Q], so gen-matmul
    must use dip:
    <pre>M1  M2  [Q]  [transpose]  dip  S   ==&gt;   M3
    </pre>
    S must construct [[Q] map popd] and then use that with T:
    <br>
    <pre>M1  M2  [Q]  [transpose]  dip  [map popd]  cons  T   ==&gt;   M3
    </pre>
    But T is just as before, T == cons map:
    <br>
    <pre>M1  M2  [Q]  [transpose]  dip  [map popd]  cons  cons map   ==&gt;   M3
    </pre>
    So we can define the general combinator
    <br>
    <pre>gen-matmul   ==   [transpose]  dip
                      [map popd]  cons  cons
                      map
    </pre>
    Now the multiplication of numerical and logical matrices can be defined just
    in terms of the corresponding scalar products:
    <pre>num-matmul   ==   [  [*] map2       0 [+]  fold]  gen-matmul
    log-matmul   ==   [[and] map2   false [or] fold]  gen-matmul
    </pre>
    Compared with the earlier, explicit version of num-matmull, this version must
    execute one extra dip and one extra cons in gen-matmul. But this is negligible compared
    with the amount of real work done later by either version, especially the I*J*K numerical
    multiplications (in [*]) and numerical additions (in [+]) required for any numerical
    matrix multiplication program. Exactly the same is true for the logical version.
    <h1>4. A cleaner multiplication combinator</h1>
    <p>
      There is really nothing one could do to improve the addition combinator gen-matadd.
      But as the two examples of applications of the gen-matmul combinator show, they
      will all have to use the map2 and the fold combinator. It would be cleaner to have
      the map2 and the fold as part of the general combinator, so that users only have
      to include what is specific to the datatype of the matrices. The specific parts
      are a binary operator [B1] (which is [*] or [and] in the examples), and also a
      zero element Z2 and a binary operator [B2] (which are 0 and [+] or false and [or]
      in the examples). The value Z2 has to be the zero element for the binary operation
      in [B2]. As gen-matmul stands, one has to provide one quotation parameter, in the
      form
    </p>
    <pre>[ [B1] map2  Z2 [B2] fold ]  gen-matmul
    </pre>
    It would be cleaner if one could provide just what is really needed:
    <br>
    <pre>[B1]  Z2 [B2]  gen-matmul
    </pre>
    The required change to gen-matmul is quite simple, all that is needed is a
    preliminary program P which changes the three simple parameters to the one complicated
    parameter:
    <pre>[B1]  Z2 [B2]   P   ==&gt;   [ [B1] map2  Z2 [B2] fold ]
    </pre>
    The preliminary program can first use [fold] cons cons, and this will at least
    change Z2 [B2] into [Z2 [B2] fold]. Following that some other changes C are needed:
    <pre>[B1]  Z2 [B2]  [fold] cons cons  C   ==&gt;   [ [B1] map2  Z2 [B2] fold ]
    </pre>
    The first parameter [B1] has to be changed to [[B1] map2], but this has to
    be done with dip just below the last constructed quotation. It is done by [[map2]
    cons] dip:
    <pre>[B1] Z2 [B2] [fold] cons cons [[map2] cons] dip F ==&gt; [ [B1] map2 Z2 [B2] fold ]
    </pre>
    where the finalisation F simply has to concatenate the two previous constructions:
    <pre>[B1] Z2 [B2] [fold] cons cons [[map2] cons] dip concat
                          ==&gt;    [B1] map2 Z2 [B2] fold ]
    </pre>
    The above program becomes a new first line in the otherwise unchanged program
    for the gen-matmul combinator:
    <pre>gen-matmul   ==   [fold] cons cons [[map2] cons] dip concat
                      [transpose] dip
                      [map popd] cons cons
                      map
    </pre>
    Now the multiplication operators for numerical and logical matrices can be
    given by the more perspicuous definitions
    <pre>num-matmul   ==   [*]  0 [+]  gen-matmul
    log-matmul   ==   [and]  false [or]  gen-matmul
    </pre>
    <h1>5. An even more general multiplication combinator</h1>
    <p>
      The two definitions just above will be suitable for many purposes. Like all definitions
      so far, they even work for degenerate matrices with no rows and no columns. Such
      degenerate matrices are of course represented as [], and their arithmetic or logical
      product is again []. For other datatypes it is often easy to give the appropriate
      definition. For example one might need an operation for the multiplication of matrices
      in which the elements are sets. In Joy set intersection is just and, wheras set
      union is just or. The zero element for union is {}. So a suitable definition is
    </p>
    <pre>set-matmul   ==   [and]  {} [or]  gen-matmul
    </pre>
    Comparing logical matrix multiplication and set matrix multiplication, the
    two are almost identical except for the different zero elements false and {}. This
    difference has the consequence that whereas the and-operator and the or-operator
    work equally for logical and set operands, we cannot define a single matrix multiplication
    operator that works equally for logical and set operations. That would be unfortunate,
    although one could first do a very simple test on the first element in the first
    row of M2 to determine whether it is a truth value or a set.
    <p>
      But there are other cases where there could not be such a simple device. Consider
      the problem of multiplying second order matrices in which the elements themselves
      are numerical matrices of compatible rows and columns but where the number of rows
      and columns are not known in advance. This means that the zero element for the
      addition of the submatrices is also not known in advance, and hence in the definition
      below the ??? is not known in advance.
    </p>
    <pre>num-matmul2   ==   [num-matmul]  ???  [num-matadd]  gen-matmul
    </pre>
    What is particularly annoying is that ??? is the only part that needs to be
    known for the definition to work.
    <p>
      There would be other cases in which the same problem arises. They will all involve
      the datatype of the Z2 zero element for the binary [B2] operation, where that datatype
      is of indeterminate size, shape or structure. There are no such restrictions on
      [B1] and [B2]. It is true that a suitable Z2 can always be constructed from two
      sample elements from M1 and M2 by applying the [B1] operation, and then making
      a suitable Z2 from that. All this would have to be encoded in a very complex version
      of gen-matmul.
    </p>
    <p>
      But there is a simpler solution. Consider again the versions of the numerical matrix
      multiplication operators of sections 1 and 2. They required, in some way or another,
      a program to compute a scalar product:
    </p>
    <pre>[ [*] map2  0 [+] fold ]
    </pre>
    In all but the degenerate case the list produced by map2 will not be null.
    So in the normal case the list can be taken apart with unswons. That will leave the
    first element of that list, followed by the rest of the list. The rest can now be
    folded by using not 0 as the initial value of the accumulator, but the first element
    of the list. In other words, the above program fragment could be replaced by
    <pre>[ [*] map2  unswons [+] fold ]
    </pre>
    So it would be possible to have just [*] and [+] as the parameters to the
    general matrix multiplication combinator, and let the combinator supply map2, unswons
    and fold. This would imply changing the first line of the last version by a program
    P which does the conversion
    <pre>[B1]  [B2]  P   ==&gt;   [ B1] map2  unswons [B2] fold ]

    The conversion is quite similar to the earlier one.
    But now only one parameter, [B1] has to be consed into [fold],
    then [B1] has to be consed, through dip, into [map2 unswons],
    and then these results concatenated.
    So the required program P is
    <pre>[fold] cons  [[map2 unswons] cons] dip  concat
    </pre> This program replaces the first line of the previous definition of the general
    matrix multiplication combinator:
    <pre>gen-matmul   ==   [fold] cons [[map2 unswons] cons] dip concat
                      [transpose] dip
                      [map popd] cons cons
                      map
    </pre> Now the multiplication operators can be defined quite simply. The first is
    for numerical matrices. In Joy0 this means just integer, in Joy1 this means integer
    or float. The second is for Boolean matrices, of either truth values or sets. The
    third is for second order matrices of numerical matrices. The fourth is for second
    order matrices of Boolean matrices.
    <pre>num-matmul     ==   [*]  [+]  gen-matmul
    bool-matmul    ==   [and]  [or]  gen-matmul
    num-matmul2    ==   [num-matmul]  [num-matadd]  gen-matmul
    bool-matmul2   ==   [bool-matmul]  [bool-matadd]  gen-matmul
    </pre> It is easy to see how multiplication of third order matrices would be defined.
    But it is doubtful whether even second order matrices arise.

    <h1>6. Polymorphic matrix operators</h1>
    <p>
      It could be useful to have a single matrix mutltiplication operator which can handle
      at least numerical and Boolean matrices. To do so, it must inspect the type of
      the elements. If they are numerical it must behave like num-matmul above, and if
      the are Boolean it must behave like bool-matmul above. If the type is neither,
      an error must be reported and the execution aborted. As a first draft, the following
      is useful:
    </p><pre>                dup first first
                    (* branching on the type,
                       push [*] and [+], or push [and] and [or],
                       or report error and abort *)
                    gen-matmul
    </pre> The branching looks like a three-way branch which could be handled by two
    ifte combinators, one inside another, or by a single cond combinator. But in Joy1
    the numerical and the Boolean cases both split into two: the numerical types are
    integer and and float, and the Boolean cases are logical and set. (In Joy0 the only
    numerical type is integer.) So each of the two correct situations can arise in two
    ways: when the element type is integer or float, and when the element type is logical
    or set. The predicates numerical and boolean test for that. The cond combinator takes
    the two normal cases and as its default the error condition.
    <pre>                [ [ [numerical]   pop   [*] [+]         ]
                      [ [boolean]     pop [and] [or]        ]
                      [ "unknown operands\n" putchars abort ] ]
                    cond
    </pre> Alternatively, the branching could be based on those four cases, with a fifth,
    default case for the error situation. Since this branching is on the basis of types,
    the opcase operator is most suitable.
    <pre>                [ [ 0     pop [*]   [+]               ]
                      [ 0.0   pop [*]   [+]               ]
                      [ true  pop [and] [or]              ]
                      [ {}    pop [and] [or]              ]
                      [ "unknown operand\n" putchars abort] ]
                    opcase
    </pre> It may be a matter of taste whether the version with the cond combinator or
    the version with the opcase combinator is preferable. Either of the two could be
    used as the insertion in the earlier skeleton.
    <p>
      But before that, it is useful to remember at this point that the gen-matmul combinator
      will fail for degenerate matrices. But that is easily fixed right here: the product
      of two degenerate matrices is just the degenerate matrix []. So the earlier skeleton
      should be wrapped inside an ifte combinator: if the top matrix is degenerate, pop
      if off and return the other matrix (which should be degenerate, too). Otherwise,
      proceed with the ordinary version.
    </p><pre>poly-matmul   ==
        [ null ]
        [ pop ] 
        [     dup first first
              [ [ [numerical] pop [*]   [+]              ]
                [ [boolean]   pop [and] [or]             ]
                [ "unknown operand type\n" putchars abort] ]
              cond
              gen-matmul ]
        ifte
    </pre> This definition of the polymorphic matrix multiplication operator is typical
    of how generality can be achieved in Joy without the use of object oriented concepts.
    The extra computation needed for such polymorphism might seem a waste, but it is
    essential for writing useful general libraries.
    <p>
      NOTE: Some of the definitions in here have been included in the matrix/vector library,
      but under quite different systematic names.
    </p>
    </pre>
  </article>
</template>

<style src="~/assets/articles.css" scoped>
</style>