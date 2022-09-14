---
layout: post
title: How to split strings using regex
date: 2022-09-13 20:30 -0300
categories:
- Pattern matching
tags:
- regex
- python
---

## Problem Statement

What if I ask you to split a paragraph in phrases delimited by ```.?!``` ?

What would be your first approach ?

Also, I want you to include the phrase itself and it's delimiters (```.?!```) in the matches. So, you can't simply call a ```split``` function like [Python's one](https://docs.python.org/3/library/stdtypes.html?highlight=split#str.split), because you're going to lose delimiters' information, once you'll get a list of strings that preceded one of the specified delimiters, and not the whole information (phrase + delimiters).

For those of you who doesn't know how to manipulate regex in Python, check it out [their documentation](https://docs.python.org/3/library/re.html). But keep reading this post to see how I use it to solve the problem.

Next, is an example of input and output as well.

### Input

(Example generated with ```$ lorem-ipsum-generator -s 50 -p 2``` command, with small modifications to add all ```.?!``` delimiters)

```
Laoreet. Etiam mi eu porta ac sit ve a mollis. Ad sollicitudin congue eu, sapien dolor adipiscing pretium nam. 
A, pellentesque cras, vivamus consequat lectus maecenas hymenaeos eros scelerisque. Urna at proin class a, 
ornare amet. Primis maecenas parturient cursus ultrices penatibus consectetuer vestibulum amet sapien aptent dis. 
Posuere eu, nisi natoque quis, ut duis mauris ad mi dis. Justo mi vitae proin vel enim id nisl quam parturient est.
Rhoncus, massa ac hac hendrerit primis aenean, leo. Lorem. Platea. Risus.

Luctus in, tortor quis et quam inceptos nulla quis et. Ante consectetuer ad ipsum in consequat felis tristique sed, 
felis. Magnis placerat hendrerit purus eu tempus ve! Ipsum in augue elit feugiat egestas erat lacus lacinia leo. 
Risus erat vivamus in, tortor at, ad augue aenean in. Vivamus nam adipiscing parturient et malesuada a lacus et 
porttitor velit. Tempor facilisis purus dictum cubilia dapibus libero. Porta ullamcorper ultrices dignissim ut 
semper. Felis tincidunt molestie duis penatibus montes maecenas ridiculus id, mauris ? Suspendisse nam. 
Adipiscing ad, erat eget in velit magnis dolor. Fusce consequat elementum eget nam cras integer iaculis consequat 
suscipit nulla vehicula nam etiam risus. Tristique parturient nec vestibulum parturient id mattis sociosqu 
ullamcorper maecenas, facilisis. Sed lacus conubia taciti. Erat ultrices mi, ipsum congue varius nam volutpat. 
Eu hymenaeos adipiscing, nec vitae tempus parturient curae duis sem mi. Vestibulum vel vestibulum tincidunt, 
porta ! Neque. Eros magnis ve dictumst.
```

### Output

(List of matches)

```
'Laoreet.'
' Etiam mi eu porta ac sit ve a mollis.'
' Ad sollicitudin congue eu, sapien dolor adipiscing pretium nam.'
' A, pellentesque cras, vivamus consequat lectus maecenas hymenaeos eros scelerisque.'
' Urna at proin class a, ornare amet.'
' Primis maecenas parturient cursus ultrices penatibus consectetuer vestibulum amet sapien aptent dis.'
' Posuere eu, nisi natoque quis, ut duis mauris ad mi dis.'
' Justo mi vitae proin vel enim id nisl quam parturient est.'
'Rhoncus, massa ac hac hendrerit primis aenean, leo.'
' Lorem.'
' Platea.'
' Risus.'
'Luctus in, tortor quis et quam inceptos nulla quis et.'
' Ante consectetuer ad ipsum in consequat felis tristique sed, felis.'
' Magnis placerat hendrerit purus eu tempus ve!'
' Ipsum in augue elit feugiat egestas erat lacus lacinia leo.'
' Risus erat vivamus in, tortor at, ad augue aenean in.'
' Vivamus nam adipiscing parturient et malesuada a lacus et porttitor velit.'
' Tempor facilisis purus dictum cubilia dapibus libero.'
' Porta ullamcorper ultrices dignissim ut semper.'
' Felis tincidunt molestie duis penatibus montes maecenas ridiculus id, mauris ?'
' Suspendisse nam.'
' Adipiscing ad, erat eget in velit magnis dolor.'
' Fusce consequat elementum eget nam cras integer iaculis consequat suscipit nulla vehicula nam etiam risus.'
' Tristique parturient nec vestibulum parturient id mattis sociosqu ullamcorper maecenas, facilisis.'
' Sed lacus conubia taciti.'
' Erat ultrices mi, ipsum congue varius nam volutpat.'
' Eu hymenaeos adipiscing, nec vitae tempus parturient curae duis sem mi.'
' Vestibulum vel vestibulum tincidunt, porta !'
' Neque.'
' Eros magnis ve dictumst.'
```


## Solution

For solving this problem, I'm going to use Python's **re** module to find all matches inside the input string. Also, I'm going to generate a file with dummy text to test my built pattern, using [**lorem-ipsum-generator**](https://code.google.com/archive/p/lorem-ipsum-generator/).

- Generating dummy text:

For the sake of simplicity I'm going to generate 2 paragraphs with a total of 50 sentences:

```$ lorem-ipsum-generator -s 50 -p 2 > test.txt```

- Building the pattern:

First, I wanna match phrases that doesn't end with any of the delimiters (```.?!```). So, I came up with the following pattern:

``` [^.!?]+ ```

If this is correct, I'll probably get something like:

(The following output ignores **'\n'** matches)

```
'Laoreet'
' Etiam mi eu porta ac sit ve a mollis'
' Ad sollicitudin congue eu, sapien dolor adipiscing pretium nam'
' A, pellentesque cras, vivamus consequat lectus maecenas hymenaeos eros scelerisque'
' Urna at proin class a, ornare amet'
' Primis maecenas parturient cursus ultrices penatibus consectetuer vestibulum amet sapien aptent dis'
' Posuere eu, nisi natoque quis, ut duis mauris ad mi dis'
' Justo mi vitae proin vel enim id nisl quam parturient est'
'Rhoncus, massa ac hac hendrerit primis aenean, leo'
' Lorem'
' Platea'
' Risus'
'Luctus in, tortor quis et quam inceptos nulla quis et'
' Ante consectetuer ad ipsum in consequat felis tristique sed, felis'
' Magnis placerat hendrerit purus eu tempus ve'
' Ipsum in augue elit feugiat egestas erat lacus lacinia leo'
' Risus erat vivamus in, tortor at, ad augue aenean in'
' Vivamus nam adipiscing parturient et malesuada a lacus et porttitor velit'
' Tempor facilisis purus dictum cubilia dapibus libero'
' Porta ullamcorper ultrices dignissim ut semper'
' Felis tincidunt molestie duis penatibus montes maecenas ridiculus id, mauris '
' Suspendisse nam'
' Adipiscing ad, erat eget in velit magnis dolor'
' Fusce consequat elementum eget nam cras integer iaculis consequat suscipit nulla vehicula nam etiam risus'
' Tristique parturient nec vestibulum parturient id mattis sociosqu ullamcorper maecenas, facilisis'
' Sed lacus conubia taciti'
' Erat ultrices mi, ipsum congue varius nam volutpat'
' Eu hymenaeos adipiscing, nec vitae tempus parturient curae duis sem mi'
' Vestibulum vel vestibulum tincidunt, porta '
' Neque'
' Eros magnis ve dictumst'
```

Wait, but this is almost the same thing as running **split** method onto my input string. Note that I've lost delimiters' information too...

To fix this, I'm going to include delimiters in my pattern by adding the ```[.!?]``` sub-pattern into the end of my first one:

``` [^.!?]+[.!?] ```

Then, the output is:

```
'Laoreet.'
' Etiam mi eu porta ac sit ve a mollis.'
' Ad sollicitudin congue eu, sapien dolor adipiscing pretium nam.'
' A, pellentesque cras, vivamus consequat lectus maecenas hymenaeos eros scelerisque.'
' Urna at proin class a, ornare amet.'
' Primis maecenas parturient cursus ultrices penatibus consectetuer vestibulum amet sapien aptent dis.'
' Posuere eu, nisi natoque quis, ut duis mauris ad mi dis.'
' Justo mi vitae proin vel enim id nisl quam parturient est.'
'Rhoncus, massa ac hac hendrerit primis aenean, leo.'
' Lorem.'
' Platea.'
' Risus.'andat lacus lacinia leo.'
' Risus erat vivamus in, tortor at, ad augue aenean in.'
' Vivamus nam adipiscing parturient et malesuada a lacus et porttitor velit.'
' Tempor facilisis purus dictum cubilia dapibus libero.'
' Porta ullamcorper ultrices dignissim ut semper.'
' Felis tincidunt molestie duis penatibus montes maecenas ridiculus id, mauris ?'
' Suspendisse nam.'
' Adipiscing ad, erat eget in velit magnis dolor.'
' Fusce consequat elementum eget nam cras integer iaculis consequat suscipit nulla vehicula nam etiam risus.'
' Tristique parturient nec vestibulum parturient id mattis sociosqu ullamcorper maecenas, facilisis.'
' Sed lacus conubia taciti.'
' Erat ultrices mi, ipsum congue varius nam volutpat.'
' Eu hymenaeos adipiscing, nec vitae tempus parturient curae duis sem mi.'
' Vestibulum vel vestibulum tincidunt, porta !'
' Neque.'
' Eros magnis ve dictumst.'
```

### Python Code

Given the final pattern, I search my input string (lines of _test.txt_ file) for all matches using the ```re.findall(...)``` [function](https://docs.python.org/3/library/re.html#re.findall).

```python
import re

# Finding all pattern's matches
matches = list()
with open('test.txt', 'r') as f:
    for line in f:
        matches += list(re.findall(r'[^.!?]+[.!?]', line))

# Printing listed matches
for x in matches:
	print(f'\'{x}\'')
```
