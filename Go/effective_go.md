### Format

use gofmt to format all the codes automatically. 

Go needs fewer parentheses. And the operator precedence hierarchy is presented by the spaces like 

> x<<8 + y<<16 // << has high precedence than + so the whole expression is segregated by the plus sign and is divided into two parts.

### Commentary

line comment is recommended. 

The block comment is used as the document of a package. Comment any one of the file of a package before the package statement. Then "go doc" command outputs this comment as the document for current package.

Go's declaration syntax allows grouping of declarations. A single doc comment can introduce a group of related constants or variables. 

### Names

1. Package names
   * package name should be good: short, concise, evocative
   * Inside a package, some functions should be named "independently from the package's name". For example, the Reader function in bufio package is not named as BufReader since the reference to this function outside the package would be "bufio.Reader" which is consice and short.  Another example is ring.New instead of ring.NewRing, the latter one does not make the code more readable.
2. Getters
   * no need to name a getter "GetXX", just "XX" would be fine if that field is exported. 
   * SetXX is the setter
3. Interface names
   * stick to the existing interfaces
4. MixedCaps
   * no underscore

### Semicolons

Semicolons are mostly omitted and the lexer adds semicolons automatically. 

If a semicolon is followed by a closing brace, it can be omitted.

### Control structures

1. If
   * if statement acception an initialization statement.
   * A typical flow control pattern in Go is like this: any error case terminates soon as they arise and successful branch goes down to the page. 
2. Redeclaration and reassignment
3. For
   * For "range" a map or an array provides a pair iterator <index, value>, where value can be omitted. (remember this rule helps to write a for-loop quickly and less error-prone)
   * use parallel assignment to control multiple variables in the loop.
4. Switch
   * pretty much like C in the flow
   * but the expressions can be the constant and variables. 
   * same as "if-else-if-else" chain.
   * Label helps to point out which flow should be terminated(whether the switch or the for loop for example).  But I think this is not so readable. 
5. Type switch
   * type assertion t.(type)

### Functions

1. Multiple return values
2. Named result parameters
   * when the result parameters are named, they become the part of the document
   * they are also returned when an unadorned return statement is used and their value are returned. 
3. Defer
   * the arguments of defer functions are determined by value at the time of the execution of defer not the execution of the function. 
   * LIFO order
   * Function-based 

### Data

1. Memory allocation with "new"

   * "new" returns a zerod storage address. 
   * the zero-value property works transitively. 

2. Constructors and composite literals

   * Zero is not good enough
   * use a constructor!
   * a composite literal are laid out in order and must all be present but labeling the fields also works and other missing fields are filled with zero. 

3. Allocation with "make"

   * creates slices, maps and channels only since these kinds of variables should be initialized before used. 
   * does not return a pointer

4. Array

   * array are value, not pointer!

5. Slice

   * slices hold a reference to an underlying array and if you assign one slice to another, both refer to the same array. 

   * See the example code of Append

     ```go
     func Append(slice, data []byte) []byte {
         l := len(slice)
         if l + len(data) > cap(slice) {  // reallocate
             // Allocate double what's needed, for future growth.
             newSlice := make([]byte, (l+len(data))*2)
             // The copy function is predeclared and works for any slice type.
             copy(newSlice, slice)
             slice = newSlice
         }
         slice = slice[0:l+len(data)]
         copy(slice[l:], data)
         return slice // slice must be returned since it is passed as a value.
     }
     ```

6. Two-dimensional slices

7. Maps

   * Like slices, maps hold references to an underlying data structure.

8. Printing

   * "%v", "%+v", "%#v"
   * a infinite recursion error on String()

9. Append

   * the underlying array may change.

   *  append a slice to a slice: 

   * ```go
     x = append(x, y...)
     ```

