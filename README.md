> Ideas for future versions:
>* React prerendering
>* Custom prerendering

# front2back
A simple program for converting and templating a frontend index.html file into a backend file (like php or whatever).

This program was originally created to be used with [create-react-app](https://github.com/facebookincubator/create-react-app) and [codeigniter](https://www.codeigniter.com/) - Usage example is a bit less pendy than the name of this repo.

## Prerequisites
1. [Node js](https://nodejs.org/) updated to latest version (needs to support es6).
2. An `index.html` file
3. Support for [rsync.js](https://www.npmjs.com/package/rsync)
  * If you are on a windows 10 PC, then I recommend getting the Windows subsystem for Linux:
    * [what's that?](https://msdn.microsoft.com/en-us/commandline/wsl/about)
    * [where do I get it?](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10)
    
## Getting started
### Installation
`npm install front-to-back -g` 

### Usage
In the root of your project (where `package.json` is), run `f2b` from the command line.

The default behaviour may not be the most desirable so the program allows a `project_src` to be declared in 3 ways:
1. As an additional argument when `f2b` is executed (e.g. `f2b ./some_file.json`)
2. As a field in `package.json` - should be called `"project_src"` and can be either a string or an object:
    * If `project_src` is a string - it should be a path to a file containing a `blueprint` object.
    * If `project_src` is an object - it should follow the blueprint layout to be useful.


### Example (Will work even without installing this package globally)
In a terminal type the following:
1. `git clone https://github.com/suli-g/front-to-back.git`
2. `cd front-to-back`
3. If you've installed `front-to-back` globally:
    * `f2b`
3. If you haven't installed `front-to-back` globally:
    * `node index.js`
4. Two new folders should have been generated:
    * `./output` <-- contains all files that will be deployed (perhaps a future version should contain only the index output file)
    * `./example` <-- This is how it would look when deployed.

### `Project_src` Layout

|Field|Type|Default
|:---|:---:|---:|
| assets| string |"{outlet}/assets"
| index_file| string |"index.php"|
| outlet| string |"."
| output | string |`"./output/"`
| src | string |"./build/"
| views| string |"{outlet}/views"
| template_input| array | `[["__","[a-zA-Z]\\w*"]]` |
| template_output| array | `[["<?=$", "?>"]]` |

#### assets
The folder containing your assets - this would contain everything except your `index_file`.

#### index_file
The name of the file output when the `node deploy` command is run (by default it is assumed to be a php file, however because the extension is required too - any backend should theoretically work).

#### outlet
The name of the root directory to which your app will be deployed.

#### output
The name of the folder to which everything will be moved **before** deploying the app (could be considered a deployment as well, since everything but the `index_file` will be the same as the `src` directory.

#### src
This is the source of your `index.html` - in the case of `create-react-app`, this would be `./build/` (the default).

#### views
The folder containing your views - this would contain your `index_file`.

#### template_input
An array of Regular expression strings to which the content of `index.html` will be matched. (see Template rules below for more information).

#### template_output
An array of replacements to matches found on comparing `index.html` with `template_input` (see Template rules below for more information).

### Template rules:
`template_input` and `template_output`:
    * should have the same length.
    * should be an array of arrays or strings

###  `template_input`:
Each entry in the `template_input` array should have an ***output*** matching `^[^()\s]+\([^()]+\)[^()\s]+$` (like `abc(sdasdasd)abc`)
* If you define a `template_input` field without a `template_output` field - then `template_input` must have a maximum length of 1 (to use default `template_output`).

Children of the `template_input` array are treated in 4 distinct ways depending upon their length:

| length | behaviour | input | output |
|:---:|---|---|---|
| 1 | treat entire entry as desired regular expression | `['__(\w*)__']` | `/__(\w*)__/g` |
| 2 | treat (0) as capture group and (1) as both template tags. | `['__', '\w*']` | `/__(\w*)__/g` |
| 3 | treat (1) as the capture group, then (0) and (2) as template tags | `['__', '\w*', _n]` | `/__(\w*)_n/g` |
| (n, k; n > 3, 0 < k < n-1) | `rule[n]` | `['_a_', ...g, _b_]` | `/_a_(...g)_b_/g` |
| `rule [0]` | treat as opening tag | `*` | `_a_` |
| `rule [k]` | treat as closing tag | `*` | `_b_` |
| `rule [n-1]` | treat as pipe separated capture group | `*` | `(g[0]\|g[1]...\|g[n-1])` |

--

#### `template_output`
The table below illustrates the behaviour of of the templater regarding the type and length of an entry in `template_output` :

|type|length|behaviour|
|:--|:---|---|
|string|1*|replace entire match|
|number|1*|replace entire match|
|array|1|treat as matching tags - use `input match` as text|
|array|2|treat as distinct tags - use `input match` as text|
|array|3|treat as distinct tags - use (1) as text|
|array|>4|throw error|

* Notice that the capture group (ex. `(stuff)`) is absolutely necessary.


####  `template_input` error handling:
The program runs through 5 checks for every `input_template` and throws an exception accordingly when a check fails.

# The 5 Checks:
1. Template tags
    * non-parenthesis match at start of regex
    * non-parenthesis match at end of regex
2. Capture group
    * left parenthesis
    * stuff between parentheses
    * right parenthesis


* Since these are sort of core checks, They haven't been made modifiable except in the source code.

Exceptions for templates use the following format:
> 
```bash 
(node:12345) Warning: template error
index1: regex1 | error1
index2: regex2 | error1
```


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
