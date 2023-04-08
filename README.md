# parsed_to_plot


## Overview

This software plots constituency tress and dependency trees given in strings, using both the id-tree crate
[id-tree](https://crates.io/crates/id_tree) and [plotters](https://crates.io/crates/plotters).
The program was written with linguistic syntax in mind, but can work on any input (such as mathematical expressions etc).

* The API expects a string input. Multiple inputs can also be delivered via the command-line in a file.
* For constituency trees, the program takes one-liners, parsed strings. These strings can be syntactic, for example
such that represent phrases and part-of-speech (like the structure of [Berkeley Neural Parser](https://pypi.org/project/benepar/)
in python). Such strings will have "double leaves" (see an example below). Alternatively, the strings can be regular,
representing for example mathematical expressions.
* For dependency trees, the programs takes a conll format, in which every token has 10 fields, separated by tab, and
presented in a new line. Sentences are separated by an empty line. (see an example below using the output of
[spaCy](https://spacy.io/) in python).
* For multiple inputs, the program will expect 3 arguments from the command line : input type (constituency or dependency),
input file path, output path. The inputs will be of the same type. See an example below.
* The API first transforms the input to an internal conll / tree, then plots the structure with recursion. It is mostly
suitable for short sentences of up to 15-20 tokens.



## Examples
### Constituency

```rust

// This is an example for a simple constituency tree of the sentence:
// The people watch the game
// (S (NP (det The) (N people)) (VP (V watch) (NP (det the) (N game))))

use parsed_to_plot::String2Tree;
use parsed_to_plot::Tree2Plot;
use parsed_to_plot::String2StructureBuilder;
use parsed_to_plot::Structure2PlotBuilder;


let mut constituency = String::from("(S (NP (det The) (N people)) (VP (V watch) (NP (det the) (N game))))");
let mut string2tree: String2Tree = String2StructureBuilder::new();
string2tree.build(&mut constituency).unwrap(); // build the tree from the string
let tree = string2tree.get_structure();

// build plot from tree and save
let save_to: &str = "constituency_plot.png";
let mut tree2plot: Tree2Plot = Structure2PlotBuilder::new(tree);
tree2plot.build(save_to);

```

### Dependency

```rust

// This is an example for a simple dependency tree of the sentence:
// The people watch the game
// 0   The the det _   _   1   det   _   _
// 1	people	people	NOUN	_	_	2	nsubj	_	_
// 2	watch	watch	VERB	_	_	2	ROOT	_	_
// 3	the	the	DET	_	_	4	det	_	_
// 4	game	game	NOUN	_	_	2	dobj	_	_

use parsed_to_plot::String2Conll;
use parsed_to_plot::Conll2Plot;
use parsed_to_plot::String2StructureBuilder;
use parsed_to_plot::Structure2PlotBuilder;

let mut dependency = [
    "0	The	the	DET	_	_	1	det	_	_",
    "1	people	people	NOUN	_	_	2	nsubj	_	_",
    "2	watch	watch	VERB	_	_	2	ROOT	_	_",
    "3	the	the	DET	_	_	4	det	_	_",
    "4	game	game	NOUN	_	_	2	dobj	_	_"
].map(|x| x.to_string()).to_vec();

let mut conll2tree: String2Conll = String2StructureBuilder::new();
conll2tree.build(&mut dependency).unwrap(); // build the conll from the vector of strings
let tree = conll2tree.get_structure();

// build plot from tree and save
let save_to: &str = "dependency_plot.png";
let mut conll2plot: Conll2Plot = Structure2PlotBuilder::new(tree);
conll2plot.build(save_to);

```


### Multiple inputs via file


// You can use multiple inputs of the same type, given in a file through the command line.
// cargo run INPUT_TYPE INPUT_FILE OUTPUT_PATH
// when:
// INPUT_TYPE should be replaced with "c" for constituency or "d" for dependency.
// INPUT_FILE should be replaced with a path to a txt file with inputs.
// OUTPUT_PATH should be replaced with a path to a requested output location.

// for example: cargo run c constituencies.txt Output
// It will saves png images of constituency trees drawn for the inputs in constituencies.txt, in an Output dir location.


#### Constituency

```rust

use parsed_to_plot::Config;
use parsed_to_plot::String2Tree;
use parsed_to_plot::Tree2Plot;
use parsed_to_plot::String2StructureBuilder;
use parsed_to_plot::Structure2PlotBuilder;
use std::env;

// collect arguments from command line
let args: Vec<String> = env::args().collect();
// note: your command line args should translate to something like the following:
// let args: Vec<String> = ["PROGRAM_NAME", "c", "Input/constituencies.txt", "ConOutput"].map(|x| x.to_string()).to_vec();

// run configuration protocol and inpectations
let sequences = match Config::new(&args) {
    Ok(sequences) => Vec::<String>::try_from(sequences).unwrap(),
    Err(config) => panic!("{}", config)
};

for (i, mut constituency) in sequences.into_iter().enumerate() {

    println!("working on input number {} ...", i);
    let save_to = &Config::get_out_file(&args[3], i.to_string().as_str());

    // build tree from consituency, and also do a DFS to build n_sub_children from trait
    let mut string2tree: String2Tree = String2StructureBuilder::new();
    string2tree.build(&mut constituency).unwrap();
    let tree = string2tree.get_structure();

    // build plot from tree
    let mut tree2plot: Tree2Plot = Structure2PlotBuilder::new(tree);
    tree2plot.build(save_to);

}

```

#### Dependency

```rust

use parsed_to_plot::Config;
use parsed_to_plot::String2Conll;
use parsed_to_plot::Conll2Plot;
use parsed_to_plot::String2StructureBuilder;
use parsed_to_plot::Structure2PlotBuilder;
use std::env;

// collect arguments from command line
let args: Vec<String> = env::args().collect();
// note: your command line args should translate to something like the following:
// let args: Vec<String> = ["PROGRAM_NAME", "d", "Input/conll.txt", "DepOutput"].map(|x| x.to_string()).to_vec();

// run configuration protocol and inpectations
let sequences = match Config::new(&args) {
    Ok(sequences) => Vec::<Vec<String>>::try_from(sequences).unwrap(),
    Err(config) => panic!("{}", config)
};

for (i, mut dependency) in sequences.into_iter().enumerate() {

    println!("working on input number {} ...", i);
    let save_to = &Config::get_out_file(&args[3], i.to_string().as_str());

    // build conll from string
    let mut string2conll: String2Conll = String2StructureBuilder::new();
    string2conll.build(&mut dependency).unwrap();
    let conll = string2conll.get_structure();

    // build plot from conll
    let mut conll2plot: Conll2Plot = Structure2PlotBuilder::new(conll);
    conll2plot.build(save_to);

}

```
