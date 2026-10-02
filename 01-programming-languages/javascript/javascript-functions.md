<table_of_contents color="gray"/>
# And let me know about Java and Javascript's marks. Like ===, ==, ?, ??. Compare to each of them.
Here’s a comparison of some common JavaScript operators, including strict equality (`===`), loose equality (`==`), the conditional (ternary) operator (`?`), and the nullish coalescing operator (`??`). I’ll explain each operator, how they work, and where they’re most useful.
---
### 1. **Strict Equality (****`===`****)**
- **Description**: Checks if two values are equal in both **value** and **type**.
- **Usage**: Best when you want exact equality without implicit type conversion.
- **Example**:
	```javascript
console.log(5 === 5);       // true (same type, same value)
console.log(5 === '5');     // false (different types)
console.log(true === 1);    // false (different types)

	```
- **Comparison to ****`==`**: Unlike `==`, `===` does not perform type coercion, so both the value and the type must match for it to return `true`.
---
### 2. **Loose Equality (****`==`****)**
- **Description**: Checks if two values are equal after **type coercion** (converting values to the same type).
- **Usage**: Only use `==` when you want to allow type conversion or when comparing to `null` or `undefined`.
- **Example**:
	```javascript
console.log(5 == '5');      // true (string '5' is converted to number 5)
console.log(true == 1);     // true (boolean true is converted to number 1)
console.log(null == undefined);  // true (null and undefined are loosely equal)

	```
- **Comparison to ****`===`**: `==` allows comparisons between different types, but it can produce unexpected results due to type conversion, so `===` is generally safer for comparisons.
---
### 3. **Conditional (Ternary) Operator (****`? :`****)**
- **Description**: A shorthand for an `if...else` statement that assigns or returns a value based on a condition.
- **Usage**: Use it when you need to quickly choose between two values or expressions based on a condition.
- **Syntax**: `condition ? expressionIfTrue : expressionIfFalse`
- **Example**:
	```javascript
const age = 18;
const canVote = (age >= 18) ? 'Yes' : 'No';
console.log(canVote);  // "Yes"

	```
- **Comparison to ****`if...else`**: The ternary operator is more concise and is typically used for simple conditions, whereas `if...else` is more readable for complex branching logic.
---
### 4. **Nullish Coalescing Operator (****`??`****)**
- **Description**: Returns the right-hand operand if the left-hand operand is `null` or `undefined`; otherwise, it returns the left-hand operand.
- **Usage**: Great for setting default values without mistakenly using falsy values like `0` or `""`.
- **Example**:
	```javascript
const name = null;
const defaultName = "Guest";
const userName = name ?? defaultName;
console.log(userName); // "Guest"

	```
- **Comparison to ****`||`**** (Logical OR)**: The `??` operator only checks for `null` or `undefined`, whereas `||` checks for all falsy values (like `0`, `false`, `""`, etc.). Use `??` when you want to allow values like `0` or `false` to pass through.
---
### Summary Table
<table>
<tr>
<td>Operator</td>
<td>Description</td>
<td>Example</td>
<td>Output</td>
<td>Key Notes</td>
</tr>
<tr>
<td>`===`</td>
<td>Strict equality, checks value & type</td>
<td>`5 === '5'`</td>
<td>`false`</td>
<td>No type coercion, must match both type/value</td>
</tr>
<tr>
<td>`==`</td>
<td>Loose equality, checks value with type coercion</td>
<td>`5 == '5'`</td>
<td>`true`</td>
<td>Allows type conversion</td>
</tr>
<tr>
<td>`? :`</td>
<td>Ternary operator, shorthand for `if...else`</td>
<td>`age >= 18 ? 'Yes' : 'No'`</td>
<td>`"Yes"` (if `age = 18`)</td>
<td>Used for concise conditional expressions</td>
</tr>
<tr>
<td>`??`</td>
<td>Nullish coalescing, returns right if `null`/`undefined`</td>
<td>`null ?? 'default'`</td>
<td>`"default"`</td>
<td>Useful for default values without `false`, `0`, or `""` interfering</td>
</tr>
</table>
---
### Choosing Between the Operators
- **`===`**: Use this as the default equality operator in JavaScript for precise comparisons.
- **`==`**: Use carefully, mostly when comparing against `null` or `undefined` in legacy code or specific use cases.
- **`? :`**** (Ternary)**: Great for quick, simple conditionals.
- **`??`**** (Nullish Coalescing)**: Use when you need a default value without accidentally overwriting falsy values like `0` or `""`.
Each operator has its place, but understanding their nuances can help you write clearer and more predictable code. Let me know if you’d like examples in more specific scenarios!
Absolutely! Here’s a deeper look into more JavaScript operators, including additional comparison, logical, assignment, and arithmetic operators, along with examples, differences, and typical use cases.
---
### 5. **Logical OR (****`||`****)**
- **Description**: Returns the first truthy value among its operands, or the last operand if all are falsy.
- **Usage**: Often used to set default values when a variable might be `null`, `undefined`, or another falsy value.
- **Example**:
	```javascript
const name = "";
const defaultName = "Guest";
const displayName = name || defaultName;
console.log(displayName);

	```
