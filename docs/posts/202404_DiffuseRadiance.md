---
date: 2024-04-17
categories:
  - Technology
authors:
  - floko

---

# Modelling Diffuse Radiation
PV systems can use two different sources of sun radiation to generate electricity. The obvious source is the direct radiation, meaning the light coming directly from the sun. The second source is the diffuse radiation, ie the radiation which is reflected by air particles and then hitting the PV panel. Hence, the blue of the sky is the second source of energy for PV systems.


<!-- more -->
## Steps
### Search for a diffuse radiation model
The goal is to have a function that gets a sun position ($\theta$ and $\phi$ in spherical coordinates) and returns the amount of diffuse radiation for every part of the sky (ie for every $\theta'$ and $\phi'$). We identified the publication [Anisotropic Sky Radiance Model based on Narrow Field of View Measurements of Shortwave Radiance](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=93f59805f64fd372d7ce5e18addbeadcc846de6b) to contain exactly that kind of model. It needs additional parameters based on the climate situation of the location of interest, ie the cloudiness.

### Implement the model
We implemented the model and generated diffuse radiation samples. Each sample is a 2D array, containing the yearly average radiation of the given $\theta'$ and $\phi'$ direction. Since sun positions depend not only on the time, but also on the location on earth, we generated one averaged sample for multiple earth positions. 

<figure>
    <img src="radiation.png"
         alt="Irradiance Plot in spherical coordiantes. The color indicates the yearly average irradiance in arbitrary units.">
    <br/>
    <figcaption>Yearly average irradiance. The color indicates the irradiance coming from the given sky segment, legend is in arbitrary units. The edge of the graph represents the horizon, the center of the graph is the center of the sky directly above the PV system. South is at the left.</figcaption>
</figure>

### Host the simulated radiance
The precalculated diffuse radiance is then hosted as json files on a server. They are named depending on their coordinates, ie "latitude/longitude.json" -> "23/44.5.json".

### Add new feature to @openpv/simshady
`@openpv/simshady` is our npm package to simulate PV yields in the browser. Originally it only used the direct radiation of the sun. We now extend the functionality as follows: The source of sun intensities can be set as an API request. The returned json is expected to contain a nested 2D array. It should contain the average yearly irradiance coming from the sky segment $\theta'$ and $\phi'$. If no such API is given, the fallback method is to sample over random sun positions and only use the direct radiation in the simulation.