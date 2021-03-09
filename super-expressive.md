# Super-expressive Snippet

Source: https://github.com/francisrstokes/super-expressive

Playground: https://sepg.netlify.app/

## First Name and Last Name
* Must not include any of the following characters :@.$#{}[]?<>+=()123456789
* A maximum of 2 spaces
* Minimum length of 2 characters

```typescript
SuperExpressive()
.startOfInput
.atLeast(2)
    .anythingButChars(' @.$#{}[]?<>+=()123456789')
.between(0,2)
    .capture
        .char(' ')
        .atLeast(1)
            .anythingButChars(' @.$#{}[]?<>+=()123456789')
    .end()
.endOfInput
.toRegex()
```

## Email regex

This must include an @ with 1 or more characters before and after the @

```typescript
SuperExpressive()
.startOfInput
.atLeast(1)
    .anyOf
        .word
        .anyOfChars('-.')
    .end()
.char('@')
.atLeast(1)
    .anyOf
        .word
        .anyOfChars('-')
    .end()
.char('.')
.atLeast(1)
    .anyOf
        .range('a','z')
        .range('A','Z')
    .end()
.endOfInput
.toRegex();
```

## Telephone regex

* A minimum of 10 characters
* Must not include any of the following characters :@.$#{}[]?<>=

```typescript
SuperExpressive()
.startOfInput
.atLeast(10)
    .anyOf
        .digit
        .range('a','z')
        .range('A','Z')
        .anyOfChars('+ ()-')
    .end()
.endOfInput
.toRegex()
```

## Product Code Matching
* 1Z -> not match
* 1Z-2B -> match
* 1Z,2B -> match
* 1A-2B,5A -> match
* 1A-2B-5A -> match
* 1A-2B,5A-3B -> match
* 1A-2A-3A-4A-5A,5B -> match

```typescript
SuperExpressive()
.startOfInput
.exactly(1)
    .range('0', '9')
.exactly(1)
    .range('A', 'Z')
.atLeast(1)
    .capture
        .anyOfChars('-,')
        .exactly(1)
            .range('0', '9')
        .exactly(1)
            .range('A', 'Z')
    .end()
.endOfInput
.toRegex()

```



