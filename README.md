# ViewReader
Read files without making new allocations for each line.

----

### How it works
We basically implement a buffered reader where the buffer is a vector of UInt8. We then stream the bytes from the file through this buffer and search for newline characters. On top these vectors we use the amazing  [StringView](http://https://github.com/JuliaStrings/StringViews.jl "StringView") package to view and compare strings without any allocations. For more detail of the actual implementation see `FileReader.jl`. NOTE, for this to work the `buffer_size` should be bigger than the longest line.

----

### Features
Currently we only have some basic features like reading a line and splitting it

#### 1. eachlineV
**`eachlineV(file_path::String; buffer_size::Int64=10_000)`**
This function can be used just like the base[ `eachline` ](https://docs.julialang.org/en/v1/base/io-network/#Base.eachline " `eachline` ")in Julia. The argument `buffer_size` determines the size of the underlaying UInt8 vector. The `buffer_size` should be bigger than the longest line in a file. If this is uknown just use a big number like 1M. This function will throw a warning if no new line is found when the eof is not reached yet - giving a clue to increase the `buffer_size`. 

**Example**
```Julia
    for line in eachlineV("test.txt", buffer_size=100_000)
        println(line)
    end
```
(*Obviously it makes more sense to do comparisons here like `like == "X"` as printing will also allocate*)

----
#### 2. splitV
**`splitV(line::Sview, delimiter::Char)`**
Sinilar to the base `split`, although we currently only support a single character (not a string).

**Example**
For example to check how often we see the string "TARGET" at column 3 in a given file 
```Julia
c = 0
for line in eachlineV("test.txt", buffer_size=100_000)
        for (i, item) in enumerate(splitV(line, '\t'))  # <- splitV
            if i == 3 && item == "TARGET"
                c += 1
            end 
        end 
    end 
    return c
```
----

#### 3. *X*Int*X*V
**`XIntXV(lineSub::Sview)`**
As it's common to parse numbers from a line, and compare these we added some examples on how to parse integers without allocating them (see `Utils.jl`)

**Example**
For example, to parse numbers as `UInt32` from a file
```Julia
c = 0
 for line in eachlineV(f)
        for item in splitV(line, '\t')
            println("X: ", item)
            c += UInt32V(item) == UInt32(10)
        end 
    end
    println(c)
	```