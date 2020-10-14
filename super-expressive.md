# Super-expressive Snippet

Source: https://github.com/francisrstokes/super-expressive

Playgroup: https://sepg.netlify.app/

## Email regex

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


