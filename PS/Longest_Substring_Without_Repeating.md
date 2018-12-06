```javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function (s) {
  return longestSubstringFinder.getLongestLength(s);
};

let longestSubstringFinder = {
  max: 0,
  inputArray: [],
  currentSubstringArray: [],
  dictionary: new Set(),

  getLongestLength(input) {
    this.initial();
    this.inputArray = [...input];

    if (this.inputArray.length === 1) {
      return 1;
    }

    for (let char of this.inputArray) {
      if (this.dictionary.has(char)) {
        this.max = this.max > this.currentSubstringArray.length ? this.max : this.currentSubstringArray.length;
        this.currentSubstringArray = [];
      }
      this.currentSubstringArray.push(char);
      this.dictionary.add(char);
    }

    return this.max > this.currentSubstringArray.length ? this.max : this.currentSubstringArray.length;
  },

  initial() {
    this.max = 0;
    this.inputArray = [];
    this.currentSubstringArray = [];
    this.dictionary = new Set();
  }
};
```