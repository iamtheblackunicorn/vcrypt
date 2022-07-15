<!--
VCRYPT by Alexander Abraham,
a.k.a. "The Black Unicorn", a.k.a. "Angeldust Duke".
Licensed under the MIT license.
-->

<!--
The main section of the app that does all the heavy lifting.
-->
<template>
 <div class="content">

  <!--Displays the result of any computations.-->
  <div class="result">
   <p class="result">{{ result }}</p>
  </div>

  <!--Gets and stores the user's input.-->
  <textarea v-model="inputText" placeholder="Your text goes here."/>
  <br/>

  <!--Displays the factor that is currently active.-->
  <p class="factor">Encryption Factor: {{ factor }}</p>

  <!--Displays the mode that is currently active.-->
  <p class="factor">Mode: {{ mode }}</p>

  <!--Displays the status of clipboard copying.-->
  <p class="factor">Result copied: {{ isCopied }}</p>

  <!--Small explanation for users.-->
  <p class="factor">The factor is the<br/>encryption/decryption key.</p>

  <br/>

  <!--Allows the user to copy text from the result field.-->
  <button @click="copyResult()">Copy</button>

  <!--Allows the user to increase the encryption factor.-->
  <button @click="increment()">Increment</button>

  <!--Allows the user to decrease the encryption factor.-->
  <button @click="decrement()">Decrement</button>

  <!--
  Allows the user to encrypt his input, 
  provided the correct mode is active.
  -->
  <button @click="encryptInput()">Encrypt</button>

  <!--
  Allows the user to decrypt his input, 
  provided the correct mode is active.
  -->
  <button @click="decryptInput()">Decrypt</button>

  <!--
  Allows the user to switch between modes.
  -->
  <button @click="switchMode()">Switch Mode</button>

  <BreakerCog/>

 </div>
 
 <BreakerCog/>
 
</template>

<!--This section actually makes all the magic happen!-->
<script>
// We import binary conversion
// functions from my library.
import zeppo from 'zeppo';

// We import the breakers for layout reasons.
import BreakerCog from './BreakerCog.vue';

export default {
  // Naming and registering the component.
  name: 'CryptCog',
  components: {
    BreakerCog
  },
  data() {
    // Setting up the app-wide
    // variables.
    return {
       // In what mode is the app working?
       mode: 'encrypt',
       // A variable to store the user's input.
       inputText: '',
       // The factor to multiply the user's 
       // input by to scramble the letters.
       factor: 1,
       // A variable to store the computational
       // result.
       result: 'Your result goes here.',
       // Tells the user if the text has been copied.
       isCopied: 'No.'
    };
  },
  methods: {

    // Encrypts a letter as a binary number.
    // The number is scrambled by the multiplication factor.
    encryptLetter(factor, letter){
      return zeppo.decToBin(zeppo.letterIndex(letter) * factor);
    },

    // Decrypts a letter from a binary number to an alphanumeric
    // character. The number is de-scrambled by the multiplication
    // factor.
    decryptLetter(factor, binaryNum){
      var integer = zeppo.binToDec(binaryNum);
      return zeppo.getLetterFromIndex(integer / factor).toString();
    },

    // Encrypts a word as a series of binary numbers.
    // Scrambles the binary numbers by the factor.
    encryptWord(factor, word){
      var charList = word.split('');
      var result = [];
      for (var i = 0; i < charList.length; i++){
        var encryptedLetter = this.encryptLetter(factor, charList[i]);
        result.push(encryptedLetter);
      }
      return result.join('|');
    },

    // Decrypts a word from a series of binary numbers
    // to an alphanumeric character sequence.
    // De-scrambles the binary numbers by the factor.
    decryptWord(factor, cipherText){
      var charList = cipherText.split('|');
      var result = [];
      for (var i = 0; i < charList.length; i++){
        var decryptedLetter = this.decryptLetter(factor, charList[i]);
        result.push(decryptedLetter);
      }
      return result.join('');
    },

    // Encrypts a phrase as a series of binary numbers.
    // Scrambles the binary numbers by the factor.
    encryptPhrase(factor, phrase){
      var wordList = phrase.split(' ');
      var result = [];
      for (var i = 0; i < wordList.length; i++){
        var encryptedWord = this.encryptWord(factor, wordList[i]);
        result.push(encryptedWord);
      }
      return result.join(';');
    },

    // Decrypts a phrase from a series of binary numbers
    // to an alphanumeric character sequence.
    // De-scrambles the binary numbers by the factor.
    decryptPhrase(factor, cipherText){
      var wordList = cipherText.split(';');
      var result = [];
      for (var i = 0; i < wordList.length; i++){
        var decryptedWord = this.decryptWord(factor, wordList[i]);
        result.push(decryptedWord);
      }
      return result.join(' ');
    },

    // Encrypts the user's input and updates the UI.
    encryptInput(){
      if (this.mode === 'encrypt') {
        this.result = this.encryptPhrase(this.factor, this.inputText);
      }
      else {
        this.result = 'Wrong mode!';
      }
    },

    // Decrypts the user's input and updates the UI.
    decryptInput(){
      if (this.mode === 'decrypt') {
        this.result = this.decryptPhrase(this.factor, this.inputText);
      }
      else {
        this.result = 'Wrong mode!';
      }
    },

    // Increments the scrambling factor.
    increment(){
      this.factor = this.factor + 1;
    },

    // Decrements the scrambling factor.
    decrement(){
      if (this.factor === 1){}
      else {
        this.factor = this.factor - 1;
      }
    },

    // Lets the user switch between
    // working modes.
    switchMode(){
      if (this.mode === 'encrypt'){
        this.mode = 'decrypt';
      }
      else {
        this.mode = 'encrypt';
      }
    },

    // Copies the text from the result field!
    copyResult(){
      navigator.clipboard.writeText(this.result);
      this.isCopied = 'Yes!';
    }
  }  
}
</script>