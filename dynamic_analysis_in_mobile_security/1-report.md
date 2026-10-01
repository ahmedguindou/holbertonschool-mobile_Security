# Report — Task 1: Hooking Native Android Functions

## Objective

The purpose of this task was to recover the hidden flag stored inside the native part of an Android application. The analysis was performed statically, without executing the APK. Both the Java bytecode and the native `.so` library were examined to understand how the secret value was generated.

## 1. Analyzing the Java Code

First, the APK was decompiled with JADX:

```bash
jadx ~/Desktop/task1_d.apk -d ~/Desktop/task1_jadx
```

After examining `MainActivity.java`, a native method called `getSecretMessage()` was identified:

```java
public final native String getSecretMessage();

static {
    System.loadLibrary("native-lib");
}
```

This indicates that the actual implementation of `getSecretMessage()` is not present in the Java source. Instead, it is located inside the native library named `native-lib`.

The method is also not directly used to display the secret in the application's interface, so analyzing the native binary became necessary.

## 2. Locating the Native Library

To extract the application's native components, apktool was used:

```bash
apktool d ~/Desktop/task1_d.apk -o ~/Desktop/task1_out
```

The available native libraries were then searched with:

```bash
find ~/Desktop/task1_out -name "*.so"
```

Several versions of `libnative-lib.so` were found for different CPU architectures, including:

* x86_64
* arm64-v8a
* x86
* armeabi-v7a

For further analysis, the x86_64 version was selected.

To identify the JNI function, the exported symbols were inspected:

```bash
nm -D ~/Desktop/task1_out/lib/x86_64/libnative-lib.so | grep -i "secret|jni"
```

The following symbol was present:

```text
0000000000000860 T Java_com_holberton_task2_1d_MainActivity_getSecretMessage
```

This confirms that the Java native method is connected to a JNI implementation inside the shared library.

## 3. Examining the Native Function

The next step was to inspect the machine code of the identified function:

```bash
objdump -d ~/Desktop/task1_out/lib/x86_64/libnative-lib.so | grep -A 80 "getSecretMessage"
```

The disassembly revealed the basic decryption procedure.

The function reads 49 bytes from the read-only data section, starting at offset `0x5f0`. Each byte is then modified according to its position in the string.

For every index `i`, the following operation is effectively performed:

```text
decoded[i] = encoded[i] - Fibonacci(i % 10)
```

The Fibonacci values used by the routine are calculated for indices 0 through 9:

```text
0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```

After processing the bytes, the resulting text is converted into a Java string using `NewStringUTF`.

## 4. Recovering the Encoded Bytes

The contents of the `.rodata` section were inspected using:

```bash
readelf -x .rodata ~/Desktop/task1_out/lib/x86_64/libnative-lib.so
# Report — Task 1: Hooking Native Android Functions

## Objective

The purpose of this task was to recover the hidden flag stored inside the native part of an Android application. The analysis was performed statically, without executing the APK. Both the Java bytecode and the native `.so` library were examined to understand how the secret value was generated.

## 1. Analyzing the Java Code

First, the APK was decompiled with JADX:

```bash
jadx ~/Desktop/task1_d.apk -d ~/Desktop/task1_jadx
```

After examining `MainActivity.java`, a native method called `getSecretMessage()` was identified:

```java
public final native String getSecretMessage();

static {
    System.loadLibrary("native-lib");
}
```

This indicates that the actual implementation of `getSecretMessage()` is not present in the Java source. Instead, it is located inside the native library named `native-lib`.

The method is also not directly used to display the secret in the application's interface, so analyzing the native binary became necessary.

## 2. Locating the Native Library

To extract the application's native components, apktool was used:

```bash
apktool d ~/Desktop/task1_d.apk -o ~/Desktop/task1_out
```

The available native libraries were then searched with:

```bash
find ~/Desktop/task1_out -name "*.so"
```

Several versions of `libnative-lib.so` were found for different CPU architectures, including:

* x86_64
* arm64-v8a
* x86
* armeabi-v7a

For further analysis, the x86_64 version was selected.

To identify the JNI function, the exported symbols were inspected:

```bash
nm -D ~/Desktop/task1_out/lib/x86_64/libnative-lib.so | grep -i "secret|jni"
```

The following symbol was present:

```text
0000000000000860 T Java_com_holberton_task2_1d_MainActivity_getSecretMessage
```

This confirms that the Java native method is connected to a JNI implementation inside the shared library.

## 3. Examining the Native Function

The next step was to inspect the machine code of the identified function:

```bash
objdump -d ~/Desktop/task1_out/lib/x86_64/libnative-lib.so | grep -A 80 "getSecretMessage"
```

The disassembly revealed the basic decryption procedure.

The function reads 49 bytes from the read-only data section, starting at offset `0x5f0`. Each byte is then modified according to its position in the string.

For every index `i`, the following operation is effectively performed:


The Fibonacci values used by the routine are calculated for indices 0 through 9:
0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```
After processing the bytes, the resulting text is converted into a Java string using `NewStringUTF`.

readelf -x .rodata ~/Desktop/task1_out/lib/x86_64/libnative-lib.so

```text
48 70 6d 64 68 77 7c 7c 83 9d
6e 62 75 6b 75 6a 67 75 84 91
```

The final `00` byte marks the end of the encoded string.
To verify the reverse-engineered algorithm, it was reproduced independently in Python:

```python
def fib(n):
    a, b = 0, 1
    for _ in range(2, n + 1):

    return b

fibs = [fib(i) for i in range(10)]

raw = [
    72, 112, 109, 100, 104, 119, 124, 124, 131, 157,
    110, 98, 117, 107, 121, 106, 103, 117, 132, 145,
    95, 101, 106, 104, 105, 106, 122, 114, 131, 150,
    95, 98, 117, 97, 100, 113, 116, 138, 0
]
decoded = ""

for i, value in enumerate(raw):
    if value == 0:

    decoded += chr(value - fibs[i % 10])

print(decoded)
```

Running this reconstruction produces the hidden message.


The recovered flag is:

```text
Holberton{native_hooking_is_no_different_at_all}
```


| Tool      | Purpose                                                   |
| --------- | --------------------------------------------------------- |
| JADX      | Decompiling the APK and inspecting Java code              |
| Apktool   | Extracting the APK contents and native libraries          |
| `nm`      | Finding exported JNI symbols                              |
| `readelf` | Examining the `.rodata` section and raw bytes             |
| `objdump` | Disassembling and studying the native function            |
| Python 3  | Recreating the decoding algorithm and confirming the flag |

## Conclusion

The flag was recovered without launching the Android application. By first tracing the Java native declaration to its JNI implementation, then analyzing the native function and the encoded data in `.rodata`, the decryption mechanism could be reconstructed. The final Python implementation confirmed the extracted flag.
## 7. Tools and Their Roles
## 6. Result
        break

    107, 106, 111, 105, 98, 110, 123, 108, 131, 145,

        a, b = b, a + b
        return n

    if n <= 1:


## 5. Reproducing the Decryption in Python
6b 6a 6f 69 62 6e 7b 6c 83 91
5f 62 75 61 64 71 74 8a 00
5f 65 6a 68 69 6a 7a 72 83 96
At offset `0x5f0`, the relevant byte sequence was found:

```

```bash
The contents of the `.rodata` section were inspected using:

## 4. Recovering the Encoded Bytes


```text
```text
```

