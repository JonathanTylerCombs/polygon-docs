The Miden read–eval–print loop (REPL) is a Miden shell that allows for quick and easy debugging of Miden assembly. After the REPL gets initialized, you can execute any Miden instruction, undo executed instructions, check the state of the stack and memory at a given point, and do many other useful things! When the REPL is exited, a `history.txt` file is saved. One thing to note is that all the REPL native commands start with an `!` to differentiate them from regular assembly instructions.

Miden REPL can be started via the CLI [repl](../intro/usage.md#cli-interface) command like so:

```sh
./target/optimized/miden repl
```

## Miden assembly instruction

All Miden instructions mentioned in the [Miden Assembly sections](../user-docs/assembly/index.md) are valid. One can either input instructions one by one or multiple instructions in one input.

For example, the below two commands will result in the same output.

```sh
>> push.1
>> push.2
>> push.3
```

```sh
push.1 push.2 push.3
```

To execute a control flow operation, one must write the entire statement in a single line with spaces between individual operations.

```sh
repeat.20
    pow2
end
```

The above example should be written as follows in the REPL tool:

```sh
repeat.20 pow2 end
```

### !help

The `!help` command prints out all the available commands in the REPL tool.

### !program

The `!program` command prints out the entire Miden program being executed. E.g., in the below scenario:

```sh
>> push.1.2.3.4
>> repeat.16 pow2 end
>> u32checked_add

>> !program
begin
    push.1.2.3.4
    repeat.16 pow2 end
    u32checked_add
end
```

### !stack

The `!stack` command prints out the state of the stack at the last executed instruction. Since the stack always contains at least 16 elements, 16 or more elements will be printed out (even if all of them are zeros).

```sh
>> push.1 push.2 push.3 push.4 push.5
>> exp
>> u32checked_mul
>> swap
>> eq.2
>> assert
```

The `!stack` command will print out the following state of the stack:

```sh
>> !stack
3072 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
```

### !mem

The `!mem` command prints out the contents of all initialized memory locations. For each such location, the address, along with its memory values, is printed. Recall that four elements are stored at each memory address.

If the memory has at least one value that has been initialized:

```sh
>> !mem
7: [1, 2, 0, 3]
8: [5, 7, 3, 32]
9: [9, 10, 2, 0]
```

If the memory is not yet been initialized:

```sh
>> !mem
The memory has not been initialized yet
```

### !mem[addr]

The `!mem[addr]` command prints out memory contents at the address specified by `addr`.

If the `addr` has been initialized:

```sh
>> !mem[9]
9: [9, 10, 2, 0]
```

If the `addr` has not been initialized:

```sh
>> !mem[87]
Memory at address 87 is empty
```

### !undo

The `!undo` command reverts to the previous state of the stack and memory by dropping off the last executed assembly instruction from the program. One could use `!undo` as often as they want to restore the state of a stack and memory $n$ instructions ago (provided there are $n$ instructions in the program). The `!undo` command will result in an error if no remaining instructions are left in the Miden program.

```sh
>> push.1 push.2 push.3
>> push.4
>> !stack
4 3 2 1 0 0 0 0 0 0 0 0 0 0 0 0

>> push.5
>> !stack
5 4 3 2 1 0 0 0 0 0 0 0 0 0 0 0

>> !undo
4 3 2 1 0 0 0 0 0 0 0 0 0 0 0 0

>> !undo
3 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0
```