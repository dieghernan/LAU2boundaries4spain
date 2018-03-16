
<!-- README.md is generated from README.Rmd. Please edit that file -->
Objetivo
--------

El objetivo de este repositorio es ofrecer los contornos de los términos municipales españoles en un contexto histórico, y ajustados a los municipios existentes a 1 de enero de cada año, de forma que cuadren exactamente con el Padrón de dicho año. La información ofrecida ha ido elaborándose durante bastante tiempo a partir de diversos trabajos (Goerlich, Mas, Azagra y Chorén 2006, 2007; Goerlich, Ruiz, Chorén y Albert 2015; Reig, Goerlich y Cantarino 2016).

Creemos que dicha base de datos es útil por varias razones. El [Instituto Geográfico Nacional](http://www.ign.es/web/ign/portal) (IGN) ofrece, a través del [Centro de Descargas](http://centrodedescargas.cnig.es/CentroDescargas/index.jsp) del Centro Nacional de Información Geográfica, los recintos municipales (también los provinciales y autonómicos) dentro de la Base de Datos de Líneas de Límite Municipal en la **Información Geográfica de Referencia**, sin embargo estas bases de datos son de actualización continua, de forma que lo que se dispone son los municipios "en el momento actual", y no existe un histórico que puede ser de utilidad por muchas razones. Así pues, hasta donde nosotros conocemos, no se dispone, por ejemplo, de una capa vectorial de los polígonos municipales del año 2006, o de la de los municipios correspondientes al censo de 2001. Esta información es necesaria si queremos combinar la información geográfica con la información estadística municipal histórica procedente, por ejemplo, del [Instituto Nacional de Estadística](http://www.ine.es/) (INE).

Este repositorio pretende cubrir esa laguna que, de momento, no ha sido satisfecha por las instituciones oficiales. El repositorio está disponible como un package de R alojado en Github: <https://github.com/perezp44/LAU2boundaries4spain>.

Información más detallada del proceso seguido para la construcción de los diferentes ficheros de lindes puede encontrase en una de las *vignettes* del *package*, concretamente [aquí](https://htmlpreview.github.io/?https://github.com/perezp44/LAU2boundaries4spain/blob/master/inst/doc/detailed-info-lau2boundaries4spain.html)

Datos
-----

El paquete proporciona:

-   un fichero con los lindes de las CC.AA (`CCAA`)
-   un fichero con los lindes provinciales (`Provincias`)
-   un fichero con los lindes municipales para cada año del periodo 2002-2018 (por ejemplo `municipios_2018`)

Los ficheros se ofrecen en formato `spatial-df` del paquete [`sf`](https://cran.r-project.org/web/packages/sf/index.html)

Instalación
-----------

Para instalar el paquete:

``` r
devtools::install_github("perezp44/LAU2boundaries4spain", force = TRUE)
library(LAU2boundaries4spain)
```

Uso
---

Para cargar los lindes provinciales y graficarlos:

``` r
library(tidyverse)
library(sf)
library(LAU2boundaries4spain)
Provincias <- Provincias
plot(Provincias, max.plot = 1)
```

Si queremos situar a Canarias cerca de España:

``` r
library(tidyverse)
library(sf)
Provincias <- Provincias
#provincias_df <- Provincias %>% st_set_geometry(NULL) #- le quitas la geometria
canarias <- Provincias %>% filter(INECodProv %in% c(35,38))
peninsula <- Provincias %>% filter( !(INECodProv %in% c(35, 38)) )
my_shift <- st_bbox(peninsula)[c(1,2)]- (st_bbox(canarias)[c(1,2)]) + c(9.5,0.5)
canarias$geometry <- canarias$geometry + my_shift
st_crs(canarias)  <- st_crs(peninsula)
provincias_a <- rbind(peninsula, canarias)  
plot(provincias_a, max.plot = 1)
```

Para cargar y graficar los lindes municipales de 2018:

``` r
library(tidyverse)
library(sf)
library(LAU2boundaries4spain)
municipios_2018 <- municipios_2018
plot(municipios_2018, max.plot = 1)
```

Si queremos graficar los lindes provinciales a partir de las geometrías municipales de 2016 (más información [aquí](https://github.com/r-spatial/sf/issues/290)):

``` r
library(tidyverse)
library(sf)
library(LAU2boundaries4spain)
Provincias_2016 <- municipios_2016 %>%  group_by(INECodProv) %>% summarise()
plot(Provincias_2016)
```

Si queremos fusionar los datos de población con el fichero de lindes:

``` r
library(tidyverse)
library(sf)
library(LAU2boundaries4spain)
library(spanishRpoblacion)
pob <- INE_padron_muni_96_17  #- datos de poblacion del Padron 
pob_2016 <- pob %>% filter(anyo == 2016) %>% # en 2016 habían 8.125 municipios
            select(INECodMuni, Pob_T, Pob_H, Pob_M)  
lindes_2016 <- municipios_2016 #- 8.125 municipios + 84 condominios
fusion_2016 <- full_join(lindes_2016, pob_2016)   
fusion_2016_df <- fusion_2016 %>% st_set_geometry(NULL) #- le quitas la geometria para verlo mejor
```

El sistema de referencia de los datos del paquete es ETRS89 sin proyección. Si queremos cambiar de sistema de referencia, por ejemplo a LAEA, hacemos:

``` r
# EPSG Projection 3035 - ETRS89 / ETRS-LAEA 
# Proj4js.defs["EPSG:3035"] = "+proj=laea +lat_0=52 +lon_0=10 +x_0=4321000 +y_0=3210000 +ellps=GRS80 +units=m +no_defs"
library(tidyverse)
library(sf)
library(LAU2boundaries4spain)
Provincias_proj <- st_transform(Provincias, crs = 3035) 
plot(Provincias_proj, max.plot = 1)
```
