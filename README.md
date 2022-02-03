# JavaScript Design Pattern

List of design patterns

-   Abstract factory
-   Builder
-   Factory Method
-   Prototype
-   Singleton

Design pattern that will be used here is **Creational Patterns**. In this repository, there is a project that created with a problem that the **Creational Patterns** will solve.

# Ten Guideline for Future-Proof Code

The ten guidelines are :

1. Write short units of code
2. Write simple units of code
3. Write code once
4. Keep unit interfaces small
5. Seperate concerns in modules
6. Couple architecture components loosely
7. Keep architecture components balanced
8. Keep your codebase small
9. Automate development pipeline and tests
10. Write celan code

# Design Patterns Intent

## Abstract Factory

> Provide an interface for creating families of related dependent object without specifying their concrete classes

The goal for _Abstract Factory_ is to create single or multiple related objects without specifying their class. The reason for this could be to create a general UI component for an application.
Let’s say this UI is a simple alert popup with text and a button. Both the text and the button
should be customizable so the alert component that take a factory that it uses to create the
text and the button. The goal is to allow the UI component create the text and the button
without specifying their classes. This allows for the developer to easily switch out the factory
with another to allow a theme change for all components that uses the factories.

## Builder

> Seperate the construction of a complex object from its representation so that the same construction process can create different representations

The Builder is used to collect multiple different small pieces and then build everything
together at the end. Note that the Builder is usable even it all data is present. Since it
collects piece by piece and then builds everything, this pattern is commonly used when
reading large file bit by bit and then calculates the result once everything has been read.

## Factory Method

> Define an interface for creating an object, but let subclasses device which class to instantiate. Factory Method lets a class defer instantiation to subclasses

Factory Method is very similar to Abstract Factory. Biggest difference is that the Factory
Method doesn’t emphasis on families. The goal for the Factory Method is to allow users to
create objects without specifying their class. It also opens up a way to easily switch out one
factory with another that creates another type of object. Like the example for Abstract
Factory with switching out factories depending on the theme

## Prototype

> Specify the kinds of objects to create using a prototypical instance, and create nwe objects by copying this prototype

Prototype is used when creating entirely new objects is too costly or some other reason that
makes the ‘new’ operator harmful. With Prototype, you clone an object in order to create a
new object by create a ‘clone’ method that create a new object and copies its properties.

## Singleton

> Ensure a class only has one instance, and provide a global point of access to it

Singleton creates one single object and allowing it to be accessed globally. One common
use-case could be to combine both Singleton and Abstract Factory to ensure only one
factory is created and can be easily access by a global point of access. Other use-cases can
also be connection to the database or loggers. This is often used when if multiple objects are
created, all of them will modify the same resource. But it doesn’t always have to

# Example of Design Pattern in JavaScript

## Abstract Factory

To show how to implement this, we are going to create a project that generates fake data
with support for different languages. It will support the following data: name and address. We
will create two factories, one will create swedish data and the other will create english data.

We will start by creating the base factory, this factory will take an array of names and
addresses. When the factory methods are called, the factory will randomize any of them and
returns the result. Let's start by defining a class with the base logic.

```js
module.exports = class DataFactory {
    constructor(name, addresses) {
        this.names = names;
        this.addresses = addresses;
    }
    createName() {
        return this.names[randomNumber(this.names.length)];
    }
    createAddress() {
        return this.addresses[randomNumber(this.addresses.length)];
    }
};
```

Now with this factory class, we can now add our language factories by extending this class
and override the constructor to add our language specific data. We are defining the names
and addresses outside the export since the class feature from ES2015 doesn’t support static
variables. But since we are using CommonJS, the variables we are defining within the file
becomes like private static variables. These names and addresses are then shared with all
swedish factories that are created.

```js
const names = ["Felix", "Robin", "Alex"];
const addresses = ["Smedjegatan 17", "Minvernavagen 20"];
module.exports = class SwedishDataFactory extends DataFactory {
    constructor() {
        super(name, addresses);
    }
};
```

### Testing

What we expected from the factory is that it only creates the data that we created the factory with.

```js
it("should only generate the same names it was created with", function () {
    const data = ["Foo", "Bar", "Baz"];
    const factory = new DataFactory(data, []);

    for (let i = 0; i < 50; ++i) {
        expect(factory.createName()).to.be.oneOf(data);
    }
});
```

The same test can be applied to addresses. For the factories that extends this don’t have any logic in it. This means that there is not a lot
to test for the factories. We just want to check that those factories actually are an instance of
the base factory

```js
it("should be an instance of DataFactory", function () {
    expect(new SwedishDataFactory()).to.be.instanceof(DataFactory);
});
```

## Builder

### Example

Imagine we are going to create a tool that reads data from files and converts that data to
other formats. For now we want to support converting to hexadecimal and binary. We first
create a class that is going to be the base of all converters. What it does is simply create an
empty result and a method available for fetching the result.

```js
module.exports = class Converter {
    constructor() {
        this.result = "";
    }
    getResult() {
        return this.result;
    }
};
```

Now we can create two modules that extends the Converter class and adds a method for converting the input and appends it to the result

```js
module.exports = class HexConverter extends Converter {
    append(data) {
        this.result += data.toString(16);
    }
};
module.exports = class BinaryConverter extends Converter {
    append(data) {
        this.result += data.toString(2);
    }
};
```

### Testing

To test the builder is simple, we want to check the properties of the returned value is equal
to what we expect. Both the tests for hexadecimal and binary looks the same so we will only
show how the tests looks like for the binary converter.

```js
it("should convert to binary", function () {
    const converter = new BinaryConverter();
    converter.append(5);

    expect(converter.getResult()).to.be.equal("101");
});
it("should be able to convert to multiple data", function () {
    const converter = new BinaryConverter();
    converter.append(5);
    converter.append(3);

    expect(converter.getResult()).to.be.equal("10111");
});
```

## Factory Method

### Example

Let's say that we have a project that needs to log information. We want to log to the console during development but during production, we want to log to a file instead. Let's say that we have a logger that takes a stream that it will write to when logging.

We will then create two factories, one that creates a logger with a stream that goes to the console, the other one will be to a file.

```js
module.exports = class ConsoleLoggerFactory {
    async create() {
        return new Logger(process.stdout);
    }
};

module.exports = class FileLoggerFactory {
    async create() {
        await fs.ensureFile(settings.logger.file);

        const stream = await fs.createWriteStream(setting.logger.file, {
            flags: "a",
        });

        return new Logger(stream);
    }
};
```

### Testing

To test whether the factories does create the expected object, we can check if the object that
is returned by the factories are instance of the expected modules.

For the ConsoleLoggerFactory, we want to check if the method actually returns an instance
of the Logger as well as that it used the stdout stream to create it. To achieve this, we will
modify what the factory receives when requiring the Logger class. We want to apply a spy to
the class to be able to check if it was called with the correct arguments.

```js
before("require the module", function () {
    loggerSpy = sinon.spy(Logger);

    ConsoleLoggerFactory = proxyquire("../src/console-logger-factor", {
        "./logger": loggerSpy,
    });
});

it("should create Logger with stdout stream", async function () {
    const logger = await new ConsoleLoggerFactory().create();

    expect(logger).to.be.instanceof(Logger);
    expect(loggerSpy).to.have.been.calledWithExactly(process.stdout);
});
```
