# methane_models

This repository has two R scripts with models to let you compare of methane and CO2 emission.

## slugger.R

The _slugger_ model is similar to the _Slugulator_ model at the University of Chicago site (<http://climatemodels.uchicago.edu/slugulator>).
It starts up and runs until the methane and CO2 concentrations reach steady states. Then at year 0, it emits slugs of methane and CO2,
where you can specify the amount and the duration of each slug (if you just want to emit one gas or the other, just set the slug size
for the other gas to zero).

To run this model with a 50 billion ton slug of methane, released over 20 years, and a 30 billion ton slug of CO2, emitted over 10 years,
you would do this:
```
# Load the script
source("slugger.R")

# Run the model. It may take a several minutes to run.
slug_data = calc_slugs(ch4_spike_gton = 50, ch4_spike_duration = 20, 
                       co2_spike_gton = 30, co2_spike_duration = 10)
                       
```
By default, the model has a 100-year spinup time and runs for 10,000 years after the _beginning_ of the slugs.
If you want to change the settings, you can. The full function specification is
```
calc_slugs(ch4_spike_gton = 50.0, ch4_spike_duration = 20.,
           co2_spike_gton = 50.0, co2_spike_duration = 20.,
           climate_sensitivity = 3.0,
           spinup_years = 100, sim_duration = 1E4)
```
This shows you the default values for any parameters you don't specify, but you can change any of these when you call the function.
For instance, to run with a 100 Gton spike of methane over 100 years, no CO2 spike, and let the model run for 100,000 years after 
the start of the spike, you would do
```
slug_data = calc_slugs(ch4_spike_gton = 100, ch4_spike_duration = 100, 
                       co2_spike_gton = 0, sim_duration = 1E5)
```
The function returns a tibble with the following columns:
* `year`: The year of the simulation. By default, the simulation starts at year -100, and starts to release the slug at year 0, 
  and then contunues until the year equals `sim_duration` (default 10,000 years)
* `ch4_gton`: The total amount of methane in the atmosphere, in billions of tons.
* `pco2`: The amount of CO2 in the atmosphere, in parts per million.
* `ch4_src`: The total methane emissions (natural emissions plus the slug), in gigatons per year.
* `ch4_sink`: The amount of methane (natural and anthropogenic) removed from the atmosphere in gigatons per year.
  When methane is removed from the atmosphere, it turns into CO2.
* `co2_src`: The excess sources of CO2 into the atmosphere, apart from the normal natural emissions. This includes both direct human
  emissions and also conversion of anthropogenic methane emissions into CO2.
* `co2_sink`: The amount of anthropogenic CO2 removed from the atmosphere in gigatons per year.
* `ch4_forcing`: The change in i_out due to changes in the methane concentration. In Watts per square meter.
* `co2_forcing`: The change in i_out due to changes in the CO2 concentration. In Watts per square meter.
* `ch4_lifetime`: The lifetime of atmospheric methane, in years. The lifetime of methane depends on many things, such as the
  amount of various other chemicals in the atmosphere, so as you add more methane, it depletes those chemicals, so the 
  lifetime increases.
* `warm_atm_co2`: The warming (in degrees Celsius) of the atmosphere due to CO2 forcing.
* `warm_atm_ch4`: The warming (in degrees Celsius) of the atmosphere due to methane forcing.
* `warm_ocn_co2`: The warming (in degrees Celsius) of the ocean due to CO2 forcing. 
  The ocean warms much more slowly than the atmosphere.
* `warm_ocn_ch4`: The warming (in degrees Celsius) of the ocean due to methane forcing. 
  The ocean warms much more slowly than the atmosphere.
  
## methane.R

This is adapted from the "Methane in the Atmosphere" model, <http://climatemodels.uchicago.edu/methane/>. This model has three time periods: 
* An initial spin-up period with only natural methane emission. 
* Then an anthropogenic period in which the methane emissions increases due to human activity and carbon dioxide emissions grow at 
  steady pace.
* And finally, a methane spike of variable size and duration (e.g., 20 gigatons, released over 10 years).

You run the model with the `calc_methane` function:
```
# Load the script
source("methane.R")

# Run the model.
methane_data = calc_methane(anthro_rate = 0.2, spike_gton = 20, spike_duration = 10)
```
As with the `slugger` model, you can also specify other parameters. The full specification and default values are:
```
calc_methane(natural_rate = 0.26, anthro_rate = 0.2,
             spike_gton = 50.0, spike_duration = 20.,
             spinup_years = 50, pre_spike_years = 50,
             sim_duration = 100)
```
Parameters: 
* `natural_rate` and `anthro_rate`: Rates (Gton/year) of natural and human methane emissions, respectively.
* `spike_gton` and `spike_duration`: Amount (Gton) and duration (years) of methane spike
* `spinup_years`, `pre_spike_years`, `sim_duration`: The model runs for `spinup_years` years with only natural emissions,
  in order to stabilize. Then it runs for `pre_spike_years` years with natural and anthropogenic emissions of methane.
  In addition, CO2 is emitted with a growth rate of 0.65% per year. All of this leads up to year zero, when a spike
  of methane is released. The simulation proceeds for `sim_duration` years after the beginning of the spike.
  Years are numbered so they are negative during spinup and the pre-spike periods and positive after the spike begins.

The model returns a tibble with the following columns:
* `year`: The year of the simulation. By default, the simulation starts at year -100, and starts to release the slug at year 0, 
  and then contunues until the year equals `sim_duration` (default 10,000 years)
* `ch4_gton`: The total amount of methane in the atmosphere, in billions of tons.
* `pco2`: The amount of CO2 in the atmosphere, in parts per million.
* `ch4_src`: The total methane emissions (natural emissions plus the slug), in gigatons per year.
* `ch4_sink`: The amount of methane (natural and anthropogenic) removed from the atmosphere in gigatons per year.
  When methane is removed from the atmosphere, it turns into CO2.
* `co2_src`: The excess sources of CO2 into the atmosphere, apart from the normal natural emissions. This includes both direct human
  emissions and also conversion of anthropogenic methane emissions into CO2.
* `co2_sink`: The amount of anthropogenic CO2 removed from the atmosphere in gigatons per year.
* `ch4_forcing`: The change in i_out due to changes in the methane concentration. In Watts per square meter.
* `co2_forcing`: The change in i_out due to changes in the CO2 concentration. In Watts per square meter.
