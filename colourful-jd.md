I wanted to compare specific field changes in two json files and wanted to genreate report  here is what I did

```
  jq '{data: {letters: (.data.letters
                    | to_entries
                    | sort_by(.key)
                    | map({(.key): {patients: ([.value.patients[].name] | sort)}})
                    | add)}}' \
   "./source1.json" > a.json

jq '{data: {letters: (.data.letters
                    | to_entries
                    | sort_by(.key)
                    | map({(.key): {patients: ([.value.patients[].name] | sort)}})
                    | add)}}' \
   "./source2.json" > b.json

icdiff -U0  a.json b.json | aha >diff.html

```

* Used jq to extract required fields and sort attributes
* icdiff to coloured diff and aha to convert it to html
