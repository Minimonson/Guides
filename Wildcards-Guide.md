# Syntax Guide

This guide will walk you through the template language used to generate dynamic prompts. It covers various features such as variants, wildcards, variables, and parameterized templates.

## Table of contents
   * [Variants](#variants)
      * [Basic Syntax](#basic-syntax)
      * [Weighting Options](#weighting-options)
      * [Choosing Multiple Values](#choosing-multiple-values)
      * [Custom Separator](#custom-separator)
      * [Range of Options](#range-of-options)
         * [Omitting Bounds](#omitting-bounds)
         * [Limitations](#limitations)
   * [Wildcards](#wildcards)
      * [Basic Syntax](#basic-syntax-1)
      * [Wildcards in Variants](#wildcards-in-variants)
      * [Variants in Wildcards](#variants-in-wildcards)
      * [Nested Wildcards](#nested-wildcards)
      * [Resolving Wildcards with Globbing](#resolving-wildcards-with-globbing)
         * [Basic Syntax](#basic-syntax-2)
         * [Example](#example)
      * [File formats](#file-formats)
         * [Text files](#text-files)
   * [Variables](#variables)
      * [Immediate Evaluation](#immediate-evaluation)
      * [Non-immediate Evaluation](#non-immediate-evaluation)
   * [Parameterized Templates](#parameterized-templates)
      * [Basic Syntax](#basic-syntax-3)
      * [Default values](#default-values)
   * [Whitespace and comments](#whitespace-and-comments)


## Variants

Variants allow you to randomly generate one or more options from a list of possibilities. They can be weighted, and you can control the number of options to be chosen.

### Basic Syntax

To create a variant, wrap your options in curly brackets {} and separate them with a vertical bar |. For example:

```
{summer|autumn|winter|spring} is coming
```

This will randomly generate one of the following:

* summer is coming
* autumn is coming
* winter is coming
* spring is coming

### Weighting Options

You can add weights to options to control their relative frequency. To do this, add a double colon `::` followed by the weight before the option:

```
`{0.5::summer|0.1::autumn|0.3::winter|0.1::spring}`
```

The weights are relative and do not have to add up to 1.
If you omit a weight, it is assumed to be 1.

### Choosing Multiple Values

To choose multiple values, add a number followed by double dollar signs `$$` before your options:

```
My favourite ice-cream flavours are {2$$chocolate|vanilla|strawberry}
```

This will generate one of the following:

* My favourite ice-cream flavours are chocolate, vanilla
* My favourite ice-cream flavours are chocolate, strawberry
* My favourite ice-cream flavours are vanilla, chocolate
* ...
etc

Values are chosen without replacement, so you won't get repeats.

### Custom Separator

You can change the default separator by adding a custom separator between the double dollar signs:

```
My favourite ice-cream flavours are {2$$ and $$chocolate|vanilla|strawberry}
```

This will generate one of the following:

* My favourite ice-cream flavours are chocolate and vanilla
* My favourite ice-cream flavours are chocolate and strawberry
* My favourite ice-cream flavours are vanilla and chocolate
* ...

### Range of Options

To provide a range for the number of options to be chosen, use a dash `-` between the lower and upper bounds:

```
My favourite ice-cream flavours are {1-2$$ and $$chocolate|vanilla|strawberry}
```

This will generate:

* My favourite ice-cream flavours are chocolate
* My favourite ice-cream flavours are strawberry
* My favourite ice-cream flavours are vanilla
* My favourite ice-cream flavours are chocolate and vanilla
* My favourite ice-cream flavours are chocolate and strawberry
* My favourite ice-cream flavours are vanilla and chocolate
* ...

#### Omitting Bounds

You can omit the lower or upper bound, and it will be treated as the minimum or maximum possible value:

```
{-2$$chocolate|vanilla|strawberry} == {1-2$$chocolate|vanilla|strawberry}
{1-$$chocolate|vanilla|strawberry} == {1-2$$chocolate|vanilla|strawberry}
```

#### Limitations

If you request more options than values in the variant, you will only get as many items as are available:

```
p{4$$chocolate|vanilla|strawberry} == chocolate, vanilla, strawberry
```

## Wildcards

### Basic Syntax

Wildcards are placeholders that inject values from a file into your prompt.
Create a file with the desired values and use double `==` to indicate the wildcard:

```
==season== is coming
```

Assuming you have a seasons.txt file with the following content:

```
# seasons.txt
summer
autumn
winter
spring
```

This prompt will randomly generate one of the following:

* summer is coming
* autumn is coming
* winter is coming
* spring is coming

### Wildcards in Variants

You can choose multiple values from a wildcard as follows:

```
My favourite ice-cream flavours are {2$$==flavours==}
```

This syntax is also possible:

```
My favourite ice-cream flavours are {2$$==flavours==|==flavours==}
```

but the first version will guarantee no duplicates.

### Variants in Wildcards

Wildcard values can also contain variants. For example, if your seasons.txt file contains:

```
# seasons.txt
summer
{autumn|fall}
winter
spring
```

The possible outputs are:

* summer is coming
* autumn is coming
* fall is coming
* winter is coming
* spring is coming

### Nested Wildcards

You can use wildcards inside other wildcards. If you have a file called people_of_the_world.txt containing:

```
# people_of_the_world.txt
==asia==
==africa==
==europe==
==north_america==
==south_america==
==australisia==
...
```

And another file called africa.txt containing:

```
# africa.txt
Zimbabwean
Namibian
Basotho
...
```

Then

```
==people_of_the_world==
```

will first select a value in people_of_the_world.txt. If that value is a wildcard, say `==africa==`,
it will then choose a value from within `africa.txt. Using nesting, you can create an sophisticated wildcard hierarchies.

### Resolving Wildcards with Globbing

Globbing allows you to match multiple wildcard files at once.
This can be useful if you have multiple files that contain similar data and want to use values from all of them in your prompts.

For example, if you have two files, colours-cold.txt and colours-warm.txt,
you can use globbing to resolve values from both of these files by using an asterisk `*` as a wildcard.

#### Basic Syntax

To use globbing, simply include an asterisk `*` in your wildcard pattern:

```
==colours*==
```

In this example, any file that starts with colours will be matched. So both colours-cold.txt and colours-warm.txt will be used to resolve values.

#### Example

Suppose you have the following files:

colours-cold.txt:

```
# colours-cold.txt
blue
green
```

colours-warm.txt:

```
# colours-warm.txt:
red
yellow
```

Using the `==colours*==` wildcard will randomly select a value from both files:

```
The colour of my shirt is ==colours*==
```

Possible outputs are:

* The colour of my shirt is blue
* The colour of my shirt is green
* The colour of my shirt is red
* The colour of my shirt is yellow

#### Recursive globbing

You can use two wildcards, e.g. `artists/**` to match any wildcard in the `artists/` hierarchy, no matter how deeply nested.

### File formats

#### Text files

The basic wildcard file is a simple text file with a `.txt` extension. It has one value per line. You can comment out a line with a `#`, e.g.

```
# this is a comment
summer
autumn
winter
spring
```

## Variables

Variables allow you to store and reuse values in your prompts. To set a variable, use the following syntax:

```
${variable_name=value}
```

### Immediate Evaluation

To force the immediate evaluation of a variable's value, add an exclamation mark ! before the value:

```
${season=!==season==}
```

or

```
${season=!{summer|autumn|winter|spring}}
```

You can then use the variable in your prompt:

```
In ${season}, I wear ${season} shirts and ${season} trousers
```

This will generate:

* In summer, I wear summer shirts and summer trousers
* In autumn, I wear autumn shirts and autumn trousers
* In winter, I wear winter shirts and winter trousers
* In spring, I wear spring shirts and spring trousers


### Non-immediate Evaluation

Without the exclamation mark, the wildcard or variant will be evaluated every time it is used:

```
${season===season==} and In ${season}, I wear ${season} shirts and ${season} trousers
```

This may produce:

In summer, I wear winter shirts and autumn trousers. Non-immediate evaluation is useful in the case that don't want to define a complex construct multiple times in the same prompt. e.g.

Instead of:

```
A {blond|redhead|brunette}, {green|blue|brown|hazel} eyes, {tall|average|short} man and a {blond|redhead|brunette}, {green|blue|brown|hazel} eyes, {tall|average|short} woman
```

you can use a variable instead:

```
${person_description={blond|redhead|brunette}, {green|blue|brown|hazel} eyes, {tall|average|short}}
A ${person_description} man and a ${person_description} woman
```

## Parameterized Templates

You can pass values into wildcard templates to create more dynamic prompts.

### Basic Syntax

Create a file called `season_clothes.txt` with the following content:

```
In ${season}, I wear ${season} shirts and ${season} trousers
```

Then, in your prompt, you can pass a value to the template:

```
==season_clothes(season=winter)==
```

Note - for now you can only pass a literal string into the template rather than an expression. This syntax will also work

```
${season={summer|autumn|winter|spring} ==season_clothes==
```

### Default values

A template will throw an error if it expects a variable that isn't set. To prevent this from happening you can set a default value.

```
In ${season:summer}, I wear ${season:summer} shirts and ${season:summer} trousers
```

Now if you forget to create the season variable, the prompt will be `In summer, I wear summer shirts and summer trousers`

## Whitespace and comments

As your prompts become more complex, the become harder to read. To prevent creating unreadable and unmaintainable prompts you can use whitespace such as newlines, which will be ignored by the parser. Python-style comments are also supported so that you can annotate your prompt.

```
    # Set the season variable
    ${season={
        summer
	|autumn
	# | fall    # commented this value out
	|winter
	|spring
    }

   In ${season:summer}, I wear ${season:summer} shirts and ${season:summer} trousers
```

Note that regular wildcard .txt files use a newline to represent new wildcard and so whitespace is not permitted.
