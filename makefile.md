# Makefile

- [Makefile](#makefile)
  - [Basic Use](#basic-use)
  - [References](#references)

## Basic Use

```
# Makefile
say_hello:
    @echo "Hello"
```

```sh
> make
> Hello

```

multiple order

```
say_hello:
        @echo "Hello World"

generate:
        @echo "Creating empty text files..."
        touch file-{1..10}.txt
```

If you run `make`, there would be only the first order to be executed.

If you wanna run all order, follow this:
```
all: say_hello generate
say_hello:
        @echo "Hello World"

generate:
        @echo "Creating empty text files..."
        touch file-{1..10}.txt
```

The terminal will show:
```
> Hello World
> Creating empty text files...
```

## References
1. https://opensource.com/article/18/8/what-how-makefile