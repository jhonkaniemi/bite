# BITE Examples

BITE examples is a collection of different biotic agents showcasing different functionalities of BITE 
and how the agent parameters are setup in Javascript. 

## Heterobasidion root rot

## Gypsy moth

European gypsy moth (from hereafter gypsy moth) is a defoliator native to Europe where it causes substantial disturbance especially on oaks (Mcmanus and Csóka, 2007). In 1869 it was also introduced to North America, where it became an invasive pest seriously threatening oak forests in the Northeastern USA (Elkinton and Liebhold, 1990). Adult gypsy moths are poor dispersers, but the first instar larvae spread passively over short distances via wind (Hunter and Elkinton, 2000). However, human-aided long distance dispersal is driving the invasion in North America (Liebhold et al., 1992). The development of a gypsy moth from egg to adult takes one season and during that development, each larvae consume about 3–4 g of foliage (Sharov and Colbert, 1996). Outbreaks of gypsy moth typically last for several years and occur both in Europe and North America in 8 – 12 year intervals (Johnson et al., 2005). Gypsy moth dispersal was approximated with a Gaussian dispersal kernel in BITE (Elderd et al., 2013), and its population dynamics was simulated with a logistic growth equation based on growth rates modified from Lustig et al. (2017). The simulated impact was consumption of foliage biomass, with preference for small over tall trees.   

Gypsy moth uses five different core modules of BITE: 1) Introduction, 2) Dispersal, 3) Colonization, 4) Population dynamics and 5) Impact. First, the details of gypsy moth life cycle  are parametrized. Gypsy moth serves as an example for an agent with periodic outbreaks. Here, the outbreak cyclicity is introduced arbitrarily with parameters, but could also be achieved with using specific functions in `BiteBiomass` describing the population dynamics. 

Introduction and dispersal are simulated with BiteDispersal item, where the agent is introduced into the landscape before the first dispersal during the first simulation year with `onBeforeSpread` using a helpful `randomSpread` function. This function can be used to introduce the agent on random points at any given time during the simulation. Dispersal here is defined to exclude human transport of gypsy moth and thus the `kernel` is simple Gaussian kernel with `maxDistance` of 50 meters. Gypsy moth can coloniza a cell when the temperature sum (GDD10) for the cell is over 500 degree days and the host species, in this case Quercus robur is present. Population dynamics are simulated with the default logistic growth equation with modifications to increase the population growth rate during outbreaks and slow it down after. The vegetation impact is simulated start from the shortest trees and continue from tree to tree until the requirement for annual consumption of foliage is met. 

