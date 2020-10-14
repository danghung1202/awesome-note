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
.oneOrMore
    .anyOf
        .word
        .anythingButChars('_')
    .end()
.oneOrMore
    .anyOf
        .word
        .anyOfChars('.-')
    .end()
.char('@')
.oneOrMore
    .anyOf
        .word
        .anyOfChars('-')
    .end()
.char('.')
.atLeast(2)
    .range('a','z')
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


