# Vector Search demo
## Prerequisites
Create a free [Couchbase Capella](https://cloud.couchbase.com/sign-up) account.  

Create a Project:  

<img width="680" alt="image" src="https://github.com/user-attachments/assets/fa93d90f-5580-438b-8907-308c13c4d40d" />

Then, click on the project name and click `Create cluster` button. That opens a cluster creation dialog, simply hit the blue `Create cluster` button:  

<img width="1664" alt="image" src="https://github.com/user-attachments/assets/73348531-cefd-485d-a24b-e38142ec2057" />  

After 5 minutes your cluster is ready to be used.

Now click on the cluster name and then on `Import data`. We're going to import a sample data set illustrating the well-known RGB model.

<img width="1530" alt="image" src="https://github.com/user-attachments/assets/93b279cd-9763-4e99-9dac-bda7aa41a006" />

Select `Load sample data` and `color-vector-sample` bucket. Then hit `Import` button.

![image](https://github.com/user-attachments/assets/1e5bcbc9-15e3-4d89-a4d3-ae1456bf31c8)


## Sample document
Each color is a JSON document. Let's open one and understand its structure.  
Click `Data tools`, select `Documents` tab, there set the context to `color-vector-sample` bucket, `color` scope and `rgb` collection.  
Now get the `000080` document and open it.  

<img width="1587" alt="image" src="https://github.com/user-attachments/assets/030cdbd8-3b58-43cf-949b-b56ef08f1608" />  

There you can see the following fields:  
* **id**: the hex code of the color
* **color**: the name of the color
* **brightness**: a calculation of the brightness to the human eye
* **colorvect_l2**: vector based on the RGB color
* **description**: a text describing the color
* **embedding_model**: the model used to encode the embedding_vector_dot vector
* **embedding_vector_dot** : vector based on the field “description” encoded via the text-embedding-ada-002 OpenAI model
* **verbs**:  list of qualifiers

Now, what is the color described in the document?  
Let's use a color picker (in this example I used whatever Google found first)  to find out.  
You can use either the hex code (the document key `000080`) or the RGB values stored in the `colorvect_l2` field (0, 0, 128).

<img width="683" alt="image" src="https://github.com/user-attachments/assets/fb5057da-4178-495d-8e70-1f5530b053a3" />


The rgb collection contains 153 documents. This is obviously just a fraction of the 16M+ colors you can encode with RGB. Here are some examples of the colors available in the rgb collection.

<img width="729" alt="image" src="https://github.com/user-attachments/assets/c3ff8b64-2417-4a06-920a-2ad83aaedbd7" />

## Simple Vector Similarity Queries
### Create your first Vector Search index
We are interested in running search queries leveraging the “colorvect_l2" vector which contains the 3 RGB dimensions of colors.  
This is a very simple example that has the benefits of being easy to understand and small enough to display the entire vector.  
Go to `Data Tools` ->  `Search` page and click `Create Search Index`.

<img width="1482" alt="image" src="https://github.com/user-attachments/assets/7b0b23f5-7dfc-47b1-b559-a6a747d20f41" />

Choose the `Quick Mode` (default), set the context to the `color-vector-sample` bucket,  `color` scope and `rgb` collection. Name your index `rgb_idx`.

<img width="1008" alt="image" src="https://github.com/user-attachments/assets/6dea5ec9-b60e-42ba-9b8e-47848ba466ad" />

In the schema area of the `Type Mappings` section, click on the field `colorvect_l2`.  
This will automatically populate the right area with the type, dimension, similarity metric, Optimized for configuration settings.  
Now, click `Add to Index`.

<img width="1504" alt="image" src="https://github.com/user-attachments/assets/5234e8ba-4608-411e-817f-b80eb0e7a585" />

This will automatically populate the right area with the type, dimension, similarity metric, optimized for configuration settings.

<img width="1289" alt="image" src="https://github.com/user-attachments/assets/c0230218-ad84-4502-8f30-f780d7de1139" />

Click `Create Index`. Your index `rgb_idx` is now available in the list of Search indexes.

### Run your first Vector Search Query
At the end of the row of your index, click the search icon 🔎  
This will open a page to run your search queries.

<img width="1603" alt="image" src="https://github.com/user-attachments/assets/9b537c27-7074-47b6-b4b6-2d7fbf11c94a" />

Let's find the top 3 nearest colors to the color `Navy`, which is encoded with the vector `[0, 0, 128]`.  
In the search area, run the following search query:

```json
{
  "query": { "match_none": {} },
  "knn": [
    { "field": "colorvect_l2",  "vector": [0, 0, 128],"k": 3 }
  ],
  "fields": ["color"]
}
```

You should see the following results:

<img width="1156" alt="image" src="https://github.com/user-attachments/assets/fe33ea4c-b8ea-4c12-acc6-d56be68dc26b" />

What are those color ids referring to?  
Let's tweak our Vector index a bit to get the colors displayed as well.  
Click `Back to Lis`t then click your `rgb_idx` to open its definition.  
Click the field `color` which contains the name of each color in the json documents.  
In the `Type Mapping` configuration area, check `Include in search results`. Click `Add to Index` and then `Update Index` at the bottom of the screen.

<img width="1253" alt="image" src="https://github.com/user-attachments/assets/fb9dde66-4b5f-4a15-92d9-087a8e626917" />

Now, run the same query as above. You should see the following results:

<img width="1174" alt="image" src="https://github.com/user-attachments/assets/0f0d0d04-a6c6-4c85-b506-d11135bce240" />

But wait, are those results accurate?  
To check that out, here is a table of both the Vector Query and results.  
The cells of the table are simply filled with their hex codes using a color picker.

<img width="719" alt="image" src="https://github.com/user-attachments/assets/1f48a15a-5588-48b3-9a44-80af4c79a3ca" />

Not only of course, Navy itself is returned as the color is already present in the database hence obviously the most similar color.  
Remember that **exact match gets extra boosting**.  
You can see that in the score itself where the color navy score is 1.7976931348623157e+308, which is way larger than the score of the other results.  
That said, the next 2 colors are very similar to the color navy!

### Run Simple Vector Search Query
We now want to run a Vector Search Query with a vector that does not exist already in the database.  
First, let's find out if the color vector `[0,0,64]` exists in the database. A simple RGB color picker tells us that this color is `#000040`.

<img width="660" alt="image" src="https://github.com/user-attachments/assets/0482e3c0-6f6f-44f5-9ba7-032383aa2f3b" />

Let's verify that this color doesn't exist in our database.  
Click `Data tools`, select `Documents` tab, there set the context to `color-vector-sample` bucket, `color` scope and `rgb` collection.  
Try to get a document with the key `000040` - there's none, so we're good to go.

<img width="1188" alt="image" src="https://github.com/user-attachments/assets/f0585d52-495c-4232-813a-71ab0db0528e" />

Now, click `Search` and then at the end of the row of your index, click the search icon 🔎  
Let's run our query to retrieve the top 3 nearest colors to `[0,0,64]` and review the results:
```json
{
  "query": {"match_none": {}},
  "knn": [
       {"field": "colorvect_l2", "vector": [0,0,64], "k": 3}
  ],
  "fields": ["color"]
}
```

Results are returned while `[0,0,64]` itself is not in the database.

<img width="717" alt="image" src="https://github.com/user-attachments/assets/d3d3f878-74e6-486e-868d-8e5dbdbf3b96" />

Interestingly enough, midnight blue comes as the top result, while we would probably consider black more similar to the color `[0.0, 0.0, 64.0]` when simply comparing the 2 colors with our own eyes. **Why is the vector search returning black in third position then?**

First of all, what is the similarity metric  used to calculate the similarity between the color vectors `colorvect_l2`? _Go back to your rgb_idx index definition and take a closer look at the type mapping for the field_  `colorvect_l2`. This is the `l2_norm`, also known as the euclidean distance.

<img width="779" alt="image" src="https://github.com/user-attachments/assets/03392abf-7f0e-489b-81d7-91dbcf510a79" />

Let's first add the RGB vector of each color in the table of results as well. You can get that information either using a color picker, enter the hex code and it will provide the RGB. Or you can click on each document in the list of results and take a look at the `colorvect_l2` vector field.

<img width="709" alt="image" src="https://github.com/user-attachments/assets/98dc808f-8b5b-4be4-b21d-c86debcb92d2" />

Let's run some simple math here.  
The euclidean distance between 2 vectors of 3 dimensions `[x1,y1,z1]` and `[x2,y2,z2]` is `√(〖(x2-x1)〗^2+〖(y2-y1)〗^2+〖(z2-z1)〗^2  )`.

* The distance between `[0,0,64]` and midnight blue `[25,25,112]` is `√(25^2+25^2+〖(112-64)〗^2) ≃ 59.6`
* The distance between `[0,0,64]` and black `[0,0,0]` is `√(64^2 ) = 64`.

In other terms, **midnight blue is indeed closer to the color [0,0,64] than black wrt the L2 similarity metric**.

Now, take a closer look at the score of each result that is displayed on the right side.

<img width="1181" alt="image" src="https://github.com/user-attachments/assets/7ec06b99-1129-44e0-a558-a47f8cdcd054" />

Note that both black and navy have the same score.  
Running the same simple math on navy we got the distance between `[0,0,64]` and navy `[0,0,128]` is `√(〖(128-64)〗^2) = 64`

Navy and midnight blue are as similar to `[0,0,64]` wrt the L2 similarity metric.   
That's why they get the same score in the list of results.  
So you might have navy either in second or third position depending on the mood of the search engine at the time of you are running this query.

### Number of Results of a Vector Query
Up to now, we have run vector queries returning the top 3 nearest neighbors of the query vector.  
We used the k parameter of the `knn` vector query to specify that.  
Let's explore the number of results of a vector query and its practical implications in more detail.

Let's get crazy and change the k parameter of the vector query to 153, which is the total number of documents in our collection rgb:
```
{
  "query": {"match_none": {}},
  "knn": [
       {"field": "colorvect_l2", "vector": [0,0,64], "k": 153}
  ],
  "fields": ["color"]
}
```

Notice that the query actually returns 153 results. Of course, the last results have a low score compared to the first ones.  
That's very intuitive as the color `white` is not similar at all to `[0,0,64]`.  
The question is now: **what is a correct value for k so that the results are relevant for the application and ultimately the end user?**  
Imagine your application is a recommendation engine. This is great to provide many recommendations, but it's better if they provide some value to the customer.

Now, let's change the k parameter of the vector query to 5.  
```json
{
  "query": {"match_none": {}},
  "knn": [
       {"field": "colorvect_l2", "vector": [0,0,64], "k": 5}
  ],
  "fields": ["color"]
}
```


<img width="718" alt="image" src="https://github.com/user-attachments/assets/f9406f38-10f4-4b06-a5fb-fe5fc6da494f" />

You can see with your bare eyes that the relevance of the results is decreasing rapidly. T 
he question is how fast? And at which point they shouldn't be considered relevant at all for the application?  
Let's take a look at the last result `dark purple #500050` and compare its score with the score of the top result, `midnight blue`.


<img width="717" alt="image" src="https://github.com/user-attachments/assets/e9c2d0c2-ccb2-442d-9669-d4087547dff1" />

What does this mean that dark purple got a score of `0.00015024038461538462`? Not much in itself.  
The point is that the interpretation of similarity is relative. In other words, midnight blue is closer to `[0,0,64]` than `dark purple`. It gets a score almost twice better than `dark purple`.  
But there is no way to define what "close enough" would be. This is why most applications would simply retrieve the top 3 results.

## Running more Advanced Vector Queries
### Run Vector Search Queries with Multiple Vectors

Run the following search query and review the results:
```json
{
  "query": {"match_none": {}},
  "knn": [
      { "field": "colorvect_l2",  "vector": [0, 0, 128],  "k": 3 },
      { "field": "colorvect_l2",  "vector": [0, 0, 64],   "k": 3 }
  ],
  "fields": ["color"]
}
```

The result is the union of both vector queries. Notice that the `knn_operator`: **or** is implicit here.  
This is the default behavior when you put multiple vector queries in the `knn` array field.

Results with vector `[0,0,128]` only:

<img width="716" alt="image" src="https://github.com/user-attachments/assets/27d7edaa-e1d3-4f7c-acdb-fde0e5fdb63e" />

Notice that navy is at the very top because this is an exact match.  
As we discussed previously, the color navy gets the highest score of `1.7976931348623157e+308`, compared to the scores of the other results.

Let's now explore a search query combining multiple vector queries with **AND**.  
Run the following search query and review the results.

```json
{
  "query": {"match_none": {}},
    "knn": [
       { "field": "colorvect_l2", "vector": [0, 0, 128],  "k": 3},
       { "field": "colorvect_l2",  "vector": [0, 0, 64],  "k": 3}
  ],
  "knn_operator": "and",
  "fields": ["color"]
}
```

The result is the intersection of both vector queries: `navy` and `midnight blue` are returned from both vector queries.  
This is `knn_operator`: **and**. Let's double check those results in more detail.

Results with vector `[0,0,128]`:

<img width="713" alt="image" src="https://github.com/user-attachments/assets/cdef520c-e779-4835-82c9-0d00b11aeaa9" />

Results are the **intersection** between results from vector `[0,0,128]` and vector `[0,0,64]`

<img width="707" alt="image" src="https://github.com/user-attachments/assets/db5d7ca0-3e10-4640-a6ca-be399dcf656c" />

Let's now explore a search query combining multiple vector queries with **explicit boosting**.

```json
{
  "query": {"match_none": {}},
  "knn": [
    { "field": "colorvect_l2", "vector": [0, 0, 127], "k":3, "boost": 0.1},
    { "field": "colorvect_l2", "vector": [0, 99, 0], "k":3, "boost": 4.0}
  ],
  "fields": ["color"]
}
```

Here we apply **boosting** for the query vector `[0, 99, 0]` - the boost value is greater than 1, and **deboosting** for the query vector `[0, 0, 127]` - the boost value is less than 1.  
Results with vector `[0,99,0]` - boost=4.0. They should get a higher score.


<img width="711" alt="image" src="https://github.com/user-attachments/assets/09165e7c-95b3-420f-a110-4ad862d680e1" />

Results of the **boosting** results from `[0,99,0]` and **deboosting** results from `[0,0,127]`


<img width="707" alt="image" src="https://github.com/user-attachments/assets/cc30f533-715d-4187-9a29-e7c8a611953c" />

Notice that most of the colors coming from the deboosted side (the blue side) are at the bottom of the list, navy is still quite high on the results.  
The reason why is because navy, encoded with `[0,0,128]`, is so close to the query vector `[0, 0, 127]` that it gets a very high score anyway.

### Run Hybrid Search and Vector Query
We want to run the query below. This is a hybrid search query between the traditional search side (query) and the vector search side (knn).  
But before we can do that, we need to create a search index that covers this query.

```json
{
  "query": {
        "field": "brightness", "min": 70, "max": 80,
        "inclusive_min": false,  "inclusive_max": true  },
        "knn": [ {"field": "colorvect_l2", "vector": [0.0, 0.0, 108.0],  "k": 5} ],
        "fields": ["color","brightness"],
        "size": 5
}
```

Click `Create Search Index`, name it `hybrid_idx` and select the usual context.

Select the field `brightness` and check the `include in search results` checkbox. Click `Add To Index`.  
Select the field `colorvect_l2`. Click `Add To Index`.  
Finally, select the field `color` and check the `Include in search results` checkbox. Click `Add To Index`.


<img width="786" alt="image" src="https://github.com/user-attachments/assets/bfaafc3d-f295-4e1d-b27c-248d502c226b" />

Click `Create Index` at the bottom of the page.

Now, let's run the hybrid search and vector query.  Insert the query in the `hybrid_idx` index search area, and review the results.
```json
{
  "query": {
        "field": "brightness", "min": 70,  "max": 80,
        "inclusive_min": false,  "inclusive_max": true  },
  "knn": [
      {"field": "colorvect_l2", "vector": [0.0, 0.0, 108.0],  "k": 5}
   ],
  "fields": ["color","brightness"],
  "size": 5
}
```

The `query` (traditional) side search and the `knn` (vector search) side are **OR**’d.  
If the same IDs are found on both sides the results are boosted to the top. Let's double-check those results.

Run the following query against the `hybrid_idx` index to get the top 5 results with vector `[0,0,108]` together with their brightness.  
```json
{
  "query": {"match_none": {}},
    "knn": [  {"field": "colorvect_l2",  "vector": [0,0,108], "k": 5} ],
  "fields": ["color","brightness"]
}
```

The results are clearly having a brightness that is not between 70 and 80.

<img width="678" alt="image" src="https://github.com/user-attachments/assets/c408c01d-795e-45c3-aa13-fe28595133b0" />

Now, let examine the results of the combined search query below.  
They all have a brightness between 70 and 80 but since the query color `[0,0,108]` has initially a brightness that seems to be closer to navy's around 14, no wonder why the results are not that close to `[0,0,108]`.


<img width="673" alt="image" src="https://github.com/user-attachments/assets/a6d99420-33ac-4e2f-9560-a561ff840671" />

### Run Hybrid SQL++ and Search Query
Now, let's say that the meaning of the query you are looking for is "I want colors similar to `[0,0,108]` **AND** having a brightness between 10 and 20.  
There is where running a hybrid SQL++ and Search query can come handy.

Go to the `Query` page, set the right context and run the following SQL++ query and review the results.

```sql
SELECT color, brightness
FROM rgb AS t1
WHERE
   brightness <= 20 AND brightness>=10
AND
  SEARCH (t1, {
    "query": {  "match_none": {} },
    "knn": [{ "field": "colorvect_l2", "vector": [0.0, 0.0, 108.0],"k": 3 }]
    }
  )
```

<img width="1181" alt="image" src="https://github.com/user-attachments/assets/8831bd8e-a59b-4d78-9fb6-320709055edb" />

The `SQL++` side and the `Search` (a pure vector search in this example) side are **AND**’d via the SQL++ syntax.  
In this case, while you were asking for the top 3 results from the vector side (search), only 2 results are returned because only those colors also satisfy the condition of brightness between 10 and 20.

Review the execution time and plan for this query. In this database, it took `778.8 ms` to run the query.

<img width="1164" alt="image" src="https://github.com/user-attachments/assets/3301f6ba-4eb6-477a-846d-1d53b86faeb9" />

Notice that the index advisor suggests another index to speed up this query. Accept the suggestion and build the index.

<img width="354" alt="image" src="https://github.com/user-attachments/assets/78373df6-8452-4203-be8e-2f1b032332d7" />

Re-run the query and observe the improved execution time.

<img width="906" alt="image" src="https://github.com/user-attachments/assets/1b0cfc06-ff0b-4933-9143-ba4e7fba585f" />

This time, the query performs an `intersectScan` between the `FTS index` and the new `GSI index`.  
Before, the execution plan leveraged only the `FTS index`.