```
var GypsyMoth = new BiteAgent({
	name: "Gypsy Moth", description: "defoliator", 
	cellSize: 10,
	climateVariables: ['GDD10'], 
	lifecycle: new BiteLifeCycle({ voltinism: 1, // max 1 generation per year, growth rate adjusts the true population growth
				dieAfterDispersal: false, // cell does not die even with dispersal
				spreadFilter: 'yearsLiving>0', // condition that needs to be met before spread 
				spreadDelay: 1, // number of years after colonization
				spreadInterval: 1,   // min. frequency of spread (1/years)
				outbreakStart: 'rnd(8,12)', //random number of years for outbreak interval, calculated after each outbreak
				outbreakDuration: 'rnd(2,4)' //random number for outbreak duration, calculated after each outbreak
		}),
				
	dispersal: new BiteDispersal({
	kernel: '(1/3.141592*21.32637^2)*exp(-(x^2/21.32637^2))', //Gaussian dispersal kernel by Elderd et al (2013)
	debugKernel: 'temp/kerneltest_gypsymoth.asc',  //raster file for debugging the kernel function
	maxDistance: 50,  // maximum spreading distance is 50m
	onBeforeSpread: function(bit) {  if (Globals.year<2) {randomSpread(1, bit.grid); console.log("added 1 px");} }  //function to introduce the agent on first simulation year in a random cell (see below that coordinates are given for landscape centerpoint
		}), 

	colonization: new BiteColonization({ 
		dispersalFilter: 'rnd(0,1) < dispersalGrid',  // stochasticity included in the dispersal and colonization processes with random number generator
		cellFilter: 'GDD10>500',   //cell must meet the criteria of 500 degree days (base temperature 10C)
		treeFilter: 'species=quro', // cell must contain Common oak (Quercus robur) as host
		initialAgentBiomass: 250/100*0.0016 // initial agent biomass is calculated assuming 250 individual per ha are needed for succesful colonization (2.5 in 10m cell) and each moth weighs on average 1.6 grams
	   }),

	growth: new BiteBiomass({
		hostTrees: 'species=quro', // // define the hosttrees as above
		hostBiomass: function(cell) {   // calculate the potential biomass available in the cell for agent consumption
		   cell.reloadSaplings(); // trees are loaded automatically, but saplings need to be reloaded here
		   cell.saplings.filter('species=quro'); // define the sapling hosts
		   var bm_saplings = cell.saplings.sum('nrep*foliagemass'); // calculate the available total biomass for saplings = foliage biomass * number of represented stems per cohort
		   var bm_trees = cell.trees.sum('foliagemass'); // calculate the available total biomass for trees
		   return bm_saplings + bm_trees;   // total sum of host biomass in the cell
		},
		mortality: function(cell){   //external mortality is applied here, antagonists cause additional mortality driving the outbreaks down. Here, the outbreakYears is used to control the mortality
			var oy = cell.value('outbreakYears');  //enter the cell value outbreakYears
			if ( oy == 0) {
			return 0;			//if there is no ongoing outbreak, the additional mortality is 0
			} else {
			var death = (0.95/(1+Math.exp(-(oy-2))));  //during outbreaks, the probability for additional mortality (antagonists) increases exponentially
			return death;
			}
		}, 			
		growthFunction: 'K / (1 + ( (K - M) / M)*exp(-r*t))', // logistic growth function, where K=hostbiomass / consumption; M=agentBiomass; r=relative growth rate coefficient; t=time
		growthRateFunction: function(cell) {   //relative growth rate coefficient, this is assumed to exponentially increase when outbreak starts
			var oy = cell.value('outbreakYears'); //enter the cell value outbreakYears
			if ( oy == 0) {
			return 0;			//if there is no ongoing outbreak, the population is in equilibrium and r=0
			} else {
			var grate = (1.3/(1+Math.exp(-(oy-1))));  //during the outbreak, the growth rate is assumed to grow exponentially to maximum 1.3 fold 
			return grate;
			}
		},
		consumption:  (0.004*520*0.5*0.1)/0.0016, // host consumption by agent unit - assumption: total consumption  by single larvae survivng to adulthood is 4 grams of leaves, the number of eggs is 520 and the survival of larvae through instars is 0.5*0.1=0.05. Each adult is assumed to weigh 1.6 grams. 
		growthIterations: 1  //only one growthiteration
		}),		
		
	impact: new BiteImpact({ 
		impactFilter: 'agentImpact>0',  //impact occurs once the agent has consumed the first biomass units of the host
		impact: [ 			//impact arrays 
		   {target: 'foliage', maxBiomass: 'agentImpact', order: "height"} // foliage of the host trees is consumed tree by tree from smallest to largest until the annual total consumption is reached
		]
	}),
	
	output: new BiteOutput({
		outputFilter: "active=true",
		tableName: 'BiteTabx',
		columns: ['yearsLiving', 'hostBiomass', 'agentImpact','agentBiomass']
		
	}),
	onYearEnd: function(agent) { 
			agent.saveGrid('yearsLiving', 'temp/gypsymoth_yliv.asc');
		agent.saveGrid('cumYearsLiving', 'temp/gypsymoth_cyliv.asc');
		agent.saveGrid('active', 'temp/gypsymoth_active.asc'); }

});

function randomSpread(n, gr) {
	for (var i=0;i<n;++i) {
		var x = Math.random()*gr.width;
		var y = Math.random()*gr.height;
    gr.setValue(250,250,1);
	}
}
```

##Roe deer

##Ash dieback

##Asian long-horned beetle

##Mastodon
