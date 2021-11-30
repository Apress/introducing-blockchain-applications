import datetime
import hashlib
from pprint import pprint

class Transaction(object):

	"""Transaction class
	"""

	def __init__(self, fromAdress, toAdress, amount):
		self.fromAdress = fromAdress
		self.toAdress = toAdress
		self.amount = amount


class Block(object):
	
	"""Block class
	"""
	
	def __init__(self, timestamp, transactions, previousHash=""):
		self.timestamp = timestamp
		self.transactions = transactions
		self.previousHash = previousHash
		self.nonce = 0
		self.hash = self.calculateHash()		

	def calculateHash(self):
		info = str(self.timestamp) + str(self.transactions) + str(self.previousHash) + str(self.nonce)
		return hashlib.sha256(info.encode('utf-8')).hexdigest()

	# Proof of work algorithm
	def mineBlock(self, difficulty):
		self.hash = self.calculateHash()
		while(self.hash[:difficulty] != "0"*difficulty):
			self.nonce += 1
			self.hash = self.calculateHash()

class BlockChain(object):

	"""Blockchain class
	"""

	def __init__(self):
		self.chain = [self.createGenesisBlock()]
		self.difficulty = 4
		self.pendingTransactions = []
		self.miningReward = 100

	def createGenesisBlock(self):
		return Block("20/03/2018", [], "0")

	def getLatestBlock(self):
		return self.chain[-1]

	def minePendingTransactions(self, miningRewardAdress):
		
		newBlock = Block(datetime.datetime.now(), self.pendingTransactions)
		newBlock.previousHash = self.getLatestBlock().hash
		# you can check if transactions are valid here
		print("mining block...")
		newBlock.mineBlock(self.difficulty)
		print("block mined:", newBlock.hash)
		print("block succesfully mined.")
		self.chain.append(newBlock)

		self.pendingTransactions = [Transaction(None, miningRewardAdress, self.miningReward)]

	def createTransaction(self, transaction):
		self.pendingTransactions.append(transaction)

	def getBalanceOfAdress(self, adress):
		balance = 0
		for block in self.chain:
			for transaction in block.transactions:
				if transaction.fromAdress == adress:
					balance -= transaction.amount
				if transaction.toAdress == adress:
					balance += transaction.amount
		return balance

	def isBlockChainValid(self):
		for previousBlock, block in zip(self.chain, self.chain[1:]):
			if block.hash != block.calculateHash():
				return False
			if block.previousHash != previousBlock.hash:
				return False
		return True

	def showBlockChain(self):
		print("blockchain: fedecoin\n")
		for block in self.chain:
			print("block")
			print("timestamp:", block.timestamp)
			pprint("transactions:", block.transactions)
			print("previousHash:", block.previousHash)
			print("hash:", block.hash, "\n")

	def showAdressBalance(self, adress):
		print(adress, "balance:", self.getBalanceOfAdress(adress))


fedecoin = BlockChain()
fedecoin.createTransaction(Transaction("adress1", "adress2", 100))
fedecoin.createTransaction(Transaction("adress2", "adress1", 50))
fedecoin.minePendingTransactions("fede_adress")
fedecoin.createTransaction(Transaction("adress1", "adress3", 100))
fedecoin.createTransaction(Transaction("adress2", "adress1", 50))
fedecoin.minePendingTransactions("fede_adress")
fedecoin.showAdressBalance("fede_adress")
print("fedecoin is valid?", fedecoin.isBlockChainValid())
fedecoin.chain[1].transactions = Transaction("adress1", "adress2", 1000)
print("fedecoin is valid?", fedecoin.isBlockChainValid())
fedecoin.chain[1].calculateHash()
print("fedecoin is valid?", fedecoin.isBlockChainValid())