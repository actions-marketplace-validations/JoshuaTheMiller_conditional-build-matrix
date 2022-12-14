# Conditional Build Matrix

[![CodeFactor](https://www.codefactor.io/repository/github/joshuathemiller/conditional-build-matrix/badge)](https://www.codefactor.io/repository/github/joshuathemiller/conditional-build-matrix) ![GitHub](https://img.shields.io/github/license/JoshuaTheMiller/conditional-build-matrix)

Enables easier *conditional* matrix builds!

---


❗ The *filter* input of this Action uses [JMESPath](https://jmespath.org/) as its JSON query language. To experiment with creating a JMESPath query, check out their [interactive examples page](https://jmespath.org/examples.html)!

*Quick Links*

* [Usage](#Usage)
  * [Filtering on Branch Name](#Filtering-on-Branch-Name)
* [Tips](#Tips)
  * [Necessary JSON File](#Necessary-JSON-File)
  * [JSON File Placement](#JSON-File-Placement)
* [Troubleshooting](#Troubleshooting)
* [Use Case](#Use-Case)
* [Inspired By](#Inspired-By)

## Usage

Inputs and Outputs are as follows:

| Input | Default Value | Description |
| - | - | - |
| `inputFile` | `.github/workflows/matrix_includes.json` | The file that contains the JSON configuration of the matrix includes elements. Must come after a Checkout. |
| `filter` | ❗ none, user must supply value! | The [JMESPath JSON](https://jmespath.org/examples.html) query to apply to the input file. |
| `addInclude` | true | Wrap the output from the JSON query in an `{include: ...}` object. |

| Output | Description |
| - | - |
| `matrix` | The filtered "strategy.matrix" statement created from filtering the "inputFile". |

### Sample

```yml
name: Sample Conditional Matrix Build

on: push

jobs:
  matrix_prep:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      # Required as the JSON input file needs to be read
      - uses: actions/checkout@v2      
      - id: set-matrix
        uses: JoshuaTheMiller/conditional-build-matrix@main        
        with:
          # inputFile: '.github/workflows/matrix_includes.json' # Default input file path
          filter: '[?runOnBranch==`${{ github.ref }}` || runOnBranch==`always`]'
          # addInclude: true
  build:
    needs: matrix_prep
    strategy:      
      matrix: ${{fromJson(needs.matrix_prep.outputs.matrix)}}
    name: Build
    runs-on: ${{ matrix.runs_on }}
    steps:
    - name: Echo
      run: echo "${{ matrix.runOn }}"

```

### Filtering on Branch Name

The following workflow sample uses the `nelonoel/branch-name` Action so that the branch name itself can be used in the filter:

```yml
name: Sample Conditional Matrix Build

on: push

jobs:
  matrix_prep:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - uses: actions/checkout@v2
      - uses: nelonoel/branch-name@v1.0.1
      - id: set-matrix
        uses: JoshuaTheMiller/conditional-build-matrix@main        
        with:          
          # The simple branch name can be used in the filter now!
          filter: '[?runOnBranch==`${{ env.BRANCH_NAME }}` || runOnBranch==`always`]'   
  build:
    needs: matrix_prep
    strategy:      
      matrix: ${{fromJson(needs.matrix_prep.outputs.matrix)}}
    name: Build
    runs-on: ${{ matrix.runs_on }}
    steps:
    - name: Echo
      run: echo "${{ matrix.runOn }}"

```

## Tips

### Necessary JSON File

This Action requires a JSON file that contains the information you would normally include in a `strategy.matrix.include` block (example is [included below](#sample-json-file)). By default, the action will look for a file in your `.github/workflows/` folder named `matrix_includes.json`. If you want to name the file differently, or place it in a different folder, set the path via the `inputFile` input.

If you need to define the entire `strategy.matrix` value, you will need to set `addInclude` to `false`. Please note that you must then ensure your **input file** and **JSON query** provide the results your workflow needs.

#### Sample JSON File

The following is the sample file (`matrix_includes.json`) that was used in the workflow above.

```json
[
    {
        "runs_on":"ubuntu-latest",        
        "runOnBranch":"always"
    },
    {
        "runs_on":"windows-latest",        
        "runOnBranch":"main"
    }
]
```

### JSON File Placement

For a cleaner repository, I recommend placing the `matrix_includes.json` file in your `.github/workflows/` folder. Doing so will also help communicate the purpose of the file. You do not have to follow this advice, of course, as this Action allows the location of the file to be passed via the `inputFile` input.

## Troubleshooting

This Action requires an output to be passed between jobs. For this Action to run properly, make sure you have defined the output of the `matrix_prep` job appropriately (or whatever you have named it), and that the other jobs have their `needs` block configured to point to the `matrix_prep` job (if they need to use the custom matrix).

## Use Case 

If you have two workflows where the only difference is the matrix elements, you may want to consider using a single workflow file with *conditional* matrix elements (which this Action makes easier):

*Build for main branch*
![Build for main branch](https://i.stack.imgur.com/O95fj.png)
*Build for v2.1-Release branch*
![](https://i.stack.imgur.com/bXFfX.png)

## Inspired By

This action was inspired by a [SO question](https://stackoverflow.com/q/65384420/1542187) on making matrix elements conditional. Thanks [lewis](https://stackoverflow.com/users/6814658/lewis) 😁.
