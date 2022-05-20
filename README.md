<div align='center'>

  <img src='https://user-images.githubusercontent.com/66398400/169051355-edd50385-e66d-4255-a6ef-e60581b30019.gif'/>

# Rust Memory Layout

Describing the Memory Layout in Rust Language

<br><br><br>

</div>

<table width="100%">
  <tr>
  <td width="20%" align=center>

<h1>Kernel Virtual Memory Space Overview</h1>

<kbd>
 <h3 align=center>&nbsp;&nbsp;&nbsp;HIGH MEMORY ADDRESS&nbsp;&nbsp;&nbsp;</h3>
</kbd>

...

| <h3>[STACK](#stack)<br>⟱</h3>                                            |
| ------------------------------------------------------------------------ |
| <div align=center>**main()**</div>                                       |
| <img width=200><br><br><div align=center>free memory</div><br><br></img> |
| <h3 align=center>⟰<br>[HEAP](#heap)</h3>                                 |
| <div align=center>**Block Started by Symbol**</div>                      |
| <div align=center>**Data Segment**</div>                                 |
| <div align=center>**Text Segment**</div>                                 |

...

<kbd>
 <h3 align=center>&nbsp;&nbsp;&nbsp;LOW MEMORY ADDRESS&nbsp;&nbsp;&nbsp;</h3>
</kbd>

  </td>

  <td width="80%">

# [Memory Address Range](#memory-address-range)

### Basic x86-64 Linux ELF Binary

- The memory address range is bounded the the `word` size of the `CPU`.
- In a **64-bits** processor the `word` size is **64/8 or 8 bytes**.
- A **32-bits** processor can only address up to **2^32 or ~4GB** of [byte-addressable](https://en.wikipedia.org/wiki/Byte_addressing) memory. On the other hand **64-bits** processor can address from **0 to 2^64-1 or 16 billion GB**. Filling up the total memory is equivalent to turn every **0 bit into 1**
- On **64-bits** `CPU` only **48-bits or ~281TB** is used for memory addressing with the remaining **16 bits** of the virtual address required to be all 0's or all 1's.
- Using the above as example only **1-bit or ~141TB** is used for `kernelspace` and the rest is used for `userspace` memory.

# [Text Segment](#text-segment)

- The text segment, aka code segment, is where the Rust code is compiled by LLVM into machine code and then execute instructions

# [Data Segment](#data-segment)

- The data segment contains the initalized variables

# [BSS](#bss)

- The Block Started by Symbol (BSS) contains the unitialized variables

  </td>
  </tr>
</table>

<br><br><br>

# The [STACK](#stack)

> SAMPLE FUNCTION

```rust
fn main() {
  let a = 48;
  let b = double(a);
  println!("{b}");
}

fn double(n: i32) -> i32 {
  n * 2
}
```

<table width="100%">
  <tr>
  <td width="20%" align=center>

| <h3>[STACK](#stack)<br>⟱</h3>                                                                                                         |
| ------------------------------------------------------------------------------------------------------------------------------------- |
| <div align=center>**main()**<br><br><kbd>&nbsp;**a = 48**&nbsp;</kbd><br><kbd>&nbsp;**b = 96**&nbsp;</kbd></div>                      |
| <div align=center>**double()**<br><br><kbd>&nbsp;**n = 48**&nbsp;</kbd><br><kbd>&nbsp;**fn return addr 0x12f = 96**&nbsp;</kbd></div> |
| <img width=200><br><br><div align=center>free memory</div><br><br></img>                                                              |
| <h3 align=center>⟰<br>[HEAP](#heap)</h3>                                                                                              |

  </td>

  <td width="80%">

# [STACK](#stack) Properties

- The `stack` is a **static memory** that requires a **fixed known size** at compile time and is stored inside the binary after compilation.
- The `stack` memory **grows downwards** starting from a [higher address](#high-memory-address) of around `0x7fffffffffff`.
- It is an **abstraction concept** that creates **processes/threads** and is very common to find in almost every program.
- Each process starts a **single thread by default** and each process has it's **own separate stack**.
- Rust `stack` size limit is **8MB** in a **64-bit** system but the default **start size is 2MB**.
- If your executition exceeds the stack limit you will be confronted with a **"stack overflow"** compile error.
- The stack **pointer size** is determined its **data type**. In this case `i32` needs **4 bytes** and `i64` needs **8 bytes**.
- Because of it's **static** nature it doesn't need any additional step to change it's size using system calls which **speeds up the execution**.

  </td>
  </tr>
</table>

🦀 **Depicting the memory allocation:**

1.  A `Stack Frame` is created and a `stack pointer` **increases by 8 bytes (for a 64-bit OS)** to keep track of the running program. The `main()` has a local variable `a` which is a `i32` so its `4 bytes` in size to be reserved in its `Stack Frame`.

2.  Next the variable `b` calls the function `double()` which creates another `Stack Frame` and the `stack pointer` **increases 8 bytes** in its own `Stack Frame`.

3.  In `double()` function the parameter `n` is a `i32` so it needs another `4 bytes` to be added on its own `Stack Frame`. The `double()` function run and returns a value. The `double()` function has a **return address** to be stored in the variable `b` inside `main()` and its a `i32` so another `4 bytes` again. The function is terminated and the **stack pointer decreases 8 bytes** deallocating `double()` to be **overwritten by another function in the future**.

4.  The **stack pointer** back to `main()` function and the variable `b` **receives the value returned** from `double()`, the `main()` function ends and the whole program terminates.

<br><br><br>

# The [HEAP](#heap)

> SAMPLE FUNCTION

```rust
fn main() {
  let heap = boxed_value();
  println!("{heap}");
}

fn boxed_value() -> Box<i32> {
  let result = Box::new(99);
  result
}
```

<table width="100%">
  <tr>
  <td width="20%" align=center>

| <h3>[STACK](#stack)<br>⟱</h3>                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------ |
| <div align=center>**main()**<br><br><kbd>&nbsp;**heap = 0x6f32**&nbsp;</kbd></div>                                                               |
| <div align=center>**boxed_value()**<br><br><kbd>&nbsp;**result = 99**&nbsp;</kbd><br><kbd>&nbsp;**fn return addr 0x6f32 (99)**&nbsp;</kbd></div> |
| <img width=200><br><br><div align=center>free memory</div><br><br></img>                                                                         |
| <div align=center><kbd>&nbsp;**heap addr 0x6f32 (99)**&nbsp;</kbd></div>                                                                         |
| <h3 align=center>⟰<br>[HEAP](#heap)</h3>                                                                                                         |

  </td>

  <td width="80%">

# [HEAP](#heap) Properties

- The `heap` is another abstraction that, unlike `stack`, has a **flexible size** that can change over time (like in **run time**).
- Instead of one `Stack Frame` for each thread the `heap` memory region has a large **shared memory with the `stack` memory region**.
- The `heap` memory **grows upwards** starting from the [lower address](#low-memory-address).
- While the program is running the `heap` memory region **grows and shrinks** as needed.
- The `heap` is **not stored** inside the binary and is discarded after the program execution.
- Its size limit is bounded the system's memory.
- Each `heap` **value size** is determined by the **data type**.
- Because of it's **dyanmic** nature it needs to make system calls to ask for more space (using the **GlobalAlloc Trait that calls C malloc**) as soon as requested thus adding a little overhead. This process is done in **chunks to make the fewer system calls as possible**. Even using this technique this process turns the `heap` slower than the `stack` memory.
- The memory allocator keeps track of the **OS memory pages** to know which pages are **free** and which are **allocated**. This process is a way to **prevent more system calls** and **reuse** the avaible memory without waiting for the OS thus speeding things up.

  </td>
  </tr>
</table>

🦀 **Depicting the memory allocation:**

1.  The `main()` function creates a new `Stack Frame`. The **stack pointer increases 8 bytes** and the variable `heap` calls the function `boxed_value()`.

2.  In the `boxed_value()` the **stack pointer increses 8 bytes again** and we also have a local variable called `result` that has a `Box` with a value inside. The **`Box` acts as a pointer** so the return value size is **8 bytes** as well (the `heap` variable in `main()` will be the same **8 bytes but as an address pointing to the `Heap Memory`**). Inside the `Box` we have a value `99` which is `i32` so we need to reserve **4 bytes but this time on the heap**. The function ends, **deallocates and decrases the stack pointer by 8 bytes**.

3.  The **pointer** back to the `main()` and the **heap variable** now receives a **copied address** of the value stored in the shared `Heap Memory`. Finally the funtion runs and terminates the program.

# [DATA TYPES](#data-types)

  <table width="100%">
  <tr>
  <td width="30%" align=center>

| <h4>&nbsp;[INTEGER TYPES](#integer-types)&nbsp;</h4> |
| ---------------------------------------------------- |

| SIGNED                        | UNSIGNED                      |
| ----------------------------- | ----------------------------- |
| <div align=center>i8</div>    | <div align=center>u8</div>    |
| <div align=center>i16</div>   | <div align=center>u16</div>   |
| <div align=center>i32</div>   | <div align=center>u32</div>   |
| <div align=center>i64</div>   | <div align=center>u64</div>   |
| <div align=center>i128</div>  | <div align=center>u128</div>  |
| <div align=center>isize</div> | <div align=center>usize</div> |

<br><br>

| <h4>&nbsp;[FLOATING POINT TYPES](#floating-point-types)&nbsp;</h4> |
| ------------------------------------------------------------------ |
| <div align=center>f32</div>                                        |
| <div align=center>f64</div>                                        |

| <h4>&nbsp;[CHARACTER TYPE](#character-type)&nbsp;</h4> |
| ------------------------------------------------------ |
| <div align=center>char</div>                           |

| <h4>&nbsp;[STRING TYPES](#string-types)&nbsp;</h4> |
| -------------------------------------------------- |
| <div align=center>String</div>                     |
| <div align=center>&str</div>                       |

| <h4>&nbsp;[BOOLEAN TYPE](#boolean-type)&nbsp;</h4> |
| -------------------------------------------------- |
| <div align=center>bool</div>                       |

| <h4>&nbsp;[TUPLE TYPE](#tuple-type)&nbsp;</h4> |
| ---------------------------------------------- |
| <div align=center>(i32, String, char)</div>    |

| <h4>&nbsp;[ARRAY TYPE](#array-type)&nbsp;</h4> |
| ---------------------------------------------- |
| <div align=center>[i32; 10]</div>              |

| <h4>&nbsp;[VECTOR TYPE](#vector-type)&nbsp;</h4> |
| ------------------------------------------------ |
| <div align=center>Vec<i32></div>                 |

| <h4>&nbsp;[UNIT TYPE](#union-type)&nbsp;</h4> |
| --------------------------------------------- |
| <div align=center>()</div>                    |

| <h4>&nbsp;[ENUM TYPE](#enum-type)&nbsp;</h4> |
| -------------------------------------------- |
| <div align=center>Color</div>                |

| <h4>&nbsp;[STRUCT TYPE](#struct-type)&nbsp;</h4> |
| ------------------------------------------------ |
| <div align=center>Point</div>                    |
| <div align=center>Point::new(0, 0)</div>         |

| <h4>&nbsp;[REFERENCE TYPE](#reference-type)&nbsp;</h4> |
| ------------------------------------------------------ |
| <div align=center>&T</div>                             |

| <h4>&nbsp;[SLICE TYPE](#slice-type)&nbsp;</h4> |
| ---------------------------------------------- |
| <div align=center>&[i32]</div>                 |

| <h4>&nbsp;[GENERIC TYPE](#generic-type)&nbsp;</h4> |
| -------------------------------------------------- |
| <div align=center>Type<T></div>                    |

  </td>

  <td width="70%">

# Properties

- **Signed Integers** and **Unsigned Integers** have **known sizes** at compile time so they can be stored fully in the `Stack` memory region.
- `isize` and `usize` are machine words of **4 bytes or 8 bytes (32 or 64 bits, respectively)** that depends on the **OS architecture**.
- The `char` type stores [UNICODE](#unicode) characters with a size of **4 bytes** .
- `Tuple` type can store **multiple other types** in the `Stack` memory region. If multiple type sizes inside a `Tuple` is less than the `Stack` memory size avaiable then a **padding will fill the remaining space** (use the `std::mem::size_of::<T>()` and `std::mem::align_of::<T>()` to understand how this better).
- The `Array` holds a **known fixed size** of **multiple values of the same type** in the `Stack` memory region.
- **Floating point numbers** are **stored in **IEEE 754\*\* format.
- **Booleans** are **stored as **1 byte\*\*.

- The `Vector` type is a **dynamic array** that can **grow and shrink** as needed. The size of the `Vector` is **not known** at compile time thus the `Vector` is allocated in the `Heap` memory region. To keep track of the `Vector` value inside the `Heap` we need to store the `Vector` in the `Stack` using 3 machine words:

<div align=center>

| addr | cap | len |
| ---- | --- | --- |

</div>

1.  The first word stores the **address** of the `Vector` in the `Heap` memory region.
2.  The second word stores the **capacity** of the `Vector` (the maximum number of elements that can be stored in the `Vector`).
3.  The third word stores the **length** of the `Vector` (the number of elements that are currently stored in the `Vector`).

> 🛈 RESIZING A VECTOR
>
> When the `Vector` overeaches its capacity then it will **grow** the `Vector` by **adding a new chunk of memory (using malloc)** to the `Heap` memory region then **copy the old values** to the new chunk of memory and finally update the **pointer** to the new avaiable space.

- `Strings` are actually `Vectors` and are stored in the `Heap` memory region as **a sequence of bytes `UTF-8` encoded**. `String Slice` or `&str` on the other hand are stored in the `Stack` memory region and have a `static` lifetime which means it'll be stored in the **binary** file and last throughout the program execution.
- The `Struct` comes in 3 kinds: `Struct` with **named fields**, **Tuple like** `Struct` and **Unit like** `Struct`. Tuple like `Structs` and named fields `Structs` act similar to a `Tuple` type. **Unit like** `Structs` **doesn't hold any data (0 bit)** and is useful for classifing without using memory.
- The `Enum` type has fields called **variants**. If a variant doesn't hold any value they are stored in the `Stack` memory region as **a sequence of integers starting from 0**. `Enums` with variant values has their size defined by the **largest variant value** and this value will be used as the size of each variant. To reduce the memory used by `Enums` you can try to use a `Box<T>` to store the variant value in the `Heap` memory region.

## [REFERENCE](#reference) and [SLICE](#slice) Type

- The `Reference` type is a **pointer (8 or 4 bytes OS dependent)** to a type that is allocated either on the `Stack` or the `Heap` memory region. It is stored on the `Stack` and is respresented by the `&T` syntax using 2 machine words, one for the starting **address** and another for its **length**.
- The `Slice` type is a **slice (or a view)** of an `Array` or `Vector` that can **only be read**. It acts like a **pointer** but needs an additional **length** machine word to know how many elements to read. This kind of pointer is also known as **fat pointer**. `Strings` can also be **sliced** using the `Slice` type becoming a `String Slice` or `&str` and **stored on the `Stack`**.

  </td>
  </tr>
</table>
