# earth-basher
BASH scripts to work with Natural Earth vectors

## earth-clipper

Inefficient but hands-free way to clip every Natural Earth layer at the same scale by a feature of your choice. Uses OGR to clip and export to a single GPKG file.

```
# select clipper
name='United States of America'
layer=ne_10m_admin_0_countries
proj='epsg:4326'

# clip layers at same scale
rm -rf natural_earth_vector_${name// /_}.gpkg
ogrinfo -sql "SELECT name FROM sqlite_master WHERE name LIKE '$(echo ${layer} | awk -F  "_" '{print $1"_"$2}')%'" /home/steve/maps/naturalearth/packages/natural_earth_vector.gpkg | grep ' = ' | sed -e 's/^.* = //g' | while read table; do
  ogr2ogr -update -append -nlt promote_to_multi -clipsrc -s_srs 'epsg:4326' -t_srs '${proj}' '/home/steve/maps/naturalearth/packages/natural_earth_vector.gpkg' -clipsrclayer ${layer} -clipsrcwhere "name = '${name}'" natural_earth_vector_${name// /_}.gpkg /home/steve/maps/naturalearth/packages/natural_earth_vector.gpkg ${table}
done

# remove empty tables
TO DO
```

## earth-to-svg

```
Coming soon
```