- **Output**: `"Guest"` (since `name` is an empty string, which is falsy).
- **Comparison to ****`??`**: `||` treats all falsy values (`0`, `""`, `false`, etc.) as “missing,” while `??` only checks for `null` or `undefined`.
---
### 6. **Logical AND (****`&&`****)**
- **Description**: Returns the first falsy value among its operands, or the last operand if all are truthy.
- **Usage**: Often used to perform an operation only if the first operand is truthy.
- **Example**:
	```javascript
const user = { name: "Alice" };
const username = user && user.name;
console.log(username);

	```
- **Output**: `"Alice"` (because `user` is truthy, so it continues to evaluate `user.name`).
- **Common Use**: Used in conditional statements where you want to check multiple conditions or execute code based on truthiness.
---
### 7. **Logical NOT (****`!`****)**
- **Description**: Converts a truthy value to `false` and a falsy value to `true`.
- **Usage**: Used to negate a boolean value.
- **Example**:
	```javascript
const isLoggedIn = false;
console.log(!isLoggedIn);  // true

	```
- **Output**: `true` (negates `false` to `true`).
- **Double NOT (****`!!`****)**: Often used to convert a value to a boolean.
	```javascript
const value = "hello";
console.log(!!value);  // true

	```
---
### 8. **Nullish Assignment (****`??=`****)**
- **Description**: Assigns a value only if the variable is `null` or `undefined`.
- **Usage**: Use when you want to set a default value only if a variable is uninitialized.
- **Example**:
	```javascript
let userColor = null;
userColor ??= "blue";
console.log(userColor);  // "blue"

	```
- **Output**: `"blue"`
- **Comparison to ****`||=`**: `||=` assigns if the variable is falsy (like `""` or `0`), while `??=` only assigns if `null` or `undefined`.
---
### 9. **Optional Chaining (****`?.`****)**
- **Description**: Safely accesses properties or methods, returning `undefined` if any part of the chain is `null` or `undefined`.
- **Usage**: Used when accessing deeply nested properties that may not exist.
- **Example**:
	```javascript
const user = { profile: { name: "Alice" } };
console.log(user.profile?.name);     // "Alice"
console.log(user.profile?.age);      // undefined
console.log(user.address?.street);   // undefined (no error)

	```
- **Output**: `"Alice"`, `undefined`, `undefined`
- **Comparison to Traditional Access**: Without optional chaining, you would need multiple checks to avoid errors, making this a cleaner solution.
---
### 10. **Exponentiation (****`*`****)**
- **Description**: Raises a number to the power of another number.
- **Usage**: Use when performing power operations, like `2^3`.
- **Example**:
	```javascript
console.log(2 ** 3);  // 8
console.log(5 ** 2);  // 25

	```
