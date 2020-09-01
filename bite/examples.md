# BITE Examples

BITE examples is a collection of different biotic agents showcasing different functionalities of BITE 
and how the agent parameters are setup in Javascript. 

## Gypsy moth

Gypsy moth

```
// create an agent:
var my_agent = new BiteAgent( agent_def );
// access the agent in Javascript:
Bite.log(agent.name);
```



See below for a description of global agent options and of Bite-Items; events are desribed at the end
of this page.


## global options
* ### `name` (string)
The name of the agent which has to be unique. The name is relevant for outputs and visualization.

* ### `description` (string)
The `description`  is a free-text element that can be used to describe the agent, or to include some
kind of versioning information.

* ### `cellSize` (integer)
The `cellSize` defines the grain at which the agent operates. Possible values are 10, 20, 50, or 100m. 
Note that multiple agents can coexist that use a different resolution.

* ### `climateVariables` (array of string)
`climateVariables` is a Javascript array that contains a list of climate variable names which can be used by the agent.
The elements of the array need to be one of the provided climate variabes (see below). Variables that are 'registered'
in this way can be subsequently used in expressions and accessed via Javascript.


Variables | Description
----------| -----------
MAT | mean annual temperature (°C)
MAP | annual precipitation sum (mm)
TMonthX | with `X` in 1..12, mean temperature of the month 1..12 (e.g. `TMonth6` for mean temp of June)
PMonthX | with `X` in 1..12, monthly precipitation sum of the month 1..12 (mm) (e.g. `PMonth6` for precipipation of June)
GDD | growing degree days (temperature sum of (t_mean-threshold) for all days with t_mean > threshold, with threshold=5°)
GDD10 | growing degree days (temperature sum of (t_mean-threshold) for all days with t_mean > threshold, with threshold=10°)
