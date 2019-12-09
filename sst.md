---
title: "Demystifying SST"
author: Luke Gregor
date: Dec 9, 2019
output:
  custom_document:
    path: ./SST_demystified.wiki
    pandoc_args: ["-t", "zimwiki"]
---

# Demistifying satellite SST products

*by Luke Gregor (9 December 2019)*

A quick guide in understanding satellite sea surface temperature. I then walk through what products might be useful. **NOTE** that [O'Carrol et al. (2019)](https://doi.org/10.3389/fmars.2019.00420) and [Merchant et al. (2019)](https://doi.org/10.1038/s41597-019-0236-x) were the two primary texts used for this and I've copy-pasted directly from these respective studies. **I, in no way, claim originality here and please cite these papers**. The O'Carrol et al. (2019) paper is a review, a very good place to start. 

**<a title="too long; didn't read">tl;dr</a>**  we should use OSTIA for most things - long time series (1985-NRT), daily, high res (0.05°), foundation temperature. 

### Table of contents
- [Summary table of products](#summary-table-of-products)
  - [Practical notes on the products](#practical-notes-on-the-products)
- [Concepts in SST](#concepts-in-sst)
  - [Sensor types](#sensor-types)
    - [Infrared (IR)](#infrared-ir)
    - [Passive Microwave (PMW)](#passive-microwave-pmw)
    - [Merged products](#merged-products)
  - [SST water column structure](#sst-water-column-structure)
    - [Use cases of different depth products](#use-cases-of-different-depth-products)
  - [Satellite product levels](#satellite-product-levels)
  - [What is GHRSST?](#what-is-ghrsst)
  - [Uncertainty estimates](#uncertainty-estimates)
  - [Night vs day](#night-vs-day)
  - [Problems at high latitudes](#problems-at-high-latitudes)
- [References and useful links](#references-and-useful-links)


## Summary table of products
This is a list I compiled based on a search mostly on the PODAAC server. It is not exhaustive. I exclude products that do not have a long time-series (many products start in the 2010's). A summary of each product is listed below taken directly from the product page. 

| Product  | space | time  | start       | end         | sensor types       | depth          | Citation                                 |
|----------|-------|-------|-------------|-------------|--------------------|----------------|------------------------------------------|
| [MUR](https://podaac.jpl.nasa.gov/dataset/MUR25-JPL-L4-GLOB-v04.2)         | 0.25  | daily | 2002 Sep 01 | DT          | IR, PMW, insitu     | foundation      | [Chin et al. 2017](https://doi.org/10.1016/j.rse.2017.07.029)  |
| [AVHRR_OI / dOISSTv2](https://podaac.jpl.nasa.gov/dataset/AVHRR_OI-NCEI-L4-GLOB-v2.0) | 0.25  | daily | 1981 Sep 01 | NRT         | IR, insitu          | bulk            | [Reynolds et al. 2007](https://doi.org/10.1175/2007JCLI1824.1); [Banzon et al. 2016](https://doi.org/10.5194/essd-8-165-2016) |
| [MW_OI](https://podaac.jpl.nasa.gov/dataset/MW_OI-REMSS-L4-GLOB-v5.0)      | 0.25  | daily | 1997 Dec 31 | NRT         | PMW                 | foundation      | [REMSS](https://doi.org/10.5067/GHMWO-4FR05)                   |
| [CMC](https://podaac.jpl.nasa.gov/dataset/CMC0.2deg-CMC-L4-GLOB-v2.0)      | 0.20  | daily | 1991 Sep 01 | 2017 Mar 18 | IR, PMW, insitu     | foundation      | [Brassnet 2008](https://doi.org/10.1002/qj.319)                |
| [SST_CCI](http://marine.copernicus.eu/services-portfolio/access-to-products/?option=com_csw&view=details&product_id=SST_GLO_SST_L4_REP_OBSERVATIONS_010_024)  | 0.05  | daily | 1981 Sep 01 | 2018 Dec 31 | IR, PMW             | at 20cm        | [Merchant et al. 2019](10.1038/s41597-019-0236-x)| 
| [OSTIA<sub>NRT</sub>](http://marine.copernicus.eu/services-portfolio/access-to-products/?option=com_csw&view=details&product_id=SST_GLO_SST_L4_NRT_OBSERVATIONS_010_001)  [OSTIA<sub>REP</sub>](http://marine.copernicus.eu/services-portfolio/access-to-products/?option=com_csw&view=details&product_id=SST_GLO_SST_L4_REP_OBSERVATIONS_010_011)    | 0.05  | daily | 1985 Jan 01 | NRT         | IR, PMW, insitu     | foundation     | [Donlon et al. 2012](10.1016/j.rse.2010.10.017)  |

Where DT = delayed time; NRT = near real time; IR = infrared; MW = microwave products; see the text below for SST measurement type descriptions. For an explaination about depths see the section about [water column structure](#sst-water-column-structure).

### Practical notes on the products 
Note that this is my opinion on the products based on what I've learnt. 

MUR
: Multiscale Ultrahigh Resolutio (MUR) has a 1km product, but this is total overkill for most applications. The authors tout this as a product with high spatio-temporal variability that is closest to reality (w.r.t. variability). However, in their publication the authors find that the CMC product is actually the most accurate with respect to GHRSST's GMPE validation dataset. I don't think we should use this product, unless we're interested in fine-scale variability. 

AVHRR / dOISSTv2
: Often refered to as daily Optimally interpolated SST uses only Infrared data (hence AVHRR). Has a long timeseries. But provides bulk temperature defined as somewhere between 1 to 2 m depth – this may be less accurate for calculating fluxes. Uses Optimal Interpolation (after [Reynolds and Smith, 1994](https://doi.org/10.1175/1520-0442(1994)007%3C0929:IGSSTA%3E2.0.CO;2)). Not the best product for calculating air-sea CO2 fluxes due to unspecified depth of the SST measurement. 

MW_OI
: A Remote Sensing Systems product that uses only Passive Microwave (PMW). Will be more robust to cloud cover, but does have gaps for heavy rainfall patches. Uses Optimal Interpolation (after [Reynolds and Smith, 1994](https://doi.org/10.1175/1520-0442(1994)007%3C0929:IGSSTA%3E2.0.CO;2)). Might be good to have as an alternative to the other products that use IR and *in situ* measurements. 

CMC
: Canadian Meteorological Center product had the lowest RMSE with respect to GHRSST's GMPE validation dataset. However, the dataset is not constantly updated and only starts in 1991 and is not updated NRT. Because of this, I don't think we should use this despite the lower RMSE's. Rather stick with better-known, loger time series products. 

SST_CCI
: The CCI product is run through the OSTIA procedure, but is endorsed and overseen by the Climate Change Initiative (CCI). It is hosted as the SST_CCI reanalysis product for 1981 to 2016 and as the C3S near rea-time (NRT) analysis product from 2017 onwards. Very similar to OSTIA, with the exception of the depth of the temperatures they estimate. The 20cm depth is aparently more consistent with bucket temperatures (older records from ships). The product reports the temperature at 10:30, which they state is equiavalent to mean daily temperatures. 

OSTIA
: The OSTIA product is hosted as two seperate products on the CMEMS website. A reanalysis product (REP) and a near real-time product. These overlap by one year. I haven't looked if these are comparible. The foundation temperature will be useful for fluxes if the Woolf et al. (2016) approaches are used to calculate air-sea CO2 fluxes. This is what I'll be using for future work on air-sea CO2 fluxes. The 0.05° resolution is probably overkill, but can always be resampled to 0.25° and 1.0°. 


## Concepts in SST
### Sensor types
There are two main sensor types: infrared and passive microwave. Within these two categories there are specific sensors, such as AVHRR, MODIS, VIRRS, AMSR-E/2 etc. The table below shows all current SST sensors in service. In the text below I give a very brief overview of the sensors types, but not the individual sensors.

![](https://www.frontiersin.org/files/Articles/434095/fmars-06-00420-HTML-r1/image_m/fmars-06-00420-t001.jpg)  
*A table from O'Carrol et al. (2019) showing the different platforms, sensor type (IR/PMW) and sensors.*

#### Infrared (IR)
- Infrared emission at the ocean surface originates in a very thin electromagnetic skin layer of <0.1 mm thickness
- The SST as derived from IR radiometers is cooler than the water beneath, on average, by ∼0.17 K, but can be several tenths of a degree cooler at low wind speeds (Donlon et al., 2002; Minnett et al., 2011). 
- Can be in geostationary or low earth orbit
- The spatial resolution of IR-derived SSTs from modern satellite instruments is typically 1–4 km at nadir
- About 80–90% of pixels are fully or partially cloudy in any IR imagery of the ocean. Clouds are colder and more reflective, more variable in space/time, and have a particular IR spectral emissivity, compared to clear ocean. Effective “cloud-screening” algorithms are needed to ensure derived SSTskin values are not tainted (Kilpatrick et al., 2001; Merchant et al., 2005). 
- Sea surface temperature retrievals from IR measurements are susceptible to forms of aerosol that absorb and scatter IR radiation, particularly mineral dust (such as lofted from the Sahara Desert) and stratospheric volcanic aerosol following major SO2-rich eruptions (the last being Mt. Pinatubo in 1991)

#### Passive Microwave (PMW)
- PMW emission comes from an electromagnetic skin layer which is several millimeters thick. 
- The resolution of microwave (MW)-derived SSTs is typically 50–75 km. 
- PMW SSTs are not normally derived within ∼100 km of land or ice. 
- A major advantage of PMW-derived SSTs over those from the IR is that the propagation of MW radiation at 6–10 GHz which is largely insensitive to the presence of aerosols and clouds, except where there is heavy rainfall. 
- SST sensitivity of the 10.65 GHz emission decreases below SSTs colder than 13°C (Gentemann et al., 2010).
- Can only be in low earth orbit

#### Merged products 
- Merging SST retrievals from IR and MW sensors together with in situ data on any given day is a widely accepted approach to derive global fields (e.g., Chin et al., 2017).
- PMW SSTs are an essential data source for producing multi-satellite merged SST products on a daily basis, though their spatial resolution is much coarser than IR SSTs. 


### SST water column structure

![](https://www.ghrsst.org/wp-content/uploads/2016/10/newerSSTdef.gif)  
*Figure showing the vertical water structure in SST measurements (taken from the [GHRSST website](https://www.ghrsst.org/ghrsst-data-services/products/))*

Interface temperature (SSTint)
: At the exact air-sea interface a hypothetical temperature called the interface temperature (SSTint) is defined although this is of no practical use because it cannot be measured using current technology.

Skin sea surface temperature (SSTskin)
: The temperature measured by an *infrared radiometer* typically operating at wavelengths 3.7-12 µm. SSTskin measurements are subject to a large potential diurnal cycle including cool skin layer effects (especially at night under clear skies and low wind speed conditions) and warm layer effects in the daytime. 

Sub-skin sea surface temperature (SSTsub-skin)
: The temperature at the base of the conductive laminar sub-layer of the ocean surface. For practical purposes, SSTsubskin can be approximated to the measurement of surface temperature by a *microwave radiometer* operating in the 6-11 GHz frequency range.

Surface temperature at depth (SSTz or SSTdepth)
: All measurements of water temperature beneath the SSTsubskin are referred to as depth temperatures (SSTdepth) measured (e.g. drifting buoys, vertical profiling floats). These temperature observations are distinct from SSTskin and SSTsubskin and must be qualified by a measurement depth in meters (e.g., or SST(z) e.g. SST5m).

Foundation temperature (SSTfnd)
: The temperature free of diurnal temperature variability, i.e., SSTfnd is defined as the temperature at the first time of the day when the heat gain from the solar radiation absorption exceeds the heat loss at the sea surface. Only in situ contact thermometry is able to measure SSTfnd and analysis procedures must be used to estimate the SSTfnd from radiometric retrievals of SSTskin and SSTsubskin taken at other times of the day. 
￼

#### Use cases of different depth products
The SSTskin is the temperature most appropriate for determining instantaneous **air-sea fluxes**, since skin SST determines the surface radiative cooling of the ocean and the temperature and humidity of the air in contact with the air-sea interface.  
For many purposes, SST estimated at a depth below the skin effect is more appropriate. The difference between skin and depth SST is typically of order tenths of kelvin, but can be larger. ***In situ* SST measurements** and the upper layers of **ocean models** typically reflect SST at depths between ~10 cm and ~10 m. In order to use satellite SSTs with the centennial SST record, estimates comparable to depths sampled by ships’ buckets and drifting buoys are needed.   
*OSTIA* and the *CCI* products convert the instantaneous skin SST to a depth of 20 cm, nominally corresponding to drifter and historic bucket temperature measurements.


### Satellite product levels
L2P 
: data are on the original viewing geometry, with gaps in SST from cloud cover. 

L3U
: gridded per orbit

L3C
: A day’s worth of L3U SSTs from a given sensor is collated to form an L3C product. 

L4
: Data from multiple sensors are merged and interpolated to give the daily gap-free SST field of the L4 analysis.

### What is GHRSST?
Group for High-Resolution SST (GHRSST) began coordination of operational production and distribution of satellite SST datasets in 2005 governed by the foundational system of shared roles and responsibilities. GHRSST has supported efforts over the past 10 years to unify in situ data for satellite validation. For example, they created the Multi-Product Ensemble (GMPE) system which was designed to allow intercomparison of near real time analyses. 

### Uncertainty estimates
There are a variety of techniques for quantifying uncertainty, in a statistical way. These fall into two classes: 
1) empirical methods in which uncertainty is deduced from the distribution of differences between alternative measured values (such as satellites versus drifting buoys; e.g., Castro et al., 2008, 2012; O’Carroll et al., 2008; Petrenko et al., 2016; Xu and Ignatov, 2016); 
2) and uncertainty modeling, in which understanding of the instrumental uncertainty, cloud screening, retrieval process and representativity effects are quantified, propagated and combined to form an uncertainty estimate (e.g., Merchant et al., 2014). 

Approximate empirical methods of estimating uncertainty are presently the approach within GHRSST format SST products. These include Sensor Specific Error Statistics (SSES), comprising the mean and standard deviation of satellite SSTs differenced from a matched in situ temperature, such as measured from drifting buoys. 


### Night vs day
The difference between a daytime hourly SST value and the corresponding foundation temperature of the previous night is defined as diurnal warming (or anomaly). Progress in understanding and quantifying diurnal variability using hourly IR-based SSTs from geostationary platforms. 
![](https://www.frontiersin.org/files/Articles/434095/fmars-06-00420-HTML-r1/image_m/fmars-06-00420-g006.jpg)  
*Figure showing the number of days where SST night vs SST day is > 1°K (O'Carrol et al., 2019)*

The following is a forum post by [Sean Baily (NASA)](https://science.gsfc.nasa.gov/sed/bio/sean.w.bailey) on  [Oceancolor](https://oceancolor.gsfc.nasa.gov/forum/oceancolor/topic_show.pl?tid=3614). Sometimes you might see two SST algorithms for MODIS, one based on the 11 micron band, and one on the 4 micron band.
The 4 micron band is ONLY processed at night, while the 11 micron is processed both day and night.

The [REMSS website](http://www.remss.com/measurements/sea-surface-temperature/oisst-description/) provides a good description of how the correction for daytime SST is done.

### Problems at high latitudes
SST retrievals at high latitudes are difficult for a number of reasons. IR and PMW SSTs require in situ datasets for algorithm development, validation, and fine-tuning, and measurements of in situ SST are sparse at high latitudes, and often lack coincident atmospheric observations critical for algorithm development. Without adequate sampling of the variable atmospheric and oceanic conditions, IR and PMW algorithms rely on the data that is too sparse in time and space, creating unknown errors in conditions outside the relatively narrow range of existing observations. There are additional complications, for example, recent PMW missions lack a 6.9 GHz channel that retains useful sensitivity below 13°C, which includes most regions poleward of 60° latitudes. Without a 6.9 GHz channel, errors increase by 0.5°C (Gentemann et al., 2010).

## References and useful links
These are not exhaustive, but are the sources I used to write this summary. 
- [OSTIA presentation (https://www.ecmwf.int/...)](https://www.ecmwf.int/sites/default/files/elibrary/2018/17975-operational-sea-surface-temperature-and-ice-analysis-ostia-system.pdf) 

- [https://www.ghrsst.org/ghrsst-data-services/products/](https://www.ghrsst.org/ghrsst-data-services/products/)

- Merchant, C.J., Embury, O., Bulgin, C.E. et al. Satellite-based time-series of sea-surface temperature since 1981 for climate applications. Sci Data 6, 223 (2019) https://doi.org/10.1038/s41597-019-0236-x

- O’Carroll, A. G., Armstrong, E. M., Beggs, H., Bouali, M., Casey, K. S., Corlett, G. K., … Wimmer, W. (2019). Observational needs of sea surface temperature. Frontiers in Marine Science, 6(JUL). https://doi.org/10.3389/fmars.2019.00420

- http://www.remss.com/measurements/sea-surface-temperature/oisst-description/