- **Output**: `8` and `25`
- **Comparison to ****`Math.pow`**: This is a more concise, modern alternative to `Math.pow(2, 3)`.
---
### 11. **Arithmetic Operators**
- **Description**: Basic operators for mathematical operations.
- **Operators**: `+` (Addition),  (Subtraction),  (Multiplication), `/` (Division), `%` (Modulus).
- **Example**:
	```javascript
console.log(10 + 5);   // 15
console.log(10 - 5);   // 5
console.log(10 * 5);   // 50
console.log(10 / 5);   // 2
console.log(10 % 3);   // 1 (remainder)

	```
- **Usage**: Used for general math operations.
---
### 12. **Assignment Operators**
- **Description**: Assign values to variables and can include arithmetic.
- **Operators**: `=`, `+=`, `=`, `=`, `/=`, `%=`, etc.
- **Example**:
	```javascript
let x = 5;
x += 3;  // x = x + 3
console.log(x);  // 8
x *= 2;  // x = x * 2
console.log(x);  // 16

	```
- **Usage**: Useful for in-place updates to variables.
---
### 13. **Comparison Operators**
- **Operators**: `>`, `<`, `>=`, `<=`
- **Description**: Used to compare two values.
- **Example**:
	```javascript
console.log(5 > 3);   // true
console.log(5 < 3);   // false
console.log(5 >= 5);  // true
console.log(5 <= 3);  // false

	```
- **Usage**: Primarily for conditions in loops and `if` statements.
---
### 14. **Bitwise Operators**
- **Operators**: `&` (AND), `|` (OR), `^` (XOR), `~` (NOT), `<<` (Left Shift), `>>` (Right Shift).
- **Description**: Perform operations at the bit level.
- **Example**:
	```javascript
console.log(5 & 1);  // 1 (0101 & 0001)
console.log(5 | 1);  // 5 (0101 | 0001)

	```
- **Usage**: Often used in low-level programming or optimizing performance.
---
### 15. **Comma Operator (****`,`****)**
- **Description**: Evaluates multiple expressions and returns the last one.
- **Example**:
	```javascript
let x = (1, 2, 3); // x becomes 3
console.log(x);    // 3

	```
- **Usage**: Rarely used, often in complex expressions.
---
### 16. **Typeof Operator (****`typeof`****)**
- **Description**: Returns the type of a variable as a string.
- **Example**:
	```javascript
console.log(typeof "hello");  // "string"
console.log(typeof 42);       // "number"
console.log(typeof true);     // "boolean"

	```
- **Usage**: Useful for checking variable types, especially when handling unknown or dynamic data.
---
### 17. **Delete Operator (****`delete`****)**
- **Description**: Removes a property from an object.
- **Example**:
	```javascript
const obj = { a: 1, b: 2 };
delete obj.a;
console.log(obj);  // { b: 2 }

	```
- **Usage**: Primarily used for removing properties from objects dynamically.
---
### Summary of Notable Operators
<table>
<tr>
<td>Operator</td>
<td>Description</td>
<td>Use Case</td>
</tr>
<tr>
<td>`===`, `==`</td>
<td>Strict vs. loose equality</td>
<td>Comparisons</td>
</tr>
<tr>
<td>\`</td>
<td></td>
<td>`, `&&`, `!\`</td>
</tr>
<tr>
<td>`??`</td>
<td>Nullish coalescing</td>
<td>Default values with null/undefined only</td>
</tr>
<tr>
<td>`?.`</td>
<td>Optional chaining</td>
<td>Safe access to deeply nested properties</td>
</tr>
<tr>
<td>`? :`</td>
<td>Ternary operator</td>
<td>Short `if...else` expressions</td>
</tr>
<tr>
<td>`+`, `-`, `*`, `/`, `%`, `**`</td>
<td>Arithmetic and exponentiation</td>
<td>Basic math operations</td>
</tr>
<tr>
<td>`>`, `<`, `>=`, `<=`</td>
<td>Comparison</td>
<td>Looping, conditions</td>
</tr>
<tr>
<td>`typeof`, `delete`</td>
<td>Type checking and property deletion</td>
<td>Dynamic data handling and object management</td>
</tr>
<tr>
<td>`+=`, `-=`, `*=`, etc.</td>
<td>Arithmetic assignment</td>
<td>In-place updates</td>
</tr>
</table>
---
These operators are foundational in JavaScript, each tailored for specific programming needs. Let me know if you’d like more detailed examples of any operator or use cases for specific programming tasks!
# How about Javascript methods not relative DOM? Like reduce, forecast, map, etc. Let me know tomorrow at least 30 over.
Here’s a start with some commonly used non-DOM JavaScript methods along with examples and outputs:
---
### 1. **Array.prototype.map**
- **Description**: Creates a new array populated with the results of calling a provided function on every element.
- **Example**:
	```javascript
