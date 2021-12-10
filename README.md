# What is blockchain?
Blockchain is a system of recording information in a way that makes it difficult or impossible to change, hack, or cheat the system.

A blockchain is essentially a digital ledger of transactions that is duplicated and distributed across the entire network of computer systems on the blockchain. Each block in the chain contains a number of transactions, and every time a new transaction occurs on the blockchain, a record of that transaction is added to every participant’s ledger. The decentralised database managed by multiple participants is known as Distributed Ledger Technology (DLT).

Blockchain is a type of DLT in which transactions are recorded with an immutable cryptographic signature called a  **Hash**




# Hashing

We  want a ‘key’ that can represent a block of data. We want a key that is  **hard to fake or brute force, but is easy to verify**. This is where hashing comes in. Hashing is a function H(x) that satisfies the following properties:

-   The same input  `x`  always produce the same output  `H(x)`.
-   Different (or even similar) inputs  `x`  should produce  **_entirely_**  different outputs  `H(x)`.
-   Computationally easy to get  `H(x)`from input  `x`, but intractable to reverse the process, i.e. getting the input  `x`  from a known hash  `H`.

This is how Google stores your ‘password’ without actually storing your password. They store the hash of your password  `H(password)`  such that they can verify your password by hashing your input and compare. Without going into too much details, we will use SHA-256 algorithm to hash our block.

# A Minimal Block

Let’s make an object class called  `MinimalBlock()`. It is initialized by providing an  `index`, a  `timestamp`, some  `data`  you want to store, and something called  `previous_hash`. The previous hash is the hash (key) of the previous block, and it acts as a pointer such that we know which block is the previous block, and hence how blocks are connected.

``` python 
import hashlib

class MinimalBlock():

def __init__(self, index, timestamp, data, previous_hash):

self.index = index

self.timestamp = timestamp

self.data = data

self.previous_hash = previous_hash

self.hash = self.hashing()

def hashing(self):

key = hashlib.sha256()

key.update(str(self.index).encode('utf-8'))

key.update(str(self.timestamp).encode('utf-8'))

key.update(str(self.data).encode('utf-8'))

key.update(str(self.previous_hash).encode('utf-8'))

return key.hexdigest()
```

In other words, `Block[x]` contains index `x`, a timestamp, some data, and the hash of the previous block x-1 `H(Block[x-1])`. Now that this block is complete, it can be hashed to generate `H(Block[x])` as a pointer in the next block.
# A Minimal Chain

Blockchain is essentially a chain of blocks, and the connection is made by storing the hash of the previous block. Therefore, a chain can be implemented using a Python list, and  `blocks[i]`  representing the {i}th block.

```python 
class MinimalChain():

def __init__(self): # initialize when creating a chain

self.blocks = [self.get_genesis_block()]

def get_genesis_block(self):

return MinimalBlock(0,

datetime.datetime.utcnow(),

'Genesis',

'arbitrary')

def add_block(self, data):

self.blocks.append(MinimalBlock(len(self.blocks),

datetime.datetime.utcnow(),

data,

self.blocks[len(self.blocks)-1].hash))

def get_chain_size(self): # exclude genesis block

return len(self.blocks)-1

def verify(self, verbose=True):

flag = True

for i in range(1,len(self.blocks)):

if self.blocks[i].index != i:

flag = False

if verbose:

print(f'Wrong block index at block {i}.')

if self.blocks[i-1].hash != self.blocks[i].previous_hash:

flag = False

if verbose:

print(f'Wrong previous hash at block {i}.')

if self.blocks[i].hash != self.blocks[i].hashing():

flag = False

if verbose:

print(f'Wrong hash at block {i}.')

if self.blocks[i-1].timestamp >= self.blocks[i].timestamp:

flag = False

if verbose:

print(f'Backdating at block {i}.')

return flag

def fork(self, head='latest'):

if head in ['latest', 'whole', 'all']:

return copy.deepcopy(self) # deepcopy since they are mutable

else:

c = copy.deepcopy(self)

c.blocks = c.blocks[0:head+1]

return c

def get_root(self, chain_2):

min_chain_size = min(self.get_chain_size(), chain_2.get_chain_size())

for i in range(1,min_chain_size+1):

if self.blocks[i] != chain_2.blocks[i]:

return self.fork(i-1)

return self.fork(min_chain_size)

```
When we initialize a chain, we automatically assign a 0th block (also known as Genesis block) to the chain with function `get_genesis_block()`. This block marks the start of your chain. Note that `previous_hash` is arbitrary in the Genesis block. Adding a block can be achieved by calling `add_block()`.

# Data Verification

Data integrity is important to databases, and blockchains provide an easy way to verify all the data. In function  `verify()`, we check the following:

-   Index in  `blocks[i]`  is  `i`, and hence no missing or extra blocks.
-   Compute block hash  `H(blocks[i])`  and cross-check with the recorded hash. Even if a single bit in a block is altered, the computed block hash would be entirely different.
-   Verify if  `H(blocks[i])`  is correctly stored in next block’s  `previous_hash`.
-   Check if there is any backdating by looking into the timestamps.

# Forking

In some case you might want to branch out of a chain. This is called forking as demonstrated as  `fork()`in the code. You would copy a chain (or root of a chain) and then go separate ways. It is vital to use  `deepcopy()`  in Python since Python list is mutable.

# The Takeaway

Hashing a block creates a unique identifier of a block, and a block’s hash forms part of the next block to establish a link between blocks. Only the same data would create the same hash.

If you want to amend data in the 3rd block, the hash of the 3rd block is changed and the  `previous_hash`  in the 4th block needs to change as well.  `previous_hash`is part of the 4th block, and hence its hash changes as well, and so on.
