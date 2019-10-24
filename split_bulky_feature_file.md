````
#split feature file into individual files
function split_feature() {
FEATURE=$1
mkdir $FEATURE
cd $FEATURE
gcsplit --prefix=$FEATURE --suffix-format="%02d.feature"  -s   ../${FEATURE}.feature '/^  Scenario Outline:.*/-1' '{*}'
for i in $(ls -1|grep -v 00)
do
cat ${FEATURE}00.feature $i >$i.new
mv $i.new $i
done
rm ${FEATURE}00.feature
}

````