const numbers = [1, 2, 3];
const doubled = numbers.map(num => num * 2);
console.log(doubled);

	```
- **Output**: `[2, 4, 6]`
### 2. **Array.prototype.forEach**
- **Description**: Executes a provided function once for each array element.
- **Example**:
	```javascript
const colors = ["red", "green", "blue"];
colors.forEach(color => console.log(color));

	```
- **Output**:
	```plain text
red
green
blue

	```
### 3. **Array.prototype.filter**
- **Description**: Creates a new array with all elements that pass a test implemented by the provided function.
- **Example**:
	```javascript
const numbers = [1, 2, 3, 4];
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens);

	```
- **Output**: `[2, 4]`
### 4. **Array.prototype.reduce**
- **Description**: Executes a reducer function on each array element, resulting in a single output value.
- **Example**:
	```javascript
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum);

	```
- **Output**: `10`
### 5. **Array.prototype.some**
- **Description**: Tests whether at least one element in the array passes the implemented test.
- **Example**:
	```javascript
const numbers = [1, 2, 3];
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven);

	```
- **Output**: `true`
### 6. **Array.prototype.every**
- **Description**: Tests whether all elements in the array pass the test implemented by the provided function.
- **Example**:
	```javascript
const numbers = [2, 4, 6];
const allEven = numbers.every(num => num % 2 === 0);
console.log(allEven);

	```
- **Output**: `true`
### 7. **Array.prototype.find**
- **Description**: Returns the first element in the array that satisfies the provided testing function.
- **Example**:
	```javascript
const numbers = [1, 3, 4, 5];
const firstEven = numbers.find(num => num % 2 === 0);
console.log(firstEven);

	```
- **Output**: `4`
### 8. **Array.prototype.findIndex**
- **Description**: Returns the index of the first element that satisfies the provided testing function.
- **Example**:
	```javascript
const numbers = [1, 3, 4, 5];
const index = numbers.findIndex(num => num % 2 === 0);
console.log(index);

	```
- **Output**: `2`
### 9. **Array.prototype.flat**
- **Description**: Creates a new array with all sub-array elements concatenated into it recursively up to the specified depth.
- **Example**:
	```javascript
const arr = [1, [2, [3, 4]], 5];
const flatArr = arr.flat(2);
console.log(flatArr);

	```
- **Output**: `[1, 2, 3, 4, 5]`
### 10. **Array.prototype.flatMap**
- **Description**: First maps each element using a mapping function, then flattens the result into a new array.
- **Example**:
	```javascript
const arr = [1, 2, 3];
const flatMapped = arr.flatMap(num => [num, num * 2]);
console.log(flatMapped);

	```
- **Output**: `[1, 2, 2, 4, 3, 6]`
---
### 11. **String.prototype.includes**
- **Description**: Checks if one string contains another string.
- **Example**:
	```javascript
const text = "Hello, world!";
console.log(text.includes("world"));

	```
- **Output**: `true`
### 12. **String.prototype.startsWith**
- **Description**: Determines if a string begins with the specified characters.
- **Example**:
	```javascript
const text = "Hello, world!";
console.log(text.startsWith("Hello"));

	```
- **Output**: `true`
### 13. **String.prototype.endsWith**
- **Description**: Determines if a string ends with the specified characters.
- **Example**:
	```javascript
