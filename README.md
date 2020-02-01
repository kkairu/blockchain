# Huduma Namba Blockchain Using Python

Blockchain prevents backdating and data tampering.

Demonstrate how to build a private blockchain for storing citizen bioData with essential operations such as:

- creating a blockchain
- verifying a chain
- forking
- comparing chains.

NOTE: no algorithms on any distributed network or Proof of Work.


```python
import copy # fork a chain
import datetime # get real time for timestamps
import hashlib # hash
```


```python
# Define Class - HudumaBlock
# -----------------------------------------------------------------------------------------------------------------------------
# - The Hash key is hard to fake or brute force, but is easy to verify
# - Computationally easy to get H(x) from input x, but intractable to reverse the process
# - Use SHA-256 algorithm to hash block

class CitizenBlock():
    def __init__(self, index, timestamp, data, previous_hash):
        self.index = index
        self.timestamp = timestamp
        self.data = data
        self.previous_hash = previous_hash
        self.hash = self.hashing()
    
    def __eq__(self, other):
        if isinstance(other, self.__class__):
            return self.__dict__ == other.__dict__
        else:
            return False
    
    def hashing(self):
        key = hashlib.sha256()
        key.update(str(self.index).encode('utf-8'))
        key.update(str(self.timestamp).encode('utf-8'))
        key.update(str(self.data).encode('utf-8'))
        key.update(str(self.previous_hash).encode('utf-8'))
        return key.hexdigest()
    
    def verify(self): # check data types of all info in a block
        instances = [self.index, self.timestamp, self.previous_hash, self.hash]
        types = [int, datetime.datetime, str, str]
        if sum(map(lambda inst_, type_: isinstance(inst_, type_), instances, types)) == len(instances):
            return True
        else:
            return False
```


```python
# Define Class - HudumaChain
# -----------------------------------------------------------------------------------------------------------------------------
# - chain of blocks, and the connection is made by storing the hash of the previous block
# - initialize chain, automatically assign a 0th block (Genesis block) 

# Verify
# -----------------------------------------------------------------
# - Index in blocks[i] is i, and hence no missing or extra blocks.
# - Compute block hash H(blocks[i]) and cross-check with the recorded hash. 
#    Even if a single bit in a block is altered, the computed block hash would be entirely different.
# - Verify if H(blocks[i]) is correctly stored in next block’s previous_hash.
# - Check if there is any backdating by looking into the timestamps.

class HudumaChain():
    def __init__(self): # initialize when creating a chain
        self.blocks = [self.get_genesis_block()]
    
    def __eq__(self, other):
        if isinstance(other, self.__class__):
            return self.__dict__ == other.__dict__
        else:
            return False
    
    def get_genesis_block(self): 
        return CitizenBlock(0, 
                            datetime.datetime.utcnow(), 
                            'Genesis', 
                            'arbitrary')
    
    def add_block(self, data): #Add Block to chain
        self.blocks.append(CitizenBlock(len(self.blocks), 
                                        datetime.datetime.utcnow(), 
                                        data,
                                        self.blocks[len(self.blocks)-1].hash))
    
    def get_chain_size(self): # exclude genesis block
        return len(self.blocks)-1
    
    def verify(self, verbose=True): 
        flag = True
        for i in range(1,len(self.blocks)):
            if not self.blocks[i].verify(): # assume Genesis block integrity
                flag = False
                if verbose:
                    print(f'Wrong data type(s) at block {i}.')
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
            if self.blocks[i-1].timestamp > self.blocks[i].timestamp: # >= compare previous and current block create time
                flag = False
                if verbose:
                    #print(self.blocks[i-1].timestamp)
                    #print(self.blocks[i].timestamp)
                    print(f'Backdating at block {i}.')
        return flag
    
    # In case you might want to branch out of a chain. 
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


```python
# Start a Citizen Chain and test
# -----------------------------------------------------------------------------------------------------------------------------

KE_CITIZEN_CHAIN = HudumaChain() # Start a chain

for i in range(1,20+1):
    #c.add_block(f'This is block {i} of my first chain.','1983-05-02','Kennedy','Kariuki')
    bioData = {'NIIMS': 'KE-2020-'+str(i),'DOB': '1960-12-01','FIRST NAME': 'Kennedy_'+str(i),'SURNAME': 'Kariuki'}
    KE_CITIZEN_CHAIN.add_block(bioData)

print('[INFO]: CITIZEN BlockChain Started...')
```

    [INFO]: CITIZEN BlockChain Started...
    


```python
#v_niims = c.blocks[3].data['NIIMS']

i = 5

print(f'   DATE/TIME  = {KE_CITIZEN_CHAIN.blocks[i].timestamp}')
print(f'CITIZEN DATA  = {KE_CITIZEN_CHAIN.blocks[i].data}')
print(f'  BLOCK HASH  = {KE_CITIZEN_CHAIN.blocks[i].hash}')
```

       DATE/TIME  = 2020-02-01 13:51:07.324425
    CITIZEN DATA  = {'NIIMS': 'KE-2020-5', 'DOB': '1960-12-01', 'FIRST NAME': 'Kennedy_5', 'SURNAME': 'Kariuki'}
      BLOCK HASH  = e3492c187634765d11e745149acf808dec3971658977334b768898709c934481
    


```python
# Get Chain Size and Verify
# -----------------------------------------------------------------------------------------------------------------------------

print(f'CITIZEN CHAIN SIZE = {KE_CITIZEN_CHAIN.get_chain_size()}')
print(f'   CHAIN VERIFIED? = {KE_CITIZEN_CHAIN.verify(verbose=False)}')
```

    CITIZEN CHAIN SIZE = 20
       CHAIN VERIFIED? = True
    


```python
# Fork Chain and check
# -----------------------------------------------------------------------------------------------------------------------------

c_forked = KE_CITIZEN_CHAIN.fork(head='latest')
print(f'FORKED SUCCESS(T) / FAIL(F) = {KE_CITIZEN_CHAIN == c_forked}')
```

    FORKED SUCCESS(T) / FAIL(F) = True
    


```python
# Add to forked Chain
# -----------------------------------------------------------------------------------------------------------------------------

c_forked.add_block('New Number for forked chain!')
print(f'KE_CITIZEN_CHAIN = {KE_CITIZEN_CHAIN.get_chain_size()} \n        c_forked = {c_forked.get_chain_size()}')
```

    KE_CITIZEN_CHAIN = 20 
            c_forked = 21
    

## Conflict Testing


```python
c_forked = KE_CITIZEN_CHAIN.fork('latest')
c_forked.blocks[9].index = -9
c_forked.verify()
```

    Wrong block index at block 9.
    Wrong hash at block 9.
    




    False




```python

c_forked = KE_CITIZEN_CHAIN.fork('latest')
c_forked.blocks[16].timestamp = datetime.datetime(2000, 1, 1, 0, 0, 0, 0)
c_forked.verify()
```

    Wrong hash at block 16.
    Backdating at block 16.
    




    False




```python

c_forked = KE_CITIZEN_CHAIN.fork('latest')
c_forked.blocks[5].previous_hash = c_forked.blocks[1].hash
c_forked.verify()
```

    Wrong previous hash at block 5.
    Wrong hash at block 5.
    




    False




```python
# Only the same data would create the same hash.

c_forked = KE_CITIZEN_CHAIN.fork('latest')
c_forked.blocks[5].data = {'NIIMS': 'KE-2020-51', 'DOB': '1960-12-01', 'FIRST NAME': 'Kennedy_XXXX', 'SURNAME': 'Kariuki'}
c_forked.verify()
```

    Wrong hash at block 5.
    




    False




```python

```
