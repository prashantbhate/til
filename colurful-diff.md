I needed to compare and share diff between two artifacts/archives ( old.war and new.war ) with the team to highlight the 
differences between old and new artifacts, both generated from two different gitlab branches => two different jenkins pipelines.

Expectation was that the changes between those two to be minimal and any differences has to be agreed and accepted.

Infact There were multiple binaries to compare ( old1.war, old2.war, old3.ear)
We also needed to repeat this multiple times 
 * Generate archive with old pipeline (this pushes old artifacts to nexus 1.0.0-SNAPSHOT)
 * Generate archive with new pipeline (this pushes new artifacts to nexus 2.0.0-SNAPSHOT)
 * Download and Unzip old to dir1
 * Download and Unzip new to dir1
 * Compare filenames and content
 * Capture the diff result
 
Solution was to scriptify some of above steps
 
 ~~~~shell
 
 VERSION_OLD=1.0.0
 VERSION_NEW=2.0.0

#download artifacts in parallel
 wget http://nexus.*/group/${VERSION_OLD}/artifactA${VERSION_OLD}.war &
 wget http://nexus.*/group/${VERSION_OLD}/artifactB${VERSION_OLD}.war &
 wget http://nexus.*/group/${VERSION_OLD}/artifactC${VERSION_OLD}.ear &

 wget http://nexus.*/group/${VERSION_NEW}/artifactA${VERSION_NEW}.war &
 wget http://nexus.*/group/${VERSION_NEW}/artifactB${VERSION_NEW}.war &
 wget http://nexus.*/group/${VERSION_NEW}/artifactC${VERSION_NEW}.ear &

#wait for download to complete

wait $(jobs -rp)

#recursive unzip
 
while [ "`find . -type f -name '*.war' -or -name '*.ear' | wc -l`" -gt 0 ]; do                                                                                    ✘ 130 develop ✱ ◼
  find . -type f \( -name "*.war"  -or -name '*.ear' \)  -exec echo {} \;  \
    -exec mkdir -p '{}.dir' \;  \
    -exec unzip -o -d '{}.dir'  -- '{}' \;   \
    -exec rm -- '{}' \;;
done

#generate text file with dir structure
ls -1d * | xargs -I {} bash -c 'cd {}; find .|sort  >../{}.txt'

#compare the filenames
icdiff -U0 artifactA${VERSION_OLD}.war.dir.txt artifactA${VERSION_NEW}.war.dir.txt |aha > artifactA.html
icdiff -U0 artifactB${VERSION_OLD}.war.dir.txt artifactB${VERSION_NEW}.war.dir.txt |aha > artifactB.html
icdiff -U0 artifactB${VERSION_OLD}.ear.dir.txt artifactC${VERSION_NEW}.ear.dir.txt |aha > artifactC.html

#compare the content
kdiff3 artifactA${VERSION_OLD}.war.dir artifactA${VERSION_NEW}.war.dir
kdiff3 artifactB${VERSION_OLD}.war.dir artifactB${VERSION_NEW}.war.dir
kdiff3 artifactC${VERSION_OLD}.war.dir artifactC${VERSION_NEW}.war.dir

 ~~~~
 
 Tools used :
 * **unix commands**
 * **icdiff** : gives colorful side by side diff
 * **aha**  : generates colour console as html
 * **kdiff3** to compare content
 
