# Report — Task 3: Revealing Hidden Android Functions

## Objective

The objective of this task was to recover a concealed flag from the Android application `task3_d.apk`. Unlike a normal application flow, the function responsible for generating the flag is never executed when the application is used normally.

Therefore, the investigation focused on static analysis of the APK to locate the unused function and understand its embedded decoding routine.

## 1. APK Decompilation and Code Search

The application was first decompiled using JADX:

```bash
jadx ~/Desktop/task3_d.apk -d ~/Desktop/task3_jadx
```

Afterwards, the generated Java sources were searched for keywords associated with hidden functionality and cryptographic operations:

```bash
grep -r "hidden|secret|flag|decrypt|encode|Holberton" \
~/Desktop/task3_jadx/sources/com --include="*.java" -l
```

The search led to the following file:

```text
com/holberton/task4_d/MainActivityKt.java
```

This file contained the function responsible for processing the hidden message.

## 2. Finding the Unused Function

Inside `MainActivityKt.java`, a private static method with an unusual name was identified:

```java
private static final void aBcDeFgHiJkLmNoPqRsTuVwXyZ123456(
    Function1<? super String, Unit> function1
)
```

The method name appears deliberately meaningless, which makes it less obvious during a quick review.

More importantly, the function is not referenced by the application's ordinary execution path.

The visible interface even provides a hint about this behavior:

```text
Hmm it seems the interesting function is never called.
```

This confirmed that simply interacting with the application's interface would not trigger the flag-generation code.

## 3. Reverse Engineering the Decryption Routine

Further inspection of the hidden method revealed that it contains the complete decoding algorithm. No remote API or external key is necessary.

### 3.1 Base64 Decoding

The function first processes the following Base64-encoded data:

```text
8CP4zSyn62t78lwwc383rxcgtv/UiMv3Pw+Mfw12LzXvorIpBypNK/oB7XvWNV0oWfoX
```

Once Base64 decoding is performed, the resulting bytes are passed through several transformations.

### 3.2 Byte-Level Transformation

For each byte at position `index`, the application applies a sequence of arithmetic and bitwise operations.

The first step is XOR:

```text
temp = value ^ 19
```

The resulting value is then rotated using a combination of bit shifts:

```text
temp2 = (((temp >> 2) | (temp << 6)) & 255) - (index * 3)
```

The result is normalized into the range of one byte:

```text
temp2 = temp2 % 256

if temp2 < 0:
    temp2 += 256
```

Finally, a multiplication and modulo operation generates the character value:

```text
char = (temp2 * 183) % 256
```

Every generated character is appended to the resulting string.

## 4. Recreating the Algorithm in Python

The complete process was reproduced in Python without launching the Android application:

```python
import base64
# Report — Task 3: Revealing Hidden Android Functions

## Objective

The objective of this task was to recover a concealed flag from the Android application `task3_d.apk`. Unlike a normal application flow, the function responsible for generating the flag is never executed when the application is used normally.

Therefore, the investigation focused on static analysis of the APK to locate the unused function and understand its embedded decoding routine.

## 1. APK Decompilation and Code Search

The application was first decompiled using JADX:

```bash
jadx ~/Desktop/task3_d.apk -d ~/Desktop/task3_jadx
```

Afterwards, the generated Java sources were searched for keywords associated with hidden functionality and cryptographic operations:

```bash
grep -r "hidden|secret|flag|decrypt|encode|Holberton" \
~/Desktop/task3_jadx/sources/com --include="*.java" -l
```

The search led to the following file:

```text
com/holberton/task4_d/MainActivityKt.java
```

This file contained the function responsible for processing the hidden message.

## 2. Finding the Unused Function

Inside `MainActivityKt.java`, a private static method with an unusual name was identified:

```java
private static final void aBcDeFgHiJkLmNoPqRsTuVwXyZ123456(
    Function1<? super String, Unit> function1
)
```

The method name appears deliberately meaningless, which makes it less obvious during a quick review.

More importantly, the function is not referenced by the application's ordinary execution path.

The visible interface even provides a hint about this behavior:

```text
Hmm it seems the interesting function is never called.
```

This confirmed that simply interacting with the application's interface would not trigger the flag-generation code.

## 3. Reverse Engineering the Decryption Routine

Further inspection of the hidden method revealed that it contains the complete decoding algorithm. No remote API or external key is necessary.

### 3.1 Base64 Decoding

The function first processes the following Base64-encoded data:

```text
8CP4zSyn62t78lwwc383rxcgtv/UiMv3Pw+Mfw12LzXvorIpBypNK/oB7XvWNV0oWfoX
```

Once Base64 decoding is performed, the resulting bytes are passed through several transformations.

### 3.2 Byte-Level Transformation

For each byte at position `index`, the application applies a sequence of arithmetic and bitwise operations.

The first step is XOR:

```text
temp = value ^ 19
```

The resulting value is then rotated using a combination of bit shifts:

```text
temp2 = (((temp >> 2) | (temp << 6)) & 255) - (index * 3)
```


```text
temp2 = temp2 % 256

if temp2 < 0:

    temp2 += 256
```


Finally, a multiplication and modulo operation generates the character value:

```text
char = (temp2 * 183) % 256
```


Every generated character is appended to the resulting string.


## 4. Recreating the Algorithm in Python

The complete process was reproduced in Python without launching the Android application:

```python
import base64

data = base64.b64decode(
    "8CP4zSyn62t78lwwc383rxcgtv/UiMv3Pw+Mfw12LzXvorIpBypNK/oB7XvWNV0oWfoX"
)


decoded_bytes = [b & 0xFF for b in data]

flag_chars = []

for index, value in enumerate(decoded_bytes):
    temp = value ^ 19

    temp2 = (((temp >> 2) | (temp << 6)) & 255) - (index * 3)

    temp2 = temp2 % 256

    if temp2 < 0:

        temp2 += 256

    flag_chars.append(chr((temp2 * 183) % 256))

print("".join(flag_chars))
```

The output of this reconstruction revealed the hidden flag.

## 5. Extracted Flag

The recovered value was:

```text
Holberton{calling_uncalled_functions_is_now_known!}

```

## 6. Security Observations

The hidden implementation relies on obscurity rather than strong protection.

Although the function is declared as `private` and is not exposed through the application's interface, its code remains present in the compiled application and can be recovered through decompilation.

The unusual method name:

```text
aBcDeFgHiJkLmNoPqRsTuVwXyZ123456
```

does not provide meaningful security. It only makes the method slightly harder to recognize manually.

Another weakness is that the entire transformation process and the encoded input are stored locally in the application. Since no external secret is required, anyone who can inspect the binary can reconstruct the same algorithm and recover the flag.

## 7. Tools Used

| Tool     | Purpose                                                      |
| -------- | ------------------------------------------------------------ |
| JADX     | Decompiling the APK and examining the generated Java code    |
| `grep`   | Locating potentially interesting methods and keywords        |
| Python 3 | Reproducing the byte transformations and recovering the flag |

## Conclusion

The flag was recovered through static analysis without running the application. The key discovery was an unused private method containing the complete decoding logic. By locating this function, understanding each byte transformation, and implementing the same operations in Python, the hidden flag was successfully reconstructed.

This task demonstrates that unused or private functions should not be considered secure simply because they are not accessible through the normal application interface.
