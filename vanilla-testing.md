# Testing Vanilla functions


## The quickest possible way
Add `chai.js` in a script-tag to your html-file via a CDN (or download to local folder):

	<script src="https://cdnjs.cloudflare.com/ajax/libs/chai/3.5.0/chai.min.js" type="text/javascript"></script>

In your js-file, add `var expect = chai.expect;` before writing the tests. »expect«-stlye goes like:

	expect(util.precise_round(0.234123, 1).to.equal(0.2);

👍 Now you can test your functions. You will see non-passing tests in your browser console.



## Adding Test Framework 'Mocha'
Adding a task runner lets you do the testing in your terminal (no browser needed.) You can also let your tests rerun every time you make a change to your specFiles, so you don't have to refresh manually.

	npm init                      # if not present yet
	npm install mocha --save-dev  # install mocha via npm
	mkdir test/                   # create test directory (mocha will look there by default)
	touch test/utilSpec.js        # create a file to write the tests in



### Extract functions to seperate file

Extract the functions you want to test from your "main.js" and browser logic. Objects like `window` or `document` will not be present to »mocha« by default.

	touch util.js                 # create a new file for extracted functions



### Writing the tests

	var assert = require('assert');          // assert style is already provided by node.js
	var util = require('../util.js');        // require file with the extracted functions

	describe('precise_round', function() {
	  it('should round a number to specified number of decimal points', function() {
	  	assert.equal(0.2, util.precise_round(0.234123, 1));
	    assert.equal(0.23, util.precise_round(0.234123, 2));
	  });
	});

### Running the tests
In your `package.json`, call mocha with a flag to watch for changes:

	"scripts": {
      "test": "mocha --watch"
  	},

Then start the task with `npm run test`.

👍🎉 You now automated your test-runner.



##### expect style
To use expect style, you need to install »chai.js« via `npm install chai --save-dev`. Then require it in your `utilSpec.js`:

	var expect = require('chai').expect;
	...

You can then write your tests like:

	describe('precise_round', function() {
  	  it('should round a number to specified number of decimal points', function() {
   	    expect(util.precise_round(0.234123, 1)).to.equal(0.2);
   	    expect(util.precise_round(0.234123, 2)).to.equal(0.23);
  	  });
	});