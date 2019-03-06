# BiteImpact


## Description

### Sequence of steps

The following steps are executed:
* ask the `impactFilter` if an impact should happen
* filter host trees (if provided)
* run the impact (either kill all trees, or run the `onImpact` handler)

### Specifying the impact
The impact item supports different impact targets (i.e. which plants/part of plants to affect), and different impact types (i.e. which part of the population to affect). The impact is a combination of impact target and impact type, and multiple impacts can be combined.

#### Impact targets (`target`):

* `tree`: Trees (>4m height) are affected (mortality)
* `foliage`: The foliage of trees (>4m) is reduced by the agent
* `roots`: The root biomass is reduced (trees >4m) (not impl)
* `sapling`: Sapling cohorts (trees <4m height) are affected (i.e. mortality)
* `browsing`: The height growth of saplings (<4m) is set to 0 for the year of impact

#### Impact types
* `all`: All trees/biomass/cohorts of host plants (see `hostFilter`)
* `fractionPerTree`: for biomass compartments (foliage, roots) a given fraction is removed (from all trees)
* `fraction`: Only a fraction of the trees/saplings are affected; for biomass compartments all biomass is removed from a given fraction of the trees (see also `order`)

#### Declaring impacts
The total impact is specified as an array of impacts:

```
...
impact: [ { impact 1}, {impact 2}, ... ],
...
```

An single impact is specified as a Javascript object with the following elements:
* `target`: The impact target as a string (e.g. `tree`, see list above)
* `type`: The impact type as a string (e.g. `fractionPerTree`, see list above)
* `value`: The fraction or fractionPerTree for the respective impact types. Not necessary for impact type `all`.
* `order`: for `fraction` impacts, order specifies the order in which the trees are processed. `order` is an Expression (provided as a string). For example, if `order` is '-dbh', then the biggest trees would be first in the list, and thus affected first. If omitted, the trees are selected randomly.

Note that the impact items are processed in the given sequence.

Example:
```
impact: [ {target: 'tree', type: 'fraction', value: 0.1}, // 10 % of all trees die (selected randomly)
          {target: 'browsing', type: 'all'}, // all saplings are browsed within the cell
          {target: 'foliage', type: 'fraction', value: 0.5, order: 'dbh' } ], // remove all foliage from 50% of trees, starting with the smallest
```

Juha: should we also allow to specify how __much__ biomass should be removed in total? (e.g. 100kg foliage)?


## Setup

* ### `hostTrees` (expression)
A filter expression to select those trees on the pixel which are considered as hosts. Note that
`BiteBiomass` includes also this filter. If omitted or empty, no filtering will happen.

* ### `impactFilter` (dynamic expression)
A dynamic expression to determine whether an impact should occur on the cell (context is cell). The expression
should return a boolean value.

* ### `verbose` (boolean)
If `true`, the log contains more details. Mainly for testing purposes. Default: false.

* ### `simulate` (boolean)
If `true` no trees are harmed.

## Properties

*no properties*

## Variables

*no variables*

## Events

* ### `onImpact(cell)` 
The `onImpact` event is responsible to implement the agents impact on a cell and should
return the number of killed trees during the calculation. 


