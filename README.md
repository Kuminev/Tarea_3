---
title: "Tarea 3"
author: "Fernando Bermúdez Kuminev"
date: "2020-11-04"
output:
  html_document:
    theme: readable    
    toc: true
    toc_depth: 3
    toc_float:
      collapsed: false
      smooth_scroll: false    
---

```{r setup, include=FALSE}
library(knitr)

knitr::opts_chunk$set(echo = TRUE)
```

Instalación de paquetes:
```{r eval = FALSE, message = FALSE}

install.packages("leaflet")
install.packages("tidyr")
install.packages("sf")
```

Carga de paquetes:
```{r message = FALSE}

library(leaflet)

library(sf)

library(tidyr)

library(dplyr)
```

## Mapa de tres capas

```{r}
mapa_1 <- leaflet()%>% 
  fitBounds(lng1 = -86, lng2 = -82, lat1 = 8, lat2 = 11)%>% 
  addTiles() %>% 
  addProviderTiles(providers$Thunderforest.OpenCycleMap, group = "Rutas") %>%
  addProviderTiles(providers$OpenTopoMap, group = "Topografia") %>%
  addProviderTiles(providers$Esri.WorldImagery, group = "Imagen satelital")  %>% 
  addLayersControl(
    baseGroups = c("Rutas","Topografia", "Imagen satelital"))

mapa_1
```

## Mapas generales

```{r}
url_base_wfs <- "http://geos.snitcr.go.cr/be/IGN_5/wfs?"

solicitud_wfs <- 
  "request=GetFeature&service=WFS&version=2.0.0&typeName=IGN_5:limitecantonal_5k&outputFormat=application/json"

sf_cantones<- st_read(paste0(url_base_wfs, solicitud_wfs))

```

## Construcción de casos positivos.

```{r message=FALSE, warning=FALSE}
#Positivos
casos_positivos <- 
  read.csv("https://raw.githubusercontent.com/pf0953-programaciongeoespacialr-2020/datos/master/covid19/ms/06_27_CSV_POSITIVOS.csv")

df_positivos<-
  casos_positivos %>%
  pivot_longer(
    cols = c(-cod_provin, -provincia, -cod_canton, -canton), 
    names_to = "fecha", 
    values_to = "positivos"
  )

df_positivos$fecha <- as.Date(df_positivos$fecha, "X%d.%m.%Y")

df_positivos_uf <- 
df_positivos %>%
filter(fecha == max(fecha), na.rm = TRUE) %>%
select(cod_canton, positivos)

sf_positivos_cantones_uf<-
  left_join(sf_cantones, df_positivos_uf, by = c('cod_canton')) %>%
  arrange(desc(positivos))

bins <- c(0, 25, 50, 75, 100)
paleta1 <- colorBin("YlOrRd", domain = sf_positivos_cantones_uf$positivos, bins = bins)

mapa_positivos <- leaflet(sf_positivos_cantones_uf) %>% 
  fitBounds(lng1 = -86, lng2 = -82, lat1 = 8, lat2 = 11) %>%
  addProviderTiles(providers$OpenStreetMap.Mapnik, group = "OpenStreetMap") %>%
  addPolygons(stroke=T, fillOpacity = 0.1,
              color="black", weight=0.2, opacity= 0.1,
              group = "Positivos",
              popup = paste(
                "Cantón: ", sf_cantones$canton, "<br>",
                "Positivos: ", sf_positivos_cantones_uf$positivos)
              )%>%
     addLegend("bottomright", pal = paleta1, values = ~positivos,
            title = "Casos positivos",
            opacity = 1
  ) %>%  
  addLayersControl(
    baseGroups = c("OpenStreetMap"),
    overlayGroups = c("Positivos"),
    options = layersControlOptions(collapsed = TRUE)    
  )

```

```{r}
#Activos

casos_activos <- 
  read.csv("https://raw.githubusercontent.com/pf0953-programaciongeoespacialr-2020/datos/master/covid19/ms/06_27_CSV_ACTIVOS.csv")

df_activos<-
  casos_activos %>%
  pivot_longer(
    cols = c(-cod_provin, -provincia, -cod_canton, -canton), 
    names_to = "fecha", 
    values_to = "activos"
  )

df_activos$fecha <- as.Date(df_activos$fecha, "X%d.%m.%Y")

df_activos_uf <- 
df_activos %>%
filter(fecha == max(fecha), na.rm = TRUE) %>%
select(cod_canton, activos)

sf_activos_cantones_uf<-
  left_join(sf_cantones, df_activos_uf, by = c('cod_canton')) %>%
  arrange(desc(activos))

bins <- c(0, 25, 50, 75, 100)
paleta2 <- colorBin("Greens", sf_activos_cantones_uf$activos, bins = bins)

mapa_activos <- leaflet(sf_activos_cantones_uf) %>% 
  fitBounds(lng1 = -86, lng2 = -82, lat1 = 8, lat2 = 11) %>%
  addProviderTiles(providers$OpenStreetMap.Mapnik, group = "OpenStreetMap") %>%
  addPolygons(stroke=T, fillOpacity = 0.1,
              color="black", weight=0.2, opacity= 0.1,
              group = "Positivos",
              popup = paste(
                "Cantón: ", sf_activos_cantones_uf$canton, "<br>",
                "Activos: ", sf_activos_cantones_uf$activos)
              )%>%
     addLegend("bottomleft", pal = paleta2, values = ~activos,
            title = "Casos activos",
            opacity = 1
  ) %>%  
  addLayersControl(
    baseGroups = c("OpenStreetMap"),
    overlayGroups = c("Activos"),
    options = layersControlOptions(collapsed = TRUE)    
  )


```