const text = "Hello, world!";
console.log(text.endsWith("!"));

	```
- **Output**: `true`
### 14. **String.prototype.repeat**
- **Description**: Constructs a string with copies of the string, repeated a specified number of times.
- **Example**:
	```javascript
const word = "hi";
console.log(word.repeat(3));

	```
- **Output**: `"hihihi"`
### 15. **String.prototype.split**
- **Description**: Splits a string into an array of substrings.
- **Example**:
	```javascript
const sentence = "Hello, world!";
const words = sentence.split(" ");
console.log(words);

	```
- **Output**: `["Hello,", "world!"]`
---
Would you like to continue with more, or should we focus on specific types or tasks?
Continuing with more non-DOM JavaScript methods:
---
### 16. **String.prototype.trim**
- **Description**: Removes whitespace from both ends of a string.
- **Example**:
	```javascript
const text = "  Hello, world!  ";
console.log(text.trim());

	```
- **Output**: `"Hello, world!"`
### 17. **String.prototype.toUpperCase**
- **Description**: Converts a string to uppercase letters.
- **Example**:
	```javascript
const text = "hello";
console.log(text.toUpperCase());

	```
- **Output**: `"HELLO"`
### 18. **String.prototype.toLowerCase**
- **Description**: Converts a string to lowercase letters.
- **Example**:
	```javascript
const text = "HELLO";
console.log(text.toLowerCase());

	```
- **Output**: `"hello"`
### 19. **Number.isInteger**
- **Description**: Determines if a value is an integer.
- **Example**:
	```javascript
console.log(Number.isInteger(4));    // true
console.log(Number.isInteger(4.5));  // false

	```
- **Output**:
	```plain text
true
false

	```
### 20. **Number.parseFloat**
- **Description**: Parses a string argument and returns a floating-point number.
- **Example**:
	```javascript
console.log(Number.parseFloat("3.14"));

	```
- **Output**: `3.14`
### 21. **Number.parseInt**
- **Description**: Parses a string argument and returns an integer.
- **Example**:
	```javascript
console.log(Number.parseInt("42px"));

	```
- **Output**: `42`
### 22. **Math.max**
- **Description**: Returns the largest of zero or more numbers.
- **Example**:
	```javascript
console.log(Math.max(1, 3, 2));

	```
- **Output**: `3`
### 23. **Math.min**
- **Description**: Returns the smallest of zero or more numbers.
- **Example**:
	```javascript
console.log(Math.min(1, 3, 2));

	```
- **Output**: `1`
### 24. **Math.floor**
- **Description**: Rounds a number down to the nearest integer.
- **Example**:
	```javascript
console.log(Math.floor(4.7));

	```
- **Output**: `4`
### 25. **Math.ceil**
- **Description**: Rounds a number up to the nearest integer.
- **Example**:
	```javascript
console.log(Math.ceil(4.3));

	```
- **Output**: `5`
### 26. **Math.round**
- **Description**: Rounds a number to the nearest integer.
- **Example**:
	```javascript
console.log(Math.round(4.5));

	```
- **Output**: `5`
### 27. **Math.random**
- **Description**: Returns a random number between 0 (inclusive) and 1 (exclusive).
- **Example**:
	```javascript
console.log(Math.random());

	```
- **Output**: A random number between 0 and 1, e.g., `0.345`
### 28. **Math.sqrt**
- **Description**: Returns the square root of a number.
- **Example**:
	```javascript
console.log(Math.sqrt(9));

	```
- **Output**: `3`
### 29. **Object.keys**
- **Description**: Returns an array of a given object's own property names.
- **Example**:
	```javascript
const obj = { a: 1, b: 2, c: 3 };
console.log(Object.keys(obj));

	```
- **Output**: `["a", "b", "c"]`
### 30. **Object.values**
- **Description**: Returns an array of a given object's own property values.
- **Example**:
	```javascript
const obj = { a: 1, b: 2, c: 3 };
console.log(Object.values(obj));

	```
- **Output**: `[1, 2, 3]`
---
Let me know if you’d like even more examples or explanations of any particular methods!
