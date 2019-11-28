# Patterns In Action

## Builder Design Pattern in javascript

[4 Dangerous Problems in JavaScript Easily Solved by the Builder Design Pattern](https://medium.com/better-programming/4-dangerous-problems-in-javascript-easily-solved-by-the-builder-design-pattern-7f0eb5b4455c)

**Without the Builder Pattern**

```javascript
    class Frog {
        constructor(name, gender, eyes, legs, scent, tongue, heart, weight, height) {
            this.name = name
            this.gender = gender
            this.eyes = eyes
            this.legs = legs
            this.scent = scent
            this.tongue = tongue
            this.heart = heart
            if (weight) {
            this.weight = weight
            }
            if (height) {
            this.height = height
            }
        }
    }
```

**With the Builder Pattern**


```javascript
    class FrogBuilder {
        constructor(name, gender) {
            this.name = name
            this.gender = gender
        }
        setEyes(eyes) {
            this.eyes = eyes
            return this
        }
        setLegs(legs) {
            this.legs = legs
            return this
        }
        setScent(scent) {
            this.scent = scent
            return this
        }
        setTongue(tongue) {
            this.tongue = tongue
            return this
        }
        setHeart(heart) {
            this.heart = heart
            return this
        }
        setWeight(weight) {
            this.weight = weight
            return this
        }
        setHeight(height) {
            this.height = height
            return this
        }
    }
```

**Validate input params without pattern**

```javascript
class Frog {
    constructor(name, gender, eyes, legs, scent, tongue, heart, weight, height) {
        if (!Array.isArray(legs)) {
            throw new Error('Parameter "legs" is not an array')
        }
        // Ensure that the first character is always capitalized
        this.name = name.charAt(0).toUpperCase() + name.slice(1)
        this.gender = gender
        // We are allowing the caller to pass in an array where the first index is the left eye and the 2nd is the right
        //    This is for convenience to make it easier for them.
        //    Or they can just pass in the eyes using the correct format if they want to
        //    We must transform it into the object format if they chose the array approach
        //      because some internal API uses this format
        this.eyes = Array.isArray(eyes) ? { left: eye[0], right: eye[1] } : eyes
        this.legs = legs
        this.scent = scent
        // Pretending some internal API changed the field name of the frog's tongue from "tongueWidth" to "width"
        //    Check for old implementation and migrate them to the new field name
        const isOld = 'tongueWidth' in tongue
        if (isOld) {
            const newTongue = { ...tongue }
            delete newTongue['tongueWidth']
            newTongue.width = tongue.width
            this.tongue = newTongue
        } else {
            this.tongue = newTongue
        }
        this.heart = heart
        if (typeof weight !== 'undefined') {
            this.weight = weight
        }
        if (typeof height !== 'undefined') {
            this.height = height
        }
    }
}

const larry = new Frog(
    'larry',
    'male',
    [{ volume: 1.1 }, { volume: 1.12 }],
    [{ size: 'small' }, { size: 'small' }, { size: 'small' }, { size: 'small' }],
    'sweaty socks',
    { tongueWidth: 18, color: 'dark red', type: 'round' },
    { rate: 22 },
    6,
    3.5,
)
```
**Readability with pattern**

```javascript
class FrogBuilder {
    constructor(name, gender) {
        // Ensure that the first character is always capitalized
        this.name = name.charAt(0).toUpperCase() + name.slice(1)
        this.gender = gender
    }
    formatEyesCorrectly(eyes) {
        return Array.isArray(eyes) ? { left: eye[0], right: eye[1] } : eyes
    }
    setEyes(eyes) {
        this.eyes = this.formatEyes(eyes)
        return this
    }
    setLegs(legs) {
        if (!Array.isArray(legs)) {
            throw new Error('"legs" is not an array')
        }
        this.legs = legs
        return this
    }
    setScent(scent) {
        this.scent = scent
        return this
    }
    updateTongueWidthFieldName(tongue) {
        const newTongue = { ...tongue }
        delete newTongue['tongueWidth']
        newTongue.width = tongue.width
        return newTongue
    }
    setTongue(tongue) {
        const isOld = 'tongueWidth' in tongue
        this.tongue = isOld
            ? this.updateTongueWidthFieldName(tongue, tongue.tongueWidth)
            : tongue
        return this
    }
    setHeart(heart) {
        this.heart = heart
        return this
    }
    setWeight(weight) {
        if (typeof weight !== 'undefined') {
            this.weight = weight
        }
        return this
    }
    setHeight(height) {
        if (typeof height !== 'undefined') {
            this.height = height
        }
        return this
    }
    build() {
        return new Frog(
            this.name,
            this.gender,
            this.eyes,
            this.legs,
            this.scent,
            this.tongue,
            this.heart,
            this.weight,
            this.height,
        )
    }
}

const larry = new FrogBuilder('larry', 'male')
    .setEyes([{ volume: 1.1 }, { volume: 1.12 }])
    .setScent('sweaty socks')
    .setHeart({ rate: 22 })
    .setWeight(6)
    .setHeight(3.5)
    .setLegs([
        { size: 'small' },
        { size: 'small' },
        { size: 'small' },
        { size: 'small' },
    ])
    .setTongue({ tongueWidth: 18, color: 'dark red', type: 'round' })
    .build()

```


**Repeated code**

```javascript
// frog
const sally = new FrogBuilder('sally', 'female')
    .setEyes([{ volume: 1.1 }, { volume: 1.12 }])
    .setScent('blueberry')
    .setHeart({ rate: 12 })
    .setWeight(5)
    .setHeight(3.1)
    .setLegs([
        { size: 'small' },
        { size: 'small' },
        { size: 'small' },
        { size: 'small' },
    ])
    .setTongue({ width: 12, color: 'navy blue', type: 'round' })
    .setHabitat('water')
    .setSkin('oily')
    .build()

// toad
const kelly = new FrogBuilder('kelly', 'female')
    .setEyes([{ volume: 1.1 }, { volume: 1.12 }])
    .setScent('black ice')
    .setHeart({ rate: 11 })
    .setWeight(5)
    .setHeight(3.1)
    .setLegs([
        { size: 'small' },
        { size: 'small' },
        { size: 'small' },
        { size: 'small' },
    ])
    .setTongue({ width: 12.5, color: 'olive', type: 'round' })
    .setHabitat('land')
    .setSkin('dry')
    .build()

// toad
const mike = new FrogBuilder('mike', 'male')
    .setEyes([{ volume: 1.1 }, { volume: 1.12 }])
    .setScent('smelly socks')
    .setHeart({ rate: 15 })
    .setWeight(12)
    .setHeight(5.2)
    .setLegs([
        { size: 'medium' },
        { size: 'medium' },
        { size: 'medium' },
        { size: 'medium' },
    ])
    .setTongue({ width: 12.5, color: 'olive', type: 'round' })
    .setHabitat('land')
    .setSkin('dry')
    .build()

```

**Avoid repeated code**

```javascript
class ToadBuilder {
    constructor(frogBuilder) {
        this.builder = frogBuilder
    }

    createToad() {
        return this.builder.setHabitat('land').setSkin('dry')
    }
}

let mike = new FrogBuilder('mike', 'male')

mike = new ToadBuilder(mike)
    .setEyes([{ volume: 1.1 }, { volume: 1.12 }])
    .setScent('smelly socks')
    .setHeart({ rate: 15 })
    .setWeight(12)
    .setHeight(5.2)
    .setLegs([
        { size: 'medium' },
        { size: 'medium' },
        { size: 'medium' },
        { size: 'medium' },
    ])
    .setTongue({ width: 12.5, color: 'olive', type: 'round' })
    .build()
```