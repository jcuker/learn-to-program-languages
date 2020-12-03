# Chapter 4 - Scanning

## Challenge 1
### Question:
The lexical grammars of Python and Haskell are not regular. What does that mean, and why aren’t they?

### Answer:
> "A regular language is a language that can be expressed with a regular expression or a deterministic or non-deterministic finite automata or state machine." - via [Brilliant](https://brilliant.org/wiki/regular-languages/)

If we take Python and Haskell, both languages have rules regarding the indentation of the program. Since a _regular_ language can be described by finite automata, and finite automata _cannot_ store state, there is no way to know what the indentation level of the program _should_ be as it depends on the parts of the program that came before. 

## Challenge 2
### Question
Aside from separating tokens — distinguishing `print foo` from `printfoo` — spaces aren’t used for much in most languages. However, in a couple of dark corners, a space does affect how code is parsed in CoffeeScript, Ruby, and the C preprocessor. Where and what effect does it have in each of those languages?

### Answer

#### Coffeescript
Referencing a StackOverflow [post](https://stackoverflow.com/questions/16314251/coffeescript-issue-with-space), spaces in Coffeescript will affect how boolean expressions are grouped together.

This Coffeescript

```Coffeescript
if eachController.indexOf("Controller.js") isnt -1
  controller = require(controllersFolderPath + eachControllerName)
  controller.register server 
```

becomes the following Javascript

```Javascript
if (eachController.indexOf("Controller.js") !== -1) {
  controller = require(controllersFolderPath + eachControllerName);
  controller.register(server);
}

```
If we add a space between `indexOf` and the opening parenthesis of `("Controller.js")` the Javascript becomes

```Javascript
if (eachController.indexOf("Controller.js" !== -1)) {
    controller = require(controllersFolderPath + eachControllerName);
    controller.register(server);
}
```

#### Ruby
A similar issue happens in Ruby. Notice the difference in spacing in following Ruby snippets.

```Ruby
render json: {
    "what" => "created", 
    "whatCreated" => "thing",
    "htmlOutput" => render_to_string (partial: "some_partial")
}


render json: {
    "what" => "created", 
    "whatCreated" => "thing",
    "htmlOutput" => render_to_string(partial: "some_partial")
}
```

The issue is that Ruby can be run with or without parenthesis. Quoting an excellent StackOverflow [explanation](https://stackoverflow.com/questions/26480823/why-does-white-space-affect-ruby-function-calls):

> for example, you can run Array.new 1,2 and ruby knows that it receives the arguments after the space. and you can also run Array.new(1,2) and ruby knows the args are inside the parentheses. 
> 
> but, when you run Array.new (1,2) , ruby thinks it will receive arguments after the space but actually it receives a tuple (1,2), and basicaly its exactly the same as Array.new((1,2)) 
>
> so bottom line:Array.new (1,2) == Array.new((1,2)) and thats a syntax error because (1, 2) literal is not a valid one

Also, another [fun whitespace problem](https://qrohlf.com/posts/ruby-whitespace-shenanigans) in Ruby, but this is due to unicode, which was probably _not_ the intent of the original question.

#### C Preprocessor
A problem in the C Preprocessor regarding whitespace can be found with [token spacing](https://gcc.gnu.org/onlinedocs/cppinternals/Token-Spacing.html). Using the examples from the above link

```C
#define PLUS +
#define EMPTY
#define f(x) =x=

+PLUS -EMPTY- PLUS+ f(=)
```

will evaluate to 

```C
→ + + - - + + = = =
```
not

```C
→ ++ -- ++ ===
```

## Challenge 3
### Question
Our scanner here, like most, discards comments and whitespace since those aren’t needed by the parser. Why might you want to write a scanner that does not discard those? What would it be useful for?

### Answer
One potential reason is to do any analysis, or even metaprogramming, over the source. Running linters that will edit the files automatically is one valid use case. 

## Challenge 4
### Question 
Add support to Lox’s scanner for C-style /* ... */ block comments. Make sure to handle newlines in them. Consider allowing them to nest. Is adding support for nesting more work than you expected? Why?

### Answer
Here is the code that needs to change to support block comments.

Within the `scanToken` switch statement, edit the `'/'` case to be the following

```Java
case '/':
   if (match('/')) {
      // A comment goes until the end of the line.
      while (peek() != '\n' && !isAtEnd()) advance();
   } else if (match('*')) 
      // this else if is the new segment
      handleBlockComment();
   } else {
      addToken(TokenType.SLASH);
   }
   break;
```

and paste the following method in your `scanner` class. 

```Java
private void handleBlockComment() {
        boolean stillInBlockComment = true;

        while (stillInBlockComment) {
            // Go until we find a '*' character
            if (peek() == '*' && !isAtEnd()) {
                // now that we've found a '*' we need to check if it ends the block comment
                // first, advance over the '*' character
                advance();

                // if the '*' character is proceeded by a '/', end the block comment
                if (peek() == '/') {
                    stillInBlockComment = false;
                }
            }

            advance();
        }
    }
```