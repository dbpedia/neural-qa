# Automatic Training Data Generation For Neural QA Model

The final output file has each row of the form:

``` bash
['Property', 'Label ', 'Range', 'Fuzzy Score', 'Comment about expr', 'URI', 'Number of Occurrences', 'MVE', 'Optimal Expression', 'SPARQL Query Template', 'Generator Query\r\n']
```

## Steps to follow

### STEP 1 - Get properties from web page

Command:

```bash
python get_properties.py --url <WEBPAGE URL> --output_file <OUTPUT FILE NAME > --project_name <PROJECT NAME>
```

- `--url <WEBPAGE URL>`: --url argument is the webpage from where property metadata is to scraped example: http://mappings.dbpedia.org/server/ontology/classes/Place

- `--output_file <OUTPUT FILE NAME >`:  --output_file argument is the file where the output data of this function will be stored in CSV format.

- `--project_name <PROJECT NAME>`:  --project_name argument is the name given to the test case, a seperate folder with ths name will be ceated where the data files related to this project will be stored.

### STEP 2 - Get number of occurrences and URI

Store only the rows of required namespace properties

### STEP 3 - Integrate STEP 2 values with their corresponding property metadata row in temp.csv

Command:

``` bash
python integrate.py --namespace --input_file temp.csv --output_file integrated_file.csv --uri_file dbpedia-201610-properties.tsv
```

- Output file: manual-annotation-updated-v2.csv (change it in the file if needed)
- Change the namespace to the required (it is 'ontology' right now) 

### STEP 4 - MVE generation

Command:

```bash
python decision_tree.py data/manual-annotation-updated-v2.csv
```

- Output file: GS_with_mve.csv

### STEP 5 - SPARQL Query Template and Generator Query generation

Command:

```bash
python sparql_generator.py GS_with_mve.csv
```

### STEP 6 - Formatting the data into required format

Command:

```bash
python final_formatting.py data/GS-v3.csv data/annotations_place_v2.csv
```

### STEP 7 - Follow the original data generation and training steps (readme of master branch)

Steps from the previous folder.

## COMPOSITIONALITY EXPERIMENT

### STEP 1 - Create template annotations (all a[i]'s)

Command:

```bash
python range_place.py data/GS-v3.csv > data/annotations_compositions_combined.csv
```

### STEP 2 - Create composite templates (a[i]○b true for all i <= sizeof list 'a')

Command:

```bash
python composite_template.py data/GS-v3.csv >> data/annotations_compositions_combined.csv
```

### STEP 3 - Follow the original data generation and training steps (readme of master branch)

Steps from the previous folder.

### STEP 4 - Choose any 10% templates and their output and shift it to new file (test data), rest of the contents of this file should be split into 90% train and 10% dev using the split_in_train_dev_test.py script.

### STEP 5 - Run the training

Command:

```bash
sh train.sh <path of directory which contains train, dev and test data>
```