# Ruby-Procs-and-Blocks
My explanation of procs and blocks

Here is my understanding and hope it guides you in the right direction.
Let's say you want to create your own ".each" method so let's see how a normal ".each" method works first.

```Ruby
[1, 2, 3].each {|n| puts n }
```

Executing the code above will output the following

```Ruby
1
2
3
=> [1, 2, 3]
```

Now I assume you know how to write this and what it will produce. Let's go over how to create this method ourselves. A naive approach might be the following.

```Ruby
def putsAll(arr)
  i = 0
  while(i < arr.length)
    puts arr[i]
    i += 1
  end
  return arr
end

putsAll([1,2,3])
```

Yay, now we have a method that "puts" all the elements from our array. But wait... something is wrong. What if we want to sum up all the numbers instead? This is a problem that procs can help solve. We need to rewrite our function so that it takes some kind of process. Here our process is to print out all our numbers (for sake of brevity I'm gonna use puts and print interchangeably.. Let's write a "Proc" that does just that. 

```Ruby
myPrintingProcess = Proc.new {|n| puts n }
```

Here we are creating a new Proc object and assigning it to the variable myPrintingProcess. Proc objects takes a block as an argument so you create them as such. Other Ruby objects like Arrays takes normal arguments; for example, Array.new(3)

Now that we have a Proc, let's create a new method that will iterate through an array's elements and execute whatever code that the proc passed in tells it to. (in this case... to print!)

```Ruby
def iterateAll(arr, someProc)
  i = 0
  while(i < arr.length) 
    someProc.call(arr[i])
    i += 1
  end
  return arr
end
```

This method takes two arguments, one is the array we want to iterate and the other is a Proc object. So say we want to print the first element of the array. How should we tell the Proc to use that element? We have to use the ".call" method! 

http://ruby-doc.org/core-2.2.0/Proc.html#method-i-call

The Ruby docs defines the .call method as such

"Invokes the block, setting the block’s parameters to the values in params using something close to method calling semantics"

Let me redefine that to "sets 'n' to whatever you passed in as the argument. So in the above example, we are setting the "n" to the first element of the array and then executing the proc. The following execution of my method should result in the same output.

```Ruby
iterateAll([1,2,3], myPrintingProcess)
```

So what is the difference between a block and a proc you say? A block is a set of statements that is passed into the method call. I like this definition from Google:
 
"A Ruby block is a way of grouping statements, and may appear only in the source adjacent to a method call"

```Ruby
def iterateAll(arr, &someBlock)
  i = 0
  while(i < arr.length) 
    yield(arr[i])
    i += 1
  end
  return arr
end

iterateAll([1,2,3]) { |n| puts n }
```

Here the method takes in an array as the first argument, and then a block. To execute the block with a given parameter similar to how Proc.call works, we have the Ruby keyword "yield". It works very much like Proc.call by setting the blocks parameter to the given argument and executing the block. The "&" symbol is required in the parameter name to tell Ruby that the argument being passed in will be a block.
 
Sorry for being so verbose and if this left you more confused than you started. Let me know if there are any questions. Thanks!

# Additional example
```Ruby

class Array                          # Monkey patch the Array class
  def my_each(&block)                # create a method called my_each that takes a block as an argument
    i = 0                            # create a variable called "i" and assign it to the number 0
    while(i < self.length)           # while the variable "i" is less than the the instance of the array's length...
      yield(self[i], i)                    # execute the block passed in with the arguments: ith element of the instace of the array and the value of "i" variable (the index)
      i += 1                         # increase the value of the variable "i" by one.
    end
  end

  def my_map(&block)                 # create a method called my_map that takes a block as an argument
    self.my_each do |n, idx|         # call the instance of the array's my_each method passing in the block that has the arguments "n" and "idx"
      self[idx] = yield(n)           # execution of the my_each block will set the instance of the array's ith element to whatever the my_map block returns
    end
    return self                      # return the instance of the array
  end
end


=begin 

  print ([1,2,3].my_map {|n| n * n })

  # I know this is ugly and not the right way to write code but this might explain what it happening

  my_each is called with the block: {|n, idx| self[idx] = yield(n) }

  for the first element the block "yielded" such that:

  n = 1,
  idx = 0 

  so self[0] = {|1| 1 * 1 } <--- this block comes from the yield call within the my_map method.
     self[0] = 1

  second element...

  n = 2
  idx = 1 

  so self[1] = {|2| 2 * 2 }
     self[1] = 4 


  third element...

  n = 3
  idx = 2 

  so self[2] = {|3| 3 * 3 }
     self[2] = 9 

  in the end we return self.

  => [1, 4, 9]

=end
```