# RECUPERADOS
```{r}
#Recuperados

casos_recupe <- 
  read.csv("https://raw.githubusercontent.com/pf0953-programaciongeoespacialr-2020/datos/master/covid19/ms/06_27_CSV_RECUP.csv")

df_recupe<-
  casos_recupe %>%
  pivot_longer(
    cols = c(-cod_provin, -provincia, -cod_canton, -canton), 
    names_to = "fecha", 
    values_to = "recupe"
  )

df_recupe$fecha <- as.Date(df_recupe$fecha, "X%d.%m.%Y")

df_recupe_uf <- 
df_recupe %>%
filter(fecha == max(fecha), na.rm = TRUE) %>%
select(cod_canton, recupe)

sf_recupe_cantones_uf<-
  left_join(sf_cantones, df_recupe_uf, by = c('cod_canton')) %>%
  arrange(desc(recupe))

bins <- c(0, 25, 50, 75, 100)
paleta3 <- colorBin("Accent", sf_recupe_cantones_uf$recupe, bins = bins)

mapa_recuperados<- mapa_recupe <- leaflet(sf_recupe_cantones_uf) %>% 
  fitBounds(lng1 = -86, lng2 = -82, lat1 = 8, lat2 = 11) %>%
  addProviderTiles(providers$OpenStreetMap.Mapnik, group = "OpenStreetMap") %>%
  addPolygons(stroke=T, fillOpacity = 0.1,
              color="black", weight=0.2, opacity= 0.1,
              group = "Recupe",
              popup = paste(
                "Cantón: ", sf_recupe_cantones_uf, "<br>",
                "Recuperados: ", sf_recupe_cantones_uf$recupe)
              )%>%
     addLegend("bottomleft", pal = paleta3, values = ~recupe,
            title = "Casos recuperados",
            opacity = 1
  ) %>%  
  addLayersControl(
    baseGroups = c("OpenStreetMap"),
    overlayGroups = c("Recuperados"),
    options = layersControlOptions(collapsed = TRUE)    
  )

```

# FALLECIDOS
```{r}
#Fallecidos

casos_fallecidos <- 
  read.csv("https://raw.githubusercontent.com/pf0953-programaciongeoespacialr-2020/datos/master/covid19/ms/06_27_CSV_FALLECIDOS.csv")

df_fallecidos<-
  casos_fallecidos %>%
  pivot_longer(
    cols = c(-cod_provin, -provincia, -cod_canton, -canton), 
    names_to = "fecha", 
    values_to = "fallecidos"
  )

df_fallecidos$fecha <- as.Date(df_fallecidos$fecha, "X%d.%m.%Y")

df_fallecidos_uf <- 
df_fallecidos %>%
filter(fecha == max(fecha), na.rm = TRUE) %>%
select(cod_canton, fallecidos)

sf_fallecidos_cantones_uf<-
  left_join(sf_cantones, df_fallecidos_uf, by = c('cod_canton')) %>%
  arrange(desc(fallecidos))

bins <- c(0, 25, 50, 75, 100)
paleta4 <- colorBin("RdYlBu", sf_fallecidos_cantones_uf$recupe, bins = bins)

mapa_recuperados<- mapa_recupe <- leaflet(sf_fallecidos_cantones_uf) %>% 
  fitBounds(lng1 = -86, lng2 = -82, lat1 = 8, lat2 = 11) %>%
  addProviderTiles(providers$OpenStreetMap.Mapnik, group = "OpenStreetMap") %>%
  addPolygons(stroke=T, fillOpacity = 0.1,
              color="black", weight=0.2, opacity= 0.1,
              group = "Recupe",
              popup = paste(
                "Cantón: ", sf_fallecidos_cantones_uf$canton, "<br>",
                "Fallecidos: ", sf_fallecidos_cantones_uf$fallecidos)
              )%>%
     addLegend("bottomleft", pal = paleta3, values = ~recupe,
            title = "Casos fallecidos",
            opacity = 1
  ) %>%  
  addLayersControl(
    baseGroups = c("OpenStreetMap"),
    overlayGroups = c("Fallecidos"),
    options = layersControlOptions(collapsed = TRUE)    
  )

```
