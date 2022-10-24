# Pyc

import pprint
import binascii
import mnemonic
import bip32utils
import requests
import random
import os
from decimal import Decimal
from multiprocessing.pool import ThreadPool as Pool
import threading
import hashlib
import secrets
import binascii
from time import sleep
class Settings():
    disable_debug = "n"
        save_empty = "y"
def userInput():
    print("""Type "start" to begin or "help" for the help prompt.
        """)
            while True:
                    user_input = input("> ")
                            if user_input == "start":
                                        start()
                                                    break
                                                            elif user_input == "help":
                                                                        helpText()
                                                                                else:
                                                                                            print("type 'help' to get help")
class Bip39Gen(object):
    def __init__(self, bip39wordlist):
            self.bip39wordlist = bip39wordlist
                    word_count = 12
                            checksum_bit_count = word_count // 3
                                    total_bit_count = word_count * 11
                                            generated_bit_count = total_bit_count - checksum_bit_count
                                                    entropy = self.generate_entropy(generated_bit_count)
                                                            entropy_hash = self.get_hash(entropy)
                                                                    indices = self.pick_words(entropy, entropy_hash, checksum_bit_count)
                                                                            self.print_words(indices)
    def generate_entropy(self, generated_bit_count):
            entropy = secrets.randbits(generated_bit_count)
                    return self.int_to_padded_binary(entropy, generated_bit_count)
    def get_hash(self, entropy):
            generated_bit_count = len(entropy)
                    generated_char_count = generated_bit_count // 4
                            entropy_hex = self.binary_to_padded_hex(entropy, generated_char_count)
        entropy_hex_no_padding = entropy_hex[2:]
        entropy_bytearray = bytearray.fromhex(entropy_hex_no_padding)
        return hashlib.sha256(entropy_bytearray).hexdigest()
    def pick_words(self, entropy, entropy_hash, checksum_bit_count):
            checksum_char_count = checksum_bit_count // 4
                    bit = entropy_hash[0:checksum_char_count]
        check_bit = int(bit, 16)
                checksum = self.int_to_padded_binary(check_bit, checksum_bit_count)
        source = str(entropy) + str(checksum)
                return [int(str('0b') + source[i:i + 11], 2) for i in range(0, len(source), 11)]
    def print_words(self, indices):
            words = [self.bip39wordlist[indices[i]] for i in range(len(indices))]
                    word_string = ' '.join(words)
                            self.mnemonic = word_string
    def int_to_padded_binary(self, num, padding):
            return bin(num)[2:].zfill(padding)
    def binary_to_padded_hex(self, bin, padding):
            num = int(bin, 2)
                    return '0x{0:0{1}x}'.format(num, padding)
def getInternet():
    try:
            try:
                        requests.get('http://216.58.192.142')
                                except requests.ConnectTimeout:
                                            requests.get('http://1.1.1.1')
                                                    return True
                                                        except requests.ConnectionError:
                                                                return False
lock = threading.Lock()
if getInternet() == True:
    dictionary = requests.get('https://raw.githubusercontent.com/bitcoin/bips/master/bip-0039/english.txt').text.strip().split('\n')
    else:
        pass
def getBalance(addr):
    try:
            response = requests.get(f'https://api.smartbit.com.au/v1/blockchain/address/{addr}')
                    return (
                                Decimal(response.json()["address"]["total"]["balance"])
                                            / 100000000
                                                    )
                                                        except:
                                                                pass
def generateSeed():
    seed = ""
        for i in range(12):
                seed += random.choice(dictionary) if i == 0 else ' ' + random.choice(dictionary)
                    return seed
def bip39(mnemonic_words):
    mobj = mnemonic.Mnemonic("english")
        seed = mobj.to_seed(mnemonic_words)
    bip32_root_key_obj = bip32utils.BIP32Key.fromEntropy(seed)
        bip32_child_key_obj = bip32_root_key_obj.ChildKey(
                44 + bip32utils.BIP32_HARDEN
                    ).ChildKey(
                            0 + bip32utils.BIP32_HARDEN
                                ).ChildKey(
                                        0 + bip32utils.BIP32_HARDEN
                                            ).ChildKey(0).ChildKey(0)
    return bip32_child_key_obj.Address()
def check():
    while True:
            mnemonic_words = Bip39Gen(dictionary).mnemonic
                    addy = bip39(mnemonic_words)
                            balance = getBalance(addy)
                                    with lock:
                                                if Settings.disable_debug == "y":
                                                                pass
                                                                            else:
                                                                                            print(f'Address: {addy} | Balance: {balance} | Mnemonic phrase: {mnemonic_words}')
                                                                                                    if balance > 0:
                                                                                                                with open('results/wet.txt', 'a') as w:
                                                                                                                                w.write(f'Address: {addy} | Balance: {balance} | Mnemonic phrase: {mnemonic_words}\n')
                                                                                                                                        else:
                                                                                                                                                    if Settings.save_empty == "n":
                                                                                                                                                                    pass
                                                                                                                                                                                else:
                                                                                                                                                                                                with open('results/dry.txt', 'a') as w:
                                                                                                                                                                                                                    w.write(f'Address: {addy} | Balance: {balance} | Mnemonic phrase: {mnemonic_words}\n')

def start():
    try:
            threads = int(input("Number of threads (1 - 666): "))
                    if threads > 666:
                                print("You can only run 666 threads at once")
                                            start()
                                                except ValueError:
                                                        print("Enter an interger!")
                                                                start()
                                                                    Settings.disable_debug = input("Disable debug? (n/y): ")
                                                                        Settings.save_empty = input("Save empty? (y/n): ")
                                                                            if getInternet() == True:
                                                                                    pool = Pool(threads)
                                                                                            for _ in range(threads):
                                                                                                        pool.apply_async(check, ())
                                                                                                                pool.close()
                                                                                                                        pool.join()
                                                                                                                            else:
                                                                                                                                    print("Told ya")
                                                                                                                                            userInput()
if __name__ == '__main__':
    welcomeBanner()
        getInternet()
            if getInternet() == False:
                    print("You have no internet access the generator won't work.")
                        else:
                                pass
                                    userInput()
