# Enigma
Python Enigma Code
Block-by-block explanation of the Enigma machine code:

Rotor Class:
python
class Rotor:
    def __init__(self, wiring, notch, ring_setting=0):
        self.wiring = [ord(char) - ord('A') for char in wiring]
        self.position = 0
        self.notch = ord(notch) - ord('A')
        self.ring_setting = ring_setting % 26
•	Constructor: Initializes with rotor wiring (converting letters to indices), notch position (where the rotor triggers the next rotor's step), and ring setting (how much the alphabet ring is offset).
•	wiring converts the string of letters into a list of indices (0-25).
•	position starts at 0, representing the current letter the rotor is set to.
•	notch is where the rotor causes the next rotor to step.
•	ring_setting adjusts the internal wiring relative to the alphabet.

python
    def forward(self, letter):
        index = (ord(letter.upper()) - ord('A') + self.position - self.ring_setting) % 26
        return chr((self.wiring[index] + self.position + self.ring_setting) % 26 + ord('A'))
•	Forward: Converts the input letter to an index, adjusts for rotor position and ring setting, then uses the rotor's wiring to find the output index, converting it back to a letter.

python
    def backward(self, letter):
        index = (ord(letter.upper()) - ord('A') + self.position) % 26
        wiring_pos = self.wiring.index((index - self.ring_setting + ord('A')) % 26)
        adjusted_index = (wiring_pos - self.position + self.ring_setting) % 26
        return chr(adjusted_index + ord('A'))
•	Backward: Similar to forward but reverses the path. It finds where the letter would map in the wiring, then adjusts for rotor position and ring setting to get back to the original input.

python
    def rotate(self):
        old_position = self.position
        self.position = (self.position + 1) % 26
        return old_position == self.notch
•	Rotate: Advances the rotor by one position and checks if this rotation should cause the next rotor to step (by comparing the old position with the notch).

Reflector Class:
python
class Reflector:
    def __init__(self, wiring):
        self.wiring = [ord(char) - ord('A') for char in wiring]

    def reflect(self, letter):
        return chr(self.wiring[ord(letter.upper()) - ord('A')] + ord('A'))
•	Constructor: Defines the reflector's fixed wiring.
•	Reflect: Maps an input letter to another letter using the reflector's wiring.

Plugboard Class:
python
class Plugboard:
    def __init__(self, connections):
        self.connections = {a: b for a, b in connections}
        self.connections.update({b: a for a, b in connections})

    def swap(self, letter):
        return self.connections.get(letter.upper(), letter)
•	Constructor: Sets up the connections for letter swapping.
•	Swap: Checks if the letter is in the connections dictionary, if so, it swaps it; otherwise, it returns the letter unchanged.

Enigma Class:
python
class Enigma:
    def __init__(self, rotors, reflector, plugboard):
        self.rotors = rotors
        self.reflector = reflector
        self.plugboard = plugboard

    def encrypt(self, text):
        encrypted = ''
        for char in text.upper():
            if char.isalpha():
                char = self.plugboard.swap(char)
                
                for rotor in self.rotors:
                    char = rotor.forward(char)
                
                char = self.reflector.reflect(char)
                
                for rotor in reversed(self.rotors):
                    char = rotor.backward(char)
                
                char = self.plugboard.swap(char)
                
                # Rotation logic with double-step mechanism
                if self.rotors[0].rotate():
                    if self.rotors[0].position == 0 or (len(self.rotors) > 1 and self.rotors[1].position == self.rotors[1].notch):
                        if len(self.rotors) > 1:
                            self.rotors[1].rotate()
                            if len(self.rotors) > 2 and self.rotors[1].position == 0:
                                self.rotors[2].rotate()
                
            encrypted += char
        return encrypted
•	Constructor: Initializes with rotors, reflector, and plugboard.
•	Encrypt: 
o	Iterates through each character in the input text:
	Swaps the letter through the plugboard.
	Passes through each rotor (forward pass).
	Reflects using the reflector.
	Reverses through the rotors (backward pass).
	Swaps through the plugboard again.
o	Implements rotor rotation with the double-step mechanism:
	The fast rotor steps every key press.
	The middle rotor steps when the fast rotor moves past its notch or when moving from 25 to 0.
	The slow rotor steps when the middle rotor is at its notch and the fast rotor moves.

This code simulates the operations of a historical Enigma machine with some key features for accuracy like ring settings and double-stepping, though actual machines had variations and potential for more complex configurations.


