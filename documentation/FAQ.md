# General

#### What are the semantics of datasetSchema.json, problemSchema.json, datasetDoc.json, problemDoc.json <a id="semantics"></a>
- datasetSchema.json - definition of an abstract dataset. It will not be a part of any actual dataset and will only reside in GitLab.
- problemSchema.json - definition of an abstract problem. It will not be a part of any actual problem and will only reside in GitLab.
- datasetDoc.json - definition of a concrete dataset. It will be an instance of datasetSchema.json. All datasets will have this file.
- problemDoc.json - definition of a concrete problem. It will be an instance of problemSchema.json. All problem instances will have this file.

Previously, we were calling both abstract and instance definitions as dataSchema and problemSchema, leading to some confusion.  

# Dataset related

#### How will performer systems know the path to a raw file that is referenced in a table?
See [Case 2: Single table, multiple raw files](datasetSchema.md#case-2)

#### Why do we have distinct "index" and "key" roles in the datasetSchema? <a id="index-key"></a>

Index corresponds to a primary key. There can be only one index column for a table. On the other hand, any column that satisfies the uniqueness constraint, can be a key. There can be multiple keys in a table. Both index and key columns can participate in the foreign key relationship with other tables. See example here [...](datasetSchema.md#case-3)

#### Why is "role" filed a list?

A column can play multiple roles. For instance, it can be a "locationIndicator" and an "attribute", a "key" and an "attribute", and so on.

#### Why are we using "suggestedTarget" role instead of "target" in datasetSchema? <a id="suggested-targets"></a>

Defining targets is inherently in the domain of a problem. The best we can do in a dataset is to make a suggestion to the problem that these are potential targets. Therefore we use "target" in the problemSchema vocabulary and "suggestedTarget" in the datasetSchema vocabulary.

#### Why are some columns whose colType is integer in datasetDoc.json are stored as floats (with .00) in the corresponding CSV files?

Because the original data came that way. If we can detect this condition when we are manually curating a dataset, then we will annotate the column as integer in the datasetDoc.json and leave the dataset as it is.

#### What is the use for the role 'boundaryIndicator'
In some datasets, there are columns that denote boundaries and is best not treated as attributes for learning. In such cases, this tag comes in handy. Here are two hypothetical examples.

Example 1
```
d3mIndex,audio_file,startTime,endTime,event
0,a_001.wav,0.32,0.56,car_horn
1,a_002.wav,0.11,1.34,engine_start
2,a_003.wav,0.23,1.44,dog_bark
3,a_004.wav,0.34,2.56,dog_bark
4,a_005.wav,0.56,1.22,car_horn
...
```
Here, 'startTime' and 'endTime' indicate the start and end of an event of interest in the audio file. These columns are good candidates for the role of boundary indicators.

Example 2
```
d3mIndex,image,left,top,height,width,object
0,i001.jpg,2,17,33,81,cloud
1,i001.jpg,14,18,33,77,building
2,i001.jpg,78,21,31,77,pedestrian
3,i002.jpg,142,22,28,73,building
4,i002.jpg,169,21,31,73,tree
...
```
Here, left, top, height, and width columns are refer to bounding boxes in an image and are good candidates for the role of boundary indicators.

# Problem related

#### Why did we include repeat and fold columns in dataSplits?

There are many challenge datasets that use k-fold CV and provide their data accordingly as folds. To take advantage of those datasets and include them in the targeted training datasets (to be released to performers), we decided to include the fold column in our splits file. Similarly, for repeat - many challenge problems provide multiple splits of the same data and will use an average performance measure. This design was inspired by OpenML's schema, which also has this feature.

#### How does this design of using fold and repeat information in dataSplits file affect D3M blind evaluation?

In D3M blind evaluation, __only holdout method is used without any repeats__. This special case can be easily handled in the current approach by setting repeat and fold columns to 0 values. Performer systems will note that repeat and fold values are all 0s and will ignore those columns.

#### How is a problem with multiple target variables handled?

The "targets" field in problemSchema is a list of dictionaries, which allows us to define multiple targets in a problem, one dictionary entry per target. See [here](problemSchema.md#target-index)


#### How to submit predictions when there are multiple target variables?

The convention for submitting predictions in a predictions.csv file, including the case of multiple targets, is given [here](problemSchema.md#predictions-file)

# Current limitations

#### CSV files should have headers
Because our dataset schema has a required 'colName'  field, all the CSV files should have a header. If there is a raw a tabular file that does not have a header, we add a header using column indexes.

#### Required 'd3mIndex' column
In our current approach learningData.csv for all datasets should have a 'd3mIndex' column. This is used in the dataSplits.csv file of a dataset's corresponding problem to indicate the train/test splits. This means that if there is raw dataset that does not have an index column (e.g., Iris dataset), it has to be added in order to use this raw dataset - the raw dataset cannot be used as is.

