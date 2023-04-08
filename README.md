# earth-basher

OGR/BASH scripts to work with Natural Earth vectors.

1. earth-clipper
2. earth-to-svg
3. earth-to-json

## 1. earth-clipper
Slow but hands-free way to clip every Natural Earth layer at the same scale by a feature of your choice and export to a single file.

```
#===============# 
# earth-clipper #
#===============#

### select clipper ###
name='United States of America'
layer=ne_110m_admin_0_countries
proj='epsg:4326'

### find & clip layers at same scale ###
rm -rf ${name// /_}.gpkg
ogrinfo -dialect sqlite -sql "SELECT name FROM sqlite_master WHERE name LIKE '$(echo ${layer} | awk -F  "_" '{print $1"_"$2}')%'" natural_earth_vector.gpkg | grep ' = ' | sed -e 's/^.* = //g' | while read table; do
  ogr2ogr -update -append -skipfailures --config OGR_ENABLE_PARTIAL_REPROJECTION TRUE -nlt promote_to_multi -s_srs 'epsg:4326' -t_srs ${proj} -clipsrc 'natural_earth_vector.gpkg' -clipsrclayer ${layer} -clipsrcwhere "name = '${name}'" ${name// /_}.gpkg natural_earth_vector.gpkg ${table}
done

### remove empty tables ###

# TO DO
```

## 2. earth-to-svg
Convert any point, line or polygon layer to svg.

```
#==============# 
# earth-to-svg #
#==============#

### select layer to convert ###
layer=ne_110m_admin_0_countries
width=1920
height=960

### get extent and start file ###
ogrinfo -dialect sqlite -sql "SELECT ST_MinX(extent(geom)) || CAST(X'09' AS TEXT) || (-1 * ST_MaxY(extent(geom))) || CAST(X'09' AS TEXT) || (ST_MaxX(extent(geom)) - ST_MinX(extent(geom))) || CAST(X'09' AS TEXT) || (ST_MaxY(extent(geom)) - ST_MinY(extent(geom))) FROM '"${layer}"'" natural_earth_vector.gpkg | grep -e '=' | sed -e 's/^.*://g' -e 's/^.* = //g' | while IFS=$'\t' read -a array; do
echo '<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" height="'${height}'" width="'${width}'" viewBox="'${array[0]}' '${array[1]}' '${array[2]}' '${array[3]}'">' > ${layer}.svg
done

### convert points, lines or polygons ###
ogrinfo -dialect sqlite -sql "SELECT fid || CAST(X'09' AS TEXT) || ST_X(ST_Centroid(geom)) || CAST(X'09' AS TEXT) || (-1 * ST_Y(ST_Centroid(geom))) || CAST(X'09' AS TEXT) || AsSVG(geom, 1) || CAST(X'09' AS TEXT) || GeometryType(geom) FROM ${layer} WHERE geom NOT LIKE '%null%'" natural_earth_vector.gpkg | grep -e '=' | sed -e 's/^.*://g' -e 's/^.* = //g' | while IFS=$'\t' read -a array; do
  case ${array[4]} in
    POINT|MULTIPOINT)
      echo '<circle id="'${array[0]}'" cx="'${array[1]}'" cy="'${array[2]}'" r="1em" vector-effect="non-scaling-stroke" fill="#FFF" fill-opacity="1" stroke="#000" stroke-width="0.6px" stroke-linejoin="round" stroke-linecap="round"/>' >> ${layer}.svg
      ;;
    LINESTRING|MULTILINESTRING)
      echo '<path id="'${array[0]}'" d="'${array[3]}'" vector-effect="non-scaling-stroke" stroke="#000" stroke-width="0.6px" stroke-linejoin="round" stroke-linecap="round"/>' >> ${layer}.svg
      ;;
    POLYGON|MULTIPOLYGON)
      echo '<path id="'${array[0]}'" d="'${array[3]}'" vector-effect="non-scaling-stroke" fill="#000" fill-opacity="1" stroke="#FFF" stroke-width="0.6px" stroke-linejoin="round" stroke-linecap="round"/>' >> ${layer}.svg
      ;;
  esac
done
echo '</svg>' >> ${layer}.svg
```

## 3. earth-to-json

```
#===============# 
# earth-to-json #
#===============#

# coming soon
```
