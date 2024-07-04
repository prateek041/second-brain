# Lexing

## Why use Lexing

To convert the code in a format that makes it easily accesible

## What is Lexing

Lexing is the process of converting the source code into tokens. It is done by a lexer (also called a tokenizer)

so, if we consider this statement as an example
`let x = 5 + 5;`, lexer will convert into an array of tokens that looks like this

```
[
LET,
IDENTIFIER("x"),
EQUAL_SIGN,
INTEGER(5),
PLUS_SIGN,
INTEGER(5),
SEMI_COLON
]
```

# Language specific things

- Identifiers: These are variable names, that are binded to an expression, or a variable.
- Keywords: let, fn.
- Numbers: Integers.
- Special Characters:

## Lexer Definition

There are a few things to be condidered when creating the Lexer struct.

We want a way to go through a given string, character by character and take out the words we care about. The words that part of our language spec. It is just like the problem of finding a matching word from a string.

Therefore the struct turns out to be looking like this.

```go
type Lexer struct{
  ch byte
  readPosition int
  Position int
  input string
}
```

### Working of Lexer

We have to create methods that read the string character by character and then another method that checks the string and returns a corresponding token

As mentioned earlier, we have identifiers, numbers, special characters and keywords. Let's see one by one how they will work.

#### special Characters

Since special characters are single, we can be sure of which one they are in just one iteration of `readChar` which updates the `readPosition` by one and update the `ch` property with the current character we are dealing with.

#### Keywords

These are comparatively tricky, reading the first character does not give a definitive answer, we have to read characters till it starts making sense. For examplee `let` is a keyword, but `letter` can be an identifier, and we won't know the difference until we read the entire thing.
