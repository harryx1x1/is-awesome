# How bit commitment works
## Bit commitment and signature
Often signature is created with this pseudo code:
```
mhash = hash(message)
private_key = some_random_number
public_key = create_public_key(private_key) // such as hash
signature = create_signature(mhash, private_key)
assert true == verify_signature(message, signature, public_key)
```
Means this signature commit to this message.

If message == 0 or 1, we would say this signature is a bit(0/1) commitment.

## Bit commitment with Lamport signature

Lamport signature is a signature scheme that can be used to do bit commitment. Let's see how it works.

### Private key and Public key
Private key and public key in Lamport signature consist of a pair of array.

Private key elements($s_i^0$, $s_i^1$) and public key elements(hash($s_i^0$), hash($s_i^1$)) are integer.

|               | 0th element   | 1st element   | 2nd element   | 3rd element   | ... |
| ------------- | ------------- | ------------- | ------------- | ------------- | --- |
| private key 0 | $s_0^0$       | $s_1^0$       | $s_2^0$       | $s_3^0$       | ... |
| private key 1 | $s_0^1$       | $s_1^1$       | $s_2^1$       | $s_3^1$       | ... |
| public key 0  | hash($s_0^0$) | hash($s_1^0$) | hash($s_2^0$) | hash($s_3^0$) | ... |
| public key 1  | hash($s_0^1$) | hash($s_1^1$) | hash($s_2^1$) | hash($s_3^1$) | ... |

For example:
```
private key 0: ['0001', '1001', '0001', '1100', ...] // length == 160

private key 1: ['1100', '1111', '1100', '1011', ...]

public key 0: [hash('0001'), hash('1001'), hash('0001'), hash('1100'), ...]

public key 1: [hash('1100'), hash('1111'), hash('1100'), hash('1011'), ...]
```

Let's make them as a table:

|               | 0th element | 1st element | 2nd element | 3rd element | ... |
| ------------- | ----------- | ----------- | ----------- | ----------- | --- |
| private key 0 | 0001        | 1001        | 0001        | 1100        | ... |
| private key 1 | 1100        | 1111        | 1100        | 1011        | ... |
| public key 0  | hash(0001)  | hash(1001)  | hash(0001)  | hash(1100)  | ... |
| public key 1  | hash(1100)  | hash(1111)  | hash(1100)  | hash(1011)  | ... |

### Signature

The logic to create the signature is as follows:
```
mhash = hash(message) = $[m_0, m_1, m_2, m3...]$
sig = []
sig[0] = $m_0$ == 0 ? $s_0^0$ : $s_0^1$
sig[1] = $m_1$ == 0 ? $s_1^0$ : $s_1^1$
sig[2] = $m_2$ == 0 ? $s_2^0$ : $s_2^1$
sig[3] = $m_3$ == 0 ? $s_3^0$ : $s_3^1$
...
```

For example, a message and its hash:
```
message: Hello world

mhash = 1111010111101001010101100110100011011010110111110110111111011110111110000101001000011111011111100001101010101000101001011110011001010000110010011111100001001001 // hash("Hello world")
```

From lowest bit to highest bit:
```
mhash[0] = 1
mhash[1] = 0
mhash[2] = 0
mhash[3] = 1
.
.
.
mhash[158] = 1
mhash[159] = 1
```

The signature for the first 4 bits are:
```
signature: ['1100', '1001', '0001', '1011', ...]
```

Bit commitment that done in bitcoin is accomplished with Lamport signature.

### Verify
Hash each element in signature and compare them with public key.
## Bit commitment in Bitcoin

Let's commit the first/lowest bit of mhash, which is `0`(`m[0]`)

Use Lamport signature to create bit commitment for the 1st bit, we need to put the corresponding public keys for 1st bit onto the stack.

```
OP_HASH160
OP_DUP

hash(1100) // hash1(the 0th element from public key 1)
OP_EQUAL
OP_DUP

OP_ROT
hash(0001) // hash0(the 0th element from public key 0)
OP_EQUAL

OP_BOOLOR
OP_VERIFY
```

See how this script works.

The initial state of stack and script is:

| Stack | Script                                                                                                                 |
| ----- | ---------------------------------------------------------------------------------------------------------------------- |
|       | OP_HASH160<br>OP_DUP<br>hash(1100)<br>OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

### If 1100 is pushed to the stack

| Stack | Script                                                                                                                 |
| ----- | ---------------------------------------------------------------------------------------------------------------------- |
| 1100  | OP_HASH160<br>OP_DUP<br>hash(1100)<br>OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

Execution: `1100 OP_HASH160`

After execution:

| Stack      | Script                                                                                                   |
| ---------- | -------------------------------------------------------------------------------------------------------- |
| hash(1100) | OP_DUP<br>hash(1100)<br>OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

Execution: `hash(1100) OP_DUP`

After execution:

| Stack                    | Script                                                                                         |
| ------------------------ | ---------------------------------------------------------------------------------------------- |
| hash(1100)<br>hash(1100) | hash(1100)<br>OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

Execution: `hash(1100) hash(1100) hash(1100)`

After execution:

| Stack                                  | Script                                                                           |
| -------------------------------------- | -------------------------------------------------------------------------------- |
| hash(1100)<br>hash(1100)<br>hash(1100) | OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

Execution: `hash(1100) hash(1100) OP_EQUAL`

After execution:

| Stack              | Script                                                               |
| ------------------ | -------------------------------------------------------------------- |
| True<br>hash(1100) | OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

Execution: `True OP_DUP`

After execution:

| Stack                      | Script                                                     |
| -------------------------- | ---------------------------------------------------------- |
| True<br>True<br>hash(1100) | OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

Execution: `True True hash(1100) OP_ROT`

After execution:

| Stack                      | Script                                           |
| -------------------------- | ------------------------------------------------ |
| hash(1100)<br>True<br>True | hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

Execution: `hash(1100) True True hash(0001)`

After execution:

| Stack                                    | Script                             |
| ---------------------------------------- | ---------------------------------- |
| hash(0001)<br>hash(1100)<br>True<br>True | OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

Execution: `hash(0001) hash(1100) OP_EQUAL`

After execution:

| Stack                 | Script                 |
| --------------------- | ---------------------- |
| False<br>True<br>True | OP_BOOLOR<br>OP_VERIFY |

Execution: `False True OP_BOOLOR`

After execution:

| Stack        | Script    |
| ------------ | --------- |
| True<br>True | OP_VERIFY |

Execution: `True OP_VERIFY`

After execution:

| Stack | Script |
| ----- | ------ |
| True  |        |

`True` is left in the stack, since the value of False is `1`, so we can say that `1100` is the bit commitment of `1`.
### If 0001 is pushed to the stack

Since the execution process is similar, we will just put the post-execution state below.

| Stack | Script                                                                                                                 |
| ----- | ---------------------------------------------------------------------------------------------------------------------- |
| 0001  | OP_HASH160<br>OP_DUP<br>hash(1100)<br>OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

| Stack      | Script                                                                                                   |
| ---------- | -------------------------------------------------------------------------------------------------------- |
| hash(0001) | OP_DUP<br>hash(1100)<br>OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

| Stack                    | Script                                                                                         |
| ------------------------ | ---------------------------------------------------------------------------------------------- |
| hash(0001)<br>hash(0001) | hash(1100)<br>OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

| Stack                                  | Script                                                                           |
| -------------------------------------- | -------------------------------------------------------------------------------- |
| hash(1100)<br>hash(0001)<br>hash(0001) | OP_EQUAL<br>OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

| Stack                 | Script                                                               |
| --------------------- | -------------------------------------------------------------------- |
| False/0<br>hash(0001) | OP_DUP<br>OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

| Stack                        | Script                                                     |
| ---------------------------- | ---------------------------------------------------------- |
| False<br>False<br>hash(0001) | OP_ROT<br>hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

| Stack                        | Script                                           |
| ---------------------------- | ------------------------------------------------ |
| hash(0001)<br>False<br>False | hash(0001)<br>OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

| Stack                                      | Script                             |
| ------------------------------------------ | ---------------------------------- |
| hash(0001)<br>hash(0001)<br>False<br>False | OP_EQUAL<br>OP_BOOLOR<br>OP_VERIFY |

| Stack                  | Script                 |
| ---------------------- | ---------------------- |
| True<br>False<br>False | OP_BOOLOR<br>OP_VERIFY |

| Stack         | Script    |
| ------------- | --------- |
| True<br>False | OP_VERIFY |

| Stack | Script |
| ----- | ------ |
| False |        |

`False` is left in the stack, since the value of False is `0`, so we can say that `0001` is the bit commitment of `0`.