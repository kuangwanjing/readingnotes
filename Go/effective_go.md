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

### Initialization

1. Constants
   * is handled in compile time so the expression must be evaluable by the compiler(Go sematics)
   * [iota](https://github.com/golang/go/wiki/Iota) helps to create complex enumeration type.
2.  Variables: created in runtime
3. The init function

### Methods

1. Pointers vs. Values
   * receiver does not have to be a struct, any data type except a pointer or an interface can be defined as the receiver of a method.
   * The rule about pointers vs. values for receivers is that value methods can be invoked on pointers and values, but pointer methods can only be invoked on pointers. (The language prevents the mistake from the language syntax)
   * [good example](https://golang.org/doc/effective_go.html#pointers_vs_values) of binding a Writer method on ByteSlice, in which ByteSlice works as a writer that can receives any bytes from the input, then the program can do any string/bytes formatting on ByteSlice.

### Interfaces and other types

1. Interfaces
   * Example: io.Writer implements writer interface.
   * a type can implement multiple interfaces
2. Conversions
   * the defined type can be converted into its underlying type.
3. Interface conversions and type assertions 
   * value.(typeName) converts the interface into a certain type. If it can not be converted, it returned a false "ok". 
4. Generality
   * If a type exists only to implement an interface and will never have exported methods beyond that interface, there is no need to export the type itself.
   * Other part of the program just treats the type as the interface that type implements so that the program stays clean. Example: crypto/cipher, the streamer is not interested in the encryption and decryption algorithm, so different encryption objects implements the Block interface and expose the interface methods and hide the detail inside. 
5. Interfaces and methods
   * see the examples
   * methods can be converted too(into a method with the same signature) -- this becomes an adapter.

### The blank identifier

 it represents a write-only value to be used as a place-holder where a variable is needed but the actual value is irrelevant.

1. The blank identifier in multiple assignment: general cases

2. Unused imports and variables

   * why import an unused package? -- for development use, sometimes the package is not needed but will be needed later. Deleting that line is annoying.

   * Example:

     ```go
     package main
     
     import (
         "fmt"
         "io"
         "log"
         "os"
     )
     
     var _ = fmt.Printf // For debugging; delete when done.
     var _ io.Reader    // For debugging; delete when done.
     
     func main() {
         fd, err := os.Open("test.go")
         if err != nil {
             log.Fatal(err)
         }
         // TODO: use fd.
         _ = fd
     }
     ```

3. Import for side effect:  it is useful to import a package only for its side effects, without any explicit use. (find more examples!)

4. Interface checks

   * type check at runtime
   * Example: json 

### Embedding

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

a union of interfaces

1. interface: an interface can embed multiple other interfaces. So that the interface can work like one of these embeded interfaces. Example: see above. 
2. Struct: it lists the types within the struct but does not give them field names. 

Basic rules for embedding in struct: if an interface is embedded in a struct, then all the interface methods should be implemented by the struct otherwise this error occurs since the method is not addressable. 

>  panic: runtime error: invalid memory address or nil pointer dereference

If an interface is not embedded but the struct tries to implement the interface, implement all the interface methods then. 

If a struct is embedded, then the outer struct can "borrow" all the method the inner struct has. 

### Concurrency

*Do not communicate by sharing memory; instead, share memory by communicating.*

buffered channel can be used to limit the throughput(semaphore).

channels of channels: the second channel is used to receive the result from the handler of the first channel. (Synchronization)

example of leaky buffer

### Error 

panic & recover

